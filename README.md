# ScriptCat UI/UX Refactor Design Specification

This repository contains the design specification and reference material for a ScriptCat UI/UX refactor.

The repo is documentation-first. It is meant to help shape the product direction, not to serve as the application source code itself.

## Design Thesis (rev. 4)

A userscript is power the user grants to someone else's code, indefinitely. Competitors treat that grant as a one-time install dialog followed by silence; ScriptCat's refactor makes it **visible, controllable, and reversible** for its whole lifetime:

- **Visible** — an activity ledger records what scripts actually do (locally, aggregates only) and shows it in plain language.
- **Controllable** — scripts × sites is a two-axis model: the popup is a per-site controller, the manager can pivot by site.
- **Reversible** — every change produces a restorable version; trash, rollback, and real undo replace confirmation friction.

Earlier revisions (rev. 1–3) synthesized the best of the reference products; rev. 4 builds ScriptCat's own model on top of that validated floor. See `SPEC.md` §2–§3 for the model and §18 for the build order.

## Contents

- `SPEC.md`: the main UI/UX design specification (rev. 4) — thesis, model, surfaces, system rules.
- `design/`: detailed per-page implementation specifications derived from `SPEC.md` (popup/site controller, manager, script hub, install/confirm, import/export/update flows, settings, logs).
- `references/`: compiled snapshots from related userscript managers, UX analyses of each, and supporting notes for comparison.
- `draft/`: exploratory visual material. (The 2026-06-10 draft predates rev. 4; it reflects the rev. 3 layouts and is kept as historical exploration.)

## How To Use This Repo

Start with `SPEC.md` Part I (§1–§3) to understand the product thesis and the model; Part II for surface-by-surface intent; the matching `design/` document for buildable detail.

Use the reference snapshots under `references/` to compare:

- popup hierarchy
- script list organization
- editor workflows
- runtime visibility and error handling
- terminology and labels

The reference notes are intentionally explicit about which sources are preferred for UX direction — and about where rev. 4 deliberately goes beyond all of them (activity ledger, version history, site axis).

## Scope

This repository focuses on the ScriptCat manager experience across desktop and mobile surfaces, including:

- installed script management (script and site axes)
- current-page runtime visibility and per-site control
- script editing, debugging, and the script detail hub
- observed script activity and trust surfaces
- update, import, export, version history, and sync flows

## Notes

- The reference files are provided for analysis and comparison.
- The spec is the source of truth for design decisions in this repository.
- Some reference assets are compiled snapshots rather than original source modules.
