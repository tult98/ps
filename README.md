# ps

Poteto-style engineering skills for Claude Code, adapted from Cursor's pstack and made project-agnostic.

| Command | What it does |
|---|---|
| `/ps:minion <task>` | Routes a task to the Bug fix, Feature, Investigation, Open a PR or Babysit playbook and runs it |
| `/ps:how` | Explains how a subsystem works and which layer owns what |
| `/ps:why` | Finds the motivation behind code from git, PRs and any connected evidence source |
| `/ps:teach` | Explains a change or subsystem plainly, built on `how` and `why` |
| `/ps:recall [topic]` | Rebuilds your recent working context from chat history, live state and the shared record, then hands back a short brief |
| `/ps:create-verification-skill` | Generates a repo-local `verify-<app>` skill that launches, drives and records the app, plus a feature map |
| `/ps:maintain-verification-skill` | Audits a verification skill against source and a live run, and hands back one branch of proven fixes |
| `/ps:deslop` | Removes AI-generated slop from the diff |
| `/ps:unslop` | Rewrites prose without AI tells |
| `/ps:no-comments` | Spawns the `stuart` agent to delete needless comments and flag workarounds |
| `/ps:technical-writing` | Layered writing standard for docs, readmes, PR descriptions and commit messages |

Agents: `kevin` (minion's delegate) and `stuart` (comment hunter).

## Install

```
/plugin marketplace add <owner>/ps
/plugin install ps@ps-marketplace
```
