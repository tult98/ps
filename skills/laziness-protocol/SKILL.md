---
name: laziness-protocol
description: Bias toward deletion and the smallest change that solves the problem. Use before shipping a diff, when evaluating diff size, or when tempted to add an abstraction, a layer, or a flag.
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Grep, Bash
---

# Laziness Protocol

The smallest change that solves the problem wins. Fewer lines beat tidy boilerplate.

- **Minimize the diff.** Write down the smallest change you expect before you build. Compare the finished diff to it. Each file or hunk beyond that estimate needs a reason that traces to the request or to runtime evidence. A reason of "it might help" fails. Revert the hunk.
- **Consolidate decisions.** Make a choice in one place. Put it behind one source of truth and pass the result as a simple flag. Do not repeat the same check in several files.
- **Prefer deletion.** Before adding an abstraction, a layer, a wrapper or signal threading, look for code to delete or reuse. Add the new thing only when the deletion route fails.
- **Take the request as written.** An unrequested extra such as a new field, flag, test harness or config is scope. Leave it out. If you think it is needed, say so and let the user decide.
