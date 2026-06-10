# Popup — Site Controller — Implementation Specification

The highest-frequency surface and the physical form of the site axis (SPEC §3.1, §5). It answers, in order: *what is affecting this page right now → is anything wrong → can I act in one click*.

## Entry points

- Click on the extension toolbar icon.
- **Icon badge** (set by background, not the popup): count of scripts running on the current page; hidden at 0. Attention severity recolors it — `--sc-danger` on error in the current tab, count retained (SPEC §3.2 roll-up, §5.1).

## Layout

Fixed **384 px wide** (4 px grid; within SPEC §11.1's 360–400 range). Max height **600 px**; the script list region is the only scroll container. No open animation (SPEC §12).

```text
┌──────────────────────────────────────┐
│ Header                         48 px │  logo+name · global switch · ⋮ menu
├──────────────────────────────────────┤
│ Site strip                     40 px │  favicon+domain · per-site switch · "N running" · errors
├──────────────────────────────────────┤
│ (contextual banner)         0/40 px  │  reload-needed / injection-failed / site off
├──────────────────────────────────────┤
│ ▾ Running on this page (3)           │  group header 28 px
│   [sw] Script name      Running [✎ ⋮]│  row 40 px
│   …                                  │
│ ▾ Background & scheduled (2)         │
│   [sw] Script name   Running  [⏹ ⋮]  │
│   [sw] Script name   Idle · 14:00 [▶ ⋮]
│ ▸ Other scripts (24)                 │  collapsed by default
├──────────────────────────────────────┤
│ Footer                         40 px │  version · Get scripts · Open Manager
└──────────────────────────────────────┘
```

## Regions

### Header (48 px)

- Left: 20 px logo + product name (14 px, medium).
- Right: **global enable switch** (tooltip + `aria-label` "Enable ScriptCat"), then one `⋮` menu containing exactly: `Create new script`, `Develop on this page`, `Get scripts`, `Settings`.
- **Develop on this page** creates a scratch script with `@match` prefilled for the current origin and opens the Script Hub at the Code tab (SPEC §5.1) — the fastest path from "this site annoys me" to a fix.
- No docs/community/report-issue links here — they live in Manager → Help (SPEC §4). Max two icon controls in the header (SPEC §5.1 DON'T).

### Site strip (40 px) — the per-site control plane

- Page favicon (16 px) + domain (13 px, `--sc-text-1`, middle-truncated beyond ~28 chars with title attr).
- **Per-site switch** (`aria-label` "Run scripts on ‹domain›"): one action turns ScriptCat off for this origin, remembered (SPEC §3.1). Off state collapses the script groups and shows the "site off" banner with **Turn back on**. This replaces per-script exclude as the primary "leave this site alone" gesture.
- Right-aligned: `N running` (12 px, `--sc-text-2`); if errors exist, a danger chip `2 errors` that scrolls to and highlights the failing rows.

### Contextual banner (one at a time, highest severity wins)

| Condition | Style | Content + action |
| --- | --- | --- |
| Toggle changed, page stale | info / `--sc-primary` tint | "Reload to apply changes" + **Reload** button |
| Injection failed | warning | "Could not inject some scripts." + **Retry in auto mode** (SPEC §13.2) |
| Site switched off | neutral | "ScriptCat is off on ‹domain›." + **Turn back on** |

### Script groups

Group headers: 28 px, 12 px semibold `--sc-text-2`, chevron, count in the header (SPEC §5.1). Groups in order:

1. **Running on this page** — expanded, pinned first.
2. **Background & scheduled** — ScriptCat differentiator, always its own labeled section (SPEC §5.1). Hidden only when the user has none at all.
3. **Other scripts** — collapsed by default; expanding reveals matched-but-disabled and the rest, visually muted but never below 4.5:1 text contrast (SPEC §15).

Script row (40 px; 44 px effective hit area via padding on touch), per the SPEC §3.2 anatomy — one switch, one state word, at most one attention badge:

```text
[switch]  Script name (14px, --sc-text-1, 1 line, ellipsis)   state/attention   [✎] [⋮]
          error line (12px, --sc-danger, only when error)
```

- **Switch** carries Off; state word carries `Running` / `Idle` / `Idle · 14:00` (scheduled next-run); attention badge (`Error`, `Update waiting`) replaces the state word when present — never both, never a stack.
- **Switch** toggling may raise the reload banner; the row never reflows the popup width (SPEC §11.1).
- **Direct actions**: `✎ Edit` is a visible icon button (tooltip + aria-label) opening the Script Hub. Background/scheduled rows show **Run once / Stop** (`▶`/`⏹`) in this slot.
- **`⋮` menu**: `View activity` (ledger glance, SPEC §3.3 — opens Hub → Activity; long-press/hover shows the one-line summary "Today: ran 3×, 14 requests to api.example.com"), `View logs`, `Check for updates`, `Exclude on this site` (the per-script fallback; per-site switch is the primary control), divider, `Delete` (danger; deletes to Trash with undo toast — no dialog, SPEC §3.4, §14.1). Order fixed across all rows.
- **Error row**: badge `Error`, second line shows the first error message (1 line, ellipsis); clicking it opens Logs filtered to that script (SPEC §13.2).

### Many-scripts state (SPEC §5.2)

- When **Other scripts** > 10: a search field (32 px) appears pinned at the top of that group; filters all groups as typed.
- When total rows > 30: list virtualizes; footer of the group shows **View all in Manager →**.
- No command palette here — the popup stays instant and minimal (SPEC §3.5 DON'T).

### Footer (40 px)

`v1.x.x` (12 px, `--sc-text-2`) · **Get scripts for this site** (text link → script-site search for current domain) · **Open Manager** (text button, `--sc-primary`).

## States

- **No scripts installed (first run)**: logo, one sentence ("Scripts customize the websites you visit."), two buttons: **Get scripts** (primary), **Create script** — never bare "No data" (SPEC §13.1).
- **Scripts installed, none match page**: "No scripts run on this site." + **Find scripts for this site** link; Background and Other groups still shown.
- **Loading**: skeleton of 3 rows in group 1; layout identical to loaded state (SPEC §13.1).

## Keyboard & accessibility

- Tab order: header switch → menu → site switch → banner action → rows (switch → Edit → ⋮) → footer. `↑/↓` move row focus; `Space` toggles focused switch; `Enter` on row name opens the Hub; `Esc` closes popup.
- Focus ring per SPEC §11.1 — never `outline: none`.
- Group headers are buttons with `aria-expanded`; state words and badges are real text, not color (SPEC §3.2, §15).

## Performance

CSS ≤ 50 KB; no Monaco/manager bundles; interactive < 100 ms; instant open (SPEC §17).

## Traceability

SPEC §2, §3.1–3.4, §5, §11–13, §15, §17, §18-P2 · VM analysis §1 (grouping, counts, failure+fix, fixed width) · TM analysis §2 (badge; anti-pattern: mini-manager) · SC analysis §2 (header cleanup, click-to-reload, background section, direct Edit).
