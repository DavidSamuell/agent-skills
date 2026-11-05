---
name: reference-driven-development
description: Uses existing source code, notebooks or prior implementations as the specification when the desired behaviour is hard to describe in words, extracting what the reference really does before copying or adapting it. Use when the user points at a repo, folder, file, notebook or old system and says "do it like this", "port this", "match this behaviour", or "same as X but in Y", when reimplementing logic across languages or frameworks, or when replacing an old pipeline whose exact behaviour must be preserved.
---

# Reference-Driven Development

## Overview

Some requirements are easier to show than to say. "Retry like the Rust client does", "split the data the way last year's experiment did", "the same layout as the old dashboard" all carry far more detail than any paragraph the user could write. Source code is the richest reference: it holds the edge cases, defaults and ordering that a screenshot or a summary loses.

The risk is copying blindly: porting a bug, a licence problem, or behaviour that only made sense in the old context. This skill makes the reference explicit first — what it actually does, in plain words — and only then decides, with the user, what to copy, adapt and leave out.

For official framework and library documentation as the source of truth, use `source-driven-development` instead. This skill is for *code and artifacts* as the reference.

## When to Use

- The user points at code, a notebook, a config, or a running system as the model for what they want
- Porting behaviour across languages or frameworks (Rust → TypeScript, notebook → package, TensorFlow → PyTorch)
- Replacing an old system whose real behaviour (not its documentation) must be preserved
- The user says they can't describe what they want, but can point at an example

**When NOT to use:**

- The reference is official docs for a library or framework — use `source-driven-development`
- The user wants something new and the "reference" is just loose inspiration — use `idea-refine`

## The Process

### Step 1: Pin the reference

Record exactly what the reference is: repo + commit or tag, file paths, notebook cell range, or system version. A moving reference makes later comparison meaningless. Check the licence if the code is external.

### Step 2: Explain what it actually does

Read the reference and write a plain-language behaviour summary *before* writing any new code:

```
REFERENCE: [repo@sha, path(s)]
WHAT IT DOES:
- [behaviour 1, including defaults and constants]
- [ordering, retries, error handling, randomness/seeding]
EDGE CASES IT HANDLES: [...]
IMPLICIT ASSUMPTIONS: [env vars, data shape, library versions, global state]
SURPRISES: [anything that looks like a bug or an accident]
```

The user reviews this summary. Misreadings caught here are free; caught after porting, they're a debugging session.

### Step 3: Decide copy / adapt / leave out

Present a three-column decision and get the user's agreement:

| Copy as-is | Adapt (and how) | Leave out (and why) |
|---|---|---|
| Backoff formula and jitter | Config from env → our settings module | Global retry counter (not thread-safe here) |

Anything in "Surprises" must land in one of the columns explicitly, never silently.

### Step 4: Build a differential check

Where the reference can still run, make it the oracle: feed the same inputs (sampled, generated, or replayed) to old and new, and compare outputs. Normalize irrelevant differences (ordering, formatting). Every remaining difference is either a bug or an intended change listed in "Adapt" / "Leave out". This check becomes an acceptance check in the spec.

Where the reference can't run, write example-based tests from Step 2's edge cases.

### Step 5: Implement and reconcile

Implement using `incremental-implementation`. Run the differential check; for each mismatch, fix it or add it to an approved-differences list with a reason. Link the reference and the approved-differences list from the spec or PR.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The reference is clear, I'll just port it" | Unstated defaults and global state are what make ports subtly wrong; write the summary first. |
| "That looks like a bug, I'll quietly fix it" | Users may rely on it. Put it in a column and let the user decide. |
| "The tests pass on the new code" | Tests you wrote encode your reading of the reference. Only a differential check compares against the reference itself. |
| "Pinning the commit is overkill" | If the reference moves, nobody can tell later whether a difference was intended. |

## Red Flags

- New code written before a behaviour summary exists
- No commit / version recorded for the reference
- Differences between old and new with no recorded reason
- External code copied without a licence check

## Verification

- [ ] Reference pinned (repo@sha or version, paths)
- [ ] Plain-language behaviour summary reviewed by the user
- [ ] Copy / adapt / leave-out table agreed, surprises all placed
- [ ] Differential check (or edge-case tests) passing
- [ ] Approved-differences list linked from spec or PR
