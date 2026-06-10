# Editor — Implementation Specification

The developer surface. Monaco stays (ScriptCat's strongest asset vs. CodeMirror competitors — SC analysis §1/§4); the refactor fixes chrome, save-state visibility, and the debug loop. (SPEC 12, 24.)

## Entry points

Manager row name / Edit · popup row Edit · error deep links (`#script=<id>&line=<n>` opens **Code** at that line) · New Script (opens with the user's template, cursor inside the metadata block).

## Layout — desktop

```text
┌──────────────────────────────────────────────────────────┐
│ ← Back · Script name ●        Saved 12:03   [▶ Run][Save ⋮]│ 48 px top bar
├──────────────────────────────────────────────────────────┤
│ Code │ Settings │ Storage │ Resources │ Logs              │ 36 px tabs
├──────────────────────────────────────────────────────────┤
│                                                          │
│                Monaco editor (fills)                      │
│                                                          │
├──────────────────────────────────────────────────────────┤
│ ⚠ 2 problems · Ln 14, Col 2 · UTF-8        [Console ▴]   │ 28 px status bar
├──────────────────────────────────────────────────────────┤
│ (dockable log/console panel, 200–400 px, resizable)       │
└──────────────────────────────────────────────────────────┘
```

### Top bar

- **Back** returns to Manager (guarding unsaved changes — see Save below).
- **Script name** from metadata; `●` dirty dot plus explicit **save-state text**: `Saved HH:MM` / `Unsaved changes` / `Saving…` — state is never ambiguous and never icon-only (SPEC 12; mobile mockup shows the same).
- **▶ Run** (primary): page scripts → toast "Applied — reload the matching tab to see changes" with **Reload tab** action when one is open; background scripts → executes once, auto-opens the docked log panel filtered to this run (SPEC 24: quick run-and-test).
- **Save** + `⋮`: Export, Check update, Copy install URL, divider, Delete (danger, confirm).

### Tabs

One scroll state each; switching never loses edits.

- **Code** — Monaco, ScriptCat API typings, metadata-block completion. **Metadata lint**: invalid `@match`, missing `@grant`, unknown keys surface as warnings in the problems strip, not only as silent failure later (SPEC 12, 24).
- **Settings** — form, common-first with **Advanced collapsed** (contextual disclosure, not a global mode — TM analysis §1 con): Enabled, Run at, Matches/Excludes (pattern list editor with live "matches example.com?" tester), Update URL, Permissions; Advanced: sandbox options, require/resource overrides.
- **Storage** — key/value table (13 px mono), inline edit, type column, Clear all (danger, confirm). Empty state explains what script storage is.
- **Resources** — `@require`/`@resource` list: URL, cached size/time, per-row re-fetch; failures show inline with retry (SPEC 14).
- **Logs** — this script's logs only; same component as the docked panel; link "All logs →" to the Logs page.

### Docked console panel (the edit-run loop)

Toggled by status bar **Console** / auto-opened by background **Run** / `Ctrl+`` `. Filter level chips + clear. Entries: time · level badge · message (mono); error entries with source position are click-to-jump into Code. Dock state and height persist per user. Rationale: don't make the developer leave the editor to see the consequence of running (SPEC 24; SC analysis §4 con). The standalone Logs page remains for cross-script work.

### Problems strip (status bar)

`⚠ N problems` opens Monaco's problems list; `F8`/`Shift+F8` cycle. Syntax + metadata issues share this one channel (SPEC 12: highlight syntax and metadata issues).

## Save model

`Cmd/Ctrl+S` saves; an in-app dialog (not only `beforeunload`) guards Back/close with **Save / Discard / Keep editing** (SPEC 12 DON'T: don't rely on browser-level confirmation). Crash-recovery draft kept until explicit save or discard.

## Keyboard shortcuts (desktop)

`Cmd/Ctrl+S` save · `Cmd/Ctrl+Enter` run · `Cmd/Ctrl+F`/`Cmd+Shift+F` find/in-files(tabs) · `F8` next problem · `Ctrl+`` ` console · `Cmd/Ctrl+1..5` tabs. Listed in the `⋮` menu ("Keyboard shortcuts") — discoverable, not folklore (SPEC 22).

## Accessibility

Monaco's screen-reader mode reachable; all top-bar controls labeled; tabs are a `tablist` with arrow-key navigation; console respects 4.5:1 in both themes; syntax theme contrast tuned per SPEC 7.2/12 (no low-contrast highlighting).

## Mobile editor (<600 px, per draft mockup)

```text
┌────────────────────────────┐
│ ← Name ●        Saved ✓    │ 48 px
│ [Code|Settings|Logs]       │ segmented, 40 px
│                            │
│   Monaco (simplified)      │
│                            │
│ [▶ Run] [↶] [↷] [🔍] [⋯]   │ 48 px toolbar, ≥44 px targets
└────────────────────────────┘
```

- Toolbar buttons labeled (text or tooltip-on-long-press); Run always visible; save state text persists in the top bar (SPEC 3.4).
- Logs open as a **bottom sheet** (50% height, draggable) immediately offered after Run — logs must not be hard to reach post-run (SPEC 3.4 DON'T).
- Storage/Resources fold into `⋯` as full-screen pages. Landscape supported; toolbar moves to the side. Metadata snippet insertion (`@match`, `@grant` templates) from the `⋯` menu (SPEC 3.4 DO).
- This is a quick-edit surface, not a shrunken IDE (SPEC 3.4): multi-pane, minimap, and breadcrumbs are disabled.

## States

Loading: top bar + tabs render immediately, Monaco region skeleton (no layout shift). Script-not-found: error state with "Back to Manager". Read-only (subscription-managed scripts): banner "Managed by subscription ‹name›" + Disable-editing rationale + **Duplicate to edit** action.

## Traceability

SPEC 3.4, 6, 7.2, 12, 14, 15, 22, 24, 27-P5 · SC analysis §4 (dock the logs; Monaco as differentiator) · TM analysis §3 (multi-script workflows; anti-pattern: flat settings form) · VM analysis (restraint in default chrome).
