---
name: implementation-notes
description: Keeps a running implementation-notes file during a build that records every decision the spec didn't cover, every deviation (taking the conservative option and continuing), surprises about the codebase, and open questions for the human. Use when starting implementation from a spec or plan, especially in a fresh session or a long or autonomous run (/build auto), when the user asks for "implementation notes", "a decision log" or "log your deviations", or when handing work between sessions or agents.
---

# Implementation Notes

## Overview

A spec never covers everything. During a build the agent meets edge cases and makes dozens of small decisions the human never sees. Those decisions are where comprehension debt and intent debt hide: the code works, the tests pass, and nobody knows that missing labels are being dropped instead of counted as errors.

Implementation notes make those decisions visible. The agent keeps a short file beside the plan, writes each unplanned decision as it happens, and when an edge case forces a departure from the spec it **takes the conservative option, logs it as a deviation, and keeps going** rather than either stopping for every question or silently improvising.

The notes serve three readers: the human reviewing the change (read Deviations first), the next session or agent picking up the work (fresh context, full history), and the human's own learning (Surprises is the fastest tour of an unfamiliar codebase).

## When to Use

- Starting `/build` or `incremental-implementation` from a spec or plan
- Any autonomous or long-running build (`/build auto`, background agents)
- Handing work between sessions: start the new session with spec + plan + notes
- The user asks for a decision log, deviation log, or implementation notes

**When NOT to use:**

- One-line fixes and mechanical changes with no decisions to record
- Architecture-level decisions that deserve an ADR — use `documentation-and-adrs` (link the ADR from the notes)

## The Process

### Step 1: Create the file at the start

Create `tasks/impl-notes.md` (or next to the existing plan) before writing code:

```markdown
# Implementation notes: [feature] ([ticket / spec link])
Spec: [path]  Plan: [path]  Started: [date]

## Decisions (not covered by the spec)
- [task #] [decision] — why: [...] — reversible: yes/no

## Deviations (conservative option taken; needs human review)
- [task #] Spec said: [...] → Did: [...] — because: [edge case] — [file:line]

## Surprises (things learned about the codebase)
- [...]

## Open questions for the human
- [ ] [...]
```

### Step 2: Write as you go, not at the end

Add an entry at the moment of the decision. Reconstructed notes miss exactly the small decisions that matter. One line each; link file:line.

### Step 3: The deviation rule

When an edge case conflicts with the spec, or the spec is silent where it matters:

1. Is it in the spec's "Ask first" or "Never" boundaries, or irreversible (data deletion, auth, payments, deploys)? → **Stop and ask.** Follow `doubt-driven-development`.
2. Otherwise → choose the **conservative** option: the one that changes the least, loses no data, is easiest to revert, and keeps existing behaviour. Log it under Deviations and continue.

"Conservative" examples: exclude-and-count rather than drop silently; keep the old default rather than invent a new one; fail loudly rather than guess.

### Step 4: Close out

At the end of the build:

- Move resolved open questions into Decisions
- Put a 3-line summary at the top: tasks done, number of deviations, the one deviation the reviewer must look at
- Link the notes from the PR description and the ticket

The reviewer (or `explain-and-quiz`) reads Deviations before the diff.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "It's a tiny decision, not worth logging" | Tiny unlogged decisions are exactly what the reviewer can't see. One line costs seconds. |
| "I'll write the notes at the end" | By the end the small decisions are forgotten or rationalized; write at the moment. |
| "The spec was ambiguous, so I picked what seemed best" | "Best" is a guess; "conservative and logged" is reviewable. |
| "Stopping to ask is safer" | Stopping for every edge case stalls the build; reserve stops for irreversible or Ask-first items. |
| "The commit messages already say this" | Commits say what changed, not what the spec didn't cover or where you departed from it. |

## Red Flags

- A finished build with an empty Deviations section on a non-trivial spec
- Notes written in one block at the end
- A deviation on an "Ask first" / irreversible item that wasn't escalated
- Notes not linked from the PR or ticket

## Verification

- [ ] Notes file exists, created before the first code change
- [ ] Every deviation has spec-said / did / because / file:line
- [ ] No irreversible or Ask-first item handled as a silent deviation
- [ ] Close-out summary at the top; open questions resolved or listed
- [ ] Linked from the PR and ticket
