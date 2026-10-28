# ps

Poteto-style engineering skills for Claude Code, adapted from Cursor's pstack and made project-agnostic.

| Command | What it does |
|---|---|
| `/ps:minion <task>` | Classifies a task as Bug fix, Feature, Investigation, Open a PR or Babysit, then runs that playbook |
| `/ps:how` | Explains how a subsystem works and which layer owns what |
| `/ps:why` | Finds the motivation behind code from git, PRs and any connected evidence source |
| `/ps:teach` | Explains a change or subsystem plainly, built on `how` and `why` |
| `/ps:deslop` | Removes AI-generated slop from the diff |
| `/ps:unslop` | Rewrites prose without AI tells |
| `/ps:no-comments` | Spawns the `stuart` agent to delete needless comments and flag workarounds |

Agents: `kevin` (minion's delegate) and `stuart` (comment hunter).

## Install

```
/plugin marketplace add <owner>/ps
/plugin install ps@ps-marketplace
```
