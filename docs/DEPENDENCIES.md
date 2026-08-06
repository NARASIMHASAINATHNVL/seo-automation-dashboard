# Dependencies

## Runtime dependencies

| Dependency | Version | How it's included | Purpose |
|---|---|---|---|
| SheetJS (xlsx.js) | Community edition, bundled ~2023 build | Inlined in full inside `<script>` at the top of `index.html` (minified) | Powers "Download Updated Excel (.xlsx)" — builds a workbook client-side |

No `npm install`, no CDN calls, no build step. The SheetJS source is
embedded directly in `index.html` so the file works fully offline and never
breaks due to a CDN going down or a version bump.

**To update SheetJS:** download a newer build of `xlsx.full.min.js` from the
SheetJS project, and replace the minified block between the `<!--!
xlsx.js ... -->` comment and the matching `</script>` near the top of
`index.html`. Test the "Download Updated Excel" button afterward.

## Browser APIs relied on

| API | Used for | Fallback if unsupported |
|---|---|---|
| `localStorage` | Auto-save of all edits within the browser | None needed — supported everywhere relevant |
| File System Access API (`window.showSaveFilePicker`) | "Connect JSON file (autosave)" — writes straight to a chosen file on disk | Falls back to `downloadJsonFallback()`, a manual `.json` download |
| `Blob` + `URL.createObjectURL` | "Download JSON now" and the CSV/XLSX downloads | None needed — supported everywhere relevant |
| `prefers-color-scheme` media query | Not used by this dashboard (this dashboard has no dark mode — that feature lives in the separate Bot Page-View Checker project) | n/a |

### Browser support for autosave-to-file

The File System Access API (used by "Connect JSON file") is supported in
Chromium-based browsers: Chrome, Edge, Brave, Opera (desktop). It is **not**
supported in Firefox or Safari as of this writing — those browsers will
automatically fall back to the "Download JSON now" button behavior.

## Build tooling

None. This is intentionally a zero-build, zero-dependency-manager project.
Editing `index.html` directly (or via Claude) is the entire workflow.

## Related project (not a dependency)

The "Bot Page-View Checker" tool in `../BOT-View/seo-bot-view-app/` is a
separate Python/Flask project with its own dependencies
(`requirements.txt`, Playwright). It does not share any code with this
dashboard — see `PROJECT_SPECIFICATION.md` §3 for scope boundaries.
