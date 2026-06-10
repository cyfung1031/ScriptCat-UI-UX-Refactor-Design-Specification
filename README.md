# ScriptCat UI/UX Refactor Design Specification

This repository contains the design specification and reference material for a ScriptCat UI/UX refactor.

The repo is documentation-first. It is meant to help shape the product direction, not to serve as the application source code itself.

## Contents

- `SPEC.md`: the main UI/UX design specification for the refactor.
- `design/`: detailed per-page implementation specifications derived from `SPEC.md` (popup, manager, editor, install/confirm, import/export/update flows, settings, logs).
- `references/`: compiled snapshots from related userscript managers, UX analyses of each, and supporting notes for comparison.
- `draft/`: exploratory visual material, if present.

## How To Use This Repo

Start with `SPEC.md` to understand the intended interaction model and design constraints.

Use the reference snapshots under `references/` to compare:

- popup hierarchy
- script list organization
- editor workflows
- runtime visibility and error handling
- terminology and labels

The reference notes are intentionally explicit about which sources are preferred for UX direction.

## Scope

This repository focuses on the ScriptCat manager experience across desktop and mobile surfaces, including:

- installed script management
- current-page runtime visibility
- script editing and debugging
- update, import, export, and sync flows

## Notes

- The reference files are provided for analysis and comparison.
- The spec is the source of truth for design decisions in this repository.
- Some reference assets are compiled snapshots rather than original source modules.
