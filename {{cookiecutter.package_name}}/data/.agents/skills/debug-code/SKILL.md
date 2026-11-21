---
name: debug-code
description: Finds the root cause of a Python bug, failing test, or traceback, and fixes it with a regression test. Use when something errors, gives wrong results, or a check fails.
argument-hint: "[error, failing test, or symptom]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Edit
  - Bash
---

# Debugging Code

Find why $ARGUMENTS happens, fix the cause rather than the symptom, and leave a test that would
have caught it.

## 1. Reproduce

- Get the exact command, input, and full traceback. Read the traceback from the bottom up: the
  last frame in this project's code is usually where to start.
- Reproduce it locally before changing anything. If it cannot be reproduced, say what you tried
  and ask for more detail rather than guessing.
- Shrink it to the smallest input that still fails.

## 2. Pin it with a failing test

Write the regression test first, following `write-tests`, and watch it fail for the reason you
expect. Useful runs:

```bash
pytest tests/test_x.py::test_y -x -v    # one test, stop at first failure
pytest --lf -x                          # rerun last failures
pytest -k "keyword" -x                  # tests matching a name
pytest --pdb -x                         # drop into the debugger on failure
pytest -l                               # show local variables in tracebacks
```

## 3. Find the cause

- Form one hypothesis at a time, and test it with the cheapest check: a `print`, an assertion,
  or `breakpoint()` at the suspect line. Remove every probe afterwards.
- Check the inputs at the boundary where good data turns bad, and work back from there.
- Usual suspects: an off-by-one or wrong comparison, `None` passed through, a mutable default
  argument, shared mutable state, a float compared with `==`, a path relative to the wrong
  directory, an exception swallowed by a broad `except`, and a dependency version change
  (`uv tree`, `git diff uv.lock`).
- When it used to work, find the change that broke it: `git log -p -- path` or `git bisect`
  with the regression test as the check.
- For a type error from ty, read the annotation it reports before changing code; the
  annotation may be the bug.

## 4. Fix

- Fix the root cause, in the smallest change that makes the regression test pass.
- Do not silence the failure: no broad `except`, no `# noqa` or `# ty: ignore`, and no weakened
  or skipped test, without asking.
- Look for the same mistake elsewhere with `rg`, and fix or report each case.

## 5. Verify and report

Run `prek run -a --stage pre-push`. Report in a few lines: the cause (with `file.py:line`), the
fix, the test that now guards it, and any related cases found.
