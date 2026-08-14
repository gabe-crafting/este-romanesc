# What counts as Romanian

The product answers one question — *is this company Romanian?* — and that question has no
single correct answer. This document works out what we can actually determine, from which
sources, and how confident we are allowed to be about it.

Nothing here is implemented yet. This is the design that the first migration should follow.

## One number won't do

"Romanian" collapses at least three different claims that people care about for different
reasons:

| Claim | What it means | Why someone cares |
| --- | --- | --- |
| **Registered here** | A Romanian legal entity, with a CUI, paying Romanian taxes | Tax revenue stays in the country |
| **Owned here** | The capital behind it belongs to Romanian people or entities | Profit stays in the country |
| **Made here** | Production physically happens in Romania | Jobs stay in the country |

These come apart constantly. A German-owned factory in Cluj is registered and producing here
but foreign-owned. A Romanian-owned brand importing everything from Asia is the reverse.
Collapsing them into one boolean means picking whose definition wins and silently discarding
the rest.

**Decision: we do not emit a single verdict.** A company gets scored on independent axes,
each with its own confidence and its own visible evidence. The UI can lead with whichever
axis a user cares about; it must never show a number without the reasoning behind it.

## Axis 1 — Registration

The easy one. Solved, cheap, reliable.

A company has a CUI, is registered in Romania, has a fiscal address and a main CAEN activity
code. This is free public data:

- **ANAF's taxpayer web service** (`webservicesp.anaf.ro/PlatitorTvaRest/api/vN/ws/tva`) —
  POST a JSON array of `{cui, data}` pairs, get back name, address, VAT status, and whether
  the taxpayer is flagged inactive. No authentication, no key. The path is versioned and the
  version has moved over the years, so pin it in config and expect to bump it.
- **Ministry of Finance** (`mfinante.gov.ro`) — per-CUI identification data plus the annual
  financial statements a company has filed, published under OMF 1420/2021. This is where
  turnover, employee counts and filing history come from.

Registration is a fact, not a score. Store it as such.

## Axis 2 — Ownership

The hard one, and the one where naive implementations quietly lie.

### The algorithm

Ownership is a graph, not a field. Company A is 60% owned by company B and 40% by a Romanian
citizen; B is 100% owned by a Dutch holding company. Answering "how Romanian is A" means
walking that graph and multiplying stakes along each path.

```
effective_ro(company, depth):
    if depth > MAX_DEPTH:          return unknown(1.0)
    if company in visited:         return unknown(1.0)     # cross-shareholding cycle

    ro, foreign, unknown = 0, 0, 0
    for each (owner, stake) in owners(company):
        if   owner is a natural person, resident RO:  ro      += stake
        elif owner is a natural person, resident XX:  foreign += stake
        elif owner is a company:
            sub = effective_ro(owner, depth + 1)
            ro      += stake * sub.ro
            foreign += stake * sub.foreign
            unknown += stake * sub.unknown
        else:                                         unknown += stake

    unknown += 1.0 - (sum of known stakes)            # unregistered remainder
    return (ro, foreign, unknown)
```

Three properties that matter more than the arithmetic:

1. **`unknown` is a first-class outcome, not zero.** The tempting bug is treating an untraced
   owner as foreign (or as Romanian). A company that is 30% Romanian, 10% foreign and 60%
   untraced is *not* 30% Romanian — it is "at least 30%, and we don't know about most of it".
   Carry all three numbers to the surface.
2. **Cycles terminate.** Cross-shareholding is legal and real; a visited-set and a depth cap
   are not optional.
3. **Stakes are time-bounded.** Ownership changes. Every edge needs `valid_from` / `valid_to`,
   and a score is only ever "as of" a date.

### The blocker

**The National Register of Beneficial Owners (RBR) is not public.** It is administered by
ONRC, and access is restricted to authorities and to persons demonstrating a legitimate
interest, through `myportal.onrc.ro`. We cannot scrape it and should not try.

So the ownership graph has to be assembled from what *is* reachable:

- **ONRC company records** — associates and administrators are obtainable, but through the
  portal and paid extracts rather than an open bulk feed.
- **Commercial aggregators** — `listafirme.ro`, `termene.ro`, `risco.ro` resell ONRC data as
  JSON APIs, including associates, administrators and company-to-company connections. Paid,
  and the licence terms decide whether we may redistribute what we derive from them. **Check
  the terms before building on one** — a scoring product that republishes derived facts is a
  different use than internal lookup.
- **Company self-declaration** — for the long tail, ask the company. Cheap, and unreliable
  unless marked as such.

Expect ownership coverage to be **partial and skewed toward large companies**. Design for
that: a mostly-unknown graph is the normal case, not an error state.

## Axis 3 — Production

The one people ask about most and the one we can support least.

### What we can establish

For a *Romanian legal entity*, reasonable proxies exist:

- **CAEN code** — the main activity classification. Codes in the 10–33 range are
  manufacturing. A company whose main CAEN is manufacturing and whose registered address is
  in Romania is very likely producing something here.
- **Secondary establishments** (`puncte de lucru`) — registered with ONRC, giving locations
  beyond the head office.
- **Employee counts** from filed financial statements — a company with 800 employees and a
  manufacturing CAEN is not a shell.

### The hard limit, stated plainly

**We cannot compute "what share of this company's factories are in Romania" for a
multinational.** Romanian registers only see the Romanian legal entity. Nothing in ANAF,
ONRC or the Ministry of Finance knows how many plants the foreign parent operates in Poland
or Vietnam. That denominator does not exist in any dataset we can reach.

Two honest options, and we should pick one rather than fudging:

- **Scope the claim to Romania.** Answer "does this company manufacture in Romania, and at
  what scale" — which is answerable — and never claim a global share.
- **Curate the multinationals by hand.** A few hundred parent companies cover most of what
  shoppers actually scan. Human research, sourced and dated, stored as regular evidence.

The first is the honest default. The second is a later enhancement, not a substitute.

## What the barcode actually tells us

Users will assume the barcode answers the whole question. It doesn't — but it is still the
single most useful thing on the package, for a different reason than people expect.

### What EAN-13 encodes

```
5 9 4 | 1 2 3 4 5 | 6 7 8 9 | 0
└─────┴───────────┘ └───────┘ └── check digit
   GS1 company prefix   item reference
   └── first digits identify the GS1 Member Organisation that issued it
```

`594` is GS1 Romania. What that establishes is that **the brand owner is a GS1 Romania
licensee** — a fact about the company, not about where the product was made.

### Why it cannot answer "made here"

- A Romanian company can register with GS1 Romania and manufacture entirely in Asia. Still 594.
- A foreign brand manufactured in a Romanian plant carries the foreign brand owner's prefix.
- **Barcode resellers** sell single numbers out of old prefixes, mostly US ones. A Romanian
  startup that bought one barcode online gets a `0`/`1` prefix regardless of where it is.
- **Private-label goods** carry the retailer's prefix, so a Romanian-made own-brand product
  reads as whatever country the chain registered in.
- **Restricted-distribution ranges** — `020–029`, `040–049`, `200–299` — are assigned in-store
  for variable-weight and counter goods. There is no global licensee at all. Deli meat, cheese
  and weighed produce land here, and they are exactly the categories where origin matters most.

So: never render "594 → Romanian" in the UI. It is not what the number means.

### The three ways it genuinely helps

**1. It is the best join key in the system.** This is its real value. A scan yields an exact
GTIN — no fuzzy matching of "SC Napolact SA" against "Napolact". Product → brand → company
becomes a primary-key lookup instead of a text-similarity problem. Everything else on this
page is easier once a product is unambiguously identified.

**2. Verified by GS1 maps a prefix to its licensee.** GS1's GEPIR service was replaced by
**Verified by GS1** at the end of 2023; it returns the licensee company name and country for a
GTIN or company prefix, across GS1's member companies worldwide. That gives an authoritative
barcode → company-name link, which we can then match to a CUI via ANAF. The free tier is
rate-limited (on the order of tens of lookups), so treat it as a **seeding and verification
tool, not a runtime dependency** — resolve in bulk offline, cache the mapping, serve from our
own tables.

**3. A weak positive on the registration axis only.** A 594 prefix does make it likely the
brand owner is a Romanian-registered entity. That is real evidence — for axis 1, worth a low
weight, and worth *nothing* on axis 3. Note the asymmetry: **594 is weak positive evidence,
but its absence is almost no evidence at all**, because of resellers and private label. Any
scoring code must treat it one-directionally or it will penalise Romanian companies that
bought a cheap barcode.

### The mark that does answer "made here"

On products of animal origin — meat, dairy, eggs, fish — EU law requires an **oval
identification (health) mark**, printed near the barcode:

```
┌─────────────┐
│     RO      │   country of the approved establishment
│  L 1234 EC  │   veterinary approval number
│     CE      │
└─────────────┘
```

This is regulated, it identifies the **approved establishment that processed or packed the
product**, and the numbers are assigned by **ANSVSA**, which authorises those establishments
and publishes lists of them. Unlike the barcode, this is a genuine production-location fact,
and it is checkable against a Romanian dataset.

Its limits, which matter:

- Animal-origin products only. Nothing for bread, beer, cosmetics, furniture.
- It identifies the **last approved establishment**, which may be a packer rather than a
  producer. Milk from anywhere bottled in Cluj still gets an `RO` oval.
- It has to be read by OCR from a photo of the package, not from a scan.

Label text is the other OCR target: `Fabricat în România`, `Produs în România`, and the origin
declarations required under EU food information rules. Read them carefully — **`Ambalat în
România` means packed, not produced**, and treating the two as equivalent is precisely the
kind of overclaim that discredits the product.

### What this implies for the app

The scan and the evidence are two different steps, and the design should say so:

1. **Scan the barcode to identify.** GTIN → product → company. Fast, exact, offline-cacheable.
2. **If the company is known**, answer from our data — ownership and registration are company
   facts, and no barcode is needed for them.
3. **If origin is the question**, ask the user to photograph the label. The oval mark and the
   origin text are where "made here" actually lives.

That also turns the unknown-product case into a contribution flow rather than a dead end: a
scan we can't resolve is an invitation to photograph the label, which is the one thing a user
standing in a shop can give us that no register can.

## Confidence and evidence

Every stored claim carries its provenance, or the product becomes a rumour engine:

- **source** — which register, aggregator, or human, with a URL or document reference
- **observed_at** — when we saw it; ownership and CAEN data go stale
- **method** — `official_register` / `aggregator` / `self_declared` / `manual_research` /
  `inferred`
- **confidence** — derived from method and staleness, not typed in by hand

Rules that follow from this:

- A score is recomputed from evidence, never edited directly.
- "Unknown" is a legitimate, displayable answer. Showing "we don't know" beats guessing, and
  for a product whose entire value is trustworthiness it is the *only* safe default.
- Anything user-visible must be able to show its sources on demand.

## What this means for the schema

The first migration needs roughly:

- `companies` — CUI, names, registration facts, main CAEN, status
- `persons` — natural persons with country of residence (personal data: see below)
- `ownership_edges` — `owner → owned`, stake percentage, `valid_from`/`valid_to`, source
- `establishments` — location, type, CAEN, employee count where known
- `evidence` — the claim-with-provenance table everything else is derived from
- `scores` — materialised per company and axis, with `computed_at`, recomputed rather than written

The ownership walk is a recursive CTE over `ownership_edges`. Postgres does this natively;
it belongs in the database, next to the search, so the app and the site return identical
numbers.

## Legal and ethical constraints

Not optional, and cheaper to design around now than to retrofit:

- **Personal data.** Shareholders who are natural persons are living people, and their names
  and stakes are GDPR-relevant even when sourced from a public register. Publishing a register
  extract is not the same as republishing it in a searchable product. Get this checked before
  any person-level data goes public.
- **Aggregator licences.** If ownership data comes from a paid API, redistribution rights are
  a contract question, not a technical one.
- **The reputational risk runs one way.** Wrongly labelling a company as foreign-owned is the
  kind of error that gets a product sued or boycotted. Under-claiming is safe; over-claiming
  is not.

## Open decisions

1. Which axis leads the UI — registration, ownership, or production?
2. Do we pay for an aggregator, and if so which, and do its terms permit our use?
3. Is person-level ownership shown publicly, or only aggregated into a percentage?
4. Do we accept company self-declarations, and are they visibly marked as such?
5. Manual curation for the top multinationals — in scope for v1, or later?

## Sources

- [ANAF taxpayer web services (community documentation)](https://github.com/robert-malai/anafpy)
- [Ministry of Finance — company identification and financial statements](https://mfinante.gov.ro/en/domenii/informatii-contribuabili/persoane-juridice/info-pj-selectie-dupa-cui)
- [Ministry of Finance datasets on data.gov.ro](https://data.gov.ro/organization/mfp)
- [ONRC — access guide to the beneficial owners register](https://www.onrc.ro/documente/ghidAccesFIRBR.pdf)
- [ONRC — trade register information](https://www.onrc.ro/index.php/ro/informatii/informatii-rc)
- [ListaFirme — beneficial owners API](https://listafirme.ro/articole/registrul-beneficiarilor-reali.asp)
- [GS1 company database / GEPIR, superseded by Verified by GS1](https://www.gs1us.org/tools/gs1-company-database-gepir)
- [GEPIR overview](https://en.wikipedia.org/wiki/GEPIR)
- [ANSVSA — authorisation and registration of establishments](https://www.ansvsa.ro/industrie-si-afaceri/autorizare-inregistrare-unitati-animal-non-animal/)
- [ANSVSA — establishments approved for intra-community trade](http://www.ansvsa.ro/unitati-schimb-intracomunitar/)
