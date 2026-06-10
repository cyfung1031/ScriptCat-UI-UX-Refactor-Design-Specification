# Manager (Script Collection) — Implementation Specification

The script axis's home (SPEC §3.1, §6): a table on desktop, cards on mobile, with the **By script / By site** view toggle, contextual batch bar, command palette, and the History & Trash section. Covers **Installed Scripts**, **History & Trash**, and the **Subscriptions** variant.

## Entry points

Popup → *Open Manager* · extension options page · deep links from error badges, toasts, and the Hub's back button · `Cmd/Ctrl+K` palette navigation. Deep links may carry a pre-applied filter (e.g. `#errors`, `#site=<origin>`, `#script=<id>` scroll-and-highlight).

## Layout — desktop (>900 px)

```text
┌────────┬───────────────────────────────────────────────────┐
│Sidebar │ Header: "Installed Scripts (37)"     [⌘K] [＋ New Script]│ 56 px
│ 220 px │ Toolbar: [Search… 280px][chips…][view: Script|Site]│ 48 px
│        │          [sort][density]                           │
│        │   — or, when rows selected —                       │
│        │ Batch bar: "3 selected" [Enable][Disable]          │ 48 px
│        │           [Update][Export] │ [Delete]   [Cancel]   │
│        ├───────────────────────────────────────────────────┤
│        │ Script table / site groups (scrolls)               │
│        ├───────────────────────────────────────────────────┤
│        │ status footer: counts · last update check · sync   │ 28 px
└────────┴───────────────────────────────────────────────────┘
```

### Sidebar (220 px; 64 px icon rail at 600–900 px)

Top: **Installed Scripts · Subscriptions · Logs · History & Trash · Settings** — icon + label, active item = `--sc-primary` tint bar + bold (SPEC §4). Bottom (separated): **Help** (docs/community/report issue) and version. No external links mixed into the top group.

### Header

Page title + total count (16 px). Right: a subtle `⌘K` affordance (opens the palette; SPEC §3.5) and **＋ New Script** — the single most prominent action on the page (solid `--sc-primary`). Next to it a `⋮` menu grouped per SPEC §14.3: *Create* (New script, Get scripts) / *Maintain* (Check updates, Import, Export).

### Command palette (SPEC §3.5)

`Cmd/Ctrl+K` from anywhere in the Manager or Hub. One input mixing actions ("Check all updates", "New script", "Enable ‹script›"), navigation ("Open ‹script›", "Settings → Sync", "History & Trash"), and script search. Collection commands rank first in the Manager. Entries show their shortcuts — the palette doubles as the shortcut reference. Every palette command also exists in visible UI; nothing is palette-only.

### Toolbar (default mode)

- **Search** (280 px, `/` focuses): matches name, description, author, domain. Persistent (SPEC §14.2).
- **Filter chips**: `Enabled` · `Errors` · `Update waiting` · `Background` · `Scheduled` · per-tag. Selected = filled chip; one **Reset** chip clears all. Result count replaces total in the title while filtering.
- **View toggle**: **By script** (flat table, default) / **By site** (SPEC §3.1) — persisted.
- **Sort** dropdown: Name / State / Last updated / Recently used. Persisted.
- **Density toggle**: comfortable 48 px rows / compact 36 px rows; persisted (SPEC §11.1).

### By-site view

Rows grouped under origin headers:

```text
▾ [favicon] github.com   [site switch]                    4 scripts
    (script rows, identical anatomy to the flat table)
▾ [favicon] *. all websites ⚠                              2 scripts
▸ Background & scheduled (no site)                         3 scripts
```

- Origin header: favicon + domain + **per-site switch** (same remembered state the popup's site strip controls, SPEC §5.1) + count.
- A script matching several sites appears under each origin; all instances act on the same underlying script and update together.
- All-sites scripts group under "all websites" with the warning glyph from the trust surfaces (SPEC §8.1); background/scheduled scripts group last under their own header.
- Row anatomy, actions, selection, and batch behavior are identical to the flat table — the view changes grouping, never grammar (SPEC §3.1 DON'T).

### Batch bar (selection mode — replaces toolbar, same height, no layout jump)

`N selected` · **Enable · Disable · Update · Export** · divider + 16 px gap · **Delete** (danger ghost) · **Cancel**. Appears only on selection (SPEC §6.1). Delete goes to **Trash** — no dialog for reversible destruction (SPEC §3.4, §14.1): toast **"3 scripts moved to Trash — Undo"** (10 s); restorable for 30 days from History & Trash.

### Script table (By-script view)

Default columns (priority order per SPEC §6.1; optional via column-config menu: Version, Author, Kind, Source):

| Col | Width | Content |
| --- | --- | --- |
| ☑ | 40 | Row checkbox; header = select-all-filtered |
| Switch | 56 | Enable/disable, `aria-label` "Enable ‹name›" |
| Name | flex ≥ 320 | 14 px medium `--sc-text-1`; description 12 px `--sc-text-2` beneath (comfortable mode only); ≤ 2 tag chips, `+N` overflow; small kind glyph for Background/Scheduled |
| Status | 130 | SPEC §3.2: one state word (`Running` / `Idle` / `Idle · 14:00`) **or** one attention badge (`Error` / `Update waiting` / `Warning`) — highest severity wins; Off = muted text via the switch, no badge. Never a dot, never a stack |
| Updated | 110 | Relative time, title = absolute |
| Actions | 96 | `✎ Edit` (always visible, not hover-only) + `⋮`: Run once/Stop (background), View activity, View logs, Check update, Export, divider, Delete |

Row interactions: click name → **Script Hub (Overview tab)**; click an attention badge → the relevant target (Error → Logs filtered; Update waiting → update review); hover = `--sc-bg-2`; selected = primary tint + visible checkbox state. The table must read as *name → state → action*, not a uniform data grid (SPEC §6.1 DON'T).

## History & Trash (sidebar entry — SPEC §6.2, §3.4)

Two sections on one page, retention rules stated in the page footer ("Trash kept 30 days · versions kept 90 days / 20 per script — change in Settings"):

- **Trash**: deleted scripts with deletion date and source; **Restore** (returns script, storage, and history) · **Delete permanently** (confirm dialog — this is the non-reversible tier, SPEC §14.1). Empty state: "Deleted scripts stay here for 30 days."
- **Recent changes**: cross-collection feed of applied updates, imports, and rollbacks; each row links to that script's **Versions** tab in the Hub; update rows offer inline **Roll back** while within the toast-grace window and beyond.

## States

- **Empty (no scripts)**: first-run state — illustration ≤ 120 px, one sentence, **Get scripts** + **Create script**, docs link (SPEC §13.1).
- **No search/filter results**: "No scripts match ‹query›" + **Reset filters**.
- **Loading**: skeleton rows at the active density; no layout shift.
- **Update check running**: progress in status footer (`Checking 12/37…`); per-row spinners only on affected rows; never blocks the table (SPEC §8.3).

## Keyboard & accessibility

`Cmd/Ctrl+K` palette · `↑/↓` row focus · `Space` select · `Enter` open Hub · `e` edit (Code tab) · `Del` move to Trash · `/` search · `Esc` clear selection. Checkbox/switch/buttons all reachable in row tab order; sort headers are buttons with `aria-sort`; site-group headers are buttons with `aria-expanded`. Compact density keeps 4.5:1 text contrast (SPEC §15).

## Responsive behavior

- **600–900 px**: sidebar → icon rail; columns drop in reverse priority (Updated first, then tags); search collapses to icon.
- **<600 px (mobile)**: bottom nav (Scripts · Logs · Settings, 56 px, persistent); table → **cards**; the By script/By site toggle survives as a segmented control under the title; FAB or header button for New Script; filter chips scroll horizontally (SPEC §6.3).

Mobile card (≥ 44 px targets):

```text
┌──────────────────────────────────┐
│ [switch]  Script name     Running│
│ description, 1–2 lines           │
│ [Edit]   [Run]          [⋮ More] │  ← text-labeled buttons
└──────────────────────────────────┘
```

Long-press = selection mode → batch bar slides up above bottom nav. No hover-dependent affordances. No palette on mobile; the header search covers navigation.

## Subscriptions variant

Same shell, simpler list: name+source URL / scripts contained (expandable) / last checked / actions (Check now · ⋮: visit page, divider, Remove — confirm dialog states the contained scripts are also removed and go to Trash, SPEC §3.4). Empty state explains what subscriptions are with a docs link.

## Traceability

SPEC §3.1–3.5, §4, §6, §8.3, §11, §13–15, §18-P1/P4 · TM analysis §3 (scale capabilities; anti-pattern: data-grid, inline trash) · VM analysis §2 (density toggle, restore, hover-only con) · SC analysis §3 (contextual batch, dot removal).
