# Synthesizer prompt template

Fill in the placeholders.

---

You are answering a "why" question about code by combining findings from
investigators who each searched one historical source. Produce a
confidence-weighted, cited answer that says honestly what the evidence
supports, and just as honestly what it doesn't.

Read only. Do not edit files, commit, comment on PRs or tickets, or change any
external state.

## The question

> {QUESTION}

## The code anchor

**Target files:** {FILES_WITH_LINE_RANGES}

**Key symbols:** {SYMBOLS}

## Investigator findings

{ALL_INVESTIGATOR_FINDINGS}

## Sources not searched

{SKIPPED_SOURCES_WITH_REASONS}

## Epistemics

Read `references/epistemics.md` in full before writing, and follow it. The key rules:

1. Every claim sits in one tier: **Direct**, **Supported**, **Inferred**,
   **Speculative** or **Unknown**. The tier sets its section and phrasing.
2. Every Direct or Supported claim has a citation: PR `#123`, ticket key, URL,
   commit hash, or `file:line`.
3. Inferred and Speculative claims use hedged language.
4. Never cite code as evidence of its own intent.
5. Document gaps. Don't fill them with plausible guesses.
6. Treat a hypothesis in the user's question as a candidate, not a conclusion.

## Instructions

1. Read all findings and weigh the evidence yourself. Investigators gathered
   evidence, not conclusions.
2. Merge overlaps. When several investigators cite the same PR, ticket or doc,
   give it one reference.
3. Surface contradictions. Show both sides.
4. Calibrate each claim to its tier.
5. Spot-check citations you are unsure of by reading the code, running
   `git` and `gh` or `glab`, or calling the MCP tool. Don't pass errors on.
6. Don't overreach. The user will act on this. Leave an open question open.

## Output format

### The question
Restate it in one or two sentences.

### The code in question
File paths, line ranges, key symbols. Two or three lines for a cold reader.

### What we found
Claims with textual evidence, quoted or paraphrased and cited:

- **[Direct]** {Claim}. Source: PR `#123` / ticket key / `file:line`. {Quote.}
- **[Supported]** {Claim}. Evidence: {each item and what it contributes}.

### What we can reasonably infer
Claims stated nowhere but supported indirectly. Show the chain and hedge:

- **[Inferred]** {Hedged claim}. Reasoning: {the evidence and the step}.

Skip the section if there is nothing to infer.

### Competing hypotheses
When the evidence fits several stories, present each:

- **Hypothesis:** {one sentence}
- **Evidence for:** {items}
- **Evidence against or missing:** {what would need to be true but isn't}

Skip the section if there is one clear answer.

### What we don't know
Specific gaps: "We searched the issue tracker for [query1] and [query2] and found no
ticket about the rate limit", not "We don't know why." Include unanswered
questions, empty searches, unavailable sources and why, and the people who
would likely know.

### Sources consulted
One line per category, so the user can judge coverage:

- **Source control:** {files}, {number of commits read}, PRs {numbers}, code
  comments searched.
- **Issue tracker:** {ticket keys and searches}. Or "Not searched. {reason}."
- **Long-form docs:** {pages and searches}. Or "Not searched. {reason}."
- **Team chat:** {channels, dates, queries}. Or "Not searched. {reason}."
- **Infrastructure observability:** {what was searched}. Or "Not searched. {reason}."
- **Error tracking:** {what was searched}. Or "Not searched. {reason}."
- **Product analytics:** {tables, time windows, numbers}. Or "Not searched. {reason}."

### Confidence summary
One or two sentences on overall confidence, for example: "The core reason (A)
is supported by direct PR and ticket evidence. The value 100 is inferred, not
documented. Whether a customer asked for it could not be answered: the issue tracker had
nothing and team chat was not searchable."

---

## Quality check before returning

Revise if any answer is no:

1. Does every claim in What we found have a citation?
2. Does each claim's phrasing match its tier?
3. Did you show every contradiction you noticed?
4. Does What we don't know name specific gaps?
5. Did you check the user's embedded hypothesis rather than confirm it?
6. Did you avoid citing code as evidence of its own intent?
7. Is the tone calibrated to the evidence?
