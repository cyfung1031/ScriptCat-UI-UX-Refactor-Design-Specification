# Cross-Product UI/UX Comparison — ScriptCat 1.3.2 · Tampermonkey 5.5.0 · Violentmonkey 2.41.0

Synthesis of the three per-product analyses:

- `UX_ANALYSIS_SCRIPTCAT_1.3.2.md` — baseline being refactored
- `UX_ANALYSIS_TAMPERMONKEY_5.5.0.md` — capability ceiling / scope checklist
- `UX_ANALYSIS_VIOLENTMONKEY_2.41.0.md` — preferred UX temperament

All DO/DON'T conclusions here are reflected in `SPEC.md` (rev. 2); section numbers refer to it.

---

## 1. At a Glance

| Dimension | ScriptCat 1.3.2 | Tampermonkey 5.5.0 | Violentmonkey 2.41.0 |
| --- | --- | --- | --- |
| Popup philosophy | Current-page-first sections, but cluttered header | Compressed manager, badge count | Current-page-first menu, grouped + counted |
| Manager list | Arco data-grid table, persistent toolbar | Dense sortable table, batch ops, scale champion | Cards ⇄ table density toggle, no batch |
| Editor | Monaco (IDE-grade, TS-aware) | CodeMirror, tabs per script | CodeMirror, simpler |
| Background/scheduled scripts | **Yes — unique differentiator** | No | No |
| Install/permission surfaces | Dedicated pages (install/confirm/import/batchupdate) | Dedicated `ask.html` permission prompt | Dedicated confirm page |
| Theming | Arco defaults, lightly tuned; weak dark mode | Monolithic 434 KB CSS, idiosyncratic dark default | 16-step token ramp, designed dark mode |
| Progressive disclosure | Little (dropdowns) | Global Novice/Beginner/Advanced mode | Restraint by omission |
| i18n | 7 locales, all bundled into one 236 KB file | 31 locales, sentence-keyed strings | 7+ locales, 276 strings, key-based |
| Recoverable deletion | Confirm dialog | Confirm-centric | Strikethrough + restore |
| Failure surfacing in popup | Partial (`click_to_reload`, blacklist state) | Limited | Injection failure + one-click fix |
| Focus visibility | Arco defaults (acceptable) | Mixed | `outline: none` globally (bad) |

---

## 2. What Each Product Proves

**Violentmonkey proves the hierarchy.** A popup grouped by relationship to the current page with counts, a three-entry sidebar, a token ramp remapped for dark mode, density choice, and visible failure-with-remedy — all at 276 strings and ~16 KB of popup CSS. The temperament SPEC 28 names ("Clear, Fast, Calm") is achievable in this product category.

**Tampermonkey proves the scope.** 694 strings and 31 locales enumerate every question a mature userscript manager eventually faces: sync backends, blacklists, externals, storage editing, per-site permissions, privacy disclosures, badge semantics. It also proves the failure mode: capability added without hierarchy yields flat settings walls, data-grid tables, and a popup that became a second manager.

**ScriptCat proves the substance.** Background/scheduled scripts, Monaco, logs, WebDAV sync, and dedicated install/import/update flow pages already exist. Nothing structural is missing — the gap is presentation: header clutter, unlabeled colored dots, default-theme dark mode, dropdown-buried actions, bare "No data" empty states.

**Direction:** Violentmonkey's hierarchy + Tampermonkey's capability checklist + ScriptCat's substance = the refactor target.

---

## 3. Consolidated DO (with sources and SPEC mapping)

| # | DO | Learned from | SPEC |
| --- | --- | --- | --- |
| 1 | Group popup scripts by current-page relationship, with counts in group headers | VM | 21 |
| 2 | Surface runtime/injection failures in the popup with a one-click fix | VM | 14 |
| 3 | Fix popup width; scroll internally; keep popup assets tiny so it opens instantly | VM (320 px, 16 KB) vs TM (434 KB) | 6.1, 16 |
| 4 | Build theming on a small token ramp with semantic aliases; tune dark mode as its own pass | VM | 6, 7 |
| 5 | Offer card/table density choice on desktop | VM | 22, 26 |
| 6 | Make deletion recoverable (restore), not merely confirmed | VM | 18.2 |
| 7 | Serve regular users by default and let depth be opted into — per context, not via a global expertise switch | TM (goal right, mechanism wrong) | 2.2, 25 |
| 8 | Use TM's string catalog as the completeness checklist for settings, errors, and edge cases | TM | 13, 15 |
| 9 | Keep desktop scale tools: sort, multi-select, contextual batch enable/disable/update/export/delete | TM capability, VM restraint as the default face | 10, 22 |
| 10 | Colocate privacy/permission consequences with the control that causes them | TM | 13.1, 18 |
| 11 | Keep badge count as ambient current-page status | TM | 21 |
| 12 | Lead with background/scheduled scripts: plain-language labels, visible Run/Stop, `Scheduled` status | SC differentiator | 4.2, 21 |
| 13 | Keep Monaco and the dedicated install/confirm/import/batch-update pages; redesign in place | SC | 12, 13, 24 |
| 14 | Close action loops the way `click_to_reload` does — state the consequence and the next step | SC | 14, 18 |
| 15 | Keep stable string keys; add glossary + native-English pass; load locales on demand | SC keys good, TM sentence-keys bad | 19, 20 |

## 4. Consolidated DON'T (with sources and SPEC mapping)

| # | DON'T | Seen in | SPEC |
| --- | --- | --- | --- |
| 1 | Don't make the popup a compressed manager mixing scripts, utilities, and navigation at equal weight | TM | 21 |
| 2 | Don't keep external/marketing links in the popup header | SC | 8.1, 25.3 |
| 3 | Don't render script lists as data grids where every column has equal weight | TM, SC | 4.1, 22 |
| 4 | Don't communicate status via unlabeled colored dots, opacity, or strikethrough alone | SC dots, VM opacity/strikethrough | 4.2, 17 |
| 5 | Don't place delete at equal prominence beside neutral row actions | TM | 18.2 |
| 6 | Don't remove focus outlines or substitute them with underlines | VM | 17 |
| 7 | Don't use sub-44 px touch targets or hover/right-click-only affordances | VM (32 px rows, right-click settings), TM (hover icons) | 6.1, 17, 23 |
| 8 | Don't grow settings as a flat wall; don't gate features behind one global expertise mode | TM | 2.2, 8 |
| 9 | Don't share one monolithic stylesheet across all surfaces | TM | 6 |
| 10 | Don't let surfaces drift into different visual languages (popup vs manager vs prompts) | TM | 28 |
| 11 | Don't ship metadata-block jargon (`@run-at`, "Matched scripts") in default-level UI | VM, SC | 19, 24 |
| 12 | Don't use two names for one concept (`Novice`/`Beginner`) or key strings by sentence text | TM | 19, 20 |
| 13 | Don't ship "No data"-style empty states | SC | 15 |
| 14 | Don't bundle all languages into every page load | SC | 20 |
| 15 | Don't bury common actions (Edit) exclusively in per-row "more" menus | SC | 18.1 |
| 16 | Don't let minimalism delete the management depth power users already rely on | VM (as cautionary opposite) | 2.2, 10 |

---

## 5. Open Tradeoffs (to resolve in design, not by copying a product)

1. **Density default.** VM defaults friendly and offers a table; TM defaults dense. ScriptCat's audience skews technical, but SPEC 2.2 says design defaults for regular users. *Recommendation: comfortable default + persistent density toggle (DO #5).*
2. **Popup capability ceiling.** VM keeps the popup thin; TM lets everything happen there. SPEC 21 sides with thin-plus-quick-actions; background-script Run/Stop is the one ScriptCat-specific power that must stay in the popup.
3. **Disclosure mechanism.** TM's global mode vs contextual "show more". SPEC 2.2/25 choose contextual; accept the cost that depth is spread across contexts rather than one switch.
4. **Editor-adjacent logs vs standalone logs page.** SC has the standalone page; SPEC 24 wants logs docked at the editor. Keep both: docked for the edit-run loop, standalone for cross-script investigation.
