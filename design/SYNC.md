# Keeping `design/` and Claude Design in step

The Classical system exists in two places: this directory, and the Claude Design project
`Classical` (`projectId 957b669d-4083-4b30-a67e-0a568f20da59`). This file records how they
relate, because the tooling does not enforce it.

## There is no two-way sync

The sync tool writes in **one direction only — local → Claude Design**. It can *read* the
remote project (list files, fetch one file), but there is no merge, no conflict detection,
and no history on the Claude Design side. If both sides are edited between syncs, whichever
side pushes last silently wins.

So pick one source of truth. **This repository is it.**

- Edit tokens and components here, in `styles.css` / `theme.json`, and commit.
- Push to Claude Design when you want the previews refreshed there.
- Treat the Claude Design project as a rendered view of this directory, not as a second
  place to author.

If you do author something in the Claude Design UI — which is a perfectly reasonable way to
explore — pull it back down into this directory and commit it *before* the next push, or it
will be overwritten.

## The mirror is currently partial

Imported so far — the actual source of the system:

- `styles.css` — the only stylesheet; tokens plus the component layer
- `theme.json` — the parameters the system was derived from
- `readme.md` — the usage guide

Still only in Claude Design, not yet here:

```
components/{buttons,cards,dialog,forms,navigation,table}.html
foundations/{color,icons,image,layout,type}.html
templates/deck/…  templates/landing/…
theme.html  thumbnail.html  assets/photo.jpg
_adherence.oxlintrc.json  _builtin.json  _ds_bundle.js  _ds_manifest.json  .thumbnail
```

Those are preview and demo pages that *consume* `styles.css` rather than define it, plus
files the Claude Design app generates for itself (`_ds_bundle.js`, `_ds_manifest.json`,
`.thumbnail`).

> **Do not push with a broad glob until the mirror is complete.** The sync tool's plan step
> takes patterns like `**/*.html` for both writes *and* deletes. Pushing `design/` as it
> stands, with a plan that covers the whole project, would delete every preview page listed
> above from Claude Design. Until the import is finished, scope any plan to the exact paths
> that exist here.

`assets/photo.jpg` is binary and cannot be round-tripped through the agent; download it from
the Claude Design project directly if it is needed in the repo.

## How this feeds the apps

`theme.json` is the machine-readable record of the theme, which makes it the natural input
for both clients:

- `apps/web/` links `styles.css` directly, or generates its CSS variables from `theme.json`.
- `apps/mobile/` cannot use CSS, so the same `theme.json` should generate the Compose theme
  (colors, type scale, spacing) — and later the SwiftUI one. Generating both from one file
  is what stops the phone app and the website drifting apart.

Neither is wired up yet.
