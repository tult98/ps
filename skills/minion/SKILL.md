---
name: minion
description: Route a task to the Bug fix, Feature, Investigation, Open a PR, or Babysit playbook and run it. Use only when the user runs /ps:minion.
user-invocable: true
disable-model-invocation: true
argument-hint: '<task description or issue key>'
allowed-tools: Bash, Read, Edit, Write, Grep, Glob, Agent, AskUserQuestion
---

# Minion

Match the task in `$ARGUMENTS` to a playbook, then run it. Playbooks live in
`playbooks/` next to this file. Bundled skills live at `../<name>/SKILL.md`
from this file: `../how/SKILL.md`, `../why/SKILL.md`, `../unslop/SKILL.md`,
`../deslop/SKILL.md`, `../no-comments/SKILL.md` and
`../technical-writing/SKILL.md`. To run one, read its `SKILL.md` in full and
resolve its `references/` paths against that skill's own directory. Never
call a bundled skill through the Skill tool, and never ask the user to run
it. Put this file's absolute path in every `ps:kevin` prompt.

Get the task text first. Plain text is used as is. An issue key (`ABC-123`,
`#45`) or issue URL with a tracker tool available (Linear, Jira, GitHub) means
you read the issue title, description, and labels. With no tracker tool, work
from the key's surrounding text and say the ticket was not read. An empty
argument means the user's most recent request in this conversation.

## Triggers

These apply at any point, in every playbook:

- Your first message on a task → tell the user, in your own words, their goal
  and, if the task ships a change, the smallest change you expect. Don't wait
  for confirmation.
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
- Docs, RFCs, readmes, plans, PR descriptions or commit messages → the
  **technical-writing** skill.
- Before commit → the **deslop** skill.
- Before review → the **no-comments** skill.
- A request to babysit a PR, get it green, or answer review comments →
  the Babysit playbook. Merely opening a PR never triggers it.
- A request to land, ship, or merge → no playbook here. Merging falls under
  Autonomy's always-pause rule, so say so and pause.

## Playbooks

Match the task to a playbook below and read its file in full. Open a todo list
whose first items are that playbook's numbered steps, copied in verbatim,
before any task-specific todos. A step you choose not to do stays in the list
with `skip: <reason>`. When a step says "Run Open a PR", read
`playbooks/open-a-pr.md` and add its steps to the todo list then. When a step
or a trigger calls a bundled skill, read its `SKILL.md` in full and add its
numbered steps the same way. End with the reply the playbook's **Reply** line
describes.

- **Bug fix.** Existing behavior that is broken, such as a crash, an error, a
  regression, or a stack trace, and the user wants it fixed. "Why is X
  broken?" is a Bug fix unless they only want the explanation. "X doesn't do
  Y" where Y was never built or specified is a Feature, so take a quick look
  at the code or ticket to see whether Y was intended. File:
  `playbooks/bug-fix.md`.
- **Feature.** New behavior, or a deliberate change to intended behavior.
  Open a PR is its last step. File: `playbooks/feature.md`.
- **Investigation.** An answer, not a change: how, why, where, "are we sure",
  or "should we X or Y". It stays an Investigation even if a change follows,
  because the change is a later task. It never opens a PR. File:
  `playbooks/investigation.md`.
- **Open a PR.** A PR or MR for work that already exists. It is also the last
  step of Bug fix and Feature, so "fix the crash and open a PR" is a Bug fix.
  File: `playbooks/open-a-pr.md`.
- **Babysit.** A PR to bring to merge-ready, with review threads answered and
  CI triaged. It never merges. After "open a PR and babysit it", run it once
  the PR is open. File: `playbooks/babysit.md`.

If no playbook fits (a chore such as commit and push, a gitignore entry,
local setup), do it directly with no todo list. For a mixed request ("fix the
crash and add a retry button"), run the playbook for the part the user
stresses and say in the reply what you left out.

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
