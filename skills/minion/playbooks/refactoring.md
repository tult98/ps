### Refactoring

**You own the contract. The structure changes. The behavior does not.** Feature adds behavior and Bug fix corrects it. This playbook does neither.

If the cleanup reveals a missing feature or a real bug, split it out. Ship the structural change first against the pin, then route the rest to Feature or Bug fix. A redesign is allowed, but name it and route it to Feature. Large or cross-cutting migration runs as a sequence of independently landable refactors. Build the pin first. Then ship each refactor as its own PR, following Open a PR's scope and stack rules. This playbook is one such refactor.

1. Pin the behavior contract first. Run the **how** skill over the affected subsystem to learn the contract. Then write a characterization test, snapshot, or equivalence harness that captures current behavior before any structure moves. If the area has no coverage, write the pin before touching structure. Type check and lint are not a pin.
2. Name the structure the code is missing. It might be a type, a state machine, a table, or a module boundary. Boring code stays when its shape is already clear and local. The reshape must delete branches or invalid states, not add indirection.
3. Name the target shape. State the module layout, types, and call graph you would build today. When the target crosses a function boundary or two structurally different shapes are plausible, run the **architect** skill. Otherwise sketch the target shape yourself.
4. Subtract before you add. Delete dead code, collapse one-caller wrappers, drop redundant validators, and remove orphan references before the new shape goes in. Design for observed usage, not speculative edge cases. A reference with no novel content gets deleted, not left as a stub. The smallest change that reaches the target shape ships. A speculative cleanup that "might help" gets reverted.
5. Move in small behavior-preserving steps. Each step starts green, makes one change, runs the pin, and stays green before the next begins. For an API reshape, migrate every caller and delete the old API in the same wave. No compatibility shims and no parallel old-and-new paths. Update tests to assert the new contract, and delete tests that only protect pre-refactor implementation details. Delegate the mechanical edits to a `ps:kevin` subagent with a specific scope: the file paths, the names being moved, the behavior to hold, and the pin to keep green. Pick its model and effort per **Subagents**, `sonnet` for mechanical moves and `opus` for hard ones. Review its diff yourself. Spot-check every rename against the real files, because renames silently miss usages in strings, prose, and back-references.
6. Prove behavior is unchanged on the real artifact. "It compiles" is not a pass, and neither is a subagent's report. For larger reshapes, script an equivalence check that a reviewer can re-run. It can diff old and new outputs, replay a recorded baseline against the new code, or smoke-run the matching surface with the repo's verification skill. When a check fails, suspect the check before the code. Run the repo's lint, typecheck, and tests for the code you touched.
7. Confirm the change is worth keeping. The measure is lower reader load. If the diff does not make the code easier to read somewhere, revert it.
8. Commit small and ordered. The subtraction commit comes first, then the reshape, then any follow-on cleanup. Each commit lands on its own and leaves the pin green, so the sequence reads as an argument.
9. Run Open a PR (`playbooks/open-a-pr.md`).

**Reply:** the structure that changed, the pin you held it against, the
equivalence proof, the reader-load delta, and what shipped and what got
reverted. No new behavior.
