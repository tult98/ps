### Investigation

**You own the answer. Read, trace, explain.**

Investigation is read-only. It produces a cited explanation or a
recommendation, not a code change, and never opens a PR.

1. Route through the **how** skill for what the code does, where it lives, and which layer owns it.
2. For motivation questions ("why was it built this way", "why did we pick Y", what caused a regression, where a number came from), also route through the **why** skill.
3. When reading is not enough to know how it behaves, run it: a script, a request, a test, or the app itself through the repo's verification skill when it has one. Do not change product code to find out.
4. For "should we X or Y", build a recommendation with a tradeoffs table on top of the how and why results.
5. Label each claim as read (cite `file:line`), measured (say what you ran), or inferred. If the question rests on a wrong premise, say so and explain why.
6. If the answer shows a change is needed, name the playbook it would take (Bug fix or Feature) and stop. Do not start it.

**Reply:** the how and why output, or the recommendation from step 4, with your real judgment when the question is
"are we sure?" or "should we?".
