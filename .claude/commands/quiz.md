---
description: Explain the change, then quiz the user on its behaviour — merge only on a perfect score
disable-model-invocation: true
---

Invoke the agent-skills:explain-and-quiz skill.

$ARGUMENTS

1. Read the spec, tasks/impl-notes.md, the check output actually run, then the diff.
2. Write a short explainer (prefer a single HTML file when visuals help): how the change fits existing code paths, the decisions that matter, behaviour before → after, deviations, what is not covered by tests.
3. Ask 5 behaviour questions (more for large changes), weighted toward behaviour that depends on existing code, incl. one on a deviation and one on a failure mode. Do NOT show answers.
4. After the user answers all of them, grade honestly with file:line evidence.
5. Perfect score → recommend /agent-skills:review. Any miss → point to what to re-read and ask two fresh questions on that area. Record the score on the PR/ticket.
