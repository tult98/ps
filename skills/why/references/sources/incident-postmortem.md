# Incident and postmortem context

Use these queries inside your own source when the target code looks defensive.

- **Git and PR host:** commits or PRs titled like "hotfix", "fix for incident",
  "add guard", "revert", or a revert followed by "re-apply with ...". Note
  merges straight to a long-lived branch outside the usual flow.
- **Issue tracker:** tickets labeled `Bug`, `incident`, `urgent`, or with priority
  Urgent, created around the date the target code was added.
- **Team chat:** incident or alerts channels around the date the target code
  was added.
- **Error tracking:** issues whose first-seen date is shortly before the
  target code was added, and whether they stopped after it shipped.
