# Changelog

All notable changes to the SEO Automation & Projects Dashboard.

## 2026-08-21 — Clickable Dashboard KPI cards with project drill-down

The six Dashboard KPI cards (Total Projects, Assigned, Ongoing, Review
Pending, Completed & Approved, Need Help) were purely decorative counters
— clicking them did nothing, so there was no quick way to see *which*
projects made up a given number without going to the Project Tracker and
filtering manually. Each card is now clickable and opens a modal listing
the exact matching projects.

### Added
- `kpiDrilldownModalOverlay` modal (new HTML block after the Analyst
  Profile modal): same overlay + `.modal-card` pattern as
  `analystProfileModalOverlay` / Tool Usage Reason modal, `max-width:640px`
  responsive card, closes on backdrop click, close button, and Escape
  (wired into the existing shared `keydown` listener).
- `KPI_DRILLDOWN_CONFIG` + `openKpiDrilldown(kind)` /
  `closeKpiDrilldownModal()`: `kind` is one of `total`/`assigned`/
  `ongoing`/`review`/`done`/`help`. Each filter mirrors `computeSummary()`
  exactly, so the count on the card and the list in the modal always
  agree — `assigned`/`ongoing`/`review`/`done` only include projects whose
  `assignedTo` is a current `ANALYSTS` entry and whose `status` matches;
  `help` includes every project with `help === "Yes"` regardless of
  status/assignee; `total` is every project. Zero-match buckets still
  open, showing a friendly "No projects in this bucket right now." empty
  state instead of being disabled. Rows show project name, assignee
  (`avatarHtml()`), status pill (`STATUS_PILL_CLASS`), and target date;
  clicking a row closes the modal and calls the existing `jumpToProject(id)`
  to scroll/flash that row in the Project Tracker.
- `.kpi-clickable` (cursor:pointer) and `.kpi-drilldown-row` (cursor:pointer
  + `:hover{background:var(--surface-1)}`, matching the existing
  `.attn-row:hover` affordance) CSS rules.

### Changed
- The six Dashboard KPI `<div class="kpi ...">` cards (Total/Assigned/
  Ongoing/Review Pending/Completed & Approved/Need Help) gained the
  `kpi-clickable` class, an `onclick="openKpiDrilldown('...')"` handler,
  and a descriptive `title` tooltip. The existing `.kpi:hover` lift/shadow
  effect (shared with the non-clickable KPI rows on My Work and the Tool
  Usage tab) is unchanged. The "N projects overdue or stale" banner above
  the cards already called `jumpToProject()` on its pills — left as-is.

## 2026-08-21 — Save confirmation for inline edits (Project Tracker + Daily Log)

Inline edits in the Project Tracker saved silently for every field except
`assignedTo` (only that one showed a toast) — analysts had no visual
confirmation that a status/priority/date/notes edit had actually been
saved, since there is no explicit Save/Submit button anywhere in these
tables. Same silent-save gap existed for the Daily Status Log's per-analyst
task cells. This adds a clear success confirmation on every successful
inline write, without adding any Save button or changing table layout.

### Added
- `flashSaved(rowId, field)` (new helper, next to `showToast`): briefly
  glows the specific `<input>`/`<select>` that was just saved with a
  `--good`-colored ring (`.save-flash` / `@keyframes field-saved-flash`,
  1.2s, auto-removes itself) — a second, at-the-cell confirmation beyond
  the toast, looked up via the row's `id` (`proj-row-<id>` /
  `daily-row-<idx>`) and a new `data-field="<field>"` attribute added to
  every editable cell's input/select (via `selectHtml()` and the
  `renderProjects()` / `renderDaily()` markup).
- `PROJECT_FIELD_SAVED_MESSAGES` lookup: per-field success toast wording
  for `updateProject()` — "Status updated ✓", "Priority updated ✓",
  "Date assigned saved ✓", "Target date updated ✓", "Completion date
  saved ✓", "Help flag updated ✓", "Notes saved ✓", "Project name saved
  ✓".

### Changed
- `updateProject()`: the success branch now shows a toast for **every**
  field, not just `assignedTo` (whose existing message wording is
  unchanged), and calls `flashSaved("proj-row-"+id, field)` after every
  successful write. The failure path (`friendlyWriteError`, red/danger
  toast) is untouched.
- `updateDaily()`: the per-analyst task-cell write (previously fully
  silent on success) now shows "Task saved ✓" / "Task cleared ✓" and
  calls `flashSaved("daily-row-"+idx, field)`; the date-row write's
  existing "Date updated" toast gained a ✓ and a matching flash. Failure
  path unchanged.

## 2026-08-21 — Project Tracker visual polish pass

Pure CSS/markup pass to fix the Project Tracker tab looking "clumsy" next
to the Dashboard/Leaderboard tabs' card-based design language. No
permission logic, `updateProject()`, `canEditProject()`, or the
assign-dropdown feature were touched — behavior is identical, only the
markup wrapping and CSS classes changed.

### Added
- `.projects-toolbar` / `.toolbar-group` (`toolbar-primary`,
  `toolbar-filters`, `toolbar-actions`): the Project Tracker toolbar is
  now three flex groups (Add Project | search + "Show only my rows" |
  Import/Download/Print) instead of one flat row split by a `.spacer`,
  so it wraps predictably at 1280/1440/1600px and stacks cleanly under
  ~900px instead of buttons falling onto random lines.
- `.status-select-wrap` / `.priority-select-wrap`: the editable Status
  and Priority `<select>` cells in `renderProjects()` are now wrapped in
  a span carrying the row's existing `STATUS_PILL_CLASS` value (Status)
  or a `data-priority` attribute (Priority), so both dropdowns render as
  colored pill/badge selects consistent with the read-only status pill
  styling, using only existing color vars (`--neutral`/`--blue`/
  `--warning`/`--good`/`--critical`).
- `.table-wrap` scroll-shadow affordance (`background-attachment:local`
  edge-gradient trick) so any horizontally-scrollable table (Project
  Tracker included) shows a subtle shadow on the side that still has
  more content — no JS needed, and it's a no-op when there's nothing to
  scroll.

### Changed
- `#projectsTable` cell padding increased (`td` 5px/10px → 9px/12px,
  `th` 8px/10px → 10px/12px) for more breathing room; Project/Notes
  columns now wrap long text (`white-space:normal;word-break:break-word`)
  instead of being squeezed, while Priority/Help Needed got a
  `max-width` so they don't over-stretch. The action column (`th:last-child`
  / `td[data-label=""]`) is now centered with the Delete button and the
  "View only" tag both stretched to the same width for consistent
  alignment down the column.
- Global `select`/`input[type=text]`/`input[type=date]` padding
  increased slightly (5px/7px → 6px/9px) and their focus ring switched
  from a hardcoded blue `rgba()` to `color-mix(in srgb, var(--accent) …)`
  so it themes correctly in dark mode — affects all tables app-wide,
  not just Project Tracker, as a consistency side-effect.
- Status stripe (`tr.status-stripe td:first-child`) inset shadow
  thickened 3px → 4px for a slightly clearer status-color cue.
- Confirmed the table header (`th{position:sticky;top:0;}` inside the
  scrolling `.table-wrap`) was already sticky — no JS change needed,
  just the padding/contrast tweaks above.

## 2026-08-21 — Direct admin assign/reassign (no pick-request needed)

Admins previously could only change a project's assignee via the
Project Tracker table's existing assignee dropdown, or indirectly by
approving an analyst-initiated pick request. This adds a direct
admin-only assign path from the sidebar too, and makes the two paths
consistent with each other.

### Added
- Sidebar (`renderProjectSidebar()`): unassigned projects with no
  pending pick request now show a compact "Assign to…" dropdown
  (admin-only, class `sidebar-assign-select`) populated with
  `ANALYSTS`, styled to match the existing compact sidebar action
  buttons. Selecting a name calls the same `updateProject(id,
  'assignedTo', value)` path used by the Project Tracker table, so it
  goes through identical validation, Supabase write, rollback-on-error,
  and toast behavior. Non-admins and rows with a pending request are
  unaffected.

### Changed
- `updateProject()`: when an admin changes `assignedTo` on a project
  that has a pending pick request (`pickStatus === 'pending'`), the
  pending request is now defensively cleared (`pick_status` reset to
  `'none'`, `pick_requested_by` to `null`) since the admin's direct
  assignment supersedes it. This secondary write is wrapped in its own
  try/catch so a missing `pick_status`/`pick_requested_by` column (the
  columns are still behind a queued migration) never rolls back the
  assignment itself. Also added a success toast ("Assigned to X" /
  "Project unassigned") specifically for the `assignedTo` field, since
  other inline-edited fields don't toast on success and this one
  benefits from explicit confirmation.

## 2026-08-20 — Responsive layout & widget sizing pass

CSS/layout-only pass fixing complaints that the UI "isn't responsive" and
that widgets look inconsistently tiny/huge, following the previous pass
that added the persistent left `#projectSidebar` (which reduced the
horizontal space available to `<main>` on every tab).

### Changed
- `.kpi-row` (used for every KPI card row across Dashboard, My Work,
  Project Tracker, Tools Usage, Leaderboard) switched from a rigid
  `grid-template-columns:repeat(6,1fr)` to
  `repeat(auto-fit,minmax(150px,220px))`. Cards now size themselves
  consistently (never squished, never stretched) and the row simply wraps
  as available width shrinks — no more separate 900px/720px step-down
  overrides needed for the base case.
- Removed the `.kpi-row.kpi-row-5` (My Work) fixed `repeat(5,1fr)` override.
  Because two-class selectors are more specific than the single-class
  `.kpi-row` rule, this override was silently winning over *both* existing
  responsive breakpoints (900px and 720px), so the My Work KPI row never
  actually shrank below 5 fixed columns on narrow screens even though it
  looked like it should. It now inherits the same auto-fit sizing as every
  other KPI row.
- Removed the inline `style="grid-template-columns:repeat(2,1fr);"` on the
  Tools Usage KPI row. With only 2 KPIs stretched across the full row width
  behind the new sidebar-narrowed `<main>`, these cards rendered
  dramatically oversized next to the compact 6-up Dashboard KPI cards —
  the clearest instance of the "some widgets are very big" complaint. Now
  uses the shared `.kpi-row` auto-fit sizing like every other tab.
- `.charts-row` and `.spotlight-row` (Leaderboard) single-column collapse
  breakpoint raised from `max-width:900px` to `max-width:1200px`. The
  272px-wide sidebar (plus its 24px margin) eats roughly 300px of viewport
  width on every tab now, so the old 900px trigger left two-column chart
  rows cramped into an effective ~600px of content width before they had a
  chance to stack.
- Dashboard "Projects per Analyst by Status" bar chart container height
  changed from `340px` to `300px` to match its side-by-side neighbor
  ("Overall Status Split" donut, already `300px`) inside the same
  `.charts-row`. Previously the two cards in the same row had visibly
  different heights.
- `main{max-width:...}` increased from `1400px` to `1600px` so wide
  monitors regain roughly the ~300px of usable content width now claimed
  by the sidebar, instead of `margin:0 auto` centering an unnecessarily
  narrow column inside `.app-shell-main`.

### Reviewed, no change needed
- `#projectSidebar` width (272px) and its `<720px` stacking behavior
  (`.app-shell{flex-direction:column}`) — both already sensible; project
  names use `text-overflow:ellipsis` so no awkward wrapping.
- `.kpi` card padding — already a single shared class reused by every tab,
  so Dashboard/My Work/Tools Usage KPI cards were already consistent.

## 2026-08-20 — Persistent left project sidebar + pick-request workflow

### Added
- New collapsible `<aside id="projectSidebar">` visible on every tab,
  listing every project from the already-loaded `projects` array with a
  status-colored dot (reusing the exact `STATUS_HEX` map from Project
  Tracker — no new colors invented) and either the assignee's name
  (`avatarHtml()` + name, same as elsewhere) or an "Unassigned" label.
  Collapsible via a toggle button in the sidebar header; collapsed/expanded
  state persists per-browser in `localStorage`
  (`seo-automation-dashboard-sidebar-collapsed`), same pattern as the
  existing theme toggle.
- Non-admin users see a "Pick this project" button on unassigned rows.
  `requestPickProject()` attempts
  `sb.from('projects').update({ pick_requested_by, pick_status:'pending' }).eq('id', projectId).is('assigned_to', null)`
  wrapped in try/catch: success shows a toast and flips the row to a
  disabled "Pending approval" button; failure (including "column doesn't
  exist yet") shows a friendly toast via `friendlyPickError()` /
  `isMissingColumnError()` and never throws or breaks the rest of the
  sidebar — same defensive pattern as `renderToolsUsage()`.
- Admins (`isAdminUser()`) see, on any row with `pickStatus === 'pending'`,
  who requested it (`profileNameById(p.pickRequestedBy)`) plus Approve
  (`approvePickRequest()` — sets `assigned_to`/`pick_status:'approved'`)
  and Reject (`rejectPickRequest()` — sets `pick_status:'rejected'`,
  clears `pick_requested_by`) buttons.
- `mapProjectRow()` now also maps `pickRequestedBy`/`pickStatus`,
  defaulting missing/undefined values to `null`/`"none"` so the sidebar
  renders correctly whether or not the migration below has landed.
- `renderProjectSidebar()` is called from the existing `renderAll()`, so
  it stays in sync with the same realtime `projects` subscription
  (`subscribeRealtime()`) already driving Project Tracker — no second
  competing subscription was added.
- Layout: `<nav>` and `<main>` are now wrapped in `.app-shell` (flex row)
  / `.app-shell-main`, with `#projectSidebar` as a sibling flex item to
  their left. `<header>` and `<footer>` are unchanged, full-width, outside
  this wrapper. On narrow viewports the existing `@media(max-width:720px)`
  breakpoint (same one `header`/`main`/`nav` already use) now also stacks
  `.app-shell` into a column and caps the sidebar to a `max-height:280px`
  scrollable strip above the nav, instead of a fixed-width side column.

### Known limitation (by design)
- **`pick_requested_by` (uuid, references `profiles(id)`, nullable) and
  `pick_status` (text, nullable — values `'pending' | 'approved' |
  'rejected'`) do not exist on the `projects` table yet.** The migration
  is queued separately. Until it lands, the sidebar's project list renders
  correctly (status dots, names, "Unassigned" labels), but the actual Pick
  / Approve / Reject button clicks will just show a "Project picking isn't
  set up yet — check back soon" toast instead of persisting anything —
  UI-complete, backend-inert.

## 2026-08-20 — Tools Usage: day x hour usage heatmap

### Added
- New "Usage Heatmap — Day & Hour" card on the Tools Usage tab, placed
  right after the Weekly/Monthly Usage `charts-row`. Shows a 7x24 grid
  (rows = day of week, columns = hour of day) of how many tool uses were
  logged in each day/hour bucket, so the team can spot when tools actually
  get used.
- `computeToolUsageHeatmap()` buckets every `tool_usage_events.used_at`
  timestamp by local day-of-week and hour-of-day into a `grid[7][24]`
  array and returns `{ grid, maxCount }`. Rows run **Mon..Sun** (row 0 =
  Monday) via `(date.getDay() + 6) % 7`, matching the week-start
  convention `mondayOf()` already uses for the weekly trend chart, rather
  than JS's native Sun-first `Date#getDay()` ordering.
- `renderToolUsageHeatmap()` renders the grid as plain CSS-grid `<div>`
  cells (no new charting library) into `#toolUsageHeatmap`. Row labels are
  Mon/Tue/.../Sun; column labels are shown only at 12am/6am/12pm/6pm to
  avoid clutter. Each cell has a native `title` tooltip with the exact
  day, hour, and count. Cell color scales from `var(--surface-2)` (zero
  uses) up through `color-mix(in srgb, var(--accent) <pct>%, var(--surface-2))`
  where `pct` scales with `count/maxCount` (floored at 14% so any nonzero
  cell stays visibly distinct from empty ones) — reuses existing CSS vars
  only, no new hardcoded colors. Empty state (table not set up, or zero
  usage logged) hides the grid and shows the same `.hint`-style message
  used elsewhere on this tab, via the existing `toolUsageAvailable` flag
  and `toolUsageStatusMessage()` helper.
- Wired into `renderToolsUsage()` alongside the existing render calls.
- New CSS: `.heatmap-scroll`, `.heatmap-grid`, `.heatmap-corner`,
  `.heatmap-hour-label`, `.heatmap-row-label`, `.heatmap-cell` — added
  next to the existing `.charts-row` rules, reusing only existing CSS
  custom properties.

## 2026-08-18 — Tools Usage: chip-style "By User" counts

### Changed
- "By User" cell in Per-Tool Breakdown now renders each person as a
  rounded chip (`.user-count-chip`) with a small solid count badge
  (`.user-count-badge`, brand-gradient background) instead of plain
  "Name &times; count" text — matches the app's existing pill/badge visual
  language instead of a raw multiplication sign.

## 2026-08-18 — Tools Usage: per-user breakdown per tool

### Added
- Per-Tool Breakdown table now has a "By User" column showing each
  person's usage count for that tool (e.g. "Lochan &times; 10, Bhagya &times; 3"),
  sorted by count descending. `computeToolUsageStats()` now tracks
  `byUser` (userId -> count) per tool alongside the existing unique-user
  Set; `renderToolBreakdownTable()` formats and renders it via the
  existing `profileNameById()` helper.

## 2026-08-18 — Tools Usage: capture a reason per logged use

### Changed
- **`tool_usage_events` planned schema** now includes a `reason` column
  alongside `id`, `tool_name`, `user_id`, `used_at` (final column list:
  `id, tool_name, user_id, used_at, reason`). The table itself is still not
  created (migration remains queued separately) — this only updates every
  JS reference to the row shape: the `toolUsageEvents` type comment, the
  `computeToolUsageStats()` per-tool aggregation (now also tracks
  `lastReason`), and the insert payload in `logToolUsage()`.
- **"Using the tool now" no longer inserts immediately.** Clicking it now
  calls `openToolUsageReasonModal(toolName)`, which opens a small modal
  (`#toolUsageReasonModalOverlay`, built from the exact same
  `.modal-overlay` / `.modal-card` / `.modal-card-header` /
  `.modal-card-body` / `.modal-card-footer` markup and CSS as the existing
  Change Password modal) asking "What are you using this for?" with a
  required text input (min 3 characters). Submitting
  (`handleToolUsageReasonSubmit()`) validates the reason client-side, then
  calls `logToolUsage(toolName, reason)`, which inserts
  `{tool_name, user_id, reason}`, closes the modal, and shows the existing
  `showToast()` confirmation on success.
  - **Cancel / overlay-click / Escape** close the modal with no insert,
    matching the existing modal close patterns in this file exactly:
    Cancel button calls `closeToolUsageReasonModal()` (same as
    `closeChangePasswordModal()`), the overlay's
    `onclick="if(event.target===this) closeToolUsageReasonModal()"` matches
    every other modal overlay, and Escape was wired into the single shared
    `document.addEventListener("keydown", ...)` listener that previously
    only handled the Analyst Profile modal — extended rather than
    duplicated.
  - **Failure handling unchanged**: if the insert fails (most likely
    because `tool_usage_events` doesn't exist yet), `logToolUsage()` still
    goes through `isMissingTableError()` / `friendlyToolUsageError()` and
    shows a clear error toast. The modal is deliberately left open on
    failure (not closed) so the analyst doesn't lose what they typed and
    can retry once the table exists.

### Added
- **Reason surfaced in the stats section**:
  - Per-Tool Breakdown table (`renderToolBreakdownTable()`) gained a
    "Last Reason" column showing the reason given on the most recent use
    of each tool (`stats.perTool[name].lastReason`), with the full text
    also available via a `title` tooltip attribute.
  - New **Recent Activity** card at the bottom of the Tools Usage tab
    (`renderToolUsageRecentActivity()` / `#toolUsageRecentActivityList`)
    listing the last 10 logged uses across all tools — tool name, user
    (via `profileNameById()` + `avatarHtml()`, same as the Top Users
    leaderboard), reason, and timestamp — styled with the same
    `.leaderboard-row` / `.lb-info` / `.lb-name` / `.lb-meta` classes used
    elsewhere on this tab. Shows the same "isn't set up yet" / "No usage
    logged yet." empty state as the rest of the tab when the table is
    missing or empty.
- No new CSS custom properties or hardcoded colors — the new modal and
  Recent Activity card reuse only existing classes/variables.

## 2026-08-18 — Analyst Profile modal (Team + Leaderboard)

### Added
- **Analyst Profile modal** — click any analyst's name (Team tab roster
  table, and Leaderboard tab per-analyst progress rows) to open a modal
  with everything already computed about that one person in one place.
  Triggered via `openAnalystProfile(profileId)`, closed via
  `closeAnalystProfileModal()`. Purely derived from `projects`/`PROFILES`
  already in memory — no new Supabase columns/tables, no new reads/writes.
  - **Header** — name, admin role pill (`profile.is_admin`, reusing
    `.role-pill.is-admin` + `SHIELD_ICON`), and a large avatar circle
    (`.ap-avatar-lg`) built the same way as the header user chip
    (`initialsOf()` + the navy/accent-2 gradient).
  - **Stats row** (`.kpi-row` / `.kpi`, same classes as Dashboard/My Work)
    — total projects, completed count, in-progress count (Ongoing +
    Review Pending), overdue/stale count (`computeStaleness()` on the
    analyst's own projects), average days-to-complete
    (`computeTimeToCompletion().perAnalyst[name]` — not reimplemented),
    and current streak (`computeStreak()`).
  - **Badges earned** (`computeBadges()`), rendered as the same
    `.badge-row`/`.badge-chip` chips used on Leaderboard/My Work, with the
    same "no badges yet" empty copy as My Work.
  - **Assigned projects table** — project name, status
    (`.status-pill`/`STATUS_PILL_CLASS`), target date, date completed —
    same columns/markup convention as the My Work table. Empty state
    ("No projects assigned yet.") when the analyst has zero projects.
  - **Close behavior** matches the existing Import Projects / Change
    Password modals exactly: overlay click (`onclick="if(event.target===
    this) closeAnalystProfileModal()"`) and an X button
    (`.modal-close-btn`). Escape-to-close is new — no other modal in this
    file previously handled the Escape key, so a single shared
    `document.addEventListener("keydown", ...)` was added that closes the
    Analyst Profile modal (only) when it's open and Escape is pressed.
  - New CSS (`.ap-header`, `.ap-avatar-lg`, `.ap-header-name`,
    `.ap-section-title`) reuses only existing custom properties
    (`--navy`, `--accent-2`, `--text-primary/secondary`, `--shadow-sm`) —
    no new hardcoded colors, no renamed variables.

## 2026-08-18 — Tools Usage tab (UI-complete, inert until `tool_usage_events` migration lands)

### Added
- New **Tools Usage** nav tab (`data-view="toolsusage"`, after Leaderboard)
  built entirely against the `tool_usage_events` table shape from the
  queued migration (`id`, `tool_name`, `user_id` -> `profiles.id`,
  `used_at`), even though that table does not exist in the database yet.
  - **Log a Tool Use** — one row per tool with a live use-count and a
    "Using the tool now" button (`logToolUsage()`). Tool catalog is the
    distinct set of `project` names already loaded from `projects`
    (`toolCatalog()`) rather than a second `select distinct` round trip,
    since `projects` is already in memory and kept in sync via the
    existing realtime subscription.
  - Clicking the button inserts `{tool_name, user_id: currentProfile.id}`
    into `tool_usage_events` (same "resolve current profile id from
    `currentProfile`" pattern used by `addProjectRow()`/comments), shows
    `showToast("Logged your use of <tool>")` on success, or a friendly
    error toast on failure.
  - **Aggregate stats**: total uses + unique users (all-time) KPI cards,
    a per-tool breakdown table (total uses / unique users / last used),
    an 8-week and a 6-month usage trend chart (`renderToolUsageTrendChart()`,
    reusing the exact Chart.js line/area config, CSS-var theming, and
    empty-state overlay pattern from `renderTrendChart()`), and a
    per-person leaderboard (`renderToolUsageLeaderboard()`), styled the
    same as the existing Leaderboard tab rows.
  - All new CSS reuses existing custom properties only (`--surface-1/2`,
    `--surface-border`, `--text-primary/secondary/muted`, `--good`,
    `--accent`, etc.) via existing classes (`.card`, `.chart-card`,
    `.kpi-row`/`.kpi`, `.leaderboard-row`, `.table-wrap`, `.hint`) — no new
    hardcoded colors were introduced.
- **Graceful "table not set up yet" handling** — every
  `sb.from('tool_usage_events')` call (`loadToolUsageEvents()`,
  `logToolUsage()`) is wrapped in try/catch. Any read/write failure
  (missing-table `42P01`, "relation does not exist", PostgREST schema-cache
  misses, RLS denials, etc. — see `isMissingTableError()` /
  `friendlyToolUsageError()`) sets `toolUsageAvailable = false` and every
  stats section falls back to a shared friendly message via
  `toolUsageStatusMessage()`: "Usage tracking isn't set up yet — check back
  soon." when the table is missing, or "No usage logged yet." when the
  table works but is empty — mirroring the existing `renderDonut()` "No
  data yet" empty-state convention. The rest of the tab (tool list, nav,
  other tabs) renders normally regardless.

### Known limitation
- **This feature is UI-complete but functionally inert until the
  `tool_usage_events` table migration (queued separately, not part of this
  change) is applied to the database.** Until then, every section of the
  Tools Usage tab will show the "Usage tracking isn't set up yet — check
  back soon." message and the "Using the tool now" button will show a
  matching error toast instead of logging anything.

## 2026-08-18 — Footer credit

### Added
- App footer (`.app-footer`, below the main content, above the toast stack):
  "Developed by Sainath Nuvvula · For suggestions and feedback, reach out at
  +91-9535745456 or narasimha.sainath@bankbazaar.com" with working `tel:`
  and `mailto:` links.

## 2026-08-18 — BankBazaar branding + internal-tool disclaimer

### Added
- Favicon now points to the official BankBazaar favicon
  (`https://www.bankbazaar.com/images/favicon.ico`) instead of a generated
  inline SVG icon.
- Header logo badge (auth screen and main app header) now shows the
  BankBazaar mark instead of the generic checkmark icon.
- Added an "Internal BankBazaar tool — for authorized team members only.
  Do not share access or data outside the organization." disclaimer line,
  shown on both the login screen and the main app header.

## 2026-08-18 — Self-service password change, Time Taken column, status row tints

### Added
- **Self-service password change** — a "Change Password" button next to
  "Sign out" in the header opens a small modal (`openChangePasswordModal()`
  / `closeChangePasswordModal()` / `handleChangePassword()`) with new
  password + confirm password fields. Validates 8+ chars and that the two
  fields match, then calls `sb.auth.updateUser({ password })` directly —
  works for any logged-in user (analyst or admin) with no admin/service-role
  access required. Shows a toast on success ("Password updated") or surfaces
  the real Supabase error on failure. Entirely separate from the existing
  admin-only "Reset password" button on the Team tab
  (`admin-manage-users` edge function) — that flow is untouched.
- **Project Tracker "Time Taken" column** (`timeTakenHtml()`) — shows days
  between Date Assigned and Date Completed for "Completed & Approved"
  projects with both dates set, "—" otherwise. The day-diff calculation
  was extracted out of `computeTimeToCompletion()` into a new shared
  helper, `projectDaysTaken()`, so both the per-row column and the
  existing aggregate stats use the exact same logic (no duplication).
- **Status-based row tinting on Project Tracker** — each row gets
  `.row-not-started` (status "Assigned") or `.row-in-progress` (Ongoing /
  Review Pending / Completed & Approved) via `color-mix()` against the
  existing `--neutral` / `--blue` CSS vars, so "not started" work is
  clearly distinct from "in progress/done" work in both light and dark
  theme. No new hardcoded colors; existing `--good`/`--warning`/
  `--critical`/`--surface-*` vars untouched.

## [Unreleased]

### Added
- `docs/` folder with full project documentation (spec, SOP, architecture,
  configuration, dependencies, troubleshooting, prompt library).
- `seo-automation-dashboard-data.sample.json` — example/reference data file.
- A registered Claude Skill for maintaining this project in future sessions.

## 2026-08-18 — Overdue banners, smart inbox, workload/pipeline analytics, predicted dates, print export

All client-side, computed from data already fetched from Supabase — no
schema changes (the DB migration for a proper audit-log table is still
pending, so the "pipeline snapshot" chart below is explicitly labeled as a
current-state snapshot, not historical cycle time).

### Added
- **Overdue banner** (`renderOverdueBanners()`) — a red/critical banner
  pinned at the top of My Work and Dashboard, listing the signed-in user's
  own overdue/stale projects (reuses `getProjectStaleness`/
  `computeStaleness`, not reimplemented), each entry clickable
  (`jumpToProject()`) to scroll straight to that row in Project Tracker
  with a brief highlight flash. Admins see a team-wide version on the
  Dashboard tab instead of just their own; the banner renders nothing at
  all when there's nothing overdue.
- **"Needs Your Attention" smart inbox** (`computeNeedsAttention()` /
  `renderNeedsAttention()`) — merges overdue/stale projects, projects with
  Help Needed set, and (admin/team-wide scope only) unassigned projects
  into one de-duplicated, urgency-sorted list. Shown on the Dashboard
  (team-wide for admins, personal for analysts) and as a personal slice on
  My Work.
- **Time-to-completion analytics** (`computeTimeToCompletion()` /
  `renderTimeToCompletion()`) — average days from Date Assigned to Date
  Completed for "Completed & Approved" projects with both dates set,
  shown overall and per analyst in a new Dashboard card.
- **Analyst workload heatmap** (`renderWorkloadHeatmap()`) — a compact
  analyst-by-status grid next to the existing "Projects per Analyst by
  Status" bar chart, cell intensity driven by `color-mix()` against the
  existing `STATUS_COLOR` CSS-variable references (no new hardcoded
  colors).
- **Pipeline snapshot funnel** (`renderFunnelChart()`) — a new Chart.js
  horizontal bar chart on the Dashboard showing current project counts
  per stage (Assigned → Ongoing → Review Pending → Completed & Approved),
  explicitly labeled as a current-state snapshot rather than historical
  cycle time.
- **Predicted completion date** (`estimateCompletionDate()` /
  `predictedBadgeHtml()`) — for Ongoing/Review Pending projects, a muted
  dashed "≈ Est. <date>" badge under Target Date in both Project Tracker
  and My Work, computed as Date Assigned + the assigned analyst's own
  historical average days-to-complete, falling back to Target Date
  (clearly marked "(target)") when that analyst has no completion history
  yet.
- **Adjustable trend range** — 4/8/12-week preset buttons above the
  Leaderboard "Completions" chart, re-calling the existing
  `computeWeeklyTrend(projects, weeks)` (already parameterized) via a new
  `trendWeeks` module variable and `setTrendRange()`.
- **Print Report** — an admin-only "Print Report" button next to
  "Download Updated Excel" that switches to the Dashboard tab and calls
  `window.print()`; a new `@media print` block hides nav/toolbar/buttons
  and lays out cards cleanly. No new PDF library added.

## 2026-08-18 — Deep design-system rewrite + Chart.js migration

Third visual pass after two rounds of tightening didn't move the needle
enough — this one is a genuine step-change rather than incremental polish.
No business logic, Supabase calls, element ids, `data-view` wiring, or CSS
custom property *names* changed; only presentation.

### Added
- Google Fonts `Inter` (weights 400–900) now actually drives the whole
  UI's typographic hierarchy: 400 body copy, 600–700 headings/labels,
  800–900 for KPI numbers and brand marks (font link tag already existed
  from a prior pass; bumped to include weight 900 and applied consistently).
- New additive CSS custom properties (existing ones like `--shadow`,
  `--brand-text`, `--good` etc. were **not** renamed or removed):
  `--brand-gradient` (rich purple-to-violet gradient, light + dark
  variants) and a layered shadow scale `--shadow-sm` / `--shadow-md` /
  `--shadow-lg`, applied to the header logo, nav active-pill, buttons,
  auth card, and the import modal for more depth than the previous flat
  `--shadow`/`--shadow-hover` pair.
- KPI cards: numbers bumped to font-weight 900, icon badges now sit in a
  soft `color-mix()` tinted rounded-square (15% of the status color)
  instead of a flat neutral swatch, with a small hover scale-up on the icon.

### Changed
- **All three dashboard charts (`renderBarChart`, `renderDonut`,
  `renderTrendChart`) migrated from hand-rolled SVG DOM manipulation to
  Chart.js 4.4.0** (loaded via `<script src="…/chart.umd.min.js">` in
  `<head>`, before the Supabase/app script at the bottom of `<body>`).
  Function names/signatures and call sites (`renderAll()`) are unchanged.
  - `#barChart` → stacked bar chart, one dataset per status
    (`STATUSES`/`STATUS_HEX`), animated entry.
  - `#donutChart` → doughnut chart with a custom `centerText` Chart.js
    plugin drawing the total + "projects" label in the hole (replaces the
    manual center `<text>` elements).
  - `#trendChart` → line/area chart with a canvas gradient fill, driven by
    the unchanged `computeWeeklyTrend()`.
  - The `<svg>` elements for all three charts became `<canvas>` elements
    with the **same ids**. The old manual SVG tooltip divs
    (`#barTooltip`/`#donutTooltip`/`#trendTooltip`) were removed — nothing
    else referenced them — since Chart.js ships its own tooltips.
  - Each chart keeps a module-level instance variable
    (`barChartInstance`/`donutChartInstance`/`trendChartInstance`) that is
    destroyed before every re-render, and still reads grid/tick/label
    colors from `getComputedStyle(document.body)` custom properties at
    render time so both light and dark theme stay correct — mirroring how
    the old SVG code sourced its colors.
  - Empty states preserved: donut/trend charts skip creating a Chart
    instance (and clear the canvas) when there's no data yet, same as the
    old "No data yet" / "No completions yet" behavior.
- Nav active-pill, header logo, primary buttons, and the auth-tab active
  state now use `var(--brand-gradient)` instead of a repeated inline
  `linear-gradient(135deg, var(--navy), var(--navy-2))` literal — same
  visual family, single source of truth.

## 2026-08-17 — Bulk project import (Excel/CSV)

Admin-only "Import Projects" button on the Project Tracker toolbar. Parses
an `.xlsx`/`.xls`/`.csv` file (reusing the already-inlined SheetJS library —
no new dependency), matching the same column layout as "Download Updated
Excel" (`List of Projects` sheet if present, else the first sheet). Every
row is validated client-side before anything touches the database:
- Project name required; blank status defaults to `Assigned` (matches the
  existing single-row Add Project behavior and the DB check constraint);
  assignee must match an existing team member name; status/priority/help
  values must match the app's canonical lists; dates are parsed leniently
  (Excel date cells via `cellDates:true`, or best-effort string parsing)
  with a non-blocking warning if a date can't be read.
- A new preview modal (`#importModalOverlay` — the app's first modal
  component) lists every parsed row with a Ready / Warning / Error tag and
  the reason, before a single batched `insert()` commits only the valid
  rows. Wired via `handleImportFile`, `validateImportRow`,
  `renderImportPreview`, `confirmImport`, `closeImportModal`.
- The Import Projects button/hint are admin-only, toggled in
  `renderProjects()` alongside the existing "only my rows" toggle.

## 2026-08-14 — Overdue/stale alerts, streaks, badges, weekly trend chart, My Work view

Purely additive analytics/personalization pass on top of the data-dense
redesign — no schema changes, no new Supabase queries; everything is
derived client-side from `projects`/`ANALYSTS`/`PROFILES`/`currentProfile`
already loaded into memory.

### Added
- **Overdue/Stale detection** — `getProjectStaleness(p)` / `computeStaleness(projects)`.
  A project is **Overdue** if it has a `targetDate` in the past and isn't
  `Completed & Approved` ("Overdue by N days"); it's **Stale** (only if not
  overdue) if it's `Ongoing`/`Review Pending`, has a `dateAssigned`, and has
  been open 14+ days ("Stale — N days open"). `Assigned` and
  `Completed & Approved` rows are never flagged. Surfaced as: a new
  Dashboard alert banner (`.dash-alert-card`, wired via
  `renderStalenessAlert()`) with an "Overdue / Stale Projects" table
  (empty state: "Nothing overdue or stale right now — clean."), and a
  small `.staleness-tag` badge next to the status pill on flagged rows in
  the Project Tracker table and the new My Work table.
- **Streaks** — `computeStreak(analystName, projects)` counts consecutive
  weeks (Monday-based) with at least one `Completed & Approved` project,
  walking backward from the current week (falling back to last week once
  if the current week has zero completions yet, so a streak isn't lost
  mid-week), capped at a 26-week lookback.
- **Badges** — `computeBadges(analystName, projects)` derives chip badges
  from completion totals and streak: 🎯 First Completion (1+), ⭐ High Five
  (5+), 🏅 Perfect Ten (10+), 🔥 On Fire (3wk streak), 💎 Consistent (6wk
  streak). Rendered as `.badge-chip` pills next to the analyst's name on
  the Leaderboard ranked list and the Team tab.
- **Weekly trend chart** — `computeWeeklyTrend(projects, weeks=8)` buckets
  `Completed & Approved` projects by ISO week of `dateCompleted`. Rendered
  by a new hand-rolled SVG line/area chart (`renderTrendChart()`, same
  plain-DOM/gridline/tooltip conventions as `renderBarChart`/`renderDonut`)
  in a `.card.chart-card` on the Leaderboard tab: "Completions — Last 8
  Weeks", with a clean "No completions yet" empty state.
- **My Work tab** — new nav button/section (`data-view="mywork"`,
  positioned first in the nav), rendered by `renderMyWork()`: a personal
  KPI strip (Assigned/Ongoing/Review Pending/Completed/Overdue-or-Stale),
  a progress bar + streak + earned badges, and a compact table of the
  signed-in user's own projects. Empty state: "No projects assigned to you
  yet." Non-admins are automatically switched to this tab on their first
  successful sign-in (`onLoggedIn()` calls `switchTab('mywork')` once);
  admins keep the existing Dashboard default, and manual tab switches are
  never overridden on subsequent re-renders.

## 2026-08-14 — Visual redesign: data-dense analytics style

Full CSS/HTML visual pass across the whole app, at the product owner's
request, moving the look from a spacious/marketing-leaning style toward a
denser BI-tool feel (Looker/Metabase/Grafana-ish) — no Supabase, data, or
business-logic changes.

### Changed
- **KPI cards (Dashboard):** tighter padding, bold tabular-figure numbers,
  a new thin proportion mini-bar per card (`.kpi-bar`/`.kpi-bar-fill`,
  colored to match each card's status), and a new small trend/context line
  (`.kpi-trend`) — Total and Completed show "▲ N new/done this wk" (derived
  from existing `dateAssigned`/`dateCompleted` fields, Monday-based week
  window), the other four show "N% of total" as scannable context.
- **Charts:** the two Dashboard chart cards now carry a `chart-card` class
  with an accent top border and a slightly wider bar-chart column
  (`.charts-row` ratio 1.4fr→1.5fr) so they read as the page's centerpiece
  rather than incidental widgets. Fixed several hardcoded hex colors in
  `renderBarChart()`/`renderDonut()` (gridlines, axis labels, "No data
  yet"/"projects" center text) to instead read `--surface-border`,
  `--text-muted`, `--text-secondary` at render time so charts now stay
  legible in dark mode instead of using fixed light-mode grays.
- **Tables:** denser rows (`td` padding 7px→5px, table font 13px→12.5px),
  tighter sticky header with a subtle bottom shadow, numeric columns
  right-aligned (`Projects Assigned` on Team, and the trailing
  action/button column via `th:empty`/`td[data-label=""]`), taller
  `.table-wrap` viewport (520px→560px). Existing hairline-divider + row
  hover + `--stripe-color` status-stripe conventions kept as the single
  consistent row treatment (no zebra striping added).
- **Layout density:** tightened header/nav margins and padding, `main`
  max-width 1320px→1400px with less vertical padding, all `.card` panels
  denser (padding/margin trimmed), section headings restyled as small
  uppercase "panel titles" (BI-tool convention) instead of larger sentence
  case.
- **Nav:** kept as the existing top pill nav (all `data-view`/`tab-btn`/
  `switchTab()` wiring untouched) but restyled from a full pill to a
  tighter rounded-rectangle toolbar with uppercase labels, matching the
  denser overall rhythm.
- **Background:** simplified the multi-layer radial-gradient body
  background (dropped one gradient layer, reduced tint intensity) for a
  more sober, professional data-tool backdrop in both themes.
- Brand palette (navy/blue/purple accents, status colors) left untouched
  per the ask — this was a density/hierarchy/typography pass, not a
  rebrand.

## 2026-08-14 — Admin-only login provisioning; self-signup removed

### Changed
- **Self-signup removed entirely.** The login screen no longer offers any
  path to create an account — `#authScreen` now shows only the username +
  password sign-in form. Removed `.auth-tabs` (the `#authTabSignin` /
  `#authTabSignup` tab switcher), the entire `<form id="signupForm">`
  block, and the `.auth-switch-hint` paragraph (`#switchToSignup` /
  `#switchToSignin`) from the HTML.
- In `<script>`: removed the now-dead `setAuthMode(mode)`,
  `handleSignup(evt)`, and `isValidUsername(username)` functions, and the
  unused `authMode` state variable. Simplified `setAuthBusy()` (no longer
  branches on signin vs signup button copy). `onLoggedOut()` no longer
  calls `setAuthMode("signin")` (redundant with its existing
  `clearAuthMessages()` call). Kept `usernameToEmail()`,
  `friendlyAuthError()`, `handleSignin()`, `showAuthError()`,
  `showAuthNotice()`, `clearAuthMessages()`, `setAuthBusy()` unchanged —
  sign-in still depends on all of them.

### Added
- **Admin-only login provisioning on the Team tab.** `renderTeam()`'s
  actions cell now shows, admin-only, either **"Set up login"** (for a
  profile with no `auth_user_id` yet) or **"Reset password"** (for a
  profile that already has one), alongside the existing Delete button.
- `promptSetupLogin(profileId, fullName)` and
  `promptResetPassword(profileId, fullName)` (defined after
  `deletePerson()`): prompt for a username/password (or just password for
  reset), validate the 6-character minimum client-side, then call the
  already-deployed `admin-manage-users` Supabase Edge Function via
  `sb.functions.invoke('admin-manage-users', { body: { action:
  'create_login'|'reset_password', ... } })`. Real server errors are
  unwrapped via `await error.context.json()` (the SDK's `error.message`
  only ever says "Edge Function returned a non-2xx status code") and shown
  via `showToast(msg, "danger")`. On successful login creation, the
  profile row is re-fetched from `profiles` and `PROFILES` is patched in
  place so the "Set up login" button flips to "Reset password" without a
  full reload.
- The Edge Function itself verifies the caller is an admin server-side
  (via session JWT + `profiles.is_admin`) — the frontend still gates
  button visibility to admins for UX, but is not the security boundary.

## 2026-08-14 — Gamified leaderboard + UI polish

### Added
- **Leaderboard tab** (5th nav item, `data-view="leaderboard"`): a new
  `<section id="leaderboard">` with two "spotlight" cards — **Champion of
  the Week** and **Champion Overall** — plus a ranked "Analyst Progress"
  list showing every analyst with a rank (medal icons for top 3), avatar,
  an animated progress bar, and "`completed / total · pct%`" text.
- `computeLeaderboard()` (in `<script>`, right after `computeSummary()`):
  purely derived from the in-memory `projects` and `ANALYSTS` arrays — no
  new Supabase queries, schema changes, or persisted state. Computes, per
  analyst, total assigned, all-time completed count, completion %, and
  completed-this-week count (Mon 00:00 → next Mon 00:00, based on
  `dateCompleted`, a `YYYY-MM-DD` string from the native date input).
  - **Champion Overall** = analyst(s) with the highest all-time completed
    count, only crowned if that count is > 0; ties are joint champions.
  - **Champion of the Week** = same logic using the weekly count,
    independently of Champion Overall.
  - Friendly, theme-safe empty states when nobody has completed anything
    yet (overall) or this week.
- `renderLeaderboard()`: renders both spotlight cards, the ranked
  progress list, and a new compact "This week's champion" callout card on
  the Dashboard tab (`#dashChampionCard`, placed between the KPI row and
  the charts row) with a "View full leaderboard →" button. Wired into
  `renderAll()` alongside the existing render calls.
- `switchTab(view)`: extracted the nav click-handler's tab-switching logic
  into a reusable function so the new dashboard callout's "View full
  leaderboard →" button reuses the exact same view-switching code as the
  nav buttons instead of duplicating it.
- New CSS: `.progress-bar` / `.progress-fill` (reusable animated
  completion bars, `transition: width .5s ease`), `.spotlight-row` /
  `.spotlight-card` (gold-gradient "trophy" cards, new `--gold` custom
  property added to both the light and `body.dark` palettes),
  `.leaderboard-row` / `.rank-medal` / `.you-tag` (ranked list styling),
  `.dash-champion-card` (dashboard callout). All new classes have explicit
  `body.dark` overrides and follow the existing `.charts-row`/`.kpi-row`
  responsive pattern (`@media(max-width:900px)` and `720px`).

### Changed (visual polish)
- `.card` now has the same hover lift as `.kpi` (`transform:
  translateY(-2px)` + `box-shadow: var(--shadow-hover)` on hover) — it
  only had a box-shadow transition before.
- `.view.active` now cross-fades/slides in (`animation: view-in .2s`,
  6px translateY + opacity) on every tab switch instead of an instant
  `display:block` snap.

### Assumptions
- `dateCompleted` is a `YYYY-MM-DD` string (confirmed via the `<input
  type="date">` binding in the Project Tracker row template) or empty —
  empty/missing values are skipped when computing the weekly count rather
  than treated as errors.

## 2026-08-06 (v3) — Team management + persistence

### Added
- **Team tab**: add/remove people from the roster, toggle whether each
  person appears in the Daily Status Log.
- `ANALYSTS` and `DAILY_ANALYSTS` are now runtime-mutable and persisted —
  previously hard-coded constants.
- Daily Status Log table header is now generated dynamically from
  `DAILY_ANALYSTS` instead of being static HTML.
- `collectData()` / the saved JSON now includes `analysts` and
  `dailyAnalysts` alongside `projects` and `dailyLog`.

### Changed
- Adding/editing/deleting a project, daily log row, or person now all
  funnel through the same `persistAll()` call, so "everything" (people +
  projects + daily log) saves to the same JSON consistently.

## 2026-08-06 (v2) — JSON persistence

### Added
- `localStorage` autosave on every edit (safety net, survives page
  reloads).
- "Connect JSON file (autosave)" — uses the File System Access API to
  write live edits straight to a real `.json` file on disk.
- "Download JSON now" — manual snapshot export, used as the fallback in
  browsers without File System Access API support.
- Sync status bar under the tab nav showing autosave state.

## 2026-07-13 (v1) — Initial dashboard

### Added
- Dashboard tab: KPI cards, per-analyst status bar chart, status-split
  donut chart, "needs help" table.
- Project Tracker tab: editable table matching the master Excel's columns
  and dropdown lists.
- Daily Status Log tab: per-analyst free-text daily log.
- "Download Updated Excel (.xlsx)" export matching the master workbook
  layout (List of Projects / Summary / Projects Daily Status sheets), via
  an inlined SheetJS library.
