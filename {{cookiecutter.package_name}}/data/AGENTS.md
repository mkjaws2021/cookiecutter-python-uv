# AI Agent Development Guide

This file covers the tooling, conventions, and workflows a contributor needs. Read it before
changing code or committing.

Keep it current: when you add a top-level directory or a configuration file, update the matching
section in the same change.

## AI Skills

This project ships agent skills in `.agents/skills/`, where Codex reads them. Claude Code reads
`.claude/skills/`, a symlink to `.agents/skills/`; edit skills through `.agents/skills/`. Load the
relevant one before working; do not change code without consulting it.

| Skill | Use when |
| --- | --- |
| `understand-code` | Learning how unfamiliar code works, before changing it |
| `plan-change` | Planning a multi-step feature, refactor, or fix before coding |
| `write-code` | Writing or reviewing Python |
| `write-docstrings` | Writing or reviewing Python docstrings |
| `write-tests` | Writing pytest tests |
| `write-method-docs` | Writing the mathematical methodology document |
| `debug-code` | Tracking down a bug, failing test, or traceback |
| `optimize-code` | Making measured slow or memory-heavy code faster or leaner |
| `cleanup-code` | Tidying without changing behavior |

## Build & check commands

Direnv activates the uv environment on `cd`, so the tools run without a `uv run` prefix.

```sh
ruff format .                   # format
ruff check . --fix              # lint, fixing what it can
ty check                        # type check
pytest                          # tests and doctests
prek run -a --stage pre-push    # every hook, tests included
uv add <package>                # add a dependency, after asking
```

`prek run -a --stage pre-push` is the full local gate and mirrors CI. Run it before pushing.

## When in doubt

- Check the relevant skill files in `.agents/skills/`
- Run individual tools to find the problem
- Ask the user to clarify ambiguous requirements

## Repository overview

Purpose: not yet written; ask the user and record it here

Structure:

- `{{cookiecutter.package_name}}/` - source code (this is a flat layout)
- `tests/` - test suite
- `.github/workflows/` - CI/CD (test{% if cookiecutter.publish_on_pypi == "Yes" %} + publish{% endif %})
- `.agents/skills/` - agent skills
- `.claude/` - Claude Code settings and hooks; `.claude/skills/` is a symlink to `.agents/skills/`
- `.codex/` - Codex hooks

Python Version: >={{cookiecutter.python_version}}

Key configuration files:

- `pyproject.toml` - project metadata, dependencies, all tool configuration (ruff, pytest, coverage)
- `prek.toml` - git hook configuration
- `.rumdl.toml` - markdown linting configuration
- `.editorconfig` - editor formatting settings

## Agent hooks

Claude Code (`.claude/settings.json`) and Codex (`.codex/hooks.json`) each run
`uv run --no-sync ruff format . && uv run --no-sync ruff check . --fix` after every file edit, so
edits need no manual formatting. Keep the two hooks identical. Codex runs a project hook only once
the project is trusted and the hook approved in `/hooks`.

ty is too slow for every edit. It runs on commit through prek, and the test suite on push and in
CI.

Claude Code's `attribution` setting is empty, so its commits and PRs carry no agent trailer.
Machine-local Claude Code overrides go in `.claude/settings.local.json` (gitignored).

## Commits and pull requests

- Create a branch for new work; do not develop on `master`. If another agent is working in this
  directory, offer a worktree (`git worktree add ../<repo>-<feature>`), and remove it once merged.
- Break work into commits that each make sense alone.
- Never credit an AI agent as an author or co-author on commits or PR messages. Commits or PRs
  that do will be rejected.
- Do not leave comments in the code detailing what was changed.
- Follow `.github/pull_request_template.md` for PR descriptions. Fill the sections that apply,
  delete empty sections, and pass the text to `gh pr create` with `--body-file`.

PRs are squash-merged. Merge with
`gh pr merge N --squash --subject "<title> (#N)" --body-file <file>` (don't use GitHub's default).
The body expresses the net change and why, measurement summaries, known limits, and breaking changes.
Branch history, verification, questions, and follow-ups stay in PR. Small PRs need only subject.
