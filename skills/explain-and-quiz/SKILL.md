---
name: explain-and-quiz
description: Turns a finished change into an explainer (or a short pitch for reviewers) and a comprehension quiz the human must pass before merging, so they understand what was built rather than just approving a diff. Use after a long or agent-driven implementation session, before merging or shipping agent-written code, when the user asks for an "explainer", "pitch", "write-up for review", "quiz me" or "check I understand this", or when the user says they can't fully explain what the agent built.
---

# Explain and Quiz

## Overview

Agent-written code can be clean, tested, and still not understood by the person who has to answer for it. That gap is comprehension debt, and it shows up later, during an incident or a refactor, when nobody can say why the code works the way it does. Diffs alone don't close it: much of a change's behaviour depends on existing code paths the diff never shows.

This skill closes it in two parts:

1. **Explain** — an explainer that gives context and intuition, or a pitch that packages spec, prototype and notes for reviewers who weren't in the room.
2. **Quiz** — questions about the change's behaviour, especially behaviour that depends on existing code, which the human answers *before* seeing the answers. **Merge only on a perfect score.** A miss shows exactly what to re-read.

The explainer is for understanding; the quiz is the proof.

## When to Use

- After any long or autonomous build session, before merge
- Before `/review` or `/ship` on agent-written code
- The user needs buy-in or approval from someone who wasn't involved (pitch mode)
- The user says "quiz me", "explain this to me", "I'm not sure I understand what you built"

**When NOT to use:**

- Trivial changes whose behaviour is fully visible in a small diff
- Code review for quality and defects — that's `code-review-and-quality` (run it too; it answers a different question)

## The Process

### Step 1: Gather the evidence first

Read, in this order: the spec and acceptance checks, `tasks/impl-notes.md` (Deviations first), the test and check output actually run, then the diff. The explainer is built from evidence, not from the agent's memory of what it intended.

### Step 2: Write the explainer (or pitch)

**Explainer** (for the person who will own the code), one page:

```markdown
# What changed: [feature]
## In one paragraph
## How it fits the existing code (the paths it touches, with file:line)
## The 3 decisions that matter, and the alternative rejected for each
## Behaviour that changed (before → after)
## Deviations from the spec (from impl-notes) and why
## What's NOT covered by tests / still uncertain
```

**Pitch** (for a reviewer or approver), one page: lead with a demo (GIF, screenshots, or a short command + output), then problem, what changed, the 3 decisions, known risks and the checks covering them, deviations needing sign-off. Write it for someone who shares the user's *starting* unknowns.

Prefer a single HTML file when visuals help (diagrams, before/after screenshots); otherwise markdown.

**Ask the user to write the "3 decisions that matter" bullets themselves first**, then compare with the agent's version. If they can't write them, that's the first finding.

### Step 3: Write the quiz

Five questions (more for large changes), each:

- About **behaviour**, not trivia ("What happens when a transcript is missing?" not "What's the function name?")
- Weighted toward behaviour that depends on **existing code paths** the diff doesn't show
- Including at least one question about a **deviation** and one about a **failure mode**
- Answerable in a sentence

Do not reveal answers. Ask the user to answer all of them first.

### Step 4: Grade honestly

Grade each answer as correct / partly / wrong. For anything less than correct: give the right answer with the file:line that proves it, and point to what to re-read. Don't round partial answers up to be polite.

### Step 5: Gate and record

- **Perfect score** → proceed to `code-review-and-quality` / `shipping-and-launch`.
- **Any miss** → user re-reads the indicated code, then answers two fresh questions on the same area.
- Record the result where the work is tracked (PR description, ticket comment, research notes): score, the question missed, and the explainer link.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "Tests pass, so understanding is optional" | Tests prove behaviour on the cases they cover; they don't let you explain or change the code later. |
| "4/5 is good enough" | The miss is where your mental model is wrong. That's the line that will bite you. |
| "I'll make the quiz easy so it doesn't block" | An easy quiz is theatre. Ask about behaviour that depends on code outside the diff. |
| "The agent's explainer is enough, no need for a quiz" | Reading a fluent explanation creates a feeling of understanding; answering cold tests it. |
| "Showing the answers alongside saves time" | Then it's a reading exercise, not a check. |

## Red Flags

- Quiz questions answerable from the explainer's headings alone
- Answers shown before the user responds
- Partial answers graded as correct
- Merge recommended after a miss without a re-check
- Explainer written without reading impl-notes or actual check output

## Verification

- [ ] Explainer or pitch built from spec, impl-notes, check output and diff
- [ ] User wrote their own "3 decisions" before seeing the agent's
- [ ] Quiz: ≥5 behaviour questions, incl. existing-code-path, deviation and failure-mode questions
- [ ] Answers hidden until the user responded; graded honestly with file:line
- [ ] Perfect score before merge, result recorded on PR / ticket
