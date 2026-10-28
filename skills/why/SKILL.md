---
name: why
description: "Use for 'why does X work this way', 'why we picked Y', design rationale, regressions, postmortems, or where a number came from. Builds a code anchor from git and its PRs or MRs, queries each available evidence source (source control, issue tracker, docs, chat, observability, error tracking, analytics) in parallel, then returns a cited, confidence-tiered answer. Use how for runtime behavior."
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Agent
---

# Why

Find the motivation behind code: why it works this way, why Y was picked, what
caused a regression, where a threshold came from. The how skill answers
what the code does. Why answers what forces gave it its shape.

Work as a careful, cautious and precise investigator. Keep what you know apart
from what you infer. Read `references/epistemics.md` in full. The synthesizer must
follow it.

## Step 1. Pin down the target and the question

The **target** is a piece of code, a pattern, a feature or a named design
decision. The **question** is a rationale, a tradeoff, a motivating edge case,
an external constraint, dead code or a broad history sweep.

If the target is vague ("why do we do it this way?"), take your best guess
from the conversation (files mentioned, recent edits, what was just discussed),
state it in one line so the user can redirect, and continue.

## Step 2. Build the code anchor

Before spawning anyone, collect the seed context yourself. Run git in the
repo that holds the file. When that repo is a submodule, run it inside the
submodule, never in the parent repo.

```bash
git blame -L <start>,<end> <file>          # last-touch commits per line
git log --follow --oneline -20 -- <file>   # recent history through renames
git log -1 --format=%B <commit>            # full message: PR ref and ticket keys
```

- **PR or MR numbers.** GitHub squash and merge commits end the subject with
  `(#123)`. GitLab merge commits carry `See merge request <group>/<project>!<iid>`
  in the body. For a commit without one, look it up:
  `gh api repos/{owner}/{repo}/commits/<sha>/pulls` or
  `glab api "projects/:id/repository/commits/<sha>/merge_requests"`.
- **Ticket keys.** Keys such as `ABC-123` or `#45` in branch names, commit
  subjects and PR titles.
- **PR discussion** for every substantive commit:
  `gh pr view <n> --comments` or `glab mr view <iid> --comments`.

The anchor is: file paths and line ranges, key symbols, the commits (newest
first), the PR numbers and the ticket keys. It goes to every investigator and
to the synthesizer.

## Step 3. Spawn investigators in parallel

Default to the full parallel investigation. The goal is a complete coverage
map. Record a null result rather than skipping the search.

### Discovery

List the MCP tools available in this session and map each to one evidence
category:

| # | Category | Best at | Source guide |
|---|---|---|---|
| 1 | Source control (git, `gh` or `glab`, code comments, tests) | Rationale captured during implementation and review | `references/sources/code-archaeology.md` |
| 2 | Issue tracker (Linear, Jira, GitHub Issues) | The product or business reason | `references/sources/issue-tracker.md` |
| 3 | Long-form docs (Notion, Confluence, Google Docs) | Design rationale written before the code | none, use the category description |
| 4 | Team chat (Slack, Teams) | Decisions made in conversation that never reached a doc | none, use the category description |
| 5 | Infrastructure observability (Datadog, Grafana) | Runtime reality behind timeouts, retries, rate limits | none, use the category description |
| 6 | Error tracking (Sentry, Rollbar) | The exceptions behind defensive code | none, use the category description |
| 7 | Product analytics (BigQuery, Snowflake, dbt) | Data behind flag-gated code, experiments, thresholds | none, use the category description |

Source control is always available. Spawn one `general-purpose` agent per
category that has a matching tool, all in a single message so they run in
parallel. Each owns exactly one source. Never ask one agent to cover two.

Each prompt is `references/investigator-prompt.md` filled in, plus:

- the category's source guide, when it has one;
- `references/sources/incident-postmortem.md` when the target code looks defensive
  (null checks, retries, timeouts, rate limits, feature flags, error handlers);
- the code anchor and the user's original question.

### When to skip an investigator

Only with a written reason, which goes in the final Sources consulted. Two
reasons are valid:

- **No tool is available for the category.** Report it as a gap, not a choice:
  "Team chat not searched. No Slack tool in this session."
- **The source is provably irrelevant**, not just probably: "Error tracking
  skipped. The target is a build script with no runtime path."

Answering without investigators is allowed only when all of these hold:
Step 2 is complete (the PR discussion and every ticket key were read), and a
PR description, ticket or code comment states the reason in so many words.
Then skip Steps 3 and 4, but still read `references/epistemics.md`, write the Step 5
output format yourself, and open Sources consulted with
"Answered without investigators: <the citation that states the reason>".
Code alone never qualifies, because code is not evidence of its own intent.

## Step 4. Synthesize

Spawn one `general-purpose` agent with `references/synthesizer-prompt.md`, filled in
with:

1. every investigator's findings, including null results and skipped
   categories with their reasons;
2. the code anchor;
3. the user's original question.

Tell it to read `references/epistemics.md` in full first.

## Step 5. Present

Present the synthesizer's output. You may lightly edit for clarity or add
context from the conversation. Do not change its confidence language.

The structure is The question, The code in question, What we found, What we
can reasonably infer, Competing hypotheses, What we don't know, Sources
consulted, Confidence summary. Keep the confidence tiers separate, and keep
Sources consulted as one line per category, including the empty and skipped
ones with their reasons.

When the question is a step toward changing this code, end with a constraint
set for the change: **Preserve**, **Change**, **Avoid**, **Risk**.

## Failure modes

- **Recency bias.** Don't treat the most recent commit as the whole story. The
  current shape is often the build-up of many earlier decisions. Trace back.
- **Code as its own evidence.** What the code does is mechanics, not motivation.
- **Confirming the user's guess.** A hypothesis in the question ("I assume it's
  for performance?") is one candidate to check, not a conclusion.
