# Logs — Implementation Specification

The cross-script console surface (SPEC §9). Three principles: errors elsewhere in the product always deep-link *into* a pre-filtered view here; this page complements — never replaces — the Script Hub's docked console (`SCRIPT_HUB.md`); and it stays a **console**, not the activity ledger — per-script behavioral data lives in the Hub's Activity tab (SPEC §3.3), and this page links there rather than duplicating it.

## Entry points

Sidebar → Logs · `Error` badges in Manager rows and popup rows (pre-filtered to that script, level=error) · Hub console "All logs →" · toast "View logs" actions · `Cmd/Ctrl+K` → "Logs". Deep-link params: `#script=<id>&level=error&t=24h`.

## Layout (desktop)

```text
┌────────┬──────────────────────────────────────────────────┐
│Sidebar │ Logs (1 248)   [Search…] [Script ▾][Level ▾][24h ▾]│
│        │ [⏸ Pause tail]                  [Export][Clear…]   │
│        ├──────────────────────────────────────────────────┤
│        │ 12:03:41  ERROR  Bilibili Evolved  TypeError: x…  │ 32 px rows
│        │ 12:03:40  INFO   AutoPagerize      page 3 loaded  │
│        │ …                                  (virtualized)  │
│        ├──────────────────────────────────────────────────┤
│        │ ▸ Detail panel (when a row is selected)            │
└────────┴──────────────────────────────────────────────────┘
```

## Toolbar

- **Search** message text; **Script** select (with per-script counts); **Level** chips `Error · Warn · Info` (multi-select, counts shown); **Time** select (1h/24h/7d/All). Active filters render as removable chips + one **Reset** (SPEC §14.2: no one-by-one manual clearing).
- **Live tail** is on by default, **auto-pauses on scroll-up** with a "Paused — N new ↓" pill that resumes on click. New rows never yank the user's reading position (SPEC §13.1: no jumping interfaces).
- **Export** (current filter result, ndjson/txt) · **Clear logs** — destructive and not trash-backed: confirm dialog states scope ("all logs" or "logs matching current filters"), red confirm button, not default-focused (SPEC §14.1).

## Log rows (32 px, virtualized; 13 px mono message)

`time (HH:MM:SS, title=full ISO) · level badge (text+color, SPEC §3.2/§15) · script name (link → Script Hub) · message (1 line, ellipsis)`. Error rows tint the row background at low alpha — the badge, not the tint, carries the meaning (never color-only). Repeated identical messages collapse into one row with a `×23` count chip (expandable).

## Detail panel (bottom split, 240 px, resizable; opens on row click)

Full message + stack trace (mono, wrapped, copyable) · context table (script, version, tab/URL when page-bound, run-id for background runs — run-id links to the matching entry in the script's **Activity** timeline, SPEC §3.3) · actions: **Open script at line** (jumps to the Hub's Code tab at source position — the direct error→editor path SPEC §13.2 requires) · **Copy** · (errors) **Report issue** prefilled (script name, version, browser, stack).

## States

- **No logs at all**: "Script activity will appear here." + one line on what gets logged + docs link (SPEC §13.1 — never blank, never technical-only).
- **No match for filters**: "No logs match" + **Reset filters**.
- **Logging disabled** (if Settings allows disabling levels): banner stating it, with **Logging settings →**.
- Retention note in the footer ("Logs are kept for 7 days / 10 000 entries — change in Settings → Privacy & activity"), so disappearance is explained, not mysterious.

## Keyboard & accessibility

`↑/↓` select row · `Enter` toggle detail · `e` open script at line · `/` search · `Esc` close detail. New-entries announcement throttled via `aria-live="polite"` only while tailing is visibly paused (avoid screen-reader flooding). Level badges are text; row tints decorative. Detail panel is `role="region"` labeled by the row.

## Responsive

<600 px (Logs is a bottom-nav destination, see `MANAGER.md`): toolbar collapses to search + one filter sheet button; rows two-line (time+level+script / message); detail opens as a bottom sheet; Export/Clear inside an overflow menu. Targets ≥ 44 px.

## Traceability

SPEC §3.2–3.3, §9, §13–15 · SC analysis §2/§4 (error→logs→editor loop; logs as first-class section) · VM analysis §1 (surfacing failure with a path to the fix) · TM analysis §3 (utilities/maintenance separated from reading flow).
