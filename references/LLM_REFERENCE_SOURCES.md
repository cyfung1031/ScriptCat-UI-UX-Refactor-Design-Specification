# LLM Reference Sources

This directory contains compiled extension snapshots that can be used as reference material when improving `SPEC.md`, the ScriptCat UI/UX refactor design specification.

Use these files to understand existing userscript-manager products, interaction patterns, information architecture, terminology, and feature coverage. Do not treat them as code to copy directly or as mandatory implementation requirements.

For this refactor, `violentmonkey-2.41.0/` is the stronger UX reference than `tampermonkey-5.5.0/`. Violentmonkey is closer to the kind of clear, compact, task-focused experience we want, while Tampermonkey is still useful for breadth of features and edge cases but is less aligned with the target UI/UX direction.

## Included Snapshots

| Directory | Product | Version | Best use |
| --- | --- | --- | --- |
| `scriptcat-1.3.2/` | ScriptCat | 1.3.2 | Baseline product behavior, current UI surfaces, terminology, and feature scope. |
| `tampermonkey-5.5.0/` | Tampermonkey | 5.5.0 | Reference for breadth, legacy expectations, advanced workflows, and edge-case coverage. |
| `violentmonkey-2.41.0/` | Violentmonkey | 2.41.0 | Preferred UX reference for clearer information architecture, simpler scanning, cleaner hierarchy, and less clutter. |

## How To Use These References

When editing `SPEC.md`, compare the reference products for:

- Popup hierarchy and current-page script visibility.
- Script list layout, filtering, sorting, grouping, and batch actions.
- Editor and developer workflow affordances.
- Install, update, import, export, sync, and subscription flows.
- Runtime status, error reporting, logs, permissions, and security prompts.
- Labels and user-facing terminology from `_locales/*/messages.json`.

Prefer extracting durable UI/UX lessons over mirroring exact layouts. The goal is to make ScriptCat clearer and more modern while preserving the power expected from a userscript manager.

When comparing Tampermonkey and Violentmonkey, favor the patterns that help an LLM infer:

- which page or view matters first
- what the primary action is
- which information is essential versus secondary
- how runtime state is surfaced without forcing too much scanning
- how to reduce visual noise while keeping advanced capability available

In practical terms, Violentmonkey is the better baseline when the question is "what should this interface prioritize?", and Tampermonkey is the better source when the question is "what additional capabilities or legacy behaviors should still be acknowledged?"

## Source-Of-Truth Rules

- `SPEC.md` is the design source of truth for this repository.
- Files in this directory are reference evidence only.
- Compiled JavaScript and CSS may be minified, bundled, or hard to trace back to original source modules.
- Product behavior may differ across browser versions, extension permissions, and runtime state.
- If references disagree, describe the design tradeoff in `SPEC.md` instead of blindly choosing one product's behavior.

## LLM Reading Order

For design-spec work, inspect the references in this order:

1. `scriptcat-1.3.2/` to understand the product being refactored.
2. `violentmonkey-2.41.0/` to establish the preferred UX baseline for clarity and hierarchy.
3. `tampermonkey-5.5.0/` to check feature breadth, compatibility expectations, and mature edge cases.

After reading, update `SPEC.md` with explicit recommendations, constraints, and rationale. Avoid vague phrases such as "make it better" unless they are backed by specific UI behavior.
