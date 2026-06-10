# Settings — Implementation Specification

One scannable page inside the Manager shell (SPEC §10). The two failure modes to avoid are both documented in the analyses: Tampermonkey's flat wall of every option, and a global expertise switch that hides what someone needs (TM analysis §1). Disclosure here is **per-group**, contextual. rev. 4 adds the **Privacy & activity** group — the user-facing contract for the activity ledger and the version system (SPEC §3.3, §3.4).

## Entry points

Sidebar → Settings · popup `⋮` → Settings · `Cmd/Ctrl+K` → "Settings → ‹group›" · deep links from features (e.g. sync error toast → `#sync`, ledger footers → `#privacy`). Hash anchors per group are stable API: `#general`, `#updates`, `#sync`, `#editor`, `#privacy`, `#advanced`.

## Layout (desktop)

```text
┌────────┬──────────────────────────────────────────┐
│Sidebar │ Settings                [Search settings…]│
│(app)   ├───────────┬──────────────────────────────┤
│        │ General   │ ┌─ General ──────────────────┐│
│        │ Updates   │ │ Language        [System ▾] ││
│        │ Sync      │ │ Theme           [Auto ▾]   ││
│        │ Editor    │ │ …                          ││
│        │ Privacy   │ └────────────────────────────┘│
│        │ Advanced  │ ┌─ Updates ─────────────…     │
│        │           │        (one scrolling column) │
└────────┴───────────┴──────────────────────────────┘
```

- Left: in-page anchor nav (scrollspy highlights the current group). Right: **one scrolling column** of group cards, max 640 px wide — settings never become a tab maze, and never an unstructured wall.
- **Search settings** filters items live across all groups, showing matches with their group label — the escape hatch that keeps one long page workable at Tampermonkey-scale option counts.

## Item anatomy

`Label (14 px) + control` on one row; helper text (12 px, `--sc-text-2`) beneath the label, ≤ 2 lines. Controls: switch (binary persistent only, SPEC §14.3), select, segmented control (≤ 4 options), button (actions). Items that take effect on next page load say so in helper text. Settings save instantly with a brief `Saved ✓` inline confirmation — no global Save button.

## Groups and contents (default set)

- **General** — Language (System default + per-locale; loads on demand, SPEC §16.2) · Theme (Auto/Light/Dark) · Density default · Show badge count on icon.
- **Updates** — Check frequency · **Auto-apply updates without new permissions** (on by default; helper: "Updates that request new access always wait for your review. Every update can be rolled back from the script's Versions tab." — the SPEC §8.2 contract in user language) · Subscription check frequency.
- **Sync** — Backend (None/WebDAV/…): config fields appear only when selected; **Test connection** button with inline success/failure + reason; status line "Last synced ‹time› · ‹n› scripts"; sync errors render here *and* as a Manager footer indicator. Helper note: activity data is never synced (SPEC §3.3).
- **Editor** — Theme, font size, tab size, format-on-save, Run-on-save default, custom script template (multiline, with the supported variables listed).
- **Privacy & activity** (SPEC §3.3, §3.4 — the transparency contract, stated where it can be controlled):
  - **Record script activity** (on by default; helper: "Counts and domains only — never page content, request data, or stored values. Kept on this device, never synced.")
  - Activity retention (7 days default; segmented 1/7/30 days).
  - Version history retention (last 20 versions / 90 days; storage size shown inline, SPEC §3.4).
  - Trash retention (30 days; link to History & Trash).
  - Per-site script permission defaults · Site blacklist (pattern-list editor, same component as the Hub's match tester).
  - "What can scripts access?" docs link.
- **Advanced** (collapsed by default; chevron, count of items) — sandbox/injection mode, compatibility toggles, developer flags. Expanding is per-visit; no global mode switch. Each item keeps its helper text — Advanced ≠ unexplained.
- **Danger zone** (visually separated card, `--sc-danger` border, bottom of page; SPEC §14.1) — **Clear all logs & activity** · **Clear version history** · **Reset settings** · **Delete all scripts & data**. Each: red ghost button → confirm dialog naming exactly what is destroyed (these bypass Trash and are the genuinely irreversible tier); delete-all requires typing `DELETE`. Reset offers a settings export first.

## States

- Sync misconfigured/failing: persistent warning banner on the Sync card with the error and a **Fix** focus jump — config errors must not hide until the next sync attempt (SPEC §13.2).
- Search with no matches: "No settings match ‹query›" + clear.
- Any group failing to load: that card shows an inline error + retry; the rest of the page works.

## Keyboard & accessibility

Anchor nav is a list of links (`aria-current`); `/` focuses settings search. Every control labeled by its visible label (`aria-labelledby`); helper text linked via `aria-describedby`. Switches announce on/off. Danger-zone confirms trap focus; destructive buttons are never the default-focused action (SPEC §14.1, §15).

## Responsive

<600 px: anchor nav becomes a sticky horizontal chip row under the title; cards full-width; controls keep ≥ 44 px targets; danger zone stays last.

## Traceability

SPEC §3.3–3.4, §8.2, §10, §13–16 · TM analysis §1 (flat-wall and global-mode anti-patterns; breadth checklist; privacy colocation) · VM analysis §2 (short settings, technical-prose con) · SC analysis §5 (locale loading).
