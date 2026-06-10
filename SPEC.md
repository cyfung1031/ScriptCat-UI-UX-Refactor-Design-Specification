# ScriptCat UI/UX Refactor Design Specification (rev. 1)

## 1. Purpose

This document defines the UI/UX design principles, do's, and don'ts for refactoring ScriptCat, a UserScript manager. The goal is to make ScriptCat feel modern, powerful, and approachable while preserving the efficiency expected by advanced users and script developers.

ScriptCat should not feel like a traditional browser extension settings panel. It should feel like a lightweight developer tool with excellent script management, clear runtime visibility, and fast access to common actions.

---

# 2. Core Design Goals

## 2.1 Make Script Management Immediately Understandable

Users should be able to quickly answer:

* Which scripts are installed?
* Which scripts are enabled?
* Which scripts are running on the current page?
* Which scripts have errors or warnings?
* What action can I take next?

### DO

* Prioritize script name, enable state, runtime state, and primary actions.
* Make the current page’s active scripts highly visible in the popup.
* Use clear labels for important states such as `Running`, `Disabled`, `Error`, `Update Available`, and `Scheduled`.
* Use progressive disclosure for advanced actions.

### DON'T

* Do not make users decode multiple colored dots without labels or tooltips.
* Do not display all advanced script metadata by default.
* Do not give equal visual weight to every piece of information.
* Do not require users to open a full management page just to know what is running on the current site.

---

## 2.2 Support Three Main User Types

ScriptCat must serve three user groups:

1. **Regular users**
   Users who mainly install, enable, disable, and update scripts.

2. **Power users**
   Users who manage many scripts, use filters, groups, sync, batch operations, and import/export.

3. **Developers**
   Users who write, debug, run, inspect logs, edit metadata, and manage script resources.

### DO

* Design the default UI for regular users.
* Make advanced tools available but not visually overwhelming.
* Keep developer workflows fast and keyboard-friendly.
* Separate management, runtime, and development concerns.

### DON'T

* Do not expose every feature at the same level.
* Do not design only for developers.
* Do not hide critical runtime information behind deep menus.
* Do not assume all users understand UserScript-specific terminology.

---

# 3. Information Architecture

## 3.1 Desktop Main Page

The desktop main page should be optimized for managing installed scripts.

Recommended layout:

```text
Sidebar
  - Installed Scripts
  - Subscriptions
  - Logs
  - Tools
  - Settings

Main Area
  - Header
  - Search
  - Filters
  - Script List
  - Contextual Batch Toolbar
```

### DO

* Keep the left sidebar simple and stable.
* Use the main area for script list management.
* Keep search globally visible.
* Show filters near the script list.
* Show batch actions only after scripts are selected.
* Make `Create New Script` the most visually prominent primary action.

### DON'T

* Do not permanently show too many batch action buttons.
* Do not place destructive actions such as delete beside neutral actions without separation.
* Do not overuse toolbar buttons.
* Do not make the table look like a database admin panel.

---

## 3.2 Popup

The popup is the highest-frequency interaction surface. It should prioritize the current browser page.

The popup should answer:

```text
What scripts are affecting this page right now?
Can I quickly disable or enable them?
Are there any errors?
How do I open the full manager?
```

### Recommended Popup Hierarchy

```text
Header
  - Product name
  - Global enable switch
  - Settings / More menu

Current Page Status
  - Current domain
  - Number of running scripts
  - Error / warning count if any

Scripts Running on This Page
  - Script name
  - Enable switch
  - Runtime status
  - Quick actions

Other Scripts
  - Collapsed by default or shown with lower priority

Footer
  - Version
  - Open manager
```

### DO

* Make current-page scripts the first visible section.
* Use strong visual distinction between current-page scripts and all installed scripts.
* Show errors and warnings clearly.
* Provide one-click enable/disable for scripts.
* Keep the popup fast, compact, and readable.
* Use collapsible sections for secondary content.

### DON'T

* Do not overload the popup with all management features.
* Do not make global scripts and current-page scripts visually identical.
* Do not hide script runtime errors.
* Do not make the user scroll too much before seeing active scripts.
* Do not place too many icons in the popup header.

---

## 3.3 Mobile Script List

Mobile should use cards instead of dense tables.

Each script card should show only essential information:

```text
Enable switch
Script name
Short description
Status label
Primary actions
```

### DO

* Use card-based layouts.
* Increase touch targets to at least 36–44 px.
* Keep script cards visually scannable.
* Use clear primary actions such as `Edit`, `Run`, or `More`.
* Allow horizontal scrolling for filter chips.
* Keep bottom navigation simple and persistent.

### DON'T

* Do not copy the desktop table layout onto mobile.
* Do not place tiny action icons too close together.
* Do not show too much metadata in each card.
* Do not rely on hover interactions.
* Do not make filters look like static labels.

---

## 3.4 Mobile Editor

The mobile editor should support quick edits, debugging, and script execution, but should not pretend to be a full desktop IDE.

### DO

* Provide a clearly visible `Run` button.
* Show save status, such as `Saved`, `Unsaved`, or `Saving...`.
* Make search, undo, redo, and formatting tools accessible.
* Support landscape mode.
* Provide easy access to logs and runtime output.
* Make syntax errors visible and actionable.
* Allow quick insertion of common UserScript metadata blocks.

### DON'T

* Do not shrink a desktop IDE directly into a phone layout.
* Do not make important actions icon-only without tooltips or labels.
* Do not hide save state.
* Do not make logs difficult to access after running a script.
* Do not use low-contrast syntax highlighting.

---

# 4. Visual Hierarchy

## 4.1 Script List Priority

Every script item should prioritize information in this order:

1. Script name
2. Enable / disabled state
3. Runtime status
4. Error or warning state
5. Primary actions
6. Metadata such as version, author, last update, tags

### DO

* Make script names visually dominant.
* Use secondary text for version, author, and description.
* Use badges for important states.
* Use icons only when their meaning is clear.
* Provide tooltips for ambiguous icons or status indicators.

### DON'T

* Do not let tags, colored dots, and minor metadata compete with the script name.
* Do not use multiple unexplained colors in one row.
* Do not make destructive actions as visually prominent as normal actions.
* Do not overload each row with icons.

---

## 4.2 Status Design

ScriptCat should have a consistent status system.

Recommended status categories:

| Status           | Meaning                             | Suggested Treatment        |
| ---------------- | ----------------------------------- | -------------------------- |
| Enabled          | Script is enabled                   | Blue or green switch       |
| Disabled         | Script is off                       | Gray switch                |
| Running          | Script is active on current page    | Green badge or indicator   |
| Error            | Script failed or threw an exception | Red badge with clear label |
| Warning          | Script may need attention           | Orange badge               |
| Update Available | New version exists                  | Blue or orange label       |
| Scheduled        | Script runs on timer                | Purple or neutral badge    |
| Background       | Background script                   | Neutral badge              |
| Unmatched        | Not active on current page          | Muted text                 |

### DO

* Use color and text together for important statuses.
* Keep status colors consistent across desktop, mobile, popup, and editor.
* Provide tooltip explanations for compact indicators.
* Use stronger visual treatment for errors than for normal states.

### DON'T

* Do not use color alone to communicate meaning.
* Do not use the same color for unrelated meanings.
* Do not use many colored dots without explanation.
* Do not make error states subtle.

---

# 5. Color System

## 5.1 Recommended Color Semantics

The product should use a restrained and consistent color system.

| Color  | Usage                                             |
| ------ | ------------------------------------------------- |
| Blue   | Primary action, selected state, active navigation |
| Green  | Running, healthy, successfully enabled            |
| Orange | Warning, pending, needs attention                 |
| Red    | Error, destructive action, deletion               |
| Gray   | Disabled, inactive, secondary information         |
| Purple | Optional category or advanced script type         |

### DO

* Use blue as the main product color.
* Use red only for errors and destructive actions.
* Use gray to reduce secondary information.
* Make dark mode colors slightly softer but still readable.
* Ensure contrast is accessible in both light and dark themes.

### DON'T

* Do not use too many bright colors in one view.
* Do not use decorative colors for critical status unless clearly defined.
* Do not use low-contrast gray text in dark mode.
* Do not make dark mode visually attractive at the cost of readability.

---

# 6. Light and Dark Mode

## 6.1 Light Mode

Light mode should feel clean, spacious, and readable.

### DO

* Use subtle borders and soft backgrounds.
* Keep main content high contrast.
* Use light blue selection states carefully.
* Ensure selected rows are clearly visible.
* Use enough whitespace between script rows.

### DON'T

* Do not make the interface feel too pale.
* Do not use weak gray text for important metadata.
* Do not rely only on background color to show selection.

---

## 6.2 Dark Mode

Dark mode should feel professional and comfortable for long usage.

### DO

* Increase contrast for secondary text compared with typical decorative dark UIs.
* Make table lines, cards, and input fields distinguishable.
* Use softer shadows and borders.
* Ensure icons are visible without being overly bright.
* Tune syntax highlighting carefully in the editor.

### DON'T

* Do not simply invert the light theme.
* Do not make text, icons, or dividers too dim.
* Do not use pure black everywhere unless intentional.
* Do not let colored badges become visually harsh on dark backgrounds.

---

# 7. Navigation

## 7.1 Sidebar

The sidebar should be stable and predictable.

### DO

* Keep top-level navigation limited.
* Use clear labels and icons.
* Highlight the current section clearly.
* Place help, feedback, and settings in predictable lower positions.

### DON'T

* Do not mix primary product sections with external links.
* Do not overload the sidebar with rarely used pages.
* Do not make icons the only navigation label on desktop.

---

## 7.2 Dropdown Menus

Dropdown menus should be grouped by intent.

Recommended grouping:

```text
Create
  - New Script
  - Get Scripts

Maintain
  - Check Updates
  - Import
  - Export

Help
  - Documentation
  - Community
  - GitHub
  - Report Issue
```

### DO

* Group menu items by purpose.
* Use separators between groups.
* Put common actions near the top.
* Put destructive actions at the bottom with clear styling.
* Use consistent icon style.

### DON'T

* Do not mix creation, maintenance, help, and external links without grouping.
* Do not put destructive actions near primary creation actions.
* Do not make long menus without visual structure.

---

# 8. Search and Filtering

Search and filtering are critical for users with many scripts.

### DO

* Keep search visible on desktop.
* Support filtering by enabled state, script type, tag, status, update state, and error state.
* Use clear selected states for filter chips.
* Allow filters to be reset easily.
* Show the number of matching scripts.
* On mobile, allow filter chips to scroll horizontally.

### DON'T

* Do not make filters look like decorative tags.
* Do not hide active filters.
* Do not make users manually clear filters one by one.
* Do not show too many filters at once on mobile.

---

# 9. Batch Actions

Batch actions are useful but should be contextual.

### DO

* Show batch actions only after one or more scripts are selected.
* Display the number of selected scripts.
* Offer common batch actions:

  * Enable
  * Disable
  * Update
  * Export
  * Delete
  * Change group
  * Add tag
* Require confirmation for destructive actions.
* Separate destructive actions from safe actions.

### DON'T

* Do not permanently display all batch actions.
* Do not make delete visually equal to export or update.
* Do not allow accidental bulk deletion.
* Do not hide the selected count.

---

# 10. Script Item Design

## 10.1 Desktop Row

Recommended script row content:

```text
Checkbox
Enable switch
Script icon or type indicator
Script name
Version / author / short description
Tags
Runtime status
Last updated
Primary actions
More menu
```

### DO

* Keep the script name aligned and easy to scan.
* Use muted text for version and author.
* Show important errors inline.
* Keep row actions consistent across all rows.
* Provide a more menu for advanced actions.

### DON'T

* Do not place too many action icons directly in the row.
* Do not make metadata visually stronger than the script name.
* Do not use ambiguous colored dots without tooltips.
* Do not make rows too short if it harms readability.

---

## 10.2 Mobile Card

Recommended mobile card content:

```text
Header
  - Enable switch
  - Script name
  - Status badge

Body
  - Short description
  - Current-page match status

Footer
  - Edit
  - Run
  - More
```

### DO

* Keep mobile cards simple.
* Make actions large enough to tap.
* Use text labels for important actions.
* Show only the most important script metadata.
* Allow expansion for details.

### DON'T

* Do not overload mobile cards with desktop-level metadata.
* Do not use tiny inline icons.
* Do not require precision tapping.
* Do not show long descriptions by default.

---

# 11. Editor UX

The script editor is for power users and developers. It should feel efficient and reliable.

## 11.1 Editor Layout

Recommended sections:

```text
Top Bar
  - Back
  - Script name
  - Save status
  - More actions

Tabs
  - Code
  - Settings
  - Storage
  - Resources
  - Logs

Editor Area
  - Code editor

Bottom / Side Actions
  - Run
  - Save
  - Search
  - Undo / Redo
  - Console / Logs
```

### DO

* Show save status clearly.
* Make `Run` easy to access.
* Provide visible access to logs.
* Keep code readable in both themes.
* Support keyboard shortcuts on desktop.
* Warn users before leaving with unsaved changes.
* Highlight syntax errors and metadata issues.

### DON'T

* Do not hide runtime output.
* Do not make save state ambiguous.
* Do not rely only on browser-level confirmation for unsaved changes.
* Do not make the editor toolbar icon-only without labels or tooltips.
* Do not use poor contrast in syntax highlighting.

---

# 12. Runtime Feedback and Error Handling

ScriptCat should communicate runtime problems clearly.

### DO

* Show script errors in the popup when they affect the current page.
* Show error counts in the main script list.
* Provide a direct path from an error to logs or editor.
* Use clear error states:

  * `Runtime Error`
  * `Permission Missing`
  * `Update Failed`
  * `Network Error`
  * `Syntax Error`
* Provide retry actions where possible.

### DON'T

* Do not hide errors only inside logs.
* Do not show generic error messages without next steps.
* Do not rely only on red color.
* Do not make users search manually for the failing script.

---

# 13. Empty, Loading, and Edge States

The UI must handle non-ideal states gracefully.

## 13.1 Required Empty States

Design empty states for:

* No scripts installed
* No scripts match the current page
* No search results
* No logs
* No subscriptions
* No scheduled scripts
* No storage data
* No network connection
* Sync not configured

### DO

* Explain the state in plain language.
* Provide a useful next action.
* Use friendly but compact illustrations if appropriate.
* Keep empty states consistent across surfaces.

### DON'T

* Do not show a blank page.
* Do not show only technical text.
* Do not blame the user.
* Do not present too many actions in an empty state.

---

## 13.2 Loading States

### DO

* Use skeleton loading for lists.
* Preserve layout during loading.
* Show progress for long operations such as import, export, update, and sync.
* Allow cancellation when appropriate.

### DON'T

* Do not make the interface jump after loading.
* Do not block the entire app for small operations.
* Do not show indefinite spinners without context.

---

# 14. Accessibility

ScriptCat should be usable by keyboard, screen readers, and users with color vision differences.

### DO

* Use sufficient color contrast.
* Provide keyboard navigation for all major actions.
* Use visible focus states.
* Provide accessible names for icon buttons.
* Avoid using color alone for status.
* Ensure switches have clear on/off labels in accessibility metadata.
* Keep touch targets large enough on mobile.

### DON'T

* Do not remove focus outlines.
* Do not rely only on hover.
* Do not use unlabeled icons.
* Do not communicate errors only through color.

---

# 15. Interaction Design

## 15.1 Primary Actions

Primary actions should be obvious.

Examples:

* Create New Script
* Enable / Disable Script
* Run Script
* Edit Script
* Open Manager
* View Logs

### DO

* Make primary actions visually stronger.
* Keep them in predictable locations.
* Use confirmation for destructive actions.
* Use toast notifications for successful lightweight actions.

### DON'T

* Do not make users search menus for core actions.
* Do not place primary actions inconsistently.
* Do not show confirmation dialogs for every minor action.
* Do not use toast messages for serious errors that require action.

---

## 15.2 Destructive Actions

Destructive actions include:

* Delete script
* Delete storage
* Remove subscription
* Clear logs
* Reset settings

### DO

* Use red styling.
* Require confirmation.
* Explain what will be deleted.
* Offer undo when technically possible.
* Separate destructive actions from normal actions.

### DON'T

* Do not place delete next to edit without spacing or confirmation.
* Do not use only an icon for delete.
* Do not make destructive actions the default button.
* Do not hide the impact of destructive actions.

---

# 16. Content and Terminology

ScriptCat should use clear, consistent product language.

### Recommended Terms

| Use                  | Avoid                           |
| -------------------- | ------------------------------- |
| Installed Scripts    | Script List if ambiguous        |
| Running on this page | Active if unclear               |
| Enable / Disable     | Open / Close                    |
| Update Available     | New version if inconsistent     |
| Logs                 | Console if not actually console |
| Storage              | Data if technically vague       |
| Resources            | Files if not accurate           |
| Subscription         | Feed if inconsistent            |

### DO

* Use consistent labels across desktop, mobile, and popup.
* Use short, direct text.
* Explain technical terms only when needed.
* Use tooltips for advanced concepts.

### DON'T

* Do not mix different terms for the same concept.
* Do not use developer-only terminology for regular user flows.
* Do not use vague labels such as `Normal`, `Other`, or `State` without context.

---

# 17. Popup-Specific Requirements

## 17.1 Popup Default View

The popup default view should prioritize current-page context.

### DO

* Show current domain.
* Show active scripts for the current page first.
* Show script-level toggles.
* Show errors and warnings prominently.
* Provide quick access to edit and logs.
* Keep all installed scripts secondary or collapsed.

### DON'T

* Do not make the popup a miniature version of the full manager.
* Do not show too much metadata.
* Do not hide the current-page script count.
* Do not require multiple clicks to disable a script on the current site.

---

## 17.2 Popup Many-Scripts State

When many scripts exist, the popup must remain usable.

### DO

* Group scripts.
* Collapse inactive groups.
* Provide search.
* Show “View all in manager” when the list is long.
* Keep current-page scripts pinned at the top.

### DON'T

* Do not create a very long unstructured list.
* Do not bury active scripts under unrelated scripts.
* Do not make users scroll through all installed scripts to find current-page scripts.

---

# 18. Desktop-Specific Requirements

### DO

* Use table layout for dense management.
* Support sorting by name, status, update time, type, and enabled state.
* Support keyboard shortcuts.
* Support multi-select.
* Keep filters and search persistent.
* Show contextual batch toolbar after selection.

### DON'T

* Do not make the desktop list visually noisy.
* Do not show every operation at all times.
* Do not overuse icons.
* Do not hide important script status in secondary columns.

---

# 19. Mobile-Specific Requirements

### DO

* Use bottom navigation.
* Use cards instead of tables.
* Make buttons touch-friendly.
* Reduce metadata density.
* Support quick enable/disable.
* Support quick access to current-page scripts.
* Use clear page titles and back navigation.

### DON'T

* Do not compress desktop UI into mobile.
* Do not use hover-only explanations.
* Do not use small click targets.
* Do not show too many columns or inline actions.

---

# 20. Developer Experience Requirements

ScriptCat should feel powerful for script authors.

### DO

* Provide a capable code editor.
* Show metadata clearly.
* Support UserScript header generation.
* Provide logs near the editor.
* Allow quick run and test.
* Show permission and match-rule issues clearly.
* Provide script storage inspection.
* Provide resource management.

### DON'T

* Do not separate debugging information too far from the editor.
* Do not make developers switch between many pages for common debugging.
* Do not hide syntax or permission errors.
* Do not treat script editing as a secondary feature.

---

# 21. Recommended UI Priority Matrix

## 21.1 High Priority

These must be very visible:

* Current-page running scripts
* Enable / disable script
* Script name
* Runtime error
* Create new script
* Search
* Edit script
* Run script

## 21.2 Medium Priority

These should be accessible but not dominant:

* Tags
* Groups
* Last updated time
* Author
* Version
* Update available
* Subscription source
* Script type

## 21.3 Low Priority

These should be hidden behind details or menus:

* Import / export
* Batch delete
* Advanced sync actions
* External links
* Debug-only metadata
* Raw internal IDs
* Rare maintenance actions

---

# 22. Component Guidelines

## 22.1 Switches

### DO

* Use switches only for binary enable/disable states.
* Keep switch position consistent.
* Show disabled state clearly.
* Provide accessible labels.

### DON'T

* Do not use switches for actions that are not persistent.
* Do not place too many switches close together without labels.

---

## 22.2 Badges

### DO

* Use badges for statuses and categories.
* Keep badge text short.
* Use consistent badge colors.
* Limit the number of badges shown by default.

### DON'T

* Do not show too many badges per script.
* Do not use badges as decoration.
* Do not use the same badge color for unrelated meanings.

---

## 22.3 Icon Buttons

### DO

* Use icons for repeated compact actions.
* Provide tooltips.
* Use labels where space allows.
* Keep icon style consistent.

### DON'T

* Do not rely on obscure icons.
* Do not make dangerous icon buttons too subtle.
* Do not use different icons for the same action across views.

---

## 22.4 Tables

### DO

* Use tables on desktop for dense script management.
* Allow sorting.
* Keep columns meaningful.
* Use row hover states.
* Use selected row states.

### DON'T

* Do not use tables on mobile.
* Do not include too many narrow columns.
* Do not make every cell visually equal.

---

## 22.5 Cards

### DO

* Use cards on mobile and popup.
* Keep cards compact.
* Use clear hierarchy inside each card.
* Make the whole card scannable.

### DON'T

* Do not overfill cards.
* Do not place too many actions in card footers.
* Do not make cards visually heavy.

---

# 23. Recommended Refactor Priorities

## Priority 1: Popup Information Hierarchy

Refactor the popup so the current page’s running scripts are the primary focus.

### Key Changes

* Add current-page summary.
* Pin current-page scripts to the top.
* Collapse unrelated scripts.
* Show errors clearly.
* Reduce header icon clutter.

---

## Priority 2: Desktop Batch Toolbar

Refactor the desktop toolbar to be contextual.

### Key Changes

* Show normal toolbar by default:

  * Search
  * Filters
  * New Script
* Show batch toolbar only after selection:

  * Enable
  * Disable
  * Update
  * Export
  * Delete

---

## Priority 3: Status and Color System

Define a consistent system for status colors, badges, and indicators.

### Key Changes

* Replace unexplained colored dots with labeled badges or tooltips.
* Use red only for errors and destructive actions.
* Use green only for healthy/running states.
* Use orange only for warning/pending states.

---

## Priority 4: Dark Mode Readability

Improve dark mode contrast and legibility.

### Key Changes

* Increase contrast of secondary text.
* Improve icon visibility.
* Tune table row borders.
* Improve code editor syntax highlighting.
* Avoid overly dim UI elements.

---

## Priority 5: Mobile Touch UX

Improve mobile usability.

### Key Changes

* Increase action touch targets.
* Reduce card metadata.
* Use text labels for key actions.
* Keep filters horizontally scrollable.
* Improve mobile editor toolbar clarity.

---

# 24. Final Design Principles

ScriptCat should be:

## Clear

The user should understand script state without guessing.

## Fast

Common actions should be available in one click or tap.

## Calm

The interface should not feel noisy, even with many scripts.

## Powerful

Advanced users should still have access to batch tools, logs, filters, storage, resources, and editor features.

## Consistent

Desktop, mobile, popup, light mode, and dark mode should share the same visual language.

## Reliable

Errors, save states, sync states, and runtime states should be visible and actionable.

---

# 25. Summary: DO and DON'T

## DO

* Prioritize current-page scripts in the popup.
* Make script name, enable state, and runtime state highly visible.
* Use contextual batch actions.
* Use cards on mobile and tables on desktop.
* Provide strong empty, error, and loading states.
* Keep status colors consistent.
* Improve dark mode contrast.
* Make touch targets large enough.
* Use clear labels and tooltips.
* Design for regular users first, then progressively reveal advanced features.

## DON'T

* Do not overload the popup with full manager functionality.
* Do not expose every advanced action by default.
* Do not rely on colored dots without explanation.
* Do not make dark mode too low contrast.
* Do not shrink desktop UI directly into mobile.
* Do not hide errors deep inside logs.
* Do not place destructive actions near common actions.
* Do not make important actions icon-only.
* Do not use inconsistent terminology.
* Do not make users guess what state a script is in.
