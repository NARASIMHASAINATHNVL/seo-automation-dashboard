# SEO Automation & Projects Dashboard

A single-file, no-server dashboard for tracking the BankBazaar SEO team's
automation projects and daily status — KPIs, charts, an editable project
tracker, and a daily status log, all matching the layout of the shared
"SEO_Automation_Tracker.xlsx" workbook.

## How to open it

Just double-click **`index.html`** in this folder (or open the deployed
GitHub Pages / Vercel URL). It's still a single static file — no build step
— but it now requires signing in before you see any data.

## Sign in

The dashboard is gated behind Supabase Auth, restricted to **@bankbazaar.com**
email addresses (enforced server-side). First time using it:

- If your name is already on the team roster (with a placeholder email like
  `name.pending@bankbazaar.local`), ask Sai to update your `profiles` row to
  your real bankbazaar.com email, then use **Create account** with that email
  to link up automatically.
- Otherwise, use **Create account** with your name, bankbazaar.com email, and
  a password. Signing up with a non-bankbazaar.com address is rejected with
  an error shown right on the form.

Once signed in, `sb.auth.getSession()` keeps you logged in on refresh; use
**Sign out** in the header to end the session.

## What's inside

- **Dashboard tab** — KPI counts (Total / Assigned / Ongoing / Review Pending
  / Completed & Approved / Need Help), a bar chart of projects per analyst by
  status, a status-split donut chart, and a table of projects flagged as
  needing help.
- **Project Tracker tab** — an editable table (project name, assigned
  analyst, status, priority, dates, help-needed flag, notes). Dropdowns are
  locked to the same analyst/status/priority lists as the team roster.
- **Daily Status Log tab** — free-text log of what each analyst worked on,
  per day.
- **Team tab** — add/remove teammates and toggle who shows up as a column in
  the Daily Status Log.

## Saving your edits

All data now lives in a Supabase Postgres database, not in this browser's
local storage. Every add/edit/delete — projects, daily log entries, team
members — is written straight to Supabase as you make it, and a toast
confirms success or failure. Anyone else signed in sees your changes appear
live (via Supabase Realtime), no refresh needed.

The **`seo-automation-dashboard-theme`** light/dark preference is the only
thing still kept in this browser's local storage — that's just a per-device
display setting, unrelated to the shared project data.

Use **Download Updated Excel (.xlsx)** in the Project Tracker tab to export
the table in the same column layout as the original master workbook.

## Notes

- The SheetJS library used for the Excel export is embedded inline, same as
  before.
- Data and auth are handled by Supabase (see `docs/CONFIGURATION.md` for the
  project URL and schema). Six of the seven team profiles were pre-seeded
  with placeholder, non-real emails until each person signs up with their
  real bankbazaar.com address.
- This is a separate, unrelated project from the "Bot Page-View Checker"
  tool — that one lives in its own folder and needs Python/Flask running.
