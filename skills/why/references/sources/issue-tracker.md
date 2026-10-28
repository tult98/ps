# Issue tracker

Linear, Jira, GitHub Issues or any tracker with tools connected in this session.

## What this source contains

- Issues describing features and bugs, and why they were wanted
- Parent and sub-issue relationships (an initiative and its tickets)
- Comments with clarifications, scope changes and decisions
- Labels (`Bug`, `customer-request`, `perf`, `incident`) that signal the kind
  of motivation
- Projects, epics and their documents (specs, PRDs)
- Attachments and linked PRs

The tracker is where the product and business reason usually lives: "the
customer asked for this", "this is for the cancel-plan retention work".

## How to search it

Use the tracker's MCP or CLI tools.

1. **Start with linked tickets.** Fetch every ticket key from the code anchor
   (`ABC-123`, `#45`) and read the whole issue, including comments.
2. **Search by keyword.** List issues matching the feature name, a key symbol,
   or the business term. Try several phrasings.
3. **Walk the tree.** From a sub-issue, fetch its parent. Sub-issues are
   tactical. Parents often hold the why.
4. **Read project or epic documents** when the issue belongs to one.
5. **Check labels and cycles.** Labels hint at the kind of motivation. Cycles,
   milestones and due dates tie work to deadlines.

## What good evidence looks like

- A description stating the problem: "Star plan users get redirected to the
  dashboard when they click submit links"
- A comment recording a decision: "We went with B because A needs a BE change"
- A parent issue named like an initiative
- An attached spec

## Pitfalls

- **Scope drift.** A ticket may have been reopened with a different scope.
  Read its history.
- **Boilerplate.** A generic "improve user experience" is not a real answer.
- **Stale tickets.** Compare the ticket's dates with when the code shipped.
- **Duplicates.** Follow duplicate-of links to the canonical ticket.
- **No access.** If you can't open an issue, record it as a gap.

## What to return

For each relevant ticket: key and title, the motivation quoted exactly, labels,
parent and project, author, created and closed dates, and the link.
