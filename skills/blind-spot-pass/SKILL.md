---
name: blind-spot-pass
description: Surfaces the user's unknown unknowns before work starts, the risks, conventions, best practices and quality bars they don't know they're missing, calibrated to their stated experience. Use when the user says "blind spot pass", "what am I missing", "unknown unknowns" or "what don't I know here", when they're entering an unfamiliar codebase, domain or technique (a new ML method, auth, deployment, frontend), or when they say they're unsure what the best approach or best practice is before any spec or plan exists.
---

# Blind Spot Pass

## Overview

Every prompt is a map; the codebase, the domain and real-world constraints are the territory. The gap between them is made of unknowns, and an agent fills each one with a plausible default. Four kinds exist:

| Kind | Example | Best tool |
|---|---|---|
| Known knowns | What's in the prompt | — |
| Known unknowns | "I haven't decided the split strategy" | `interview-me` |
| Unknown knowns | "I'll know the layout I want when I see it" | `idea-refine` + throwaway prototypes |
| **Unknown unknowns** | "I didn't know speaker leakage across splits was a thing" | **This skill** |

Unknown unknowns are the expensive ones: nobody asks about them, so they surface during implementation or in production. A blind spot pass hunts them deliberately, *before* the spec, using what the agent knows about common failure modes in the domain plus a quick look at the actual codebase.

This skill does not decide anything and does not propose a solution. Its output is a ranked list of things to check, which then feeds `interview-me`, `spec-driven-development`, or the user's own reading.

## When to Use

- The user explicitly asks: "blind spot pass", "what am I missing?", "unknown unknowns", "what should I know before I start?"
- The user is new to the codebase, the domain, or the technique (says so, or it's evident)
- The user is unsure which approach is standard or best practice for a problem
- Before `spec-driven-development` on any medium or large task in unfamiliar territory

**When NOT to use:**

- The user already knows the domain well and the task is routine
- The requirement itself is vague (who / why / success unclear) — run `interview-me` first; blind spots in a fuzzy goal are noise
- A plan already exists and needs stress-testing — use `doubt-driven-development`

## The Process

### Step 1: Calibrate to the person

Ask, or extract from what they already said, in one message:

- Their role and strengths ("ML engineer, strong on PyTorch and evaluation")
- Their gaps ("little frontend, never touched this repo's auth")
- The task in one sentence, and their own predicted approach if they have one

If they've already given this, don't ask again. The calibration decides what counts as a blind spot: leakage is obvious to a senior ML researcher and a real risk for a backend engineer training a first model.

### Step 2: Look before you list

Spend a few minutes grounding the list in reality:

- Skim the relevant parts of the codebase: existing patterns, configs, tests, data loaders, CI
- If the domain is fast-moving or the user named a specific library or technique, check current official docs (`source-driven-development` rules apply: cite, mark unverified claims)

A blind spot list built only from general knowledge is a generic checklist. One grounded in *this* repo ("your eval script reads `data/all.csv` and splits randomly, but rows repeat per speaker") is a finding.

### Step 3: Generate, then rank

Produce candidate blind spots across these lenses, then keep only those that plausibly apply:

- **Data** — leakage, split strategy, label quality, imbalance, privacy/consent, versioning
- **Correctness** — edge cases, failure modes, silent defaults, numerical issues
- **Quality bar** — how good "good" can be; what experts would consider table stakes
- **Integration** — how this touches existing code paths, contracts, other teams
- **Ops** — performance, cost, monitoring, rollback, train/serve skew
- **Vocabulary** — the standard names for this problem, so the user can search for themselves

Rank by *cost if missed × likelihood it applies here*. Cap the output at about 10 items; a 40-item list is a way of hiding the 3 that matter.

### Step 4: Report in this format

```
BLIND SPOT PASS — [task]
Calibrated for: [role, strengths, gaps]

1. [Blind spot, in one line]
   Why it matters here: [specific to this repo/task, with file:line if found]
   How to check: [a concrete, cheap check — a query, a test, a doc to read]
   Kind: unknown unknown | likely-unknown known
2. ...

Terms worth searching: [3–6 standard names]
Not checked: [what you didn't look at, so the user knows the limits]
```

Do not propose a solution or architecture. End by asking which items the user wants to resolve now, and suggest the next skill (`interview-me` for decisions, `spec-driven-development` to record them).

### Step 5: Record

Each blind spot the user accepts becomes either an open question, a decision, or an acceptance check in the spec. If the project keeps an unknowns log (Jira comment, Notion database, `docs/unknowns.md`), add the items there with their kind and how they were resolved.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I'll just list the standard best practices" | A generic list is not calibrated to this person or this repo; ground every item in Step 2 or drop it. |
| "Better to be exhaustive" | Forty items bury the three that matter. Rank and cap. |
| "The user asked what they're missing, so I should also tell them what to do" | Proposing a solution anchors them before they've weighed the unknowns. Report, then hand off. |
| "They're experienced, they won't have blind spots" | Experts have blind spots at the edges of their expertise; calibrate, don't skip. |
| "I'm not sure this pitfall really exists" | Then mark it unverified and give the term to search, don't state it as fact. |

## Red Flags

- The list contains items that would apply to any project in any language
- No file paths or repo specifics anywhere in the report
- A recommended architecture appears in the output
- More than ~10 items, or items not ranked
- Claims about libraries or papers with no source and no "unverified" marker

## Verification

- [ ] Calibration (role, strengths, gaps) is stated at the top
- [ ] At least some items cite specifics from the codebase or current docs
- [ ] Each item has "why it matters here" and a concrete "how to check"
- [ ] Items are ranked and capped (~10)
- [ ] Search terms provided; limits of the pass stated
- [ ] No solution proposed; next skill suggested
