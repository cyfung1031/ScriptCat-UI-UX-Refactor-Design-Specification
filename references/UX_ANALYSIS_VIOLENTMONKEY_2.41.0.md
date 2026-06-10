# Violentmonkey 2.41.0 — UI/UX Analysis

Role in this repo: **preferred UX baseline** for clarity, hierarchy, and restraint (see `LLM_REFERENCE_SOURCES.md`).

> **Evidence basis.** This analysis is grounded in the compiled snapshot under `violentmonkey-2.41.0/` (HTML shells, compiled CSS, `_locales/en/messages.json`, manifest) combined with known shipped product behavior. The snapshot was not executed or rendered for this analysis; claims that come only from product knowledge rather than files in this repo are phrased as product behavior.

## Surface Inventory (from snapshot)

| Surface | Files | Notes |
| --- | --- | --- |
| Popup | `popup/index.{html,css,js}` | 320 px fixed-width body; CSS is only ~16 KB |
| Manager | `options/index.{html,css,js}` | Sidebar nav: Installed scripts / Settings / About; CSS ~40 KB |
| Install confirm | `confirm/index.{html,css,js}` | Dedicated page per install |
| Locales | `_locales/` (en, de, ja, ru, vi, zh_CN, zh_TW, …) | 276 strings total — a deliberately small surface area |

---

## 1. Popup

### Pros

* **Current-page first.** Scripts are grouped under explicit headings driven by match state: `Matched scripts`, `Matched disabled scripts`, `Sub-frames only scripts` (`menuMatchedScripts`, `menuMatchedDisabledScripts`, `menuMatchedFrameScripts`). The user never wonders which scripts concern this page.
* **Group counts.** `.menu-group [data-totals]:after { content: ": " attr(data-totals) }` — each group header shows its script count without opening it.
* **Failure is surfaced with a fix.** `menuInjectionFailed` ("Could not inject some scripts.") is shown in the popup itself, paired with a one-click remedy: `menuInjectionFailedFix` ("Retry in auto mode"). Error + next action in one place.
* **Fixed 320 px width.** Content never reflows while the user toggles scripts; long lists scroll internally.
* **Primary tasks are one tap away.** `Open Dashboard`, `Create a new script`, `Find scripts for this site` are flat menu items, not buried in submenus.
* **Configurable ordering** (`Enabled first`, `Group by @run-at stage`) lets power users tune the list without complicating the default.

### Cons

* **Touch targets are small.** `.menu-item { min-height: 2rem }` = 32 px rows with 14 px icons — fine for mouse, below the 44 px minimum for touch.
* **Hidden popup settings.** `popupSettingsHint` = "Right-click: popup settings" — right-click as the only entry to popup configuration is undiscoverable, especially on touch devices.
* **Hover-dependent emphasis.** Disabled/unmatched rows render at `opacity: .65` and only reach full opacity on hover/focus; meaning is conveyed by subtle dimming.
* **Encoded, unlabeled states.** Removed scripts use `text-decoration: line-through red` — a visual code with no text label or tooltip explaining it.
* **Icon-only header actions.** The top button row (`.menu-buttons`, 20 px icons) relies on icon recognition plus tooltip.
* **Mild jargon.** "Matched scripts" is `@match`-developer vocabulary; regular users think "running on this page."

### DO (adopt for ScriptCat)

* Group popup scripts by relationship to the current page, with counts on group headers (SPEC 21).
* Surface injection/runtime failures in the popup with a one-click remediation (SPEC 14).
* Fix the popup width; scroll internally (SPEC 6.1).
* Keep Create / Find / Open Manager as flat, always-visible popup actions.

### DON'T (avoid repeating)

* Don't use 32 px rows where touch is expected — 44 px minimum (SPEC 6.1).
* Don't hide settings behind right-click or other gesture-only entry points.
* Don't communicate state via opacity, strikethrough, or color alone without a label or tooltip (SPEC 4.2, 17).
* Don't label sections with metadata-block jargon when plain language exists (SPEC 19).

---

## 2. Manager (Options / Dashboard)

### Pros

* **Simple, stable navigation.** Three sidebar items (`sideMenuInstalled`, Settings, About). Nothing competes with the script list.
* **User-selectable density.** The script list supports both a card grid and a compact table mode (`.scripts[data-table]`, `data-columns="1..4"`), with responsive column counts and zebra striping computed in CSS. One list, two densities — power users get compactness without forcing it on everyone.
* **Responsive column hiding** (`.hidden-sm`, `.hidden-xs`) — narrow windows lose secondary columns rather than breaking layout.
* **Clear name hierarchy.** `.script-name { font-size: 14px; font-weight: 500 }`, disabled names drop to a muted fill tone — name dominates, state is visible but secondary.
* **Recoverable deletion.** Removed scripts remain visible (strikethrough) with restore, instead of vanishing on click.
* **Search is a first-class citizen** (`labelSearchScript` = "Search scripts..."), with an explicit empty result string (`labelNoSearchScripts` = "No script is found.").
* **Settings page is short.** 276 total strings product-wide means settings stay scannable on one page with groups, not a tab maze.

### Cons

* **Restraint costs capability.** No script groups/tags, no batch toolbar, limited filtering — users with 100+ scripts outgrow it (this is where Tampermonkey/ScriptCat are stronger).
* **Action icons appear on hover** on cards in the shipped product — undiscoverable on touch and invisible to keyboard-first scanning.
* **Settings prose leans technical.** Strings like `labelScriptOptionRequiredGlobal` ("this option must be also enabled in global settings") expose internal option coupling to the user.

### DO

* Offer card/table density toggle on desktop rather than choosing for the user (SPEC 22, 26.4–26.5).
* Keep the sidebar to a handful of stable entries (SPEC 8.1).
* Make deletion visibly recoverable (restore affordance) (SPEC 18.2).
* Hide secondary columns first as width shrinks (SPEC 6.4).

### DON'T

* Don't make hover the only way to reveal row/card actions (SPEC 17, 23).
* Don't let minimalism remove the management tools (filters, batch, tags) that ScriptCat's power users already rely on — ScriptCat must keep them, but contextual (SPEC 10).

---

## 3. Visual System and Theming

### Pros

* **Systematic token scale.** A 16-step neutral ramp (`--fill-0` … `--fill-15`) with `--bg`/`--fg` aliases; dark mode remaps the same ramp under `@media (prefers-color-scheme: dark)`. One system, two themes — exactly the model SPEC 6/7 calls for.
* **Dark mode is designed, not inverted:** dark base is `#1e1e1e` (not pure black), tooltips gain a border (`--tooltip-border-color: #8888`) to stay legible, link blue lightens (`#1e90ff` → `#7baaff`), inputs get dedicated `--input-bg`.
* **CJK-aware system font stack** (PingFang SC, Hiragino Sans GB, Microsoft YaHei alongside system-ui) — fits ScriptCat's audience.
* **Tiny CSS payloads** (16 KB popup / 40 KB options) keep the popup effectively instant.

### Cons

* **Focus outlines are globally removed.** `:focus { outline: none }`, replaced by underline/border-color changes — far below a 3:1-contrast visible focus indicator; keyboard users lose their place (SPEC 17 explicitly requires better).
* **A nearly colorless UI.** The neutral ramp is elegant but status communication leans on dim/bright and decoration rather than the labeled, color+text badge system SPEC 4.2 specifies.
* **Hover inversion color (`#6495ed`) is hard-coded**, outside the token system — even disciplined systems leak one-offs; budget for token governance.

### DO

* Adopt a small token ramp with semantic aliases that dark mode remaps (SPEC 6, 7).
* Treat dark mode as its own tuning pass: softer base, border-driven separation, adjusted accents (SPEC 7.2).
* Keep popup CSS small enough that opening feels instant (SPEC 16: "do not animate the popup opening").

### DON'T

* Don't remove focus outlines — ever (SPEC 17).
* Don't rely on the neutral ramp alone for status; pair color with text labels (SPEC 4.2).

---

## 4. Terminology & i18n

### Pros

* Small string surface (276) keeps terminology consistent by construction.
* Standard `_locales` flow; 7+ languages maintained.

### Cons

* Several user-facing strings are developer-voiced: "Matched scripts", "Group by @run-at stage", `msgReinstallScripts` ("Please re-save or reinstall these script(s):") — accurate, but assumes metadata-block literacy.

### DO / DON'T

* DO keep the translatable surface small and reuse defined terms (SPEC 19, 20).
* DON'T ship `@`-prefixed metadata vocabulary in default-level UI; reserve it for developer surfaces (SPEC 19, 24).

---

## Summary

Violentmonkey demonstrates the **target temperament**: current-page-first popup, tiny stable IA, a real token system, designed dark mode, recoverable deletion, and density choice. Its weaknesses are the cost of that restraint — small touch targets, hover- and gesture-dependent affordances, removed focus outlines, near-zero batch/filter capability, and occasional developer jargon. ScriptCat should copy its hierarchy and visual discipline while keeping its own deeper management features behind progressive disclosure.
