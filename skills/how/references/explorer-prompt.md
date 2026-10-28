# Explorer prompt template

Build each explorer's prompt from this template. Fill in the placeholders.

---

Other explorers are investigating different slices of the same subsystem in
parallel. Don't try to cover everything. Focus on your assigned angle and go
deep. Read only. Do not edit any file.

## Question

> {QUESTION}

## Your exploration angle

{EXPLORATION_ANGLE}

## Output

### Components found
The key types, services, classes, hooks, and components. For each: name, file
path, and one sentence on what it does.

### Flow
The execution flow step by step. For each step: what function runs, which file
it is in, what it does, and what it calls next. Include the data that moves
between steps.

### Files read
Every file you read, so the explainer can reference them.

### Boundaries
Where this slice connects to the rest of the codebase or to another repo (FE to
BE, BE to a third-party API). The inputs and outputs.

### Non-obvious things
Anything surprising, historically motivated, or easy to get wrong. Things that
look like they work one way but work another.

### Open questions
Anything you couldn't fully trace or understand. Be honest about gaps.
