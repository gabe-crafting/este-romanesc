# MVP

**Scan or search a product, see whether the company behind it is Romanian-owned.**

Nothing else. No production axis, no group display, no scores across axes. Three phases.

This narrows [plan-aplicatie.md](plan-aplicatie.md): that plan shipped axes 1, 2 and 4 and
held ownership back. The MVP does the opposite — ownership is the whole product. The reasoning
below is why that can work anyway.

## The problem this MVP has to solve first

Ownership is the least available of the four things we can show. The beneficial owners register
is not public, ONRC data comes through paid extracts, and coverage across 1.5M Romanian
companies will never be close to complete.

Built naively, this MVP answers **"nu știm"** to almost every scan. That is not a soft failure —
an app whose one feature usually returns nothing reads as broken, and never gets a second use.

### The answer: curate the shelf, not the country

You do not need ownership for every Romanian company. You need it for the **few hundred brands
that are actually on a supermarket shelf**. That changes the problem completely:

- **~300 companies covers most real scans.** Dairy, beer, bread, snacks, water, cleaning,
  cosmetics. A person in a shop scans products from a small set of large producers.
- **For companies that size, ownership is genuinely public** — annual reports, Bucharest Stock
  Exchange filings, company investor pages, press coverage. OMV Petrom publishes its shareholder
  structure. No aggregator purchase and no scraping needed.
- **No GDPR problem at that scale.** Large-company ownership is corporate: firm owns firm. We
  are not naming private individuals, which is where the legal exposure lives.
- **It is honest work, not a shortcut.** Each entry is researched, sourced, dated and stored as
  regular evidence, exactly like ingested data.

So: **manual curation for the shelf, ANAF for everything else, and an honest "nu știm" beyond
that.** The aggregator decision, the persons table and the full graph walk all stay deferred.

## Phase 1 — companies and search

Build order is the reverse of the user flow: there is nothing to scan into until companies
exist.

- Minimal schema: `companies`, `evidence`.
- Ingest ANAF's taxpayer service — free, unauthenticated, 100 CUIs per request at 1 request per
  second, so a national sweep is an overnight job. Adds name, CUI, address, active/inactive.
- Search in Postgres: full-text plus trigram similarity, over names and diacritic-free variants,
  so "napolact" finds "SC NAPOLACT SA".
- Every fact lands in `evidence` with source, method and `observed_at`.

**Done when:** you can type a company name into a page and get the right company back from our
own database, fast, with no external call on the request path.

## Phase 2 — the ownership answer

The actual product.

- `ownership_edges`: `owner_company_id → owned_company_id`, stake percentage, `valid_from` /
  `valid_to`, source. Company-to-company only — no `persons` table in the MVP.
- Curate the top ~300 consumer-facing companies by hand. For each: who owns it, what percentage,
  where that owner is resident, and the URL the claim came from.
- Compute a simple verdict per company: `romanian` / `foreign` / `mixed` / `unknown`, with the
  percentage where the curation supports one.
- **`unknown` is a real, designed state**, not an error page. It says what we do know — the
  company is registered in Romania, here is its CUI and county — and offers the user a way to
  ask us to research it. That request queue is also the priority list for the next curation
  batch.

**Done when:** for a curated company the app states who owns it and how Romanian that is, with a
visible source and date; for everything else it says clearly that we do not know yet.

## Phase 3 — the scan

Now the barcode has something to resolve into.

- `products`: GTIN, name, brand, `company_id`.
- Barcode scanning in the Android app. The scan yields a GTIN, which is a primary-key lookup
  into `products` — exact, fast, offline-cacheable.
- Seed GTINs for the curated companies' products. Verified by GS1 resolves a prefix to its
  licensee, but its free tier is a few tens of lookups, so it runs offline in a queue and the
  result is cached forever. Never on the request path.
- **Unknown barcode is a contribution, not a dead end:** fall back to name search, and let the
  user photograph the label so we can add the product.

**Done when:** scanning a product from a curated brand shows the ownership answer in one step,
and scanning anything else degrades to search rather than to an error.

## Out of scope for the MVP

- Production axis, group display, any combined score
- `persons` table, individual shareholders, the recursive `effective_ro` walk
- Paid aggregators, ONRC extracts, anything touching the beneficial owners register
- The Next.js site — the MVP is the Android app plus whatever admin UI curation needs
- iOS

## Assumption worth confirming

The MVP shows ownership as the headline answer, but still shows **registration** as a fallback
line when ownership is unknown — "înregistrată în România, proprietari neurmăriți încă" — because
the alternative is a blank screen on most scans. Registration data is free, complete and already
ingested in phase 1, so this costs nothing.

If you would rather the app say strictly nothing when ownership is unknown, that is one line to
change — but it makes most scans return an empty result.

## The wording risk carries over

With only one axis shown, users will read the ownership verdict as a verdict on the whole
company. It is not. A foreign-owned factory employing 2,000 people in Ploiești will read as
"not Romanian" in this MVP, while a Romanian-owned importer of Chinese goods reads as
"Romanian". That is a real limitation of shipping this axis alone, and the FAQ should say so
plainly rather than letting the app imply otherwise.
