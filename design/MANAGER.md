# Manager (Script List) — Implementation Specification

The desktop home of script management; becomes the card-based mobile list below 600 px. Covers the **Installed Scripts** page plus the **Subscriptions** variant. (SPEC 3.1, 22, 23.)

## Entry points

Popup → *Open Manager* · extension options page · deep links from error badges, toasts, and the editor's back button. Deep links may carry a pre-applied filter (e.g. `#errors`, `#script=<id>` scroll-and-highlight).

## Layout — desktop (>900 px)

```text
┌────────┬───────────────────────────────────────────────┐
│Sidebar │ Header: "Installed Scripts (37)"   [＋ New Script]│ 56 px
│ 220 px │ Toolbar: [Search… 280px][chips… ][sort][density]│ 48 px
│        │   — or, when rows selected —                    │
│        │ Batch bar: "3 selected" [Enable][Disable]       │ 48 px
│        │           [Update][Export] │ [Delete]  [Cancel] │
│        ├───────────────────────────────────────────────┤
│        │ Script table (scrolls)                          │
│        ├───────────────────────────────────────────────┤
│        │ (status footer: counts · last update check)     │ 28 px
└────────┴───────────────────────────────────────────────┘
```

### Sidebar (220 px; 64 px icon rail at 600–900 px)

Top: **Installed Scripts · Subscriptions · Logs · Tools · Settings** — icon + label, active item = `--sc-primary` tint bar + bold (SPEC 8.1). Bottom (separated): **Help** (docs/community/report issue relocated from popup — SC analysis §2) and version. No external links mixed into the top group (SPEC 8.1 DON'T).

### Header

Page title + total count (16 px). Right: **＋ New Script** — the single most prominent action on the page (solid `--sc-primary`; SPEC 3.1). Next to it a `⋮` menu grouped per SPEC 8.2: *Create* (New script, Get scripts) / *Maintain* (Check updates, Import, Export) / no Help here (sidebar owns it).

### Toolbar (default mode)

- **Search** (280 px, `/` focuses): matches name, description, author, domain. Persistent (SPEC 9).
- **Filter chips**: `Enabled` · `Errors` · `Update available` · `Background` · `Scheduled` · per-tag. Selected = filled chip; active filters always visible; one **Reset** chip clears all (SPEC 9). Result count replaces total in the title while filtering.
- **Sort** dropdown: Name / Status / Last updated / Recently used. Persisted.
- **Density toggle**: comfortable 48 px rows / compact 36 px rows; persisted (SPEC 22; VM analysis §2 pro).

### Batch bar (selection mode — replaces toolbar, same height, no layout jump)

`N selected` · **Enable · Disable · Update · Export** · divider + 16 px gap · **Delete** (danger ghost) · **Cancel**. Appears only on selection (SPEC 10; SC analysis §3 con: no persistent batch buttons). Delete → dialog listing script names → on confirm, toast **"3 scripts deleted — Undo"** (10 s); deleted scripts are restorable from Tools for 30 days (SPEC 18.2; VM restore lesson).

### Script table

Default columns (priority order per SPEC 4.1; optional columns via column-config menu: Version, Author, Type, Source):

| Col | Width | Content |
| --- | --- | --- |
| ☑ | 40 | Row checkbox; header = select-all-filtered |
| Switch | 56 | Enable/disable, `aria-label` "Enable ‹name›" |
| Name | flex ≥ 320 | 14 px medium `--sc-text-1`; description 12 px `--sc-text-2` beneath (comfortable mode only); ≤ 2 tag chips, `+N` overflow |
| Status | 130 | One labeled badge: `Running` / `Error` / `Update` / `Scheduled 14:00` / `Disabled` (muted text, no badge). Never a bare dot (SPEC 4.2, 27-P3) |
| Updated | 110 | Relative time, title = absolute |
| Actions | 96 | `✎ Edit` (always visible, not hover-only — VM analysis §2 con) + `⋮`: Run once/Stop (background), View logs, Check update, Export, divider, Delete |

Row interactions: click name → editor; click `Error` badge → Logs filtered to script; hover = `--sc-bg-2`; selected = primary tint + visible checkbox state (not background-only, SPEC 7.1). The table must read as *name → state → action*, not a uniform data grid (SPEC 4.1; TM analysis §3 con, SC analysis §3 con).

## States

- **Empty (no scripts)**: first-run state — illustration ≤ 120 px, one sentence, **Get scripts** + **Create script**, docs link (SPEC 15.3).
- **No search/filter results**: "No scripts match ‹query›" + **Reset filters**.
- **Loading**: skeleton rows at the active density; no layout shift (SPEC 15.2).
- **Update check running**: progress in status footer; per-row spinners only on affected rows; never blocks the table (SPEC 13.2).

## Keyboard & accessibility

`↑/↓` row focus · `Space` select · `Enter` open editor · `e` edit · `Del` delete (dialog) · `/` search · `Esc` clear selection. Checkbox/switch/buttons all reachable in row tab order; sort headers are buttons with `aria-sort`. Density/compact mode must keep 4.5:1 text contrast.

## Responsive behavior

- **600–900 px**: sidebar → icon rail; columns drop in reverse priority (Updated first, then tags); search collapses to icon.
- **<600 px (mobile, per draft mockup)**: bottom nav (Scripts · Logs · Settings, 56 px, persistent); table → **cards**; FAB or header button for New Script; filter chips scroll horizontally (SPEC 23).

Mobile card (≥ 44 px targets, SPEC 6.1):

```text
┌──────────────────────────────────┐
│ [switch]  Script name    [badge] │
│ description, 1–2 lines            │
│ [Edit]   [Run]          [⋮ More] │  ← text-labeled buttons
└──────────────────────────────────┘
```

Long-press = selection mode → batch bar slides up above bottom nav. No hover-dependent affordances (SPEC 23).

## Subscriptions variant

Same shell, simpler list: name+source URL / scripts contained (expandable) / last checked / actions (Check now · ⋮: visit page, divider, Remove — confirm dialog states the contained scripts are also removed, SPEC 18.2). Empty state explains what subscriptions are with a docs link (SPEC 15.1).

## Traceability

SPEC 3.1, 4.1–4.2, 6, 8–10, 11.1–11.2, 15, 17, 18.2, 22–23, 27-P2/P3/P5 · TM analysis §3 (scale capabilities; anti-pattern: data-grid, inline trash) · VM analysis §2 (density toggle, restore, hover-only con) · SC analysis §3 (contextual batch, dot removal).
