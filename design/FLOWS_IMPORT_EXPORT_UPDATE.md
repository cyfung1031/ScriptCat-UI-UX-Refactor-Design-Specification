# Import / Export / Batch Update — Implementation Specification

Redesigns the existing `import.html` and `batchupdate.html` surfaces plus the export path. Shared rule for all three: **preview → explicit apply → honest summary**; nothing applies silently and partial failure is never reported as success (SPEC §8.3). In rev. 4 the version system underwrites these flows: overwrites and updates record versions, so the flows can be confident without being scary (SPEC §3.4).

## 1. Import

Entry: Manager `⋮` → Import · Settings → Sync/Backup · drag-drop of a backup file onto the Manager.

### Step 1 — Choose file

Centered drop zone ("Drop a backup file here, or browse") accepting zip/JSON backups. Parse errors state file name + reason, with **Choose another file**.

### Step 2 — Preview (the core step)

```text
Import from scriptcat-backup-2026-06-01.zip          37 items
[☑] all  Name              Version   Status        Action
[☑]      Bilibili Evolved  2.1.0     New           Import
[☑]      AdGuard Extra     1.0.3     Conflict ▾    [Skip|Overwrite|Keep both]
[☐]      broken.user.js    —         Invalid ⚠     —
Apply to all conflicts: [Skip] [Overwrite] [Keep both]
                              [Cancel]  [Import 35 items]
```

- Status values: **New** · **Conflict** (same name/UUID exists; expandable row shows both versions + last-modified to inform the choice) · **Invalid** (reason inline, unchecked, uncheckable).
- Per-row conflict resolution + apply-to-all. Default for conflicts: **Skip** — import never overwrites silently. **Overwrite records a version** of the existing script first; the row says so ("Current version kept in history") — reversibility replaces fear (SPEC §3.4).
- Settings/storage data contained in the backup is listed as its own checkbox group ("Include script storage data").
- Primary button always states the real count: **Import 35 items**.

### Step 3 — Progress & summary

Determinate progress with per-item ticks; cancellable between items (SPEC §13.1). Summary: `32 imported · 3 skipped · 2 failed` — failures expandable with reasons and a **Retry failed** action. The import lands as one entry in Manager → History & Trash → Recent changes (SPEC §6.2). Toast on success; the summary screen itself is the record (toasts are not the only record of failure, SPEC §13.3).

## 2. Export

Entry: Manager `⋮` → Export · batch bar → Export (pre-selects the selection).

One dialog, no wizard: script list with checkboxes (pre-checked = current selection or all) · options: **Include script storage data** (off by default; the helper text explains it may contain site data — privacy consequence colocated with the control, SPEC §8.3; TM analysis §1) · **Include settings** · format note (zip). Primary: **Export N scripts** → file download + toast. Empty selection disables the button with inline hint.

Terminology: this flow is **Backup/Export** — distinct from automatic **History/Versions** (SPEC §16.1); the dialog links to History & Trash for users who actually wanted rollback.

## 3. Batch update (check + apply)

Entry: Manager `⋮` → Check updates · `Cmd/Ctrl+K` → "Check all updates" · scheduled background checks land their results here.

### Checking

Non-blocking: progress lives in the Manager status footer (`Checking 12 / 37…`) with per-row spinners only on rows being checked; the table stays usable (SPEC §8.3 DON'T).

### Review screen (when updates exist)

```text
Updates available (4)
[☑] Name             1.4.2 → 1.5.0   ✓ No new permissions     [View changes]
[☑] Name             0.9 → 1.0      ⚠ New: runs on *.bank.com [Review required]
                       [Later]  [Update 3 scripts]
```

- Each row states the escalation verdict up front: **✓ No new permissions** or **⚠ New access** with the specific addition (SPEC §8.2).
- Escalating rows are excluded from one-click bulk update; their button routes to the full update-confirmation page (`INSTALL_CONFIRM.md`, update mode). Until reviewed they carry the `Update waiting` attention state (SPEC §3.2).
- Non-escalating rows update in bulk; **each records a version** and the result toast offers **Roll back** (SPEC §3.4).
- **View changes** opens the same confirmation page for anyone who wants the diff.

### Result

Per-item progress → summary (`3 updated · 1 awaiting review · 0 failed`); failures with reason + retry (`Update Failed` is a defined state, SPEC §13.2). Manager rows refresh their attention states immediately; applied updates appear in History & Trash → Recent changes with inline rollback.

## States & edge cases (all three flows)

- **Nothing to do**: "All scripts are up to date" / "This backup contains no scripts" — plain language + dismiss (SPEC §13.1).
- **Network loss mid-flow**: completed items stay completed; remainder marked `Failed — network`; summary reflects the truth; **Retry failed** re-runs only the remainder.
- **Large sets (100+)**: lists virtualize; progress shows count, not just a bar.

## Keyboard & accessibility

Dialogs trap focus, `Esc` cancels (with confirm if work is in flight). Checkbox lists support `Space`/`Shift+click` ranges. Status words (`New`, `Conflict`, `Invalid`, `Failed`) are text + icon, never color-only (SPEC §15). Progress announces via `aria-live="polite"`; the final summary via `role="status"`.

## Traceability

SPEC §3.2, §3.4, §6.2, §8.2–8.3, §13, §15, §16.1, §18-P5 · TM analysis §1 (privacy colocation), §3 (utilities separated from daily management) · VM analysis (import/export data options) · SC analysis §1 (existing `import.html`/`batchupdate.html` redesigned in place).
