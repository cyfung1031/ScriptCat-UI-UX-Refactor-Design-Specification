# ScriptCat 1.3.2 — UI/UX Analysis (Baseline)

Role in this repo: **the product being refactored**. This document records what the current release does well (preserve it) and where the refactor must intervene (see `SPEC.md`).

> **Evidence basis.** Grounded in the compiled snapshot under `scriptcat-1.3.2/` (HTML shells, bundle composition, i18n keys extracted from `popup.js` and `translation_json.js`, manifest) combined with known shipped product behavior. The snapshot was not executed or rendered; the UI is React + Arco Design (`arco-card`, `arco-collapse`, `arco-dropdown` class names appear in the popup bundle).

## Surface Inventory (from snapshot)

| Surface | Files | Notes |
| --- | --- | --- |
| Popup | `src/popup.html` + `popup.js` (52 KB) | Arco collapse/card-based |
| Manager | `src/options.html` + `options.js` (260 KB) + Monaco bundles | Manager and editor share the options app |
| Install / confirm | `src/install.html`, `src/confirm.html` | Dedicated install + permission surfaces |
| Import / batch update | `src/import.html`, `src/batchupdate.html` | Dedicated flow pages |
| Editor | Monaco (`lib_monaco-*.js`, ~10 chunks) | Full IDE-grade editor |
| i18n | `translation_json.js` (236 KB, all languages bundled); `_locales/` holds only name/description | 7 locales |

---

## 1. Product Scope — the Differentiators to Protect

### Pros

* **Background and scheduled scripts are first-class.** Keys like `background_script`, `enabled_background_scripts`, `run_once`, `stop` show a runtime model no competitor surfaces this directly: scripts that run as services or on cron, controllable (run once / stop) from the popup. This is ScriptCat's core differentiation and the refactor must make it *more* legible, not bury it.
* **Monaco editor.** TypeScript-aware, IDE-grade editing in the browser — strictly stronger developer tooling than the CodeMirror-based competitors. Justifies SPEC 24's "treat script editing as a primary feature."
* **Dedicated flow pages already exist** for install, confirm/permission, import, and batch update — the refactor can redesign these surfaces (SPEC 13) without inventing them.
* **Permission requests are modeled** (`request_permission` flows in the popup bundle), enabling per-script permission UX rather than silent grants.
* **Modern component foundation.** React + Arco Design provides accessible-by-default primitives, theming hooks, and consistent components — a workable base for a design-token refactor (SPEC 6).

### Cons

* **Differentiators are explained in insider vocabulary.** "Background script", "run once", "scheduled" appear without onboarding; a regular user installing a page script never learns why these sections exist (SPEC 2.2, 19).
* **Heavy bundles for management surfaces.** Monaco's chunks and a 260 KB options app are justified for editing, but they weigh on first paint of the manager; the popup (52 KB) is acceptable but Arco brings component CSS the popup only partly uses.

### DO

* Lead the refactor with the background/scheduled-script story: clear status (`Running`, `Scheduled`, `Stopped`), plain-language section names, visible run/stop controls (SPEC 4.2, 21).
* Keep Monaco and the dedicated flow pages; redesign them in place (SPEC 12, 13, 24).

### DON'T

* Don't let parity-chasing with Tampermonkey/Violentmonkey dilute the background-script differentiation.
* Don't make the manager's first paint wait on editor-grade bundles.

---

## 2. Popup

### Pros

* **Already current-page-first in structure:** collapsible `current_page_scripts` and `enabled_background_scripts` sections (Arco collapse) put page context at the top — the right skeleton, matching SPEC 21.
* **Rich per-script controls** via dropdown: edit, delete (with `confirm_delete_script` confirmation — destructive confirmation already practiced), check update, exclude on/off, run once, stop.
* **Page-level states exist:** `page_in_blacklist`, `click_to_reload` (after toggling, the popup tells the user a reload applies the change — closing the loop competitors leave open), update-check feedback (`checking_for_updates`, `latest_version`).

### Cons

* **Header carries external links** — `project_docs`, `community`, `report_issue` sit in the highest-frequency surface alongside `create_script`/`get_script`. Navigation, creation, and marketing compete in one icon row (the exact "reduce header icon clutter" target of SPEC 27 Priority 1).
* **Collapse-card chrome spends vertical space.** Arco card headers, borders, and paddings around each section reduce how many scripts are visible before scrolling in a popup-height surface.
* **Dropdown is the only path to common actions.** One-click acts (toggle aside) hide behind a per-row "more" menu — fast actions like Edit deserve direct affordances (SPEC 18.1).
* **Empty popup state is bare** (`no_data` = "No data") — technical, dead-end text where SPEC 15 requires explanation plus next action.

### DO

* Keep the two-section skeleton; tighten it: counts in headers, less card chrome, more rows visible (SPEC 21).
* Keep `click_to_reload`-style closing of the action loop; generalize it to other stale-state moments.
* Promote Edit (and error/log access when present) to direct row actions (SPEC 18.1, 21.1).

### DON'T

* Don't keep docs/community/issue links in the popup header — move them to the manager's help area (SPEC 8.1, 25.3).
* Don't ship "No data" as an empty state anywhere (SPEC 15.1).

---

## 3. Manager (Options)

### Pros

* **Full management feature set:** sidebar navigation (scripts, subscriptions, logs, tools, settings), sortable script table with enable switches, batch operations, import/export, sync/WebDAV (`lib_webdav.js` ships in the bundle), logs as a first-class section.
* **Arco table gives sorting/selection for free**, aligned with SPEC 22's desktop requirements.

### Cons

* **The table reads as a data grid.** Many columns at similar weight (version, author, source, last-update, type, actions); colored dots and tags compete with names; the row answers "what fields exist" rather than "what needs my attention" — the motivating critique behind SPEC 4.1 and the draft mockup.
* **Persistent toolbar instead of contextual batch bar.** Batch actions render as standing buttons rather than appearing on selection (SPEC 27 Priority 2 exists because of this).
* **Default Arco theming, lightly tuned.** Dark mode in particular shows low-contrast secondary text and dim dividers — the direct subject of SPEC 27 Priority 4.
* **Status communication leans on unlabeled colored dots** in the list (SPEC 27 Priority 3's target).

### DO

* Keep the sidebar IA and the feature set; re-weight the table around name → state → status → actions (SPEC 4.1, 22).
* Make the batch toolbar contextual on selection with count shown (SPEC 10).
* Run the SPEC 6/7 token pass over Arco's theme variables rather than fighting the framework.

### DON'T

* Don't expose every metadata column by default; default to the high-priority set, let users add columns (SPEC 25).
* Don't keep unlabeled colored dots as the status language (SPEC 4.2).

---

## 4. Editor & Developer Experience

### Pros

* Monaco with TypeScript services (`ts.worker.js`), multiple themes, and ScriptCat API typings — best-in-class authoring among the three.
* Logs exist as a manager section; storage and resources are inspectable.

### Cons

* **Debugging is spread across the manager.** Logs live in a separate sidebar section rather than docked at the editor — the round-trip SPEC 24 warns about ("do not separate debugging information too far from the editor").
* **Editor chrome inherits the manager's density issues**; save state and run affordances are not as prominent as SPEC 12 requires.

### DO

* Dock logs/runtime output at the editor; keep the standalone logs page for cross-script investigation (SPEC 12, 24).
* Make Run and save state the editor's strongest signals (SPEC 12.1).

### DON'T

* Don't require leaving the editor to see the consequence of running a script.

---

## 5. Terminology & i18n

### Pros

* 7 locales maintained, including both Chinese variants — the core audience is served.
* Key-based string IDs (`current_page_scripts`) are stable and edit-safe, unlike Tampermonkey's sentence keys.

### Cons

* **All languages ship to every user** in one 236 KB `translation_json.js` instead of per-locale loading via `_locales` (which holds only the store-listing strings). Cost grows linearly with every locale added.
* **Translation-quality drift:** the bundled catalog shows raw or machine-flavored entries in places, and terminology is not enforced across surfaces ("script" vs "userscript", section names varying between popup and manager).
* English strings read as direct translations in spots — the refactor's terminology table (SPEC 19) needs a native-English pass.

### DO

* Keep stable string keys; add a glossary and a native-speaker review pass per SPEC 19/20.
* Load locales on demand.

### DON'T

* Don't bundle every language into every page load.
* Don't let popup and manager translate the same concept differently.

---

## Summary

ScriptCat 1.3.2 already has the **right skeleton and the strongest feature substance**: current-page-first popup structure, background/scheduled scripts, Monaco, dedicated install/import/update flow pages, logs, sync. Its problems are presentation-layer: header clutter in the popup, data-grid tables where hierarchy should be, unlabeled colored dots, default-theme dark mode, dropdown-buried actions, bare empty states, and bundle/i18n weight. That is precisely why this is a UI/UX refactor and not a rebuild — `SPEC.md` rev. 2 sections 4, 6, 7, 13, 15, 21, 22, and 27 map one-to-one onto the cons above.
