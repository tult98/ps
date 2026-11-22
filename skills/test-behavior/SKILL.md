---
name: test-behavior
description: Assert what users of the code can observe, not how the code is built. Use when you write, change, review or keep a test.
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Grep, Edit, Write, Bash
---

# Test Behavior, Not Implementation

A test is a claim about what users of the code can observe through its interface. That means return values, errors, state they can read back, and effects on outside systems. Private helpers, internal call order and intermediate data are implementation.

Judge each test against two imagined edits.

- **Break the behavior.** Picture a wrong value, or a skipped or doubled effect. If the test still passes, it proves nothing. The usual causes are a loose assertion, an expected value computed by the code under test, and an assertion on data the test built itself.
- **Change the internals.** Keep every observable result the same. If the test fails, it is pinned to the implementation.

Keep a test that catches the first edit and survives the second. Rewrite or delete the rest.

- Drive the code through its public interface with a concrete input. Derive the expected value from the requirement.
- What an external service receives is behavior. A call to an internal helper is not.
- Pair "this never happens" with a case where it does happen.
- Assert the error type callers branch on, not the message.
- Name a test for the behavior it protects. The name never says how the dependency is mocked.
