---
name: how
description: "Use for \"how does X work\", code walkthroughs before changing something, and placement / ownership / layering questions (\"where should this live\", \"which repo or module owns this\", \"is this the right layer\"). Explains subsystem architecture, runtime flow and onboarding mental models. Use why for motivation."
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Agent
---

# How

Answer "how does X work?", "where does X live?" and "which layer owns X?" at
the level of a senior engineer onboarding onto a subsystem: enough to build a
working mental model, not annotated source code.

## 1. Assess complexity

If the scope is ambiguous, state your reading of it and explore. The user can
redirect.

- **Simple** (one module, a small utility, a narrow question such as "how does
  function X work"): no subagents. Read the code yourself and write the answer
  with `references/explainer-prompt.md`'s output format. Go to step 4.
- **Complex** (a subsystem across several files or repos, a cross-cutting
  feature, a full architectural overview): go to step 2.

When in doubt, take the simple path.

## 2. Explore (complex only)

Split the question into 2 to 4 angles, each a distinct slice of the subsystem
(for example the FE flow, the BE endpoint, the data model). Spawn one `Explore`
agent per angle, all in a single message so they run in parallel. Build each
prompt from `references/explorer-prompt.md` with its angle filled in, and ask for
"very thorough" breadth.

## 3. Synthesize (complex only)

When every explorer has returned, spawn one `general-purpose` agent with
`references/explainer-prompt.md` and all the explorers' findings filled in. Tell it not to
edit any file.

## 4. Present

Present the explanation. Light edits for clarity or for context from the
conversation are fine. Do not rewrite it substantially.

The sections are Overview, Key Concepts, How It Works, Where Things Live and
Gotchas, as defined in `references/explainer-prompt.md`. Drop any that do not apply.
