# Troubleshooting

## "Connect JSON file (autosave)" button does nothing / errors

**Cause:** your browser doesn't support the File System Access API
(Firefox, Safari, or an older Chromium browser).

**Fix:** use **"Download JSON now"** instead, and re-download periodically
or after important edits. See `DEPENDENCIES.md` for the supported-browser
list. Switching to Chrome or Edge enables true autosave.

## My edits disappeared after reopening the file

**Cause:** you were relying on localStorage only (no file connected), and
either opened the dashboard in a different browser / different profile /
incognito window, or cleared browser data.

**Fix:** localStorage is per-browser-profile. If you need edits to survive
across browsers or machines, click **"Connect JSON file (autosave)"** (or at
minimum, **"Download JSON now"** regularly) so the data lives in an actual
file, not just the browser.

## The status bar under the tabs shows "Couldn't write to \<file\>: ..."

**Cause:** the connected file was moved, renamed, or deleted after you
connected it, or permission was revoked.

**Fix:** click **"Connect JSON file (autosave)"** again and re-select (or
recreate) the file.

## Two people have different data and I don't know which is current

**Cause:** this dashboard has no built-in multi-user sync (see
`ARCHITECTURE.md`) — each browser/file is independent until someone
exports and someone else imports.

**Fix:** agree on one "source of truth" JSON file (e.g. store it in a
shared OneDrive folder and have everyone connect to that same file), or
have one person be the sole editor and others view read-only exports.

## The Project Tracker dropdown for "Assigned to" doesn't show someone I removed

**Expected behavior.** Removing a person from the Team tab removes them
from the dropdown going forward, but any project row that already had them
assigned keeps the name as plain text until you manually change it. Re-add
the person (Team tab) if you want the dropdown option back.

## Excel export doesn't match the master workbook

**Check:**
1. Are you using **"Download Updated Excel (.xlsx)"** (Project Tracker tab)
   and not a screenshot/manual copy?
2. Has the master workbook's "Lists" sheet (status/priority options)
   changed recently? If so, `CONFIGURATION.md` explains how to update the
   matching lists in `index.html`.

## The page looks broken / unstyled

**Cause:** `index.html` was opened from a `file://` path with browser
security settings that block inline `<style>`/`<script>` execution (rare),
or the file was partially downloaded/corrupted.

**Fix:** re-copy `index.html` from a known-good source, or re-download it.

## I asked Claude to change something and it says the file is "read-only"

**Cause:** this happens if Claude tries to edit the file through certain
tools while it's marked as read-only content in that session (e.g. cached
skill/plugin content). It does not happen with a normal user file.

**Fix:** ask Claude to use its filesystem tools (Desktop Commander) instead
of the standard file editor, or just say "the file tool says it's
read-only, try the other filesystem tool" — this has a known workaround
used throughout this project's build history (see `CHANGELOG.md`).
