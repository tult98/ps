### Bug fix

**You own the fix. Plan, review, verify.** Delegate the fix to a subagent, stay in the lead.

Every line you ship traces to runtime evidence. A change that "might help" is a
hypothesis, not a fix, and it does not ship. When evidence refutes a
hypothesis, revert what it motivated. Ship the smallest change the evidence
justifies.

1. Reproduce it yourself on the surface where it was reported, using the repo's verification skill when it has one (for example `.claude/skills/verify-<app>/`), following its recipe for the feature. If the repo has none, drive the surface with the repo's own test tooling and say so in the reply. For pure logic, a test or script is enough. Ask the user to reproduce only after driving the surface as far as it goes, and say exactly what you could not reach.
2. Find the root cause. Seed the candidate hypotheses with the **how** skill over the affected subsystem and the **why** skill for regression history. Then rule them out one at a time with runtime evidence: logs and added instrumentation. Trace the symptom back to the first place the system went wrong. A guard is a fix only when the state it handles is legitimate.
3. Plan the fix there: the smallest change that removes the cause. Delegate implementation to a `ps:kevin` subagent with a specific scope: the file paths, the root cause and its evidence, the planned change, and the repro it must turn green. Pick its model and effort for the job, per **Subagents**. The brief tells it to follow the surrounding code's conventions and write comments per **Comments**.
4. Review the diff yourself against the brief: it fixes the cause at the place step 2 found, adds nothing the evidence does not justify, and follows **Comments**. Write your own summary of what changed. For each problem, give a fresh `ps:kevin` a fix round with consolidated scope, per **Subagents**, and review again until the diff holds.
5. Verify on the same surface and with the same recipe as step 1: the original repro now passes. Unit tests alone do not prove the bug is gone. Run the repo's lint, typecheck, and tests for the code you touched.
6. Commit in order: the failing test first when there is a cheap local test for it, the fix on top.
7. Run Open a PR (`playbooks/open-a-pr.md`).

**Reply:** what was broken, the root cause, the fix, and how you verified it.
Paste the repro output from before and after the fix.
