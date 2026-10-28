# Source playbooks

The why skill spawns one investigator per available evidence category. Each
reads the guide for its category below. A category with no guide uses its row
in the Discovery table of the why skill's `SKILL.md`.

| Category | Guide | Tool |
|---|---|---|
| Source control | [`code-archaeology.md`](./sources/code-archaeology.md) | git, `gh` or `glab` |
| Issue tracker | [`issue-tracker.md`](./sources/issue-tracker.md) | Linear, Jira or GitHub Issues tools, when connected |
| Long-form docs | none | Notion, Confluence or Google Docs MCP, when connected |
| Team chat | none | Slack or Teams MCP, when connected |
| Infrastructure observability | none | Datadog or Grafana MCP, when connected |
| Error tracking | none | Sentry or Rollbar MCP, when connected |
| Product analytics | none | BigQuery, Snowflake or dbt MCP, when connected |

Cross-cutting:

- [`incident-postmortem.md`](./sources/incident-postmortem.md). Add it when the
  target code looks defensive (null checks, retries, timeouts, rate limits,
  feature flags, error handlers).
