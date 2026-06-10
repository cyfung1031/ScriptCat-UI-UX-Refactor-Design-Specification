# Settings — Implementation Specification

One scannable page inside the Manager shell. The two failure modes to avoid are both documented in the analyses: Tampermonkey's flat wall of every option, and a global expertise switch (Novice/Advanced) that hides what someone needs (TM analysis §1). Disclosure here is **per-group**, contextual.

## Entry points

Sidebar → Settings · popup `⋮` → Settings · deep links from features (e.g. sync error toast → `#sync`). Hash anchors per group are stable API: `#general`, `#updates`, `#sync`, `#editor`, `#security`, `#advanced`.

## Layout (desktop)

```text
┌────────┬──────────────────────────────────────────┐
│Sidebar │ Settings                [Search settings…]│
│(app)   ├───────────┬──────────────────────────────┤
│        │ General   │ ┌─ General ──────────────────┐│
│        │ Updates   │ │ Language        [System ▾] ││
│        │ Sync      │ │ Theme           [Auto ▾]   ││
│        │ Editor    │ │ …                          ││
│        │ Security  │ └────────────────────────────┘│
│        │ Advanced  │ ┌─ Updates ─────────────…     │
│        │           │        (one scrolling column) │
└────────┴───────────┴──────────────────────────────┘
```

- Left: in-page anchor nav (scrollspy highlights the current group). Right: **one scrolling column** of group cards, max 640 px wide — settings never become a tab maze, and never an unstructured wall (SPEC 8; TM analysis §1 con).
- **Search settings** filters items live across all groups, showing matches with their group label — the escape hatch that makes one long page workable at Tampermonkey-scale option counts.

## Item anatomy

`Label (14 px) + control` on one row; helper text (12 px, `--sc-text-2`) beneath the label, ≤ 2 lines. Controls: switch (binary persistent only, SPEC 26.1), select, segmented control (≤ 4 options), button (actions). Items that take effect on next page load say so in helper text. Settings save instantly with a brief `Saved ✓` inline confirmation — no global Save button.

## Groups and contents (default set)

- **General** — Language (System default + per-locale; loads on demand, SPEC 20) · Theme (Auto/Light/Dark) · Density default · Show badge count on icon.
- **Updates** — Check frequency · **Auto-apply updates without new permissions** (on by default; helper: "Updates that request new access always wait for your review" — the SPEC 13.2 escalation contract stated in user language) · Subscription check frequency.
- **Sync** — Backend (None/WebDAV/…): config fields appear only when selected; **Test connection** button with inline success/failure + reason; status line "Last synced ‹time› · ‹n› scripts"; sync errors render here *and* as a Manager footer indicator (SPEC 28 "Reliable").
- **Editor** — Theme, font size, tab size, format-on-save, custom script template (multiline, with the supported variables listed — VM's `descScriptTemplate` precedent).
- **Security** — Site blacklist (pattern list editor, same component as the editor's match tester) · per-site script permission defaults · "What can scripts access?" docs link.
- **Advanced** (collapsed by default; chevron, count of items) — sandbox/injection mode, compatibility toggles, developer flags. Expanding is per-visit; no global mode switch. Each item keeps its helper text — Advanced ≠ unexplained.
- **Danger zone** (visually separated card, `--sc-danger` border, bottom of page; SPEC 18.2) — **Clear all logs** · **Reset settings** · **Delete all scripts & data**. Each: red ghost button → confirm dialog naming exactly what is destroyed; delete-all requires typing `DELETE`. Reset offers a settings export first.

## States

- Sync misconfigured/failing: persistent warning banner on the Sync card with the error and a **Fix** focus jump — config errors must not hide until the next sync attempt (SPEC 14).
- Search with no matches: "No settings match ‹query›" + clear.
- Any group failing to load: that card shows an inline error + retry; the rest of the page works.

## Keyboard & accessibility

Anchor nav is a list of links (`aria-current`); `/` focuses settings search. Every control labeled by its visible label (`aria-labelledby`); helper text linked via `aria-describedby`. Switches announce on/off. Danger-zone confirms trap focus; destructive buttons are never the default-focused action (SPEC 18.2).

## Responsive

<600 px: anchor nav becomes a sticky horizontal chip row under the title; cards full-width; controls keep ≥ 44 px targets; danger zone stays last.

## Traceability

SPEC 8, 13.2, 14, 17, 18.2, 20, 26.1 · TM analysis §1 (flat-wall and global-mode anti-patterns; breadth checklist; privacy colocation) · VM analysis §2 (short settings, technical-prose con) · SC analysis §5 (locale loading).
