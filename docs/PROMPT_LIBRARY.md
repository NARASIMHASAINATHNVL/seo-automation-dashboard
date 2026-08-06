# Prompt Library — Extending the SEO Automation Dashboard with Claude

Ready-to-use prompts for asking Claude (in Cowork) to work on this specific
project. Point Claude at this folder (or mention "seo-automation-dashboard")
and use one of these, editing the bracketed parts.

## Maintenance & fixes

- "Open the seo-automation-dashboard and fix [describe the visual/behavior
  bug], the same way you fixed the h1 title / dark theme before."
- "The [Dashboard/Project Tracker/Daily Status Log/Team] tab looks broken
  in [Chrome/Edge/Firefox] — check it like a senior full-stack dev and fix
  what's wrong."
- "Audit the seo-automation-dashboard for accessibility issues and fix
  them, same standard as before (keyboard nav, aria labels, focus states)."

## New features

- "Add a [column name] field to the Project Tracker table, persisted the
  same way as the existing fields."
- "Add a filter/search box to the Project Tracker table like the one in the
  Bot Page-View Checker."
- "Add a 'Completed this week' stat card to the Dashboard tab."
- "Let me reorder project rows by drag-and-drop, and persist the new
  order."
- "Add a confirmation dialog before deleting a project row, like the one
  for deleting a person."

## Data & persistence

- "Give me the current data from the dashboard as a JSON file."
- "Add [import from JSON / merge two JSON exports] to the dashboard."
- "Change the autosave so it also keeps the last 5 versions, not just the
  latest."

## Reporting

- "Summarize the current project tracker data: who's overloaded, what's
  overdue, what needs help."
- "Turn the current dashboard data into a weekly status email for
  leadership."

## Documentation

- "Update CHANGELOG.md with today's changes."
- "Regenerate ARCHITECTURE.md to reflect the new [feature]."

## Tips for good results

- Mention the exact tab/feature by name — this app has four tabs
  (Dashboard, Project Tracker, Daily Status Log, Team) and Claude will
  target edits faster with a specific pointer.
- If something looks visually wrong, say what you see ("black rectangle
  behind the title", "buttons overlap on mobile") rather than just "it
  looks off" — concrete descriptions map directly to CSS bugs.
- For anything destructive (deleting data, removing a tab), ask for a plan
  first before Claude edits the file.
