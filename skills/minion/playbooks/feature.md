### Feature

**You own the design. Plan, review, verify.** Delegate implementation. Stay in the lead.

1. Read the request and its sources: the ticket and its acceptance criteria, and any linked designs, PRs or MRs. Run the **how** skill over the affected subsystem.
2. Name the data shape before writing logic: the types, the state, and how they are organized. Prefer a state machine over scattered booleans, a table or registry over branching, and a typed model over repeated shape assumptions.
3. Plan the change: the files to touch and the order. When the design has more than one reasonable shape, compare two options in a short table and pick one. Ask the user only for a product or preference call. If running something can answer the question, run it.
4. Delegate code-writing to a `ps:kevin` subagent with a specific scope: the file paths, the data shape from step 2 and its organizing structure, the plan from step 3, and success criteria (the acceptance criteria it serves and the checks it must pass). Pick its model and effort for the job, per **Subagents**. Mandatory: no skip-with-reason escape, because the gain is review separation, not lines saved. The brief tells it to build in small units that each end in a check (typecheck, a test, or a look in the browser), follow the surrounding code's conventions, write comments per **Comments**, and commit liberally.
5. Review the diff yourself against the brief: the data shape, the plan, the surrounding code's conventions, and **Comments**. Write your own summary of what changed. For each problem, give a fresh `ps:kevin` a fix round with consolidated scope, per **Subagents**, and review again until the diff holds.
6. Verify on the real surface against each acceptance criterion, using the repo's verification skill when it has one (for example `.claude/skills/verify-<app>/`), following its recipe for the feature. If the repo has none, drive the surface with the repo's own test tooling and say so in the reply. "It compiles" is not a pass. Run the repo's lint, typecheck, and tests for the code you touched. Note the evidence dir the verify run wrote. Open a PR attaches from it.
7. Arrange the commits so each one is small, ordered, and leaves the build green.
8. Run Open a PR (`playbooks/open-a-pr.md`).

**Reply:** what you built, what you chose and why, how you verified each
acceptance criterion, and any open decisions. Use a table for design
alternatives.
