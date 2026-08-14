# How we get the data

Which sources we can call live, which we have to ingest into our own database, and why.

See [what-counts-as-romanian.md](what-counts-as-romanian.md) for what the data is *for*.

## The rule

Ask four questions about a source. Any single "no" means ingest it.

1. **Is it off the user's critical path?** Search results cannot wait on a third party.
2. **Is the rate limit per-user rather than shared?** This is the one that decides most cases.
3. **Does the answer change fast enough to need a fresh fetch?** Most of this data doesn't.
4. **Does the licence permit storing it?** Occasionally the pressure runs the other way.

In practice, for this product: **the read path never touches an external API.** Ever. Users
hit Postgres, and background jobs keep Postgres current.

### Why the rate limit settles it

ANAF's taxpayer service allows **1 request per second, 100 CUIs per request** — and that
limit is *global to us*, not per user. Ten people searching at once means the tenth waits.
A hundred means the product looks broken. A shared rate limit is not a performance problem
you can optimise around; it's a hard ceiling on concurrent users.

Turned around, though, that same limit is generous for **batch** work:

```
100 CUIs/request × 1 request/second = 360,000 companies/hour
```

Romania has on the order of 1.5M registered entities, so a **full refresh of every company
is roughly a 4–5 hour job**. That is an overnight cron, not an obstacle. The exact same
budget that makes live calls impossible makes bulk ingestion easy.

## Source by source

| Source | Access | Limit | How fast it changes | Strategy |
| --- | --- | --- | --- | --- |
| **ANAF taxpayer service** | Free, no auth | 100/req, 1 req/s, shared | VAT + inactive flags: weeks | **Ingest.** Nightly targeted, full sweep quarterly |
| **Ministry of Finance** filings | Free, bulk + per-CUI | Bulk download | Annual filings | **Ingest.** After each filing season |
| **Verified by GS1** | Free tier | ~tens of lookups | Licensee almost never | **Seed and cache.** Queue new GTINs, resolve offline |
| **Aggregators** (termene, listafirme, risco) | Paid, quota | Contractual | Ownership: rarely | **Ingest, licence permitting.** Read the terms first |
| **ONRC beneficial owners (RBR)** | Restricted | n/a | — | **Not automatable.** Manual, out of band |
| **ANSVSA approved establishments** | Published lists | — | Occasionally | **Ingest.** Monthly scrape |
| **User label photos** | Ours | — | — | Our own tables from the start |

Two notes on that table:

- **GS1 is the clearest "never live" case.** A free tier measured in tens of lookups cannot
  sit behind a scan button. Resolve unknown GTINs from a queue, store the mapping, and serve
  it locally forever after — the licensee behind a barcode prefix effectively never changes.
- **Aggregator licences can forbid what we want to do.** Everything else here is a technical
  decision; that one is a contract. Check before building on it.

## The two paths

```
READ PATH — what a user triggers                    must be local, always
  app / site  →  Supabase (Postgres)  →  answer
                     ▲
                     │
INGEST PATH — what a schedule triggers              may be slow, may be rate-limited
  pg_cron / scheduled Edge Function
      →  ANAF · MF · GS1 · aggregators · ANSVSA
      →  write to evidence + recompute scores
```

Clients never call an external API directly. Every outbound request goes through an Edge
Function, which is the only place that holds keys, the only place that respects a rate limit,
and the only place that can decide something is already cached.

### The one exception

A **single-company "refresh"**, triggered explicitly by a user or an admin on one record.
One CUI, one request, well inside the budget, and the user has asked for it so a two-second
wait is legible rather than mysterious. Rate-limit it per user and never let it fan out.

## Refresh cadences

Match the cadence to how fast the truth actually moves — refreshing yearly data nightly is
just burning quota:

| Data | Cadence | Why |
| --- | --- | --- |
| VAT status, inactive flag | Nightly for the top few thousand; quarterly full sweep | Changes, and "inactive" matters |
| Financial statements | Yearly, after filing season | Filed annually |
| Ownership | Quarterly, or on demand | Changes rarely, expensive to fetch |
| GTIN → licensee | Once, on first sighting | Effectively static |
| ANSVSA establishments | Monthly | Slow-moving list |

## What this means in the schema

- Every ingested fact carries `observed_at` and its source — a value with no timestamp is not
  usable evidence, because we can't tell whether it's a year stale.
- **Staleness is visible in the UI.** "As of March 2026" is honest; an undated number implies
  a freshness we don't have.
- Ingestion writes to `evidence`; scores are **recomputed** from it, never written directly.
  Re-running a job must be safe, so upserts key on `(subject, source, observed_at)`.
- Keep a per-source job log — last run, rows touched, failures. When a number looks wrong,
  the first question is always which job last touched it.

## Practical order of work

1. **ANAF ingest first.** Free, unauthenticated, covers every company, and gives the
   registration axis outright. Nothing else is blocked on a decision.
2. **Ministry of Finance next** — turnover and employee counts, which feed the production axis.
3. **GS1 resolution** once products exist to resolve.
4. **Ownership last.** It needs a paid-source decision and a licence review, and it's the axis
   we can least complete. Don't let it block the other two.
