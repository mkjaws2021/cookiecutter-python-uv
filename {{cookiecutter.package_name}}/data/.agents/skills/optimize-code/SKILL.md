---
name: optimize-code
description: Finds and fixes measured performance problems in Python (speed and memory), keeping behavior unchanged. Use when code is slow, uses too much memory, or the user asks to optimize it.
argument-hint: "[function, module, or slow command]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Edit
  - Bash
---

# Optimizing Code

Make $ARGUMENTS faster or leaner without changing what it returns. Measure first, change one
thing at a time, and keep the measurements.

## Rules

- No optimization without a measurement showing the problem, and a second one showing the gain.
- Tests must pass before and after. If the code has no tests covering the behavior, add them
  first with `write-tests`; an optimization that changes results is a bug.
- Prefer a better algorithm or data structure over micro-tuning. A change from O(n²) to O(n)
  beats any amount of inlining.
- Keep the code readable. A small gain that makes code much harder to follow is not worth it;
  report it and let the user decide.
- Do not add a dependency (numpy, numba, a profiler) without asking.

## 1. Reproduce and measure

Pick a realistic input and record a baseline with the standard library:

```bash
python -m timeit -s "from pkg.mod import func, DATA" "func(DATA)"     # small hot calls
python -m cProfile -s cumulative -m pkg.cli args | head -40          # whole program
python -X importtime -c "import pkg" 2> import-times.txt              # slow imports
```

For memory, take a `tracemalloc` snapshot around the call and print the top lines by size.

`pytest --durations=5` (already on in `pyproject.toml`) points at slow tests, which often point
at slow code.

## 2. Find the hot spot

Read the profile for the functions with the most cumulative time, then their own time. Optimize
only those. Common causes in Python:

- repeated work inside a loop: a lookup, a regex compile, an attribute chain, or a call whose
  result does not change
- `x in some_list` in a loop where a `set` or `dict` would do
- building a string or list with `+=` in a loop instead of `"".join(...)` or a comprehension
- reading a whole file or result set into memory when a generator would stream it
- recomputing a pure function's result: `functools.cache` (bound it with `lru_cache` when
  inputs vary widely)
- many small I/O calls, or I/O inside a loop that could be batched
- per-element Python loops over numeric arrays, when numpy is already a dependency

## 3. Change one thing, then re-measure

Make one change, rerun the same benchmark on the same input, and keep the change only if it
helps clearly beyond run-to-run noise. Revert anything that does not.

## 4. Verify and report

Run `prek run -a --stage pre-push`. Then report a short table:

| Change | Before | After |
| --- | --- | --- |
| `set` lookup in `match_ids` | 2.4 s | 0.08 s |

Give the input and command used, so the numbers can be reproduced. Say what was tried and did
not help, and any trade-off made, such as more memory for a cache.
