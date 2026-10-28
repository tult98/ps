---
name: deslop
description: Remove AI-generated code slop and clean up code style
user-invocable: true
allowed-tools: Bash, Read, Edit, Grep, Glob
---

# Remove AI code slop

Check the diff against the base branch (see **Base branch**) and remove AI-generated slop introduced in the branch.

## Base branch

The branch this work was cut from, which is not always the repo's default. Take the first that applies, and say which source you used:

1. The base the caller names.
2. The target branch of the current branch's open PR or MR: `gh pr view --json baseRefName -q .baseRefName` on GitHub, `glab mr view --output json | jq -r .target_branch` on GitLab.
3. The repo's default branch from `git remote show origin`. If `git log --oneline <default>..HEAD` holds commits that are not this branch's work, the branch was cut from another base, so ask for it instead of guessing.

## Focus Areas

- Extra comments that are unnecessary or inconsistent with local style
- Defensive checks or try/catch blocks that are abnormal for trusted code paths
- Casts to `any` used only to bypass type issues
- Deeply nested code that should be simplified with early returns
- Other patterns inconsistent with the file and surrounding codebase

## Guardrails

- Keep behavior unchanged unless fixing a clear bug.
- Prefer minimal, focused edits over broad rewrites.
- Keep the final summary concise (1-3 sentences).
