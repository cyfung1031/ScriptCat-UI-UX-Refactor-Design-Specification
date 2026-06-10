# Script Hub — Implementation Specification

Every script's home (SPEC §7): clicking a script anywhere lands here. The editor is the **Code** tab — Monaco stays (ScriptCat's strongest asset — SC analysis §1/§4) — but management questions (*what is this, where does it run, what has it done, what changed*) get tabs of their own. This replaces the rev. 3 editor-as-home model.

## Entry points

Manager row name → **Overview** · Manager/popup `✎ Edit` → **Code** · error deep links (`#script=<id>&line=<n>` → Code at that line) · popup **View activity** → **Activity** · History & Trash change rows → **Versions** · New Script / Develop on this page (opens Code with the user's template, `@match` prefilled when from the popup, cursor inside the metadata block).

## Layout — desktop

```text
┌──────────────────────────────────────────────────────────────┐
│ ← Back · [icon] Script name [switch]  Running    Saved 12:03 │ 48 px top bar
│                                          [▶ Run] [Save  ⋮]   │
├──────────────────────────────────────────────────────────────┤
│ Overview │ Code │ Settings │ Activity │ Storage │ Resources │ Versions │ 36 px tabs
├──────────────────────────────────────────────────────────────┤
│                      (active tab fills)                      │
└──────────────────────────────────────────────────────────────┘
```

### Top bar

- **Back** returns to Manager (guarding unsaved Code changes — see Save model).
- Identity: `@icon` · name · **enable switch** · state word per SPEC §3.2.
- `●` dirty dot plus explicit **save-state text**: `Saved HH:MM` / `Unsaved changes` / `Saving…` — never ambiguous, never icon-only (SPEC §7.1).
- **▶ Run**: page scripts → applies and, when **Run on save** is off, toasts "Applied — reload the matching tab" with **Reload tab** action; background scripts → executes once and auto-opens the Code tab's console filtered to that run.
- **Save** + `⋮`: Export, Check update, Copy install URL, Keyboard shortcuts, divider, Delete (danger → Trash + undo toast, SPEC §3.4).

## Tabs

One scroll state each; switching never loses edits (SPEC §7.1).

### Overview — the management home

- **Status block**: kind glyph + label, state, and **all** attention items expanded with their next steps (the list-row one-badge rule pays off here, SPEC §3.2) — e.g. `Error: TypeError… → View in logs`, `Update waiting: 1.5.0 adds *.bank.com → Review`.
- **Where it runs**: live matched-site list (favicon + origin + per-site toggle, same state as popup/Manager site switches, SPEC §3.1); `+N more` expands; all-sites scope shown with the §8.1 warning glyph.
- **Activity summary** (last 7 days, from the ledger, SPEC §3.3): runs count, top network domains with counts, storage size — plain language, link **Full activity →**.
- **Update**: channel/URL, last checked, **Check now**.
- A regular user never needs to leave this tab.

### Code — the editor

- Monaco, ScriptCat API typings, metadata-block completion. **Metadata lint**: invalid `@match`, missing `@grant`, unknown keys → warnings in the problems strip, not silent failure later.
- Status bar (28 px): `⚠ N problems` (opens Monaco problems list; `F8`/`Shift+F8` cycle — syntax + metadata share this one channel) · `Ln, Col` · encoding · **Console** toggle.
- **Docked console panel** (200–400 px, resizable, state persists): filter level chips + clear; entries time · level badge · mono message; error entries click-to-jump to source position. Auto-opens on background Run. The edit-observe loop never leaves the tab (SPEC §7.1 DON'T).
- **Run on save** toggle (status bar): page scripts re-apply to the matched open tab on save — implemented as reload + reapply and labeled honestly ("Reloads the tab and reapplies"). Background scripts: save runs lint only; Run stays explicit.

### Settings

Form, common-first with **Advanced collapsed per group** (contextual disclosure, not a global mode — TM analysis §1 con): Enabled, Run at, Matches/Excludes (pattern-list editor with live "matches example.com?" tester), Update URL, Permissions; Advanced: sandbox options, require/resource overrides.

### Activity — the per-script ledger (SPEC §3.3)

- **Runs timeline**: when, where (origin), duration; injection failures inline with retry.
- **Network**: target domains with request counts; domains outside `@connect` flagged with the consent decision taken (Allowed / Once / Blocked) — **revocable here**.
- **Storage / clipboard / notifications / downloads**: counts and sizes. Aggregates only; payloads are never recorded or shown (SPEC §3.3 privacy contract).
- Retention note in the footer ("Activity kept 7 days — change in Settings → Privacy & activity").
- Empty state is affirmative: "No activity recorded in the last 7 days."

### Storage

Key/value table (13 px mono), inline edit, type column, Clear all (danger, confirm — value destruction is not versioned). Empty state explains what script storage is.

### Resources

`@require`/`@resource` list: URL, cached size/time, per-row re-fetch; failures inline with retry (SPEC §13.2).

### Versions — reversibility made visible (SPEC §3.4)

- List: timestamp · source badge (`Edit` / `Update` / `Import` / `Rollback`) · version string · one-line summary (scope changes called out, e.g. `+ *.bank.com`).
- Selecting a version shows a diff against current (unified, mono 13 px).
- **Restore this version** — creates a *new* version; nothing is destroyed by rolling back. Restores code, metadata, and settings the version touched.
- Retention stated in the footer ("Last 20 versions / 90 days").

## Save model

`Cmd/Ctrl+S` saves and records a version; an in-app dialog (not only `beforeunload`) guards Back/close with **Save / Discard / Keep editing**. Crash-recovery draft kept until explicit save or discard.

## Keyboard shortcuts (desktop)

`Cmd/Ctrl+K` palette (script-local commands rank first here, SPEC §3.5) · `Cmd/Ctrl+S` save · `Cmd/Ctrl+Enter` run · `Cmd/Ctrl+F` find · `F8` next problem · `Ctrl+`` ` console · `Cmd/Ctrl+1..7` tabs. Listed in the palette and the `⋮` menu — discoverable, not folklore.

## Accessibility

Monaco's screen-reader mode reachable; all top-bar controls labeled; tabs are a `tablist` with arrow-key navigation; console and Activity respect 4.5:1 in both themes; syntax theme contrast tuned per SPEC §11.3 (no low-contrast highlighting); state words and badges are text, never color-only (SPEC §15).

## Mobile Hub (<600 px)

```text
┌────────────────────────────┐
│ ← Name ●        Saved ✓    │ 48 px
│ [Overview|Code|Activity]   │ segmented, 40 px
│                            │
│   (active tab)             │
│                            │
│ [▶ Run] [↶] [↷] [🔍] [⋯]   │ 48 px toolbar (Code tab), ≥44 px targets
└────────────────────────────┘
```

- Three segments: **Overview · Code (simplified Monaco) · Activity**; Settings/Storage/Resources/Versions fold into `⋯` as full-screen pages (SPEC §7.1 DON'T: not a shrunken IDE — multi-pane, minimap, breadcrumbs disabled).
- Toolbar buttons labeled (text or tooltip-on-long-press); Run always visible; save-state text persists in the top bar.
- Console opens as a **bottom sheet** (50 % height, draggable) immediately offered after Run — logs must not be hard to reach post-run.
- Landscape supported; toolbar moves to the side. Metadata snippet insertion (`@match`, `@grant` templates) from `⋯`.

## States

Loading: top bar + tabs render immediately; tab region skeleton (no layout shift); Monaco loads only when Code opens (SPEC §17). Script-not-found: error state with "Back to Manager". Read-only (subscription-managed): banner "Managed by subscription ‹name›" + rationale + **Duplicate to edit**.

## Traceability

SPEC §3.1–3.5, §7, §11–13, §15, §17, §18-P3 · SC analysis §4 (dock the logs; Monaco as differentiator) · TM analysis §1/§3 (anti-patterns: flat settings form, global modes) · VM analysis (restraint in default chrome).
