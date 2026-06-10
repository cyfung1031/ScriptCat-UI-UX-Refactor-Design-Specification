# Install / Update Confirmation — Implementation Specification

Redesigns the existing `install.html` / `confirm.html` surfaces. Installing is a trust decision: the page must answer *what is this, where will it run, what can it access* before showing code (SPEC 13).

## Entry points

Navigating to a `*.user.js` URL · "Install" from a script site · drag-drop of a script file · update prompts (manual check or scheduled check finding a scope change) · permission grants requested at runtime reuse the same scope-card component in a dialog.

## Layout

Full page, centered column **max 720 px**, generous whitespace. Sticky bottom action bar so the decision is always reachable without scrolling past code.

```text
┌──────────────────────────────────────────────┐
│ [icon] Script Name                 v1.4.2     │  header
│ by Author · from greasyfork.org [favicon]     │
├──────────────────────────────────────────────┤
│ WHERE IT RUNS                                 │
│  ⚠ Runs on all websites                       │  scope card
│   – or –  Runs on: github.com, gist.github.com│
│ WHAT IT CAN DO                                │
│  • Cross-origin requests to api.example.com ⚠ │
│  • Clipboard access ⚠                         │
│  • Store data · Show notifications            │
├──────────────────────────────────────────────┤
│ ▸ Code preview (1 240 lines)                  │  collapsed section
├──────────────────────────────────────────────┤
│            [Cancel]        [Install script]   │  sticky, 64 px
└──────────────────────────────────────────────┘
```

## Regions

### Header

Script `@icon` (40 px, fallback glyph) · name (18 px semibold) · version · author · **source origin** as favicon + domain chip — users must be able to recognize unfamiliar hosts (SPEC 13.1). Unknown/IP/data-URL sources get a neutral "Unverified source" note, not alarm styling.

### Scope card — the heart of the page

- **Where it runs**: `@match`/`@include` rendered in plain language (SPEC 13.1; VM analysis §4 jargon con). Patterns are summarized (`github.com and 2 more` expands); `<all_urls>`-class patterns render as **"Runs on all websites"** with a `--sc-warning` icon — emphasis, not fear (SPEC 13.1 DON'T: no alarming language for narrow scopes).
- **What it can do**: derived from `@grant`/`@connect`, translated to user language ("Cross-origin requests to ‹domains›", "Read and modify clipboard", "Run in background on a schedule"). Powerful grants carry a ⚠ and a tooltip explaining the risk in one sentence. Plain capabilities listed without warning decoration.
- Background/scheduled scripts state it here: "Runs in the background ‹cron summary, e.g. daily at 14:00›" — the user learns the ScriptCat model at the moment it matters (SC analysis §1).

### Code preview

Collapsed `▸ Code preview (N lines)` section; expanding lazy-loads a read-only highlighter (asset budget, design/README). Always *available*, never *required* — code is evidence, not the explanation (SPEC 13.1; TM/VM ship code-first pages).

### Action bar (sticky, 64 px)

**Cancel** (secondary, left) · **Install script** (primary, right). Both visible at every scroll position (SPEC 13.1). After install: success state with **Open ‹first matched site›** / **Done**; if the script declares background/cron, note where to find it ("It appears under Background scripts in the popup").

## Update mode differences (SPEC 13.2)

Header shows `v1.4.2 → v1.5.0`. A **What changed** section is inserted *above* the scope card:

- **Scope changes first, called out**: added matches/permissions/connect domains rendered as `+ Runs on: *.bank.com ⚠` in `--sc-warning`; removals in muted green. Escalations are never findable only inside the diff (SPEC 13.2 DON'T).
- **Code diff** as a collapsed section (unified, mono 13 px) replacing the plain preview.
- Primary button: **Update script**; with escalation present, the button reads **Update and allow new access** — the click acknowledges the escalation explicitly.
- Auto-update policy: non-escalating updates may apply silently per user setting; *any* scope escalation parks the update and surfaces it via badge + Manager `Update` status until reviewed here (SPEC 13.2).

## States

- **Fetching script**: header skeleton + spinner with the source URL shown · Cancel active.
- **Fetch failed**: error card (URL, status), **Retry** (SPEC 14).
- **Invalid script**: "This file is not a valid userscript" + first parse error, mono; no Install button.
- **Already installed (same version)**: banner "Already installed" + **Reinstall** (secondary) / **Open in editor**.

## Keyboard & accessibility

`Esc` = Cancel · `Tab` order: header links → expanders → Cancel → Install (Install is **not** auto-focused — prevents Enter-mash installs). Scope card is a list with real text labels; warning icons carry `aria-label`; diff colors paired with `+`/`−` glyphs (never color-only, SPEC 17).

## Traceability

SPEC 13, 14, 17, 27-P6 · TM analysis §1 (dedicated permission prompts, consent at the decision point) · VM analysis §4 (plain language over `@`-jargon) · SC analysis §1 (existing surfaces redesigned in place; background-script disclosure).
