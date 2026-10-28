# Epistemics

How to reason about confidence when evidence is historical, fragmentary and
sometimes contradictory, and how to report it without flattening it into false
certainty. Code shows what it does, not why it exists. The why lives in
commits, PRs, tickets, docs and conversations, which are all incomplete, biased
and sometimes missing.

## Confidence tiers

Every claim sits in exactly one tier. The tier sets its section and its
phrasing.

### 1. Direct

An author wrote down why, explicitly. Not "the code does X, so the author must
have wanted X". Examples: a PR description "this fixes the bug where users with
more than 1000 links couldn't paginate"; a ticket "we're adding this because
the customer asked for it in their security review"; a code comment "clamp to
100 because the upstream API rejects larger values".

Phrasing: confident, present tense, cited. "This exists because X."

### 2. Supported

Several pieces of indirect evidence point to one conclusion. No single source
states it, but together they make it likely. Example: the PR title says
"improve performance", the ticket has a `perf` label, and the surrounding
commits all touch the same hot path.

Phrasing: confident but visibly derived, citing each item. "The evidence
points strongly to X: [the items]."

### 3. Inferred

A reasonable reading of the context that nothing explicitly supports. It is
your interpretation, not a fact from the record. Example: the PR doesn't say
why, but it merged the same day an error spiked, so it was probably a hotfix.

Phrasing: hedged, with the chain visible. "Given A and B, C seems likely
because D."

### 4. Speculative

A plausible hypothesis on thin evidence, where other explanations fit equally
well. Worth stating, because the user may know which is right, but mark it as a
guess.

Phrasing: "One possibility is X, but we found no direct evidence." Usually
belongs in Competing hypotheses.

### 5. Unknown

You looked and couldn't find out. This is a valid and important result.

Phrasing: "We searched X, Y and Z and found no evidence of why." Name what you
searched: "We searched the issue tracker for A and B, read the 6 PRs that touched this
file since 2024, and grepped the repos for the threshold value. None gave a
reason."

## Phrasing guide

**Confidence words** claim Direct or Supported evidence and need a citation
right next to them. Never use them for inferences: "because", "the reason is",
"was designed to", "fixes", "addresses", "solves", "the team decided".

**Hedges** mark interpretation. Use them freely in What we can reasonably
infer: "appears to", "seems to", "likely", "suggests", "is consistent with",
"one reading is", "plausibly", "may have been".

**Words to avoid:** "obviously", "clearly", "of course", "just" (as in "it's
just for performance"), "I think", "I believe". Write "the evidence suggests"
instead.

## Avoid rationalization

Code that makes sense today may have been written for reasons that no longer
apply, or that were wrong at the time. Don't retrofit a clean rationale onto
messy history. Don't assume a consistent pattern was intentional when it may be
copy-paste. Don't turn absence of evidence into evidence of absence.

## The sycophancy trap

Users often embed a guess: "Why do we do it this way, I assume it's for
performance?" Treat the guess as one candidate and check it independently. If
the evidence supports it, say so with citations. If not, say so, and present
what the evidence does support.

## When evidence contradicts

Show both sources with citations. Don't pick the one that makes a tidier
story. Both may be true (the ticket gave the reason, the PR gave the author's
framing), or one may be wrong. Let the user decide.

## When evidence is missing

An honest "we don't know" is one of the most useful results. It tells the user
the answer isn't in the obvious places and who to ask instead: the original
author, the product owner, the team lead. Filling a gap with a confident guess
harms the user, because they will act on it. Name each gap concretely: the
question, the sources searched, what you searched for, and what you found.

## Before finalizing

Check every claim in What we found and What we can reasonably infer:

- It has a citation, or it moves to Inferred or Competing hypotheses.
- Its phrasing matches its tier.
- It does not use code as evidence of its own intent.

The output must have a What we don't know section. If it names no gaps, be
suspicious: either the evidence was unusually complete or something is being
hidden.
