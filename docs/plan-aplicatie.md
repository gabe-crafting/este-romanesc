# Build plan

What we build first, what we hold back, and why. Decided 2026-08-15.

> **Narrowed for the first release by [mvp.md](mvp.md)**, agreed later the same day. The MVP is
> axis 1 alone — is this company registered in Romania — because it is the only axis with no
> blocker: free source, full coverage, nothing to buy, no GDPR exposure. This document stays as
> the plan for what comes after; the axis ordering and the wording rules below still apply.

## The scope decision

Of the four things the app can show (see [de-unde-luam-datele.md](de-unde-luam-datele.md)):

| | Axis | v1 |
| --- | --- | --- |
| 1 | Company and where it is registered | **Build** |
| 2 | The group it belongs to | **Build** |
| 4 | How much it produces in Romania | **Build — the useful one** |
| 3 | What percentage of ownership is Romanian | **On hold** |

Axis 3 is paused as a feature, not cancelled. The data model still has to leave room for it,
because axis 2 is built on the same table.

## Why this split works

Axes 2 and 3 read the same graph but need very different things from it, and that is what
makes deferring 3 cheap:

- **Axis 2 asks "who controls this?"** — walk up the company-to-company edges until nobody
  above holds a controlling stake. It needs the *shape* of the chain.
- **Axis 3 asks "how much reaches Romanians?"** — walk every path, multiply the stakes, and
  trace all the way down to **natural persons**. It needs the chain to be *complete*.

Three things fall away with axis 3:

1. **No `persons` table yet.** Naming individual shareholders is the GDPR-heavy part of this
   product. A group chain is companies talking about companies — far less exposure. Deferring
   3 defers that whole legal question.
2. **Lower coverage bar.** "Who is the parent" is answerable from partial data. "45% Romanian"
   is only meaningful if the graph is near-complete, which it will not be for a long time.
3. **No unknown-percentage UI.** The hardest display problem in this product — showing
   `ro / foreign / unknown` honestly without it reading as evasion — is postponed.

What does **not** fall away: `ownership_edges` still has to exist and be populated, because
axis 2 lives on it. We are deferring the arithmetic and the completeness requirement, not the
data acquisition.

## The risk this creates, and the mitigation

**Showing a group without a percentage is more misleading than showing nothing**, unless the
wording is careful. "Part of an Austrian group" will be read as "an Austrian company" — which
is precisely the error the documentation spends three files correcting. OMV Petrom is
controlled from Austria and roughly 45% Romanian-owned; a bare group label communicates the
first and hides the second.

Rules for the group field in v1:

- Say **control**, not ownership: "face parte din grupul X" / "controlată de X (Austria)".
  Never "firmă austriacă", never a flag icon next to the company name.
- Put one line under it stating that this says nothing about how much of the company Romanians
  own, linking to the FAQ entry.
- Do not colour the group field, or rank/sort by it. It is information, not a verdict.
- Never combine axes into a single badge or score. That temptation is exactly what axis 3 was
  meant to make honest, and it is now absent.

## Build order

Free and unblocked first; the axis that needs a purchasing decision last.

### Phase 0 — schema

`companies`, `establishments`, `products`, `evidence`, `ownership_edges`.

`ownership_edges` is created now with `owner_company_id → owned_company_id`, a stake
percentage, `valid_from` / `valid_to` and a source. The owner column is **nullable-by-design
for a future person owner** — add the `persons` foreign key when axis 3 is picked up, rather
than reshaping the table then.

Everything ingested lands in `evidence` with source, method and `observed_at`. Scores are
recomputed from it, never written directly.

**Done when:** migrations apply from empty, and `supabase db reset` reproduces the schema.

### Phase 1 — axis 1, registration

Ingest ANAF's taxpayer service — free, no auth, 100 CUIs per request at 1 request/second, so a
full national sweep is a 4–5 hour overnight job. Add Ministry of Finance filings for CAEN,
turnover and employee counts.

**Done when:** any Romanian company can be looked up by CUI or name from our own database, with
a date attached, and no user request touches an external API.

### Phase 2 — axis 4, production

The one you called the useful one, and it is mostly free data we already have after phase 1:

- manufacturing CAEN codes
- registered `puncte de lucru` from ONRC
- employee counts from the filings
- ANSVSA approved-establishment lists, for the oval mark on animal-origin products

**Done when:** the app can say whether a company produces in Romania and at roughly what scale,
scoped to Romania only — never a share of a foreign group's global plants.

### Phase 3 — axis 2, the group

Needs the source decision first: which aggregator, at what price, and whether its licence
permits storing and displaying what we derive. That is a contract question, so it gates the
phase.

Populate company-to-company edges, then resolve upward to a controlling parent.

**Done when:** a company page can name the group above it and that group's country, with the
wording rules above applied.

### Later — axis 3, ownership percentage

Revisit when **all three** are true:

1. Ownership coverage is good enough that `unknown` is not the largest number for most
   companies people actually search.
2. The GDPR question on person-level data has an answer we are comfortable publishing.
3. There is a UI design that shows `ro / foreign / unknown` without it reading as evasion.

When it is picked up, `effective_ro` gets tested against OMV Petrom first — we already know the
expected answer is ≈45% Romanian, ≈51.2% foreign, and that is a good regression case.

## What we are explicitly not building yet

- `persons` table and any person-level shareholder display
- the `effective_ro` recursive walk
- percentage UI, and any combined score across axes
- barcode scanning as a product feature (the GTIN mapping can be seeded earlier)

## Open items this plan does not settle

- Which aggregator, and whether its terms allow our use. Blocks phase 3.
- Whether the FAQ's current wording — that "made here" matters more than "owned by" for the
  money question — stays stated outright or is left to emerge from the layout.
