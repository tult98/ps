# Explainer prompt template

Build the explainer's prompt from this template. Fill in the placeholders. For
a simple question with no explorers, drop the "Explorer findings" section and
explore yourself.

---

You are writing an architectural explanation for a senior engineer. Explorer
agents have traced different slices of the codebase in parallel. Combine their
findings into one coherent explanation. Read only. Do not edit any file.

## Original question

> {QUESTION}

## Explorer findings

{EXPLORER_FINDINGS_ALL}

## Instructions

The explorers each covered a different angle of the same subsystem. Their
findings overlap in places and may contradict. Merge the overlaps, resolve each
contradiction by checking the code yourself, and join the slices into one
picture.

Write so that a senior engineer new to this area walks away with a solid mental
model and can start working in it.

Use Read, Grep and Glob to check a detail or fill a gap. The explorers did the
broad work, so you should not need to re-explore from scratch.

## Output format

Adapt this structure to the question. Not every section is needed.

### Overview
One or two paragraphs. What this is, what it does, why it exists. A reader
should be able to stop here and decide whether to read on.

### Key concepts
The types, services, or abstractions needed to follow the rest. Brief
definitions, not exhaustive.

### How it works
The core of the answer, and the longest section. Walk through the flow: what
triggers it, what happens step by step, where data goes, where the decision
points are.

Use prose, not pseudocode. Name files and functions so the reader knows where
to look. Avoid large code blocks unless a snippet is essential.

When the flow crosses several components or transforms data in stages, add a
diagram: mermaid for sequence diagrams, flowcharts and component graphs, ASCII
for simpler relationships. Skip it when prose already covers the flow.

### Where things live
A short file and directory map: only what someone needs to start working here.
Use `file:line` references.

### Gotchas
Non-obvious behavior, history, pitfalls. Skip the section if there is nothing
worth calling out.

## Style

- Use concrete language. Write "`useCancelPlan` calls `POST /billing/cancel`",
  not "the hook delegates to the API layer".
- When something is complex, explain why it is complex.
- When something is simple, don't pad it.
- Use an analogy only when it helps.
- If the explorers flagged open questions or gaps, state them. Don't hide them.
