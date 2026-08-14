# faq/

The public "how it works" page, in Romanian, built on the Classical design system.

- `index.html` — the FAQ itself. Open it directly in a browser; there is no build step.

## Conventions

- It links `../design/styles.css` and takes every color, font, spacing and radius from that
  file's variables. The only page-level CSS is layout (measure, the numbered contents list,
  table scrollers) and it uses `var(--…)` throughout — no hard-coded hex or px values, per
  the rule in [`design/readme.md`](../design/readme.md).
- Body copy is justified, dividers are hairlines, tables use `.table`, callouts use `.card`.
  If a new pattern is needed, take it from the design system rather than inventing one here.
- Wide tables sit inside a `.scroller` so the page never scrolls sideways on a phone.

## Where the content comes from

The answers are the plain-Romanian version of the reasoning in `docs/`:

- [`docs/explicatie-simpla.md`](../docs/explicatie-simpla.md) — the same material as prose
- [`docs/what-counts-as-romanian.md`](../docs/what-counts-as-romanian.md) — the three axes, the barcode
- [`docs/ownership-and-where-the-money-goes.md`](../docs/ownership-and-where-the-money-goes.md) — the tax and dividend mechanics
- [`docs/data-retrieval.md`](../docs/data-retrieval.md) — where the data comes from

**Keep them in step.** If a number changes in `docs/`, it changes here too — this page is the
version users actually read.

## Later

This is a standalone page so it can exist before `apps/web/` does. When the Next.js site is
scaffolded it should move there and become a route, reusing the same stylesheet rather than a
copy of it.

Fiscal figures carry a date for a reason: Romanian tax rates move often, and a stale number on
a public page is worse than no number.
