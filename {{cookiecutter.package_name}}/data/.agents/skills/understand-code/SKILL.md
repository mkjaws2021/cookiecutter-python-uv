---
name: understand-code
description: Builds an accurate picture of how a Python codebase, module, or feature works, from entry points to data flow. Use when onboarding, before changing unfamiliar code, or when asked how something works.
argument-hint: "[directory, module, or question]"
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
---

# Understanding Code

Answer how $ARGUMENTS works (or the whole package if no argument is given), from the code
itself. Read only; change nothing.

## Start from the edges

1. `pyproject.toml`: name, Python version, dependencies, `[project.scripts]` entry points, and
   tool configuration.
2. `AGENTS.md` and `README.md`: stated purpose and conventions. Treat them as claims to check
   against the code, not as fact.
3. The package's `__init__.py`: what is exported is the public API.
4. Entry points: `[project.scripts]`, `__main__.py`, and `if __name__ == "__main__":` blocks.

## Follow the code

- Map the modules: `rg --files -t py`, then each module's docstring and top-level names
  (`rg -n "^(class|def) " <package>`).
- Trace one real call from an entry point to its result. Note each function it passes through
  and what data it hands on.
- Find every caller of a name with `rg -n "\bname\b"` before saying what it is for. Callers
  define what a function is relied on to do.
- Read the tests and doctests for the code in question. They show intended inputs, outputs, and
  edge cases, often more precisely than the docs.
- Run things when reading is not enough: `python -c "..."` in the project environment, or a
  single test with `pytest tests/test_x.py::test_y -v`.
- Check history for why code looks odd: `git log -L :function_name:path/to/file.py` or
  `git log -p -- path/to/file.py`.

## What to look for

- Where state lives and who mutates it
- Where I/O happens: files, network, subprocesses, environment variables
- Where errors are raised, and where they are caught
- Invariants that the code relies on but does not check
- Code with no tests, and code with no callers

## Report

Lead with a short answer to the question asked. Then, as needed:

- **Structure:** modules and what each holds, as a short list
- **Flow:** the path of one call, step by step, citing `path/to/file.py:line`
- **Key types:** the main classes or records, and what they represent
- **Gaps and surprises:** untested code, dead code, and anything that contradicts the docs

Cite file and line for every claim about behavior. Say plainly when something is inferred
rather than read or run. Keep it to what the reader needs for their next step.

If `AGENTS.md` still says the purpose is not yet written, offer the user a one-line purpose
based on what you found.
