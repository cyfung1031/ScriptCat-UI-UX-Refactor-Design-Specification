# ScriptCat UI/UX Design Specification (rev. 4)

> rev. 4 is a structural revision, not an increment. rev. 3 synthesized the best of the reference products (`references/UX_ANALYSIS_*.md`); rev. 4 keeps everything that synthesis validated — tokens, accessibility targets, flow safety, terminology — but reorganizes the product around a thesis of its own instead of a parity checklist. The major changes:
>
> 1. **Two-axis model (§3.1)** — sites are a first-class management axis beside scripts, not just a popup framing.
> 2. **Three-dimensional status (§3.2)** — *kind / state / attention* replaces the flat nine-badge status table.
> 3. **Activity ledger (§3.3)** — observed script behavior becomes first-class, local-only data; trust is continuous, not a one-time install decision.
> 4. **Reversibility as a system guarantee (§3.4)** — per-script version history, trash, and rollback replace scattered per-feature undo.
> 5. **Script Hub (§7)** — the script detail page is the script's home; the editor is its Code tab.
> 6. **Command layer (§3.5)** — one palette across manager and hub instead of growing toolbars.
>
> Per-page implementation specifications live under `design/` and cite this document by section number.

---

# Part I — Direction

# 1. Purpose

This document defines the UI/UX design model, principles, and do's/don'ts for refactoring ScriptCat, a userscript manager. ScriptCat should feel like a lightweight developer tool with excellent script management, clear runtime visibility, and fast access to common actions — never like a browser settings panel.

The reference analyses established the floor: Violentmonkey proves the hierarchy and temperament are achievable; Tampermonkey enumerates the capability scope; ScriptCat already has the substance (Monaco, background/scheduled scripts, logs, sync, dedicated flow pages). rev. 4 defines what sits *above* that floor.

---

# 2. Product Thesis: Transparent Automation

A userscript is power the user grants to someone else's code, running inside their browsing session, indefinitely. Every competitor treats that grant as a one-time install dialog followed by silence. ScriptCat's differentiation is to make the grant **visible, controllable, and reversible** for its whole lifetime.

Three promises, which every surface must serve:

1. **You can always see it.** What runs where, what it did, and what changed — without opening DevTools or reading source.
2. **You can always control it.** Per-script and per-site control are each one action away, from anywhere the script is visible.
3. **You can always take it back.** No user action, and no script update, is irreversible. History replaces fear.

These promises are why ScriptCat's unique substance matters: background and scheduled scripts are *more* invisible than page scripts, so transparency is not a nice-to-have here — it is the category ScriptCat is actually in.

## 2.1 Three User Types (unchanged from rev. 3, re-grounded)

1. **Regular users** — install, enable, disable, update. The promises give them confidence without vocabulary: a plain-language answer to "what is this thing doing?"
2. **Power users** — many scripts, filters, batch operations, sync. The two-axis model and command layer give them speed without clutter.
3. **Developers** — write, run, debug, inspect. The Script Hub, live run loop, and activity ledger shorten the edit-observe cycle.

### DO

* Design the default UI for regular users; let depth be opted into per context.
* Keep developer workflows fast and keyboard-first.
* Separate management, runtime, and development concerns.

### DON'T

* Do not gate features behind a global expertise mode (Tampermonkey's documented failure).
* Do not let minimalism delete the management depth power users rely on (Violentmonkey's documented gap).
* Do not assume users understand userscript terminology (`@match`, `@grant`, "metadata block") in default-level UI.

---

# 3. The Model

The concepts in this section are the spine of rev. 4. Every surface specification in Part II is an application of them.

## 3.1 Two Axes: Scripts × Sites

Users hold two mental models, and the product must serve both natively:

* **Script axis** — "manage my collection": install, update, organize, develop. The Manager's default view.
* **Site axis** — "what is happening on this site?": the question behind almost every popup open, and behind most trust anxiety.

### DO

* Make the popup the **site controller**: everything about the current origin — running scripts, per-site enable/disable, errors, find-scripts-for-this-site — in one surface (§5).
* Give the Manager a **By site** view: scripts grouped under the origins they touch, with per-site controls (§6).
* Make per-site disable a remembered, first-class state ("ScriptCat is off on example.com"), not a per-script exclude buried in menus.
* Keep the two axes consistent: a script row looks and acts the same whichever axis it is reached from.

### DON'T

* Do not treat site context as popup-only framing while the Manager stays a flat script table.
* Do not implement per-site control as N separate per-script excludes the user must maintain by hand.
* Do not invent a third organizational axis (workspaces, profiles) — two is the model.

## 3.2 Script Anatomy: Kind / State / Attention

A script's status is three orthogonal dimensions, never one badge:

| Dimension | Values | What it answers | Rendering |
| --- | --- | --- | --- |
| **Kind** | Page · Background · Scheduled | What is it? (fixed at install) | Small type glyph + label in detail contexts; grouping in lists |
| **State** | Running · Idle · Off | What is it doing right now? | The enable switch carries Off; one plain state word carries the rest. `Idle` = enabled but not active here/now; `Scheduled 14:00` is Idle with a next-run time |
| **Attention** | Error · Warning · Update waiting · (none) | What needs me? | At most **one** badge per row — the highest severity. Remaining attention items appear in the row's detail/hub |

### DO

* Render exactly: one switch, one state word, at most one attention badge per list row.
* Keep the dimensions consistent across popup, manager, hub, and mobile — same words, same colors, same order.
* Roll attention up: group headers, the Manager title, and the extension icon badge show the highest attention severity present.
* Pair every status color with its text label (also §15).

### DON'T

* Do not mix kind into the state column ("Background" is not a state).
* Do not show multiple competing badges on one row.
* Do not use unlabeled colored dots, opacity, or strikethrough as the only status signal.
* Do not invent per-surface status vocabularies.

## 3.3 The Activity Ledger

ScriptCat mediates every privileged API a script uses. That position makes **observed behavior** cheap to record and uniquely honest to show. The ledger is the mechanism behind promise 1.

**What is recorded (aggregates, never payloads):**

* Cross-origin requests: target domain + count (per `GM_xmlhttpRequest` family).
* Storage: keys touched, total size.
* Clipboard reads/writes, notifications shown, downloads initiated — counts.
* Runs: when, where (origin), duration; injection failures.

**Privacy contract (non-negotiable):** local-only, never synced or transmitted; aggregate counts and domains, never request/response bodies or values; user-configurable retention (default 7 days); a single off switch in Settings → Privacy. The ledger watches scripts on the user's behalf — it must never become telemetry about the user.

**Where it surfaces:**

* **Script Hub → Activity tab** (§7): the full per-script ledger.
* **Popup row → View activity**: one-glance summary ("Today: ran 3×, 14 requests to api.example.com").
* **Install/update pages** (§8): the *declared* contract (`@match`, `@grant`, `@connect`) that observation is later compared against.
* **Declared vs observed enforcement**: a request to a domain not covered by `@connect` is blocked pending a one-time consent prompt ("‹Script› wants to contact ‹domain›. Allow / Once / Block"), and the event is recorded. Escalation between what was promised and what was attempted is surfaced, not silently allowed or silently dropped.

### DO

* Translate ledger entries to plain language ("Sent 14 requests to api.example.com", "Stored 2 KB of data") — the ledger is for regular users first.
* Keep recording cost negligible: counters and domain sets, batched writes; sampling for high-frequency events.
* Show "No activity recorded" as an affirmative, informative state.
* Let developers use the ledger too — it doubles as a lightweight "what did my script actually do" trace.

### DON'T

* Do not record payloads, URLs beyond origin/domain, or page content. Ever.
* Do not turn the ledger into alarm UI — observed activity within declared scope is presented neutrally.
* Do not make the consent prompt for undeclared domains a scary modal wall; it is one clear sentence with three buttons.
* Do not let metering measurably slow script execution; the budget is approximately zero.

## 3.4 Reversibility as a System Guarantee

rev. 3 scattered safety across features (delete-undo toast, restorable trash, update review). rev. 4 unifies them on one mechanism: **every change to a script produces a restorable version.**

* **Version history per script**: kept on save (editor), update (any channel), and import-overwrite. Entries: timestamp, source (edit/update/import), version string, diff available. Default retention: last 20 versions or 90 days.
* **Trash**: deleted scripts (including their storage and history) restorable for 30 days; surfaced in Manager → History & Trash.
* **Rollback** is one action from the Hub's Versions tab and from any update toast ("Updated to 1.5.0 — Roll back").
* **Undo toasts are real**: every "Undo" in a toast maps to a restore point, not a best-effort inverse operation.

This is also what makes **silent auto-update acceptable**: non-escalating updates can apply automatically *because* rollback is one click. Safety enables convenience, not the other way around.

### DO

* State the guarantee in UI at the moments of fear: the delete dialog says "You can restore this for 30 days"; the update toast offers rollback.
* Keep history storage bounded and visible (size shown in Settings → Privacy & storage).
* Roll back *everything* the version touched: code, metadata, and settings changed by an update.

### DON'T

* Do not confirm-dialog actions that are cheaply reversible — reversibility replaces friction, that's the point. Confirmation remains only for bulk destruction and trash-purge.
* Do not let history quietly grow unbounded, and do not silently drop it either — show retention rules where history is shown.
* Do not offer "Undo" anywhere it cannot actually restore prior state.

## 3.5 The Command Layer

One command palette (`Cmd/Ctrl+K`) across the Manager and Script Hub: actions ("Check all updates", "New script", "Toggle ‹script›"), navigation ("Open ‹script›", "Settings → Sync"), and search in one input. The palette is the pressure valve that keeps toolbars from growing.

### DO

* Make every palette command also reachable through visible UI — the palette is acceleration, never the only path.
* Scope intelligently: in the Hub, script-local commands rank first; in the Manager, collection commands rank first.
* Show shortcuts beside palette entries; the palette doubles as the shortcut reference.

### DON'T

* Do not put palette-only features anywhere.
* Do not add a toolbar button for every new feature — palette + menus absorb low-frequency actions.
* Do not ship the palette on the popup; the popup stays instant and minimal.

---

# Part II — Surfaces

# 4. Information Architecture Overview

```text
Popup (site controller)         — the site axis, current origin
Manager                         — the script axis (default) + By site view
  ├─ Installed Scripts
  ├─ Subscriptions
  ├─ Logs
  ├─ History & Trash
  └─ Settings
Script Hub (per script)         — Overview · Code · Settings · Activity · Storage · Resources · Versions
Trust surfaces                  — install / update / import / export / batch update
```

Sidebar rules (unchanged in spirit from rev. 3): limited, stable top-level entries; Help and version at the bottom, separated; no external links mixed into product navigation; current section clearly highlighted; icons never the only label on desktop.

---

# 5. Popup — the Site Controller

The highest-frequency surface. It answers, in order: *what is affecting this page right now → is anything wrong → can I act in one click*. It is the site axis (§3.1) made physical.

## 5.1 Hierarchy

```text
Header        product name · global switch · ⋮ (New script, Get scripts, Settings)
Site strip    favicon + domain · per-site switch · "N running" · error chip
(banner)      one contextual banner max: reload-needed / injection-failed / site off
Running on this page (n)        expanded, pinned first
Background & scheduled (n)      ScriptCat differentiator — always its own labeled section
Other scripts (n)               collapsed by default
Footer        version · Get scripts for this site · Open Manager
```

### DO

* Put the **per-site switch** in the site strip: one action turns ScriptCat off for this origin, remembered (§3.1). The global switch stays in the header for the rare global case.
* Keep current-page scripts first, with counts in group headers and collapsed secondary groups.
* Render rows per §3.2: switch · name · state word/attention badge · visible Edit · ⋮ menu.
* Surface injection failures and reload-needed states as the single contextual banner with a one-click fix.
* Offer **View activity** in the row menu — the ledger glance (§3.3).
* Offer **Develop on this page** in the popup ⋮ menu: creates a scratch script with `@match` prefilled for the current origin and opens the Hub — the fastest path from "this site annoys me" to a fix.
* Show the extension icon badge: count of scripts running on the current page; attention severity recolors it.
* Keep the popup instant: fixed width, internal scroll, no open animation, no manager/editor bundles.

### DON'T

* Do not make the popup a compressed manager — no batch tools, no settings forms, no palette.
* Do not bury per-site disable inside per-script exclude menus.
* Do not make current-page scripts and other scripts visually identical.
* Do not require scrolling before the first running script is visible.
* Do not exceed two icon controls in the header.

## 5.2 Many-Scripts and Edge States

* "Other scripts" > 10 → search field pinned at the top of the list, filtering all groups.
* Total rows > 30 → virtualize; show "View all in Manager →".
* First run: one sentence + **Get scripts** / **Create script** (never "No data").
* Scripts installed but none match: say so, offer **Find scripts for this site**; background group still shown.

---

# 6. Manager

The script collection's home: a table on desktop, cards on mobile, with the **By script / By site** view toggle (§3.1).

## 6.1 Layout (desktop)

```text
Sidebar │ Header: "Installed Scripts (37)"            [⌘K] [+ New Script]
        │ Toolbar: [Search][filter chips…][view: Script|Site][sort][density]
        │   — or, when rows are selected —
        │ Batch bar: "3 selected" [Enable][Disable][Update][Export] | [Delete] [Cancel]
        │ Script table / site groups (scrolls)
        │ Status footer: counts · last update check · sync state
```

### DO

* Keep **+ New Script** the single most prominent action on the page.
* **By site view**: rows grouped under origin headers (favicon + domain + per-site switch + count); a script matching several sites appears under each, acting on the same underlying script.
* Status column per §3.2: one state word or one attention badge, never a badge stack and never a bare dot.
* Keep search persistent (`/` focuses), filter chips with one-tap Reset, sort and density (comfortable/compact) persisted.
* Show batch actions only after selection, with count, destructive action separated; bulk delete goes to Trash with an undo toast (§3.4).
* Keep Edit visible per row (not hover-only, not menu-only); advanced actions in the row ⋮.
* Keep update checks non-blocking: progress in the status footer, spinners only on affected rows.

### DON'T

* Do not render the list as a data grid where every column has equal weight — it must read *name → state → action*.
* Do not permanently show batch buttons.
* Do not hide important status in secondary columns or behind hover.
* Do not let the By-site view become a separate page with different row anatomy.

## 6.2 History & Trash

A sidebar entry, the visible face of §3.4: deleted scripts (restore / purge with confirmation), and recent cross-collection changes (updates applied, imports) each linking to the script's Versions tab. Retention rules stated on the page.

## 6.3 Mobile (<600 px)

Bottom navigation (Scripts · Logs · Settings, persistent); cards instead of tables; card anatomy: switch + name + state/attention, one-line description, text-labeled Edit / Run / More; long-press enters selection mode with the batch bar above the bottom nav; filter chips scroll horizontally; touch targets ≥ 44 px; no hover-dependent affordances. The By-site toggle survives as a segmented control.

---

# 7. Script Hub

Every script's home — reached by clicking a script anywhere. The editor is its Code tab, not the destination itself. This replaces rev. 3's editor-as-home model: management questions ("what is this script, what has it done, what changed") deserve a surface that is not a code window.

## 7.1 Structure

```text
← Back · icon · Script name · switch     state word · [▶ Run] [Save ⋮]
Tabs: Overview · Code · Settings · Activity · Storage · Resources · Versions
```

* **Overview** — kind, state, attention items (all of them, expanded — this is where the one-badge list rule pays off); *Where it runs*: live matched-site list with per-site toggles; activity summary (last 7 days); update channel + last checked; quick actions. A regular user never needs to leave this tab.
* **Code** — Monaco, ScriptCat API typings, metadata completion and lint (invalid `@match`, missing `@grant`, unknown keys → problems strip). Status bar: problems count, cursor, **Console** toggle for the docked log panel (resizable, persists). **Run on save** toggle: page scripts re-apply to the matched open tab on save (reload + reapply, stated honestly); background scripts run once and auto-open the console filtered to that run. The edit-observe loop never requires leaving the tab.
* **Settings** — common-first form (enabled, run-at, matches/excludes with a live pattern tester, update URL, permissions); Advanced collapsed per group.
* **Activity** — the per-script ledger (§3.3): runs timeline, network domains with counts, storage/clipboard/notification events, undeclared-domain consent decisions (revocable here).
* **Storage** — key/value table, inline edit, Clear all (confirm).
* **Resources** — `@require`/`@resource`: URL, cached size/time, re-fetch, inline failures with retry.
* **Versions** — §3.4 history: list of versions with source and diff view; **Restore this version** (creates a new version; nothing is ever destroyed by rolling back).

### DO

* Show save state as explicit text (`Saved 12:03` / `Unsaved changes` / `Saving…`) plus the dirty dot; guard close/back with an in-app Save / Discard / Keep editing dialog; keep a crash-recovery draft.
* Keep one scroll state per tab; switching tabs never loses edits.
* Keep keyboard-first: `Cmd/Ctrl+S` save, `Cmd/Ctrl+Enter` run, `F8` problems, `Ctrl+\`` console, `Cmd/Ctrl+K` palette; all listed in the palette.
* Tune syntax-highlighting contrast in both themes.

### DON'T

* Do not make developers leave the Hub to see the consequence of running a script.
* Do not make save state ambiguous or icon-only.
* Do not rely on browser `beforeunload` alone for unsaved changes.
* Do not shrink the full IDE into mobile: the mobile Hub is Overview · Code (simplified) · Activity, with a labeled toolbar (Run always visible), logs as a draggable bottom sheet offered right after Run, and the rest behind ⋯ as full-screen pages.

---

# 8. Trust Surfaces: Install, Update, Import/Export

Installing is a trust decision; updating can be a scope escalation; importing is bulk trust. Shared rule: **plain-language contract → explicit consent → honest summary.** These pages also establish the *declared* baseline the activity ledger observes against (§3.3).

## 8.1 Install Confirmation

Order: name/version/author/source origin → **where it runs** (match patterns in plain language; all-sites scope gets a warning icon, not alarm copy) → **what it can do** (grants/connect translated: "Cross-origin requests to api.example.com", "Read clipboard ⚠"; background/cron stated as "Runs in the background, daily at 14:00") → collapsed code preview → sticky Cancel / Install.

### DO

* Keep both decision buttons visible at every scroll position; never auto-focus Install.
* Show source origin as favicon + domain; unknown sources get a neutral "Unverified source" note.
* State where a background script will appear after install ("under Background scripts in the popup").
* Keep the code preview available but secondary — code is evidence, not the explanation.

### DON'T

* Do not auto-install without confirmation.
* Do not present raw code as the only description of behavior.
* Do not use alarming language for narrow, normal scopes.

## 8.2 Update Confirmation

* Header `v1.4.2 → v1.5.0`; **scope changes called out first** (`+ Runs on *.bank.com ⚠`), never findable only inside the diff; code diff collapsed below.
* With escalation, the primary button reads **Update and allow new access** — consent is explicit.
* Non-escalating updates may auto-apply (default on, per Settings) *because rollback is one click* (§3.4); every applied update produces a version and a toast with **Roll back**.
* Any escalation parks the update behind an `Update waiting` attention state until reviewed.

## 8.3 Import / Export / Batch Update

* Import: preview list with per-row status (New / Conflict / Invalid), per-row + apply-to-all conflict resolution (default Skip; overwrite creates a version, §3.4), real count on the primary button, determinate progress, honest summary (`32 imported · 3 skipped · 2 failed` + retry failed).
* Export: one dialog; checkbox list; "Include script storage data" off by default with a privacy note colocated with the control.
* Batch update: non-blocking check; review screen states each row's escalation verdict (`✓ No new permissions` / `⚠ New access: …`); escalating rows are excluded from one-click bulk and route to §8.2.

### DON'T

* Do not import or update silently over anything that expands scope.
* Do not report success when items failed; the summary screen is the record, not a toast.
* Do not block the UI while checking many scripts.

---

# 9. Logs

The cross-script console surface; the Hub's Activity tab and docked console are per-script complements, not replacements.

* Toolbar: search · script select (with counts) · level chips · time range; active filters as removable chips + Reset.
* Live tail on by default; auto-pause on scroll-up with "Paused — N new ↓"; reading position never yanked.
* Rows: time · level badge (text + color) · script name (→ Hub) · message; identical repeats collapse with a `×N` chip.
* Detail panel: full message/stack, context (script, version, tab, run-id), **Open script at line** (→ Code tab at position), Copy, prefilled Report issue.
* Every error badge elsewhere in the product deep-links into a pre-filtered view here or into the Hub's Activity/Code — there is always a path from an error to its source.
* Retention stated in the footer; Clear is a confirmed, scoped destructive action.

---

# 10. Settings

One scannable page in the Manager shell: anchor nav + one scrolling column of group cards + live settings search. Per-group contextual disclosure; no global expertise mode; no tab maze; no flat wall.

Groups: **General** (language — loaded on demand; theme; density; badge) · **Updates** (frequency; auto-apply without new permissions, with the §8.2 contract stated in user language) · **Sync** (backend config appears when selected; Test connection; status line; errors surface here *and* in the Manager footer) · **Editor** (theme, font, template) · **Privacy & activity** (NEW: ledger on/off, retention, history retention and size, per-site permission defaults, site blacklist, "What can scripts access?" docs) · **Advanced** (collapsed; every item keeps helper text) · **Danger zone** (separated, red-bordered, type-to-confirm for delete-all; reset offers settings export first).

Items save instantly with inline `Saved ✓`; helper text states when a change needs a page reload.

---

# Part III — System

# 11. Visual Foundations

## 11.1 Tokens

```text
Spacing: 4 px base — steps 4 / 8 / 12 / 16 / 24 / 32
Radius: 8 px cards & dialogs · 4 px buttons, inputs, badges · full for chips & switches
Type: 14 px primary · 12 px secondary · 16 px page titles · 13 px monospace (code, logs, patterns)
       system font stack for UI; monospace only for code/logs/patterns; line-height ~1.5 (more for CJK)
Elevation: two levels only (floating, modal); borders over shadows in dark mode
Touch targets: ≥ 44 × 44 px on touch surfaces
Breakpoints: <600 px mobile (cards, bottom nav) · 600–900 px condensed (icon rail, fewer columns) · >900 px full desktop
Popup: fixed width 360–400 px · max height ~600 px, internal scroll
Desktop rows: 48 px comfortable / 36 px compact · mobile card padding 12–16 px
Focus ring: 2 px, primary color, 2 px offset, ≥ 3:1 contrast — never removed
```

### DO

* Derive every margin/padding from the base unit; keep row heights consistent within a view.
* Keep terminology, ordering, and the status system identical across breakpoints.

### DON'T

* Do not use one-off spacing values, mixed radii in one view, or more than one UI font family.
* Do not switch interaction models at different widths on different pages.

## 11.2 Color Semantics

| Color | Usage |
| --- | --- |
| Blue | Primary action, selection, active navigation |
| Green | Running, healthy |
| Orange | Warning, pending, update waiting |
| Red | Error, destructive |
| Gray | Off, idle-muted, secondary information |
| Purple | Background/Scheduled kind accent (optional) |

Red only for errors/destruction; green only for healthy/running; color never carries meaning without a text label (§3.2, §15).

## 11.3 Light and Dark

Light: clean and readable — subtle borders, high-contrast content, visible selection (not background-tint-only). Dark: a designed pass, not an inversion — secondary text stays ≥ 4.5:1, borders carry separation, badges softened, syntax highlighting tuned separately, no pure black unless intentional.

# 12. Motion

```text
Micro (switch, hover, press): 100–150 ms
Menus, popovers, sections: 150–250 ms
Dialogs, page transitions: 200–300 ms max
Ease-out entering · ease-in exiting · prefers-reduced-motion → fades
```

Never block input on animation; never animate large lists item-by-item; never animate the popup opening; no decorative motion.

# 13. States and Feedback

## 13.1 Empty, Loading, First-Run

* Required empty states: no scripts; none match this page; no search results; no logs; no activity; no subscriptions; no versions yet; no storage; sync not configured; network lost. Each: plain language + one useful next action; never blank, never "No data", never blaming.
* Loading: skeletons that preserve layout; progress with counts for long operations; cancellation where meaningful; no post-load jumps.
* First run is a designed state: one sentence, **Get scripts** + **Create script**, docs link. No onboarding wizard; no permission/sync requests before the user has a reason to care.

## 13.2 Errors

Defined states: `Runtime Error` · `Permission Missing` · `Update Failed` · `Network Error` · `Syntax Error`. Every error is visible where it matters (popup for current page, Manager attention badge, Hub Overview), states a next step, offers retry where possible, and links to its source (§9). Never color-only; never buried in logs alone; never generic text without a path forward.

## 13.3 Toasts

Confirm lightweight successes; one consistent position per platform away from primary actions; auto-dismiss successes; **Undo/Roll back in the toast whenever a restore point exists (§3.4)**; errors requiring action persist until acknowledged and are never recorded *only* as a toast. Never stack many; never cover bottom nav or the batch bar.

# 14. Interaction Standards

## 14.1 Primary and Destructive Actions

Primary actions (New Script, enable/disable, Run, Edit, Open Manager) are visually strong, predictably placed, one click/tap away. Destructive actions are red, separated from neutral actions, never icon-only, never the default-focused button, and follow §3.4: reversible destruction (single delete) gets an undo toast instead of a dialog; bulk or permanent destruction (purge trash, delete-all) gets an explicit confirmation naming exactly what is destroyed.

## 14.2 Search, Filtering, Batch

Search persistent on desktop; filters for state, kind, attention, tag, update status; selected chips obvious; one-tap reset; matching count shown. Batch bar only on selection, with count, destructive separated — full rules in §6.

## 14.3 Components (condensed)

* **Switches**: binary persistent state only; accessible labels; never for momentary actions.
* **Badges**: one per row (§3.2); short text; consistent colors; never decorative.
* **Icon buttons**: tooltip + accessible name always; labels where space allows; consistent icon set; dangerous icon buttons never subtle.
* **Tables**: desktop only; sortable; hover + selected states; columns weighted by priority, not equal.
* **Cards**: mobile and popup; compact; clear internal hierarchy; few actions in footers.
* **Menus**: grouped by intent (Create / Maintain / Help) with separators; destructive last; common first.

# 15. Accessibility

* WCAG 2.1 AA: ≥ 4.5:1 normal text, ≥ 3:1 large text, icons, and component boundaries — in both themes, including compact density and muted "Other scripts" rows.
* Full keyboard paths on every surface; visible focus per §11.1 — `outline: none` is forbidden.
* Accessible names on all icon buttons and switches ("Enable ‹name›"); group headers expose `aria-expanded`; live regions polite and throttled (log tail, progress).
* Status never by color alone — §3.2's text labels are the accessibility mechanism, not a styling preference.
* Touch targets ≥ 44 px; no hover-only or right-click-only affordances.

# 16. Language and Localization

## 16.1 Terminology

| Use | Avoid |
| --- | --- |
| Installed Scripts | Script List |
| Running on this page | Active |
| Enable / Disable | Open / Close |
| Update waiting | New version |
| Activity | Telemetry, Analytics (it is neither) |
| Logs | Console (unless it is the console) |
| Storage | Data |
| Resources | Files |
| Subscription | Feed |
| History / Versions / Trash | Backup (history is automatic; backup is the export flow) |

One term per concept everywhere; plain language at default level; `@`-jargon only inside developer contexts (Code tab, pattern editors).

## 16.2 Localization

Layouts tolerate 30–50 % expansion; pluralized counts translatable; locale-aware dates/numbers; CJK verified on every surface; locales loaded on demand, never all bundled; no concatenated sentence fragments; no text in images; truncation always with full-text tooltip; stable string keys.

# 17. Performance Budgets

| Surface | Budget |
| --- | --- |
| Popup | CSS ≤ 50 KB · no Monaco/manager bundles · interactive < 100 ms · no open animation |
| Manager | CSS ≤ 150 KB · Monaco loads only when a Code tab opens |
| Trust pages | CSS ≤ 100 KB · highlighter/diff lazy-loads below the fold |
| Activity ledger | ~zero runtime overhead: counters + domain sets, batched writes, sampling for high-frequency events |

Per-surface bundles; one monolithic stylesheet across surfaces is forbidden (Tampermonkey's 434 KB counter-example).

---

# 18. Refactor Priorities (re-ranked for rev. 4)

1. **P1 — Three-dimensional status (§3.2).** Cheapest, most visible coherence win; unblocks every other surface. Replace dots and badge stacks everywhere.
2. **P2 — Popup as site controller (§5).** Highest-frequency surface; adds the per-site switch and clean hierarchy.
3. **P3 — Script Hub + Versions (§7, §3.4).** The structural change: detail home, editor as Code tab, version history with rollback. Enables safe auto-update (§8.2).
4. **P4 — Manager views + command layer (§6, §3.5).** By-site view, contextual batch bar, History & Trash, palette.
5. **P5 — Trust surfaces (§8).** Plain-language install/update, escalation gating, honest import/export summaries.
6. **P6 — Activity ledger (§3.3).** The differentiator; ship after the surfaces that display it exist. Start with run/injection events, add network/storage metering next.
7. **P7 — System polish (§11–§13).** Dark-mode contrast pass, mobile touch pass, motion, empty states.

---

# 19. Design Principles

ScriptCat should be:

* **Clear** — script state is understood without guessing (§3.2).
* **Fast** — common actions in one click; developer loops never leave the surface (§5, §7).
* **Calm** — no noise even with many scripts; one badge, one banner, restrained color.
* **Powerful** — batch, filters, palette, storage, versions, full editor — opted into, not imposed.
* **Consistent** — one visual language and one vocabulary across popup, manager, hub, mobile, both themes.
* **Transparent** — what runs, where, and what it did is always visible (§2, §3.3).
* **Reversible** — nothing the user or an update does is permanent by accident (§3.4).

## Summary DO

* Serve both axes: site controller popup, by-site manager view.
* Render status as kind/state/attention — one switch, one word, one badge.
* Show observed behavior, locally and privately, in plain language.
* Version every change; put Undo/Roll back where fear lives.
* Keep one command layer; keep toolbars from growing.
* Explain scope before install; gate escalations; report flows honestly.
* Build every surface from the same tokens, terms, and budgets.

## Summary DON'T

* Don't make the popup a mini-manager or the manager a data grid.
* Don't stack badges, use bare dots, or mix kind into state.
* Don't record payloads, sync activity data, or alarm users about declared behavior.
* Don't confirm what can be undone; don't "undo" what can't be restored.
* Don't gate depth behind a global mode; don't ship palette-only features.
* Don't apply scope-expanding updates without explicit consent.
* Don't let any surface drift from the shared tokens, vocabulary, or budgets.
