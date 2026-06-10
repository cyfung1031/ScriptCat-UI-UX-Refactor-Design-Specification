# Import / Export / Batch Update — Implementation Specification

Redesigns the existing `import.html` and `batchupdate.html` surfaces plus the export path. Shared rule for all three: **preview → explicit apply → honest summary**; nothing applies silently and partial failure is never reported as success (SPEC 13.3, "fail loud").

## 1. Import

Entry: Manager `⋮` → Import · Settings → Sync/Backup · drag-drop of a backup file onto the Manager.

### Step 1 — Choose file

Centered drop zone ("Drop a backup file here, or browse") accepting zip/JSON backups. Parse errors state file name + reason, with **Choose another file**.

### Step 2 — Preview (the core step)

```text
Import from scriptcat-backup-2026-06-01.zip          37 items
[☑] всё  Name              Version   Status        Action
[☑]      Bilibili Evolved  2.1.0     New           Import
[☑]      AdGuard Extra     1.0.3     Conflict ▾    [Skip|Overwrite|Keep both]
[☐]      broken.user.js    —         Invalid ⚠     —
Apply to all conflicts: [Skip] [Overwrite] [Keep both]
                              [Cancel]  [Import 35 items]
```

- Status values: **New** · **Conflict** (same name/UUID exists; expandable row shows both versions + last-modified to inform the choice) · **Invalid** (reason inline, unchecked, uncheckable).
- Per-row conflict resolution + apply-to-all (SPEC 13.3). Default for conflicts: **Skip** — import never overwrites silently.
- Settings/storage data contained in the backup is listed as its own checkbox group ("Include script storage data", VM's `labelImportScriptData` precedent).
- Primary button always states the real count: **Import 35 items**.

### Step 3 — Progress & summary

Determinate progress with per-item ticks; cancellable between items (SPEC 15.2). Summary: `32 imported · 3 skipped · 2 failed` — failures expandable with reasons and a **Retry failed** action. Toast on success; the summary screen itself is the record (toasts are not the only record of failure, SPEC 26.6).

## 2. Export

Entry: Manager `⋮` → Export · batch bar → Export (pre-selects the selection).

One dialog, no wizard: script list with checkboxes (pre-checked = current selection or all) · options: **Include script storage data** (off by default; explains it may contain site data — privacy colocation, TM analysis §1) · **Include settings** · format note (zip). Primary: **Export N scripts** → file download + toast. Empty selection disables the button with inline hint.

## 3. Batch update (check + apply)

Entry: Manager `⋮` → Check updates · scheduled background checks land their results here.

### Checking

Non-blocking: progress lives in the Manager status footer (`Checking 12 / 37…`) with per-row spinners only on rows being checked; the table stays usable (SPEC 13.2 DON'T: don't block while checking).

### Review screen (when updates exist)

```text
Updates available (4)
[☑] Name             1.4.2 → 1.5.0   ✓ No new permissions     [View changes]
[☑] Name             0.9 → 1.0      ⚠ New: runs on *.bank.com [Review required]
                       [Later]  [Update 3 scripts]
```

- Each row states the escalation verdict up front: **✓ No new permissions** or **⚠ New access** with the specific addition (SPEC 13.2).
- Escalating rows are excluded from one-click bulk update; their button routes to the full update-confirmation page (`INSTALL_CONFIRM.md`, update mode). Non-escalating rows update in bulk.
- **View changes** opens the same confirmation page for anyone who wants the diff.

### Result

Per-item progress → summary (`3 updated · 1 awaiting review · 0 failed`); failures with reason + retry (SPEC 14: `Update Failed` is a defined state). Manager rows refresh their `Update` badges immediately.

## States & edge cases (all three flows)

- **Nothing to do**: "All scripts are up to date" / "This backup contains no scripts" — plain language + dismiss (SPEC 15.1).
- **Network loss mid-flow**: completed items stay completed; remainder marked `Failed — network`; summary reflects the truth; **Retry failed** re-runs only the remainder.
- **Large sets (100+)**: lists virtualize; progress shows count, not just a bar.

## Keyboard & accessibility

Dialogs trap focus, `Esc` cancels (with confirm if work is in flight). Checkbox lists support `Space`/`Shift+click` ranges. Status words (`New`, `Conflict`, `Invalid`, `Failed`) are text + icon, never color-only (SPEC 17). Progress announces via `aria-live="polite"`; the final summary via `role="status"`.

## Traceability

SPEC 13.2–13.3, 14, 15, 17, 26.6, 27-P6 · TM analysis §1 (privacy colocation), §3 (utilities separated from daily management) · VM analysis (import/export data options) · SC analysis §1 (existing `import.html`/`batchupdate.html` redesigned in place).
