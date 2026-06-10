# Popup — Implementation Specification

The highest-frequency surface. It answers, in order: *what is affecting this page right now → is anything wrong → can I act in one click* (SPEC 3.2, 21).

## Entry points

- Click on the extension toolbar icon.
- **Icon badge** (set by background, not the popup): count of scripts running on the current page; hidden at 0. On script error in the current tab, badge background switches to `--sc-danger` with the count retained. (SPEC 3.2 DO; TM analysis §2.)

## Layout

Fixed **384 px wide** (4 px grid; within SPEC 6.1's 360–400 range). Max height **600 px**; the script list region is the only scroll container. No open animation (SPEC 16).

```text
┌──────────────────────────────────────┐
│ Header                         48 px │  logo+name · global switch · ⋮ menu
├──────────────────────────────────────┤
│ Page status strip              36 px │  favicon+domain · "N running" · errors
├──────────────────────────────────────┤
│ (contextual banner)         0/40 px  │  reload-needed / blacklist / injection-failed
├──────────────────────────────────────┤
│ ▾ Running on this page (3)           │  group header 28 px
│   [sw] Script name      [status] [✎ ⋮]   row 40 px
│   …                                  │
│ ▾ Background scripts (1)             │
│   [sw] Script name   [Running] [⏹ ⋮] │
│ ▸ Other scripts (24)                 │  collapsed by default
├──────────────────────────────────────┤
│ Footer                         40 px │  version · Get scripts · Open Manager
└──────────────────────────────────────┘
```

## Regions

### Header (48 px)

- Left: 20 px logo + product name (14 px, medium).
- Right: **global enable switch** (labeled by tooltip + `aria-label` "Enable ScriptCat"), then one `⋮` menu containing exactly: `Create new script`, `Get scripts`, `Settings`. Nothing else.
- **No docs/community/report-issue links here** — they move to Manager → Help (SC analysis §2 con; SPEC 27 Priority 1). Max two icon controls in the header (SPEC 3.2 DON'T).

### Page status strip (36 px)

- Page favicon (16 px) + domain (13 px, `--sc-text-1`, middle-truncated beyond ~28 chars with title attr).
- Right-aligned: `N running` (12 px, `--sc-text-2`); if errors exist, a danger chip `2 errors` that scrolls to and highlights the failing rows.

### Contextual banner (one at a time, highest severity wins)

| Condition | Style | Content + action |
| --- | --- | --- |
| Toggle changed, page stale | info / `--sc-primary` tint | "Reload to apply changes" + **Reload** button (generalizes 1.3.2's `click_to_reload`; SC analysis §2 pro) |
| Injection failed | warning | "Could not inject some scripts." + **Retry in auto mode** (VM analysis §1 pro; SPEC 14) |
| Page blacklisted | neutral | "ScriptCat is off for this site." + **Manage** link |

### Script groups

Group headers: 28 px, 12 px semibold `--sc-text-2`, chevron, **count in the header** (VM `data-totals` pattern; SPEC 21.2). Groups in order:

1. **Running on this page** — expanded, pinned first.
2. **Background scripts** — running/scheduled background scripts; ScriptCat differentiator, always its own labeled section (SPEC 3.2). Hidden only when the user has none at all.
3. **Other scripts** — collapsed by default; expanding reveals matched-but-disabled and the rest, visually muted but never below 4.5:1 text contrast (don't copy VM's opacity-only dimming).

Script row (40 px; 44 px effective hit area via padding on touch):

```text
[switch]  Script name (14px, --sc-text-1, 1 line, ellipsis)   [status badge]  [✎] [⋮]
          error line (12px, --sc-danger, only when error)
```

- **Switch** toggles enable; `aria-label`: "Enable ‹name›". Toggling may raise the reload banner; the row itself never reflows the popup width (fixed width, SPEC 6.1).
- **Status badge**: text label per SPEC 4.2 (`Running`, `Error`, `Scheduled 14:00`, `Stopped`). No bare colored dots.
- **Direct actions**: `✎ Edit` is a visible icon button (tooltip + aria-label), not buried in the menu (SC analysis §2 con; SPEC 18.1). Background rows swap `✎` position to include **Run once / Stop** (`▶`/`⏹`).
- **`⋮` menu**: `View logs`, `Check for updates`, `Exclude on this site`, divider, `Delete` (danger style, confirm dialog). Order fixed across all rows.
- **Error row**: badge `Error`, second line shows first error message (1 line, ellipsis); clicking it opens Logs filtered to that script (SPEC 14: direct path from error to logs).

### Many-scripts state (SPEC 21.2)

- When **Other scripts** > 10: a search field (32 px) appears pinned at the top of that group; filters all groups as typed.
- When total rows > 30: list virtualizes; footer of the group shows **View all in Manager →**.

### Footer (40 px)

`v1.x.x` (12 px, `--sc-text-2`) · **Get scripts for this site** (text link → script-site search for current domain) · **Open Manager** (text button, `--sc-primary`).

## States

- **No scripts installed (first run)**: logo, one sentence ("Scripts customize the websites you visit."), two buttons: **Get scripts** (primary), **Create script** — never bare "No data" (SPEC 15.3; SC analysis §2 con).
- **Scripts installed, none match page**: "No scripts run on this site." + **Find scripts for this site** link; Background/Other groups still shown.
- **Loading**: skeleton of 3 rows in group 1; layout identical to loaded state (SPEC 15.2).

## Keyboard & accessibility

- Tab order: header switch → menu → banner action → rows (switch → Edit → ⋮) → footer. `↑/↓` move row focus; `Space` toggles focused switch; `Enter` on row name opens editor; `Esc` closes popup.
- Focus ring per foundations — **never** `outline: none` (VM analysis §3 con).
- Group headers are buttons with `aria-expanded`; badge text is real text, not color (SPEC 17).

## Performance

CSS ≤ 50 KB; no Monaco/manager bundles; interactive < 100 ms; instant open (no entry animation). Counter-example: TM's popup loading the whole 434 KB product stylesheet (TM analysis §2).

## Traceability

SPEC 3.2, 6.1, 14, 15, 16, 17, 18, 21, 27-P1 · VM analysis §1 (grouping, counts, failure+fix, fixed width) · TM analysis §2 (badge; anti-pattern: mini-manager) · SC analysis §2 (header cleanup, click-to-reload, background section, direct Edit).
