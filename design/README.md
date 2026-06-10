# Page-Level Design Implementation Specifications

These documents translate `SPEC.md` (rev. 3) into buildable, per-page specifications. Each spec is self-contained for the implementer of that page but defers all cross-cutting rules (status system, terminology, accessibility, motion) to `SPEC.md`. Where a decision came from reference-product evidence, the spec cites `references/UX_ANALYSIS_*.md`.

## Documents

| Page | File | Draft mockup coverage |
| --- | --- | --- |
| Popup | `POPUP.md` | Popup Light/Dark, Many-Scripts |
| Manager (script list, desktop + mobile) | `MANAGER.md` | Main Page v2 Light/Dark, Mobile Script List |
| Editor (desktop + mobile) | `EDITOR.md` | Mobile Editor Light/Dark |
| Install / Update confirmation | `INSTALL_CONFIRM.md` | — |
| Import / Export / Batch update | `FLOWS_IMPORT_EXPORT_UPDATE.md` | — |
| Settings | `SETTINGS.md` | — |
| Logs | `LOGS.md` | — |

## Shared Foundations

All pages use the same tokens. ScriptCat is built on Arco Design; semantic tokens map onto Arco palette variables so implementation is a theme configuration, not a fork (see `references/UX_ANALYSIS_SCRIPTCAT_1.3.2.md` §1).

### Color tokens (SPEC 5, 7)

| Token | Light | Dark | Use |
| --- | --- | --- | --- |
| `--sc-primary` | `#165DFF` (arcoblue-6) | `#3C7EFF` | Primary actions, selection, active nav |
| `--sc-success` | `#00B42A` (green-6) | `#27C346` | Running, healthy |
| `--sc-warning` | `#FF7D00` (orange-6) | `#FF9A2E` | Warning, pending, update available |
| `--sc-danger` | `#F53F3F` (red-6) | `#F76560` | Error, destructive |
| `--sc-scheduled` | `#722ED1` (purple-6) | `#8E51DA` | Scheduled / background category |
| `--sc-bg-1/2/3` | white → gray layers | `#17171A` base, never pure black | Surface layers |
| `--sc-text-1` | `gray-10` | ≥ 4.5:1 on `--sc-bg-1` | Primary text |
| `--sc-text-2` | `gray-8` | ≥ 4.5:1 — dark mode secondary text must not drop below AA (SPEC 7.2, 17) | Secondary text |
| `--sc-border` | `gray-3` | `gray-7` equivalent; borders carry separation in dark mode | Dividers |

Status colors are never used without a text label (SPEC 4.2).

### Layout tokens (SPEC 6)

```text
Spacing steps: 4 / 8 / 12 / 16 / 24 / 32 px
Radius: 8 px cards & dialogs · 4 px buttons, inputs, badges · full for chips & switches
Type: 14 px primary · 12 px secondary · 16 px page titles · 13 px monospace (code, logs, patterns)
Touch target: ≥ 44 × 44 px on touch surfaces
Focus ring: 2 px, --sc-primary, 2 px offset, ≥ 3:1 against adjacent colors — never removed
Breakpoints: <600 px mobile · 600–900 px condensed · >900 px full desktop
Motion: 100–150 ms micro / 150–250 ms panels / ease-out in, ease-in out / prefers-reduced-motion → fades
```

### Per-surface asset budgets (SPEC 6.1; counter-example: Tampermonkey's single 434 KB stylesheet, `UX_ANALYSIS_TAMPERMONKEY_5.5.0.md` §2)

| Surface | CSS budget | JS note |
| --- | --- | --- |
| Popup | ≤ 50 KB | No Monaco, no manager bundles; interactive < 100 ms |
| Manager | ≤ 150 KB | Monaco loads only when the editor opens |
| Install/confirm | ≤ 100 KB | Read-only highlighter may lazy-load below the fold |

Locale data loads per-locale on demand (SPEC 20); never ship all languages to every page (counter-example: ScriptCat 1.3.2's 236 KB bundled catalog).

### Spec template

Each page spec follows: **Purpose → Entry points → Layout → Regions & components → States → Keyboard & accessibility → Responsive behavior → Traceability**. Wireframes are ASCII; dimensions are the implementation default, not pixel-perfect mandates.
