---
name: minion
description: Classify a task as Bug fix, Feature, Investigation, Open a PR, or Babysit, then run that playbook. Use only when the user runs /ps:minion.
user-invocable: true
disable-model-invocation: true
argument-hint: '<task description or issue key>'
allowed-tools: Bash, Read, Edit, Write, Grep, Glob, Agent, AskUserQuestion
---

# Minion

Classify the task in `$ARGUMENTS` into exactly one playbook, then run it.
Playbooks live in `playbooks/` next to this file. Bundled skills live at
`../<name>/SKILL.md` from this file: `../how/SKILL.md`, `../why/SKILL.md`,
`../unslop/SKILL.md`, `../deslop/SKILL.md` and `../no-comments/SKILL.md`.
To run one, read its `SKILL.md` in full and resolve its `references/` paths
against that skill's own directory. Put this file's absolute path in every
`ps:kevin` prompt.

## Triggers

These apply at any point, in every playbook:

- Nontrivial change, architecture decision, or "are we sure?" → the **how**
  skill.
- A question about why code is the way it is (a design choice, a regression,
  where a number came from) → the **why** skill.
- About to ask the user "which approach", "how should I" or "what should this
  do" → first check whether running something can answer it (behavior,
  timing, layout, output). If it can, run it and let the result decide. Ask
  only for a product or preference call no experiment can settle.
- Any prose surface, your reply to the user included → the **unslop** skill.
  Write the reply per **Writing the reply**.
- Before commit → the **deslop** skill.
- Before review → the **no-comments** skill.
- A request to babysit a PR, get it green, or answer review-bot comments →
  the Babysit playbook.

## Step 1. Get the task text

- Plain text: use it as is.
- An issue key (`ABC-123`, `#45`) or issue URL and a tracker tool is
  available (Linear, Jira, GitHub): read the issue title, description, and
  labels. If no tracker tool is available, classify from the key's
  surrounding text and say the ticket was not read.
- Empty: classify the user's most recent request in this conversation.

Read code only when the request names something whose current behavior decides
the type (see "Tie-breakers"). Keep it to a quick look.

## Step 2. Classify by deliverable

Ask what the user expects to have when the work is done.

| Type | Playbook | Deliverable | Typical signals |
|---|---|---|---|
| **Bug fix** | `playbooks/bug-fix.md` | A code change that makes existing behavior match what it was meant to do | "broken", "doesn't", "error", "crash", "regression", stack trace, steps to reproduce, a `Bug` label |
| **Feature** | `playbooks/feature.md` | A code change that adds behavior or deliberately changes intended behavior | "add", "support", "allow", "change X to Y", "new page", design link, story ticket with ACs |
| **Investigation** | `playbooks/investigation.md` | An answer, not a change: an explanation or a recommendation | "how does", "why", "where is", "are we sure", "should we X or Y", "what would it take" |
| **Open a PR** | `playbooks/open-a-pr.md` | A PR or MR for work that already exists | "open a PR", "create an MR", "PR from develop -> staging", "push and open a PR" |
| **Babysit** | `playbooks/babysit.md` | A PR brought to merge-ready, with review threads answered and CI triaged | "babysit", "get it green", "address the bugbot comments", "is it green", "check on PR X" |

Open a PR is also the last step of Bug fix and Feature. Investigation never
opens one. Babysit never merges, and opening a PR never triggers it.

### Tie-breakers

- **"Why is X broken?"** Bug fix. The user wants it fixed. Investigation only if
  they say they only want the explanation.
- **"X doesn't do Y"** where Y was never built or specified: Feature, not Bug
  fix. A quick look at the code or ticket decides whether Y was intended.
- **"Should we X or Y?"** Investigation, even if a change follows. The change
  is a later task.
- **Work plus a PR** ("fix the crash and open a PR"): classify the work. Open a
  PR runs as that playbook's last step.
- **Open a PR and babysit it:** Open a PR. Run Babysit after, on request.
- **"Land it", "ship it", "merge it":** no playbook here. Merging falls under
  Autonomy's always-pause rule, so say so and pause.
- **Mixed request** ("fix the crash and add a retry button"): pick the type of
  the part the user leads with or stresses, and list the other part under
  Notes.
- **Still unclear:** pick the closest type and name the runner-up. Do not ask.

### No playbook

A chore that is none of the five (commit and push, a gitignore entry, local
editor or shell setup) gets `Type: No playbook`. Do it directly, with no todo
list.

## Step 3. Report the classification

Start the reply with this block:

```
Type: <Bug fix | Feature | Investigation | Open a PR | Babysit | No playbook>
Why: <one sentence naming the signal that decided it>
Runner-up: <type, and what would flip it>   (omit if the call was clear)
Notes: <a split-off part of a mixed request, or "ticket not read">   (omit if none)
```

## Step 4. Run the playbook

1. Read the playbook file in full.
2. Open a todo list whose first items are the playbook's numbered steps, copied
   word for word, before any task-specific todos. A step you choose not to do
   stays in the list, marked `skip: <reason>`.
3. Work through the steps in order. When a step says "Run Open a PR", read
   `playbooks/open-a-pr.md` and add its steps to the todo list then. When a
   step or a trigger calls a bundled skill, read its `SKILL.md` in full,
   add its numbered steps to the todo list the same way, then follow them.
4. End with the reply the playbook's **Reply** line describes.

## Subagents

**Use `subagent_type: "ps:kevin"` for any subagent you spawn inside a playbook
step** (code-writing delegates, ad-hoc helpers). The chat that receives
`/ps:minion` runs the playbook. Never spawn a `kevin` to run the playbook
itself. The how and why skills set their own subagents: `Explore` agents for
how's explorers, `general-purpose` agents for why's investigators and
synthesizer. Use what the skill prescribes. Don't swap in your own, and don't
answer in their place.

**Defaults for every Agent call.** File pointers, not inlined context: name
the files, the plan, and the ticket for the subagent to read instead of
pasting their contents. Pick the model and effort for the job. The hardest
changes (cross-cutting design, gnarly concurrency, subtle algorithms) go to
`opus` with effort raised one step, up to `max`. Routine work keeps the
default. Trivial mechanical edits go to `sonnet` with effort lowered one step.

You own every subagent's work. Review the diff and write your own summary,
don't pass through what it said. A second opinion is the same prompt against
a different model. Agreement is high-signal.

**Fresh subagents by default.** Give new work to a fresh subagent with
consolidated scope, meaning the original brief, every later directive, and the
prior agent's report and branch. This holds for a fix round, a follow-up, a
retry, and the next queue item. Resume or message an existing subagent only
when the new work strictly needs state that lives in that agent and is costly
to move: its local checkout, its uncommitted changes, or a process it still
runs, such as a dev server. A stop or hold order to a running agent is not
reuse. Interrupt-chained resumes silently drop directives, so fire a fresh
subagent with consolidated scope rather than trusting a "done" summary.

## Autonomy

**Just do it.** Use any MCP tool. Reversible work and external actions
(issue tracker updates, team chat, pushing a feature branch, opening a PR)
proceed without asking.

**Always pause** for irreversible or shared writes:

- force-push to a shared branch
- merging a PR or MR
- pushing directly to the default branch or any long-lived branch such as
  `develop`, `staging` or `main`
- deploys
- data deletion, including remote branches
- customer messages

**Session overrides:** "Don't stop" / "going to bed" / "run until done" / "be
fully autonomous" → keep going.

**No is an acceptable answer.** Asked whether to do something, invited to add
scope, or shown an approach, reply with your real judgment. Decline, push back,
or say "this doesn't earn its place" when true. A recommendation is a
judgment, not a validation. Agreement is not the default, candor over
sycophancy.

## Writing the reply

Write the reply clean as you draft it. A cleanup pass after drafting does not
remove these patterns.

- **Short declarative sentences.** One thought per sentence, ended with a
  period.
- **No long-dash character anywhere.** Write a file-list bullet as a sentence
  ("`main.js` owns persistence and the IPC handlers") and a bold section header
  as its own sentence ("**Verification.** End to end through the verify
  skill").
- **No colon as a mid-sentence connector.** A colon before a list is fine.
- **Terse is not an excuse to drop content.** Short sentences, but every
  section the playbook's reply names stays: details, tradeoffs, choices, open
  decisions.
- **Frame impact for the consumer and the maintainer.** Name who the work is
  for (an end user, a teammate calling the API) and what changes for them
  before any implementation detail. Then what the next engineer who owns this
  code inherits. If you can't say what either would notice, the work or the
  explanation is off.
- **Never fabricate a link, citation, or transcript reference.** Link only
  artifacts you produced or read this session.
- **Every claim carries its evidence or its label in the same sentence.**
  Measured, inferred, or guess. A prediction or an unseen cause is a guess.
  Never hand the user a check you could run.

Every playbook ends with a reply written this way, with the PR as its full
URL. Each playbook's own **Reply** line names only the content unique to
that playbook.

## Comments

Comments follow the same rule as the reply. Write them clean as you go. Keep a
comment only for a non-obvious *why* the code can't show. A verify or test
script gets no phase-narrating comments such as `// Phase 1: add cards`. The
assertion or log string documents the step, as in
`assert(ok, 'persisted across restart')`. This applies to every file you
produce, including a subagent's diff.
