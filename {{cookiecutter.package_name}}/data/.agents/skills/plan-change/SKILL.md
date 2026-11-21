---
name: plan-change
description: Plans a feature, refactor, or fix before any code is written, breaking it into small, testable steps. Use when a task touches several modules, changes a public API, or has an unclear approach.
argument-hint: "[feature, issue, or goal]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

# Planning a Change

Plan the work in $ARGUMENTS before editing anything. A plan is for the user to approve; do not
start implementing until they do.

Use `understand-code` first when the affected code is unfamiliar.

## When a plan is worth it

Plan when the change touches more than one module, alters a public function's signature or
behavior, adds a dependency, or has more than one reasonable approach. A one-line fix needs no
plan; make it.

## Gather before deciding

1. Restate the goal in one or two sentences, and what done looks like from the caller's side.
2. Find the code it touches: `rg` for the names involved, then read the callers as well as the
   definitions. Note every public name whose behavior changes.
3. Find the tests that cover that code, and what they leave uncovered.
4. Check `pyproject.toml` for the Python version and the dependencies already available. Prefer
   the standard library and existing dependencies; a new one needs the user's approval.
5. List what you do not know. Ask about anything that changes the design; pick a sensible
   default for the rest and say so.

## Choose an approach

When there is more than one way, compare at most two or three on what matters here:
correctness, how much existing code changes, public API impact, and cost to test. Recommend one
and say why in a sentence. Do not survey options you would not pick.

## Write the plan

```md
## Goal
One or two sentences, and the observable result.

## Approach
The chosen design and why, in a short paragraph.

## Steps
1. <change> in `module.py`, with the test that proves it
2. ...

## Public API changes
New, changed, or removed names, and whether any change breaks callers.

## Risks and open questions
What could go wrong, and what still needs an answer.
```

Each step:

- leaves the project passing `prek run -a --stage pre-push`, so it can be its own commit
- names the files it touches
- names the test that proves it; write that test first when fixing a bug (see `write-tests`)

Order steps so the riskiest assumption is tested earliest. Put pure refactors that keep
behavior in their own steps, before the steps that change behavior.

## After approval

Work through the steps in order with `write-code` and `write-tests`. When a step turns out
wrong, stop and update the plan with the user rather than drifting from it. Finish with
`cleanup-code`.
