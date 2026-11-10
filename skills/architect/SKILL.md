---
name: architect
description: "Sketch the caller's usage, types and module boundaries before code, design it twice with parallel runners, pick a base and graft, then implement against the sketch. Use for 'architect this', 'design this', or non-trivial work where jumping to code would lock in the wrong shape."
user-invocable: true
disable-model-invocation: true
argument-hint: '<the feature, module, or change to design>'
allowed-tools: Read, Grep, Glob, Bash, Edit, Write, Agent
---

# Architect

**You sketch types, signatures and module boundaries with `not implemented` bodies and pseudocode, pick one shape from independent candidates, then fill in code against it.**

Bundled skills live at `../<name>/SKILL.md` from this file: `../how/SKILL.md`, `../why/SKILL.md` and `../unslop/SKILL.md`. To run one, read its `SKILL.md` in full and resolve its `references/` paths against that skill's own directory. Put the absolute path of `../minion/SKILL.md` in every `ps:kevin` prompt.

Open a todo list with one entry per phase: Ground, Sketch, Agree, Implement, Scrap.

## Phase A: Ground

Run the **how** skill over every subsystem the new code touches. Naming a file is not grounding. If the design redefines ownership or layering, also run the **why** skill on the existing shape, so its reasons become constraints. Write the grounding to `${TMPDIR:-/tmp}/architect-<slug>/grounding.md`. Skip this phase only for greenfield work.

## Phase B: Sketch

**Design it twice.** Get at least two structurally distinct whole-shape candidates, even when the first looks sufficient. A point fix inside one shape does not count. Spawn two `ps:kevin` runners in one message as background tasks. Do not pin a model, so they inherit the session's. Give each the task, [`references/runner-prompt.md`](references/runner-prompt.md) and the grounding by file path, plus its own directory `${TMPDIR:-/tmp}/architect-<slug>/candidate-<n>/`. Never a shared directory. Give the two runners different framings so they diverge, such as "optimize for the smallest public API" and "optimize for the fewest moving parts".

**Pick.** While runners work, derive a rubric of 3 to 6 gradeable criteria from the task. Read every candidate end to end and screen it against [`references/design-red-flags.md`](references/design-red-flags.md). Assume the next contributor is an agent that copies the nearest example and takes the shortest path that compiles. Score criterion by criterion. Pick the base a future maintainer can extend most easily. On a tie, prefer the one that hides more behind a smaller public API.

**Graft.** Take one or two strengths from each loser and fold them in by hand, so the result has one mental model. If candidates converge, say so and ship the consensus. If they wildly diverge, the framing was under-specified, so reframe and re-run. Record the base, the grafts and what you rejected in the rationale's "Synthesis decision" section.

## Phase C: Agree (opt-in)

The default is to proceed. Pause for sign-off only when the invoker asks, as in "show me before implementing".

For a one-way-door design (a new module boundary, a public API, a data model), spawn one fresh `ps:kevin` as a background task. Brief it to break the synthesized sketch, and keep only findings it shows with a concrete scenario. This is optional, and mandatory when the invoker asks for adversarial pressure.

Pushback on the shape is Phase A evidence. Re-ground and re-run Phase B before writing more code.

## Phase D: Implement against the sketch

Delegate the fill-in to a `ps:kevin` per minion's Subagents section. The sketch is the contract. The brief says to surface every deviation in the report, never absorb it. A parameter the sketch did not anticipate means the sketch was wrong, a requirement was missed, or the code overreaches. Decide which. Review the diff against the sketch yourself.

## Phase E: Scrap when the architecture is wrong

When implementation keeps producing friction the sketch cannot absorb, throw the sketch out. Fix the root cause, which is the shape, and do not bolt fixes onto it. The signal is a pattern, not one instance. Tells:

- The same shape of workaround appears across unrelated code.
- Several unrelated edge cases each need a special-case branch.
- Types need escape hatches (`any`, casts, optional fields always set in practice) to compile.
- The "we need a lock" reflex appears where the sketch said state was not shared.
- Callers must know the abstraction's internal rules to use it.
- Two or more independent Phase D deviations share one shape.

A few edge cases do not condemn an architecture. Complexity in the data is not complexity in the design.

To scrap:

1. Re-run the **how** skill over what was built.
2. Redesign as if the new constraints had been day-one assumptions.
3. Subtract before adding. The new sketch is smaller than the old one before it grows.
4. Return to Phase B.

## Outputs

Write the caller's usage first and derive the type sketch from it. The rationale ships alongside, per [`references/rationale-template.md`](references/rationale-template.md).

Follow the repo's convention for where design docs or types live. With none, write the type sketch next to the code it will become and the rationale as a sibling markdown file, and say where. A small change gets one sketch file. Larger work gets a module map plus type definitions.

Write the rationale and every reply through the **unslop** skill.

**Reply:** the chosen shape, the base and what was grafted, alternatives rejected and why, tradeoffs accepted, open questions as questions, and the next implementation step.
