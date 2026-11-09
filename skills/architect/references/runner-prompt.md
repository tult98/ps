# Architect runner prompt

The orchestrator passes this file to every Phase B runner along with the task, the grounding path, any framing, and the runner's own output directory. Runners never share a directory.

You are producing one candidate design. Read the architect skill at [`../SKILL.md`](../SKILL.md) in full first. Do not edit the repo. Write only inside your output directory. Produce the usage sketch, the type sketch and signatures, a module map when the work spans modules, and a rationale shaped per [`rationale-template.md`](rationale-template.md). Leave its "Synthesis decision" section empty.

Apply this discipline. The orchestrator scores candidates on these axes to pick a base.

- **Caller's usage first.** Write the README-style usage and two or three real call sites before the types, then derive the type sketch from them. The usage is the spec. When the two disagree, reconcile the sketch to the usage, not the reverse.
- **Data structures first.** Get the core types right and the code becomes obvious. Trace each dominant access pattern through the proposed structure. If the answer is "we'll add a map, index or cache later", the structure is wrong.
- **Interface depth.** Compare the capability hidden behind the public API to the size of that API. Prefer a simple interface that pulls complexity into the callee, even when the implementation gets harder. Keep transport and wire types off the public API. Parse them into domain types behind the interface.
- **Separate before serializing shared state.** If two actors might write the same state, ask what happens. If the answer is not "nothing", give each actor its own state and merge where it is read. Serialize with a lock or a single writer only when one shared write target is a real invariant.
- **Visible boundaries.** Use `not implemented` errors for bodies, `// TODO` pseudocode for tricky logic, and doc comments that state intent and invariants. A reader should trace data from input to output through types and signatures alone.
- **Encode invariants in structure.** Hard-to-misuse types beat runtime checks, and runtime checks beat prose comments. When a rule can live in a type, the type is the whole fix.
- **Boundary discipline.** Validate and parse at system boundaries such as CLI args, config, the network and external APIs. Inside, trust the types and skip re-validation. Business logic lives in pure functions and the shell stays thin.
- **One source of truth per invariant.** Derive instead of sync.
- **Idempotent operations.** Every state change answers what happens if it runs twice or crashes halfway. Derive each step from the state that exists, so a repeat finds nothing left to do.
- **Smallest change that works.** Fewer lines beat elegant boilerplate. Make each decision in one place and pass the result along.
- **Low reader load.** If tracing a flow needs more than three files, flatten it. Collapse wrappers with one caller and layers that only pass arguments through. A new reader should answer "where does X come from?" and "what can change X?" in under 30 seconds.

You are one of several runners. Produce the best design you can and don't hedge against the others. If you got a framing, commit to it fully. Differences between candidates are the signal for picking a base. A safe-looking middle defeats the exploration.

End your run with the path to your package and one paragraph naming the shape, the alternatives you considered, and why you rejected them.
