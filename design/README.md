# Page-Level Design Implementation Specifications

These documents translate `SPEC.md` (rev. 4) into buildable, per-page specifications. Each spec is self-contained for the implementer of that page but defers all cross-cutting rules — the two-axis model, the kind/state/attention status system, the activity ledger, reversibility, terminology, accessibility, motion — to `SPEC.md`, cited as `SPEC §n`. Where a decision came from reference-product evidence, the spec cites `references/UX_ANALYSIS_*.md`.

HTML illustration boards for every surface live in `../draft/html/` (open `index.html`); each board cites the spec it visualizes.

## Documents

| Surface | File | Carries rev. 4 pillar |
| --- | --- | --- |
| Popup (site controller) | `POPUP.md` | Site axis (§3.1): per-site switch, develop-on-this-page |
| Manager (collection, desktop + mobile) | `MANAGER.md` | By-site view, command palette (§3.5), History & Trash (§3.4) |
| Script Hub (detail home + editor) | `SCRIPT_HUB.md` | Overview/Activity/Versions tabs (§3.3–3.4); editor as Code tab |
| Install / Update confirmation | `INSTALL_CONFIRM.md` | Declared contract (§3.3), escalation gating, rollback-backed updates |
| Import / Export / Batch update | `FLOWS_IMPORT_EXPORT_UPDATE.md` | Version-backed overwrites; honest summaries |
| Settings | `SETTINGS.md` | Privacy & activity group — the ledger/history contract |
| Logs | `LOGS.md` | Console surface; links into per-script Activity |

> rev. 4 note: `EDITOR.md` became `SCRIPT_HUB.md` — the script detail page is now the script's home and the editor is its Code tab (SPEC §7).

## Shared Foundations

All pages use the same tokens. The visual identity is **ScriptCat's own design system** — source of truth: `visual/scriptcat.pen`, rendered references in `visual/export/` — built around the ScriptCat brand blue and quiet neutral surfaces. Tokens are plain CSS custom properties. The design assumes **no third-party UI framework**: whatever component stack implements these pages must consume the tokens below, never impose its own palette, radii, or component anatomy.

### Color tokens (SPEC §11.2–11.3 · canonical values: `visual/scriptcat.pen`)

| Token | Light | Dark | Use |
| --- | --- | --- | --- |
| `--sc-primary` | `#1296DB` (brand blue) | `#3AACEF` | Primary actions, selection, active nav |
| `--sc-primary-hover` | `#0A7DB8` | — | Hover/pressed on primary controls |
| `--sc-primary-tint` | `#D6ECFA` | `#1E3040` | Selection background, active-nav tint; row-hover wash `#EDF5FC` / `#162430` |
| `--sc-success` | `#34C759` | same · tint `#1E3520` | Running, healthy; labeled Background kind chip |
| `--sc-warning` | `#FF9500` | same · tint `#352C1E` | Warning, pending, update waiting; labeled Scheduled kind chip |
| `--sc-danger` | `#E7000B` | `#FF6669` | Error, destructive |
| `--sc-bg-1` | `#FAFAFA` | `#1E1E1E` | App background — never pure black |
| `--sc-bg-2` | `#FFFFFF` (card & sidebar) | `#151515` card · `#1A1A1A` sidebar | Cards, panels, sidebar |
| `--sc-bg-3` | `#F0F0F0` | `#282828` | Inputs, muted fills |
| `--sc-text-1` | `#1A1A1A` | `#E5E5E5` | Primary text |
| `--sc-text-2` | `#666666` | `#B5B5B5` — dark-mode secondary text must not drop below AA ≥ 4.5:1 (SPEC §11.3, §15) | Secondary text |
| `--sc-text-3` | `#888888` | `#8A8A8A` | Muted text, placeholders |
| `--sc-border` | `#E5E5E5` light · `#D0D0D0` strong | `#2A2A2A` · hover `#3A3A3A`; borders carry separation in dark mode | Dividers, input borders |

Kind (Page · Background · Scheduled) has **no dedicated hue**: it renders as a small glyph plus a labeled tint chip — green tint for Background, orange tint for Scheduled. Status colors are never used without a text label; lists render one switch, one state word, at most one attention badge (SPEC §3.2).

### Layout tokens (SPEC §11.1)

```text
Spacing steps: 4 / 8 / 12 / 16 / 24 / 32 px
Radius: 6 px buttons, inputs, badges (sm) · 8 px controls & small cards (md) · 12 px cards & dialogs (lg) · full for chips & switches
Type: 14 px primary · 12 px secondary · 16 px page titles · 13 px monospace (code, logs, patterns)
Touch target: ≥ 44 × 44 px on touch surfaces
Focus ring: 2 px, --sc-primary, 2 px offset, ≥ 3:1 against adjacent colors — never removed
Breakpoints: <600 px mobile · 600–900 px condensed · >900 px full desktop
Motion: 100–150 ms micro / 150–250 ms panels / ease-out in, ease-in out / prefers-reduced-motion → fades
```

### Per-surface asset budgets (SPEC §17; counter-example: Tampermonkey's single 434 KB stylesheet, `UX_ANALYSIS_TAMPERMONKEY_5.5.0.md` §2)

| Surface | CSS budget | JS note |
| --- | --- | --- |
| Popup | ≤ 50 KB | No Monaco, no manager bundles; interactive < 100 ms |
| Manager / Hub | ≤ 150 KB | Monaco loads only when a Code tab opens |
| Trust pages | ≤ 100 KB | Read-only highlighter/diff may lazy-load below the fold |
| Activity ledger | — | ~zero runtime overhead: counters + domain sets, batched writes |

Locale data loads per-locale on demand (SPEC §16.2); never ship all languages to every surface (counter-example: ScriptCat 1.3.2's 236 KB bundled catalog).

### Spec template

Each page spec follows: **Purpose → Entry points → Layout → Regions & components → States → Keyboard & accessibility → Responsive behavior → Traceability**. Wireframes are ASCII; dimensions are the implementation default, not pixel-perfect mandates.
