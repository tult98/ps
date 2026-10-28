---
name: no-comments
description: "Spawn Stuart, fix accepted findings, and offer encodings for claimed constraints."
user-invocable: true
disable-model-invocation: true
argument-hint: '[files or diff scope]'
allowed-tools: Bash, Read, Edit, Write, Grep, Glob, Agent, AskUserQuestion
---

# No comments

Spawn Stuart. Act on accepted findings.

Defer to Stuart's fresh perspective.

Bundled skills live at `../<name>/SKILL.md` from this file. To run one, read
its `SKILL.md` in full and resolve its `references/` paths against that skill's
own directory.

## Scope

Use the caller's files or diff. Otherwise use the current diff against the base branch, including the working tree.

## Base branch

The branch this work was cut from, which is not always the repo's default. Take the first that applies, and say which source you used:

1. The base the caller names.
2. The target branch of the current branch's open PR or MR: `gh pr view --json baseRefName -q .baseRefName` on GitHub, `glab mr view --output json | jq -r .target_branch` on GitLab.
3. The repo's default branch from `git remote show origin`. If `git log --oneline <default>..HEAD` holds commits that are not this branch's work, the branch was cut from another base, so ask for it instead of guessing.

## Steps

1. Spawn an Agent with `subagent_type: "ps:stuart"`. Pass the scope as explicit files or a `<base>...HEAD` diff plus the working tree, and the absolute path of the skills directory that holds this file. Pick its model and effort for the job, per the Subagents section of the **minion** skill (`../minion/SKILL.md`). Do not restate its rules.
2. Inspect its report and diff. Reject application-code edits, scope escapes, exception-protected deletions, misstated `MUST KILL` reasons, and flags that treat kept intentional code as guilty. Reshape flags on our-code surprises stay actionable. Do not restore those comments. A keep survives only with proof it is about something we cannot change. Audit missed scoped lint and TypeScript suppressions. Correctness or safety suppressions stay actionable `MUST KILL`s. Restore deletions only with exact exceptions and scoped proof. Before accepting thin `IMPORTANT` or `do not remove` kills or keeps, run the **how** or **why** skill on their symbol. If a kill is ambiguous, do not restore. If a keep is refuted or still ambiguous, delete it. Revert and rerun one rejected report with the failure named. Reject a second, report it open, and fail `/ps:no-comments`.
3. Fix trivial accepted flags directly by deleting a dead path, dropping a parameter, or using the real API. If any fix needs a shape, sketch the types, signatures, and module boundaries once for the accepted set and surrounding code. Stop at the sketch. The sketch shapes. Step 4 implements.
4. Implement the smallest root-cause fix in scope. Remove every named workaround. If the root cause is out of scope, land the smallest in-scope fix and report the rest open. Fixing the root cause guides intent only. It never authorizes widening the fence nor fixing instances outside it. Never bolt on symptom guards.
5. Constraint comments say `do not remove`, `do not change wording`, or `talk to X before changing`. Leave keeps about things we cannot change. Offer the cheapest in-scope type, runtime, test, or CI lint. Wait for interactive approval. Unattended runs require the caller's pre-approval. If approved, encode then delete. Otherwise delete, report the constraint open, and sketch out-of-scope work.
6. Report the deletion count, restored comments, reruns, the shape sketch, fixes, encoding offers, encodings, unenforced constraints, and other open work.
