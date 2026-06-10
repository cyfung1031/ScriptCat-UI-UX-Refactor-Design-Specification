# Logs — Implementation Specification

The cross-script investigation surface. Two principles: errors elsewhere in the product always deep-link *into* a pre-filtered view here (SPEC 14), and this page complements — never replaces — the editor's docked console (`EDITOR.md`; SPEC 24).

## Entry points

Sidebar → Logs · `Error` badges in Manager rows and popup rows (pre-filtered to that script, level=error) · editor Logs tab "All logs →" · toast "View logs" actions. Deep-link params: `#script=<id>&level=error&t=24h`.

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

- **Search** message text; **Script** select (with per-script counts); **Level** chips `Error · Warn · Info` (multi-select, counts shown); **Time** select (1h/24h/7d/All). Active filters render as removable chips + one **Reset** (SPEC 9: no one-by-one manual clearing).
- **Live tail** is on by default, **auto-pauses on scroll-up** with a "Paused — N new ↓" pill that resumes on click. New rows never yank the user's reading position (SPEC 15.2: no jumping interfaces).
- **Export** (current filter result, ndjson/txt) · **Clear logs** — destructive: confirm dialog states scope ("all logs" or "logs matching current filters"), red confirm button, not default-focused (SPEC 18.2).

## Log rows (32 px, virtualized; 13 px mono message)

`time (HH:MM:SS, title=full ISO) · level badge (text+color, SPEC 4.2) · script name (link → editor) · message (1 line, ellipsis)`. Error rows tint the row background at low alpha — the badge, not the tint, carries the meaning (never color-only, SPEC 17). Repeated identical messages collapse into one row with a `×23` count chip (expandable).

## Detail panel (bottom split, 240 px, resizable; opens on row click)

Full message + stack trace (mono, wrapped, copyable) · context table (script, version, tab/URL when page-bound, run-id for background runs) · actions: **Open script at line** (jumps to editor Code tab at source position — the direct error→editor path SPEC 14 requires) · **Copy** · (errors) **Report issue** prefilled (script name, version, browser, stack) — the relocated, purposeful home of the old popup "report issue" link (SC analysis §2).

## States

- **No logs at all**: "Script activity will appear here." + one line on what gets logged + docs link (SPEC 15.1 — never blank, never technical-only).
- **No match for filters**: "No logs match" + **Reset filters**.
- **Logging disabled** (if Settings allows disabling levels): banner stating it, with **Logging settings →**.
- Retention note in the footer ("Logs are kept for 7 days / 10 000 entries — change in Settings"), so disappearance is explained, not mysterious.

## Keyboard & accessibility

`↑/↓` select row · `Enter` toggle detail · `e` open script at line · `/` search · `Esc` close detail. New-entries announcement throttled via `aria-live="polite"` only while tailing is visible-paused (avoid screen-reader flooding). Level badges are text; row tints decorative. Detail panel is `role="region"` labeled by the row.

## Responsive

<600 px (Logs is a bottom-nav destination, see `MANAGER.md`): toolbar collapses to search + one filter sheet button; rows two-line (time+level+script / message); detail opens as a bottom sheet; Export/Clear inside an overflow menu. Targets ≥ 44 px.

## Traceability

SPEC 4.2, 9, 14, 15, 17, 18.2, 24 · SC analysis §2/§4 (error→logs→editor loop; logs as first-class section) · VM analysis §1 (surfacing failure with a path to the fix) · TM analysis §3 (utilities/maintenance separated from reading flow).
