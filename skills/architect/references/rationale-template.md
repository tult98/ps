# Rationale template

The prose that ships alongside the type sketch. One page. Sentence-case headings, no boilerplate. Replace the italic notes with real content.

## Problem

*One paragraph. What we're trying to do, and what about the existing system or constraints makes the shape non-obvious. If [Phase A](../SKILL.md#phase-a-ground) surfaced constraints the design must honor, such as existing types to interop with, callers we can't break, or invariants that cross our boundary, name them here so the reader sees the same constraints you saw.*

## Usage (caller's view)

*Write this first, before the type sketch. Show the README or quickstart the consumer reads, plus two or three realistic call sites in their own code. Show what they import, what they call, and what comes back. The type sketch in [Shape](#shape) is derived from this, and the two must agree. When they diverge, reconcile the sketch to the usage, not the reverse. The caller's experience is the spec. The types serve it.*

## Shape

*The recommended architecture. Data structures first. Then how data flows through the signatures. Name the load-bearing decisions. State which invariants are encoded in types, where validation lives, and what the system deliberately does not do. Judge interface depth explicitly. State what complexity the public API hides, what stays exposed to callers, and why the interface is no larger than needed. Name the rule behind each decision in a few words, such as "validated at the boundary" or "derived, not synced". Don't restate the rule.*

## Synthesis decision

*Filled in by the orchestrator in Phase B's pick step, never by a runner. Records which candidate became the base and why, scored against the rubric. Names each graft and the candidate it came from, what was rejected and why, and any convergence or dropout.*

## Tradeoffs accepted

*One bullet per tradeoff the chosen shape makes, in the form "we accept X in exchange for Y". Name anything a future reader might mistake for an oversight, including things that look like premature optimization or premature simplification.*

## Alternatives considered

*Required. Name at least one concrete alternative shape, with one line on why it lost. Judge each alternative on interface depth, not implementation simplicity alone. Name the complexity it exposes to callers and the complexity it hides. Two or three alternatives belong here when the design space had real contenders. One is fine when the constraints forced the answer, phrased as "this was the only viable shape because...". Avoid listing flavors of the same shape. This section covers design alternatives the chosen shape rejected, not the other runner candidates.*

## Open questions and risks

*Things you noticed during the sketch that the human needs to weigh in on, and risks worth flagging before implementation starts. Phrase each as a question, not an assertion, so the human's answer is the resolution.*

## Next implementation step

*The first thing to build against the sketch, in one sentence. What you would start writing right after the pick, or after sign-off when a checkpoint was opted into.*
