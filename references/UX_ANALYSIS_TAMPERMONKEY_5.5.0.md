# Tampermonkey 5.5.0 — UI/UX Analysis

Role in this repo: **breadth reference** — feature coverage, legacy expectations, mature edge cases (see `LLM_REFERENCE_SOURCES.md`). Less aligned with the target UI/UX direction.

> **Evidence basis.** Grounded in the compiled snapshot under `tampermonkey-5.5.0/` (HTML shells, `style.css`, `editor.css`, `_locales/en/messages.json`, manifest) combined with known shipped product behavior. The snapshot was not executed or rendered; all markup is generated at runtime (every `.html` file ships an empty `<body>`), so structural claims rely on strings, CSS, and product knowledge.

## Surface Inventory (from snapshot)

| Surface | Files | Notes |
| --- | --- | --- |
| Popup | `action.html` + `extension.js` (535 KB) | Empty shell; loads the full 434 KB `style.css` |
| Manager | `options.html` + same bundles | Tabbed dashboard, JS-rendered |
| Permission prompt | `ask.html` | Loads editor + extension bundles even for a prompt |
| Editor | `editor.js` (410 KB) + `editor.css` (CodeMirror) | CodeMirror-based |
| Locales | `_locales/` — **31 languages, 694 strings** | Largest i18n surface of the three |

---

## 1. Feature Breadth and Progressive Disclosure

### Pros

* **Config mode is real progressive disclosure.** A single global setting (`Config_Mode`: `Novice` / `Beginner` / `Advanced`) gates how much of the settings and per-script options surface is visible. Regular users see a short page; experts opt into everything. This is the most successful idea in Tampermonkey's UX and directly validates SPEC 2.2.
* **Every edge case has a home.** 694 strings cover blacklists, sync backends, externals/resources, storage editing, update intervals, badge behavior, context-menu modes, per-site permissions. As a checklist of what a mature userscript manager must eventually answer, nothing else comes close.
* **Privacy implications are disclosed at the decision point.** The badge/search option explains that "the tab's URL is automatically transferred to the search website" with a pointer to the privacy policy — consent text colocated with the setting that triggers it.
* **Permission prompting is a dedicated surface** (`ask.html`): cross-origin and similar grants interrupt with an explicit allow/deny step rather than failing silently.

### Cons

* **Disclosure by global mode is blunt.** Hiding features behind a mode switch means a Novice user who needs one Advanced option must globally re-expose everything. Disclosure per-context ("More options" near where it's needed) scales better.
* **Breadth without hierarchy.** In Advanced mode the settings page becomes a very long, visually flat form; finding one option means scrolling and reading everything. Capability outgrew the information architecture.
* **Inconsistent vocabulary within one feature.** Both `Novice` and `Beginner` exist as adjacent mode names; `Action_Menu` ("Action Menu") names the popup in settings-speak. Exactly what SPEC 19 forbids.

### DO

* Adopt the *goal* of config modes — defaults for regular users, opt-in depth — but implement it as contextual disclosure, not one global switch (SPEC 2.2, 25).
* Use Tampermonkey's string catalog as a completeness checklist when scoping ScriptCat's settings and error states.
* Colocate privacy/permission consequences with the control that causes them (SPEC 13.1, 18).

### DON'T

* Don't gate individual needs behind an all-or-nothing expertise switch.
* Don't let the settings page grow as a flat form; group, search, and prioritize it.
* Don't ship two names for one concept (SPEC 19).

---

## 2. Popup ("Action Menu")

### Pros

* **Dense and complete:** current-tab scripts with enable toggles, version info, script commands, plus Dashboard / new-script / find-scripts entries — heavy users can do almost everything without leaving it.
* **Badge count** on the toolbar icon communicates "how many scripts run here" before the popup is even opened.

### Cons

* **It is a compressed manager, not a page-status view.** Rows mix scripts, commands, utilities, and navigation at similar visual weight; the question "what is affecting this page and is anything wrong?" takes scanning. SPEC 21's "do not make the popup a miniature manager" is written against this pattern.
* **Icon-dense rows with small hit areas**, hover-revealed affordances, and meaning carried by icon shape — weak for touch and accessibility.
* **The popup pays the whole product's style cost:** `action.html` loads the single 434 KB `style.css` shared by every surface. Monolithic styling also makes per-surface design evolution risky — popup, dashboard, and prompts are coupled to one file.
* **Dark, idiosyncratic default theme** that matches neither the OS theme by default nor the dashboard's look — the product's surfaces feel like different eras.

### DO

* Keep badge-count-on-icon as ambient current-page status (SPEC 21).
* Let power users act from the popup (toggle, commands) — but visually subordinate to current-page status (SPEC 21.1).

### DON'T

* Don't replicate the manager inside the popup; the popup answers "this page, right now" (SPEC 21).
* Don't share one monolithic stylesheet across all surfaces; budget styles per surface so the popup stays light (SPEC 6).
* Don't let popup, manager, and prompts drift into different visual languages (SPEC 28 "Consistent").

---

## 3. Manager (Dashboard)

### Pros

* **The installed-scripts table is genuinely efficient for scale:** sortable columns, enable toggles, inline version/size/last-update, multi-select with batch operations, per-row actions. With hundreds of scripts this remains workable — the strongest large-collection story of the three products.
* **Utilities are separated** (import/export, backup to cloud, ZIP) into their own tab instead of polluting the script list.
* **Tab-per-script editing** lets developers keep several scripts open — a real multi-file workflow.

### Cons

* **Database-admin aesthetics.** Many narrow columns, small icon buttons per row, near-equal visual weight across cells — the table communicates data, not priority. Script name does not dominate (contrast SPEC 4.1).
* **Trash/delete sits among neutral row icons** at identical size and prominence — destructive adjacency SPEC 18.2 prohibits.
* **State is icon/color-coded without labels** in places (sync state, update state), requiring legend knowledge.
* **The editor's settings tab** is another long flat form mixing common fields (name, run-at) with rare ones.

### DO

* Match its *capability list* for desktop: sort, multi-select, batch enable/disable/update/export/delete, per-row quick actions (SPEC 10, 22).
* Keep maintenance utilities in a separate area from daily script management (SPEC 8.2, 25.3).
* Support multi-script editing workflows for developers (SPEC 24).

### DON'T

* Don't render the list as an undifferentiated data grid — name and state must dominate (SPEC 4.1, 22).
* Don't place delete inline with neutral actions at equal weight (SPEC 18.2).
* Don't rely on unlabeled icon/color codes for sync/update state (SPEC 4.2).

---

## 4. Terminology & i18n

### Pros

* **31 locales** — the widest reach; proof that full localization of a complex tool is feasible and worth planning layout tolerance for (SPEC 20).
* Long-lived terms (`Dashboard`, `Utilities`, `Enabled`/`Disabled`) are entrenched user vocabulary worth not contradicting gratuitously.

### Cons

* **Sentence-keyed message IDs** (entire sentences with placeholders baked into the key, e.g. the badge privacy string) make strings brittle to edit and prone to stale duplicates.
* Mode names (`Novice` vs `Beginner`) and settings-speak (`Action Menu`) show terminology drift across 694 strings with no enforced glossary.

### DO

* Plan string architecture (stable keys, glossary, plural rules) before the catalog grows (SPEC 20).

### DON'T

* Don't key messages by their English sentence text.
* Don't let the string count grow without a terminology owner — drift is gradual and permanent.

---

## Summary

Tampermonkey is the **capability ceiling**: its feature catalog, table-at-scale management, multi-locale reach, config-mode disclosure, and explicit permission prompts define what ScriptCat must eventually answer. Its failures are of hierarchy, not function — a popup that became a second manager, a data-grid aesthetic where priority should live, flat endless settings, destructive actions beside neutral ones, coupled monolithic styling, and terminology drift. Use it as a scope checklist; do not use it as a layout reference.
