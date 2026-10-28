---
name: check
description: Run the project's type-check, lint, and test suite and report results concisely. Works across Node, Python, and Rails projects by using whatever checks the project defines. Use when you want to verify the project builds cleanly and tests pass - after making changes, before committing, or as a quick health check.
---

Run the check suite and report results.

## 0. Locate the project

Run from the repository root, wherever that is. This also does the right thing inside a git
worktree, where a hardcoded path would silently check the wrong copy:

```bash
cd "$(git rev-parse --show-toplevel)"
```

## 1. Work out what to run

Take the first of these that applies, and say which one you used:

1. **The project's own instructions.** If its `AGENTS.md` or `CLAUDE.md` says how to type-check,
   lint, or test, do exactly that.
2. **Node** (`package.json`). Run whichever of the `typecheck`, `check`, `lint`, and `test`
   scripts exist, in that order. Use the package manager the lockfile names: `pnpm-lock.yaml`
   means pnpm, `yarn.lock` means yarn, `bun.lock` or `bun.lockb` means bun, otherwise npm.
   Do not add runner-specific flags; Jest and Vitest accept different ones.
3. **Python** (`pyproject.toml`, `requirements.txt`, or `manage.py`). Prefix commands with
   `uv run` when there is a `uv.lock`. Run `ruff check` if ruff is a dependency, `mypy` if it is
   configured, then tests: `pytest` if it is a dependency, otherwise `python manage.py test` for
   a Django project.
4. **Ruby** (`Gemfile`). Run `bundle exec rubocop` or `bundle exec standardrb`, whichever is in
   the bundle, then `bundle exec rspec` if there is a `spec/` directory, otherwise
   `bin/rails test`.
5. **Makefile.** If it has `check`, `lint`, or `test` targets, run those.

If none of these applies, ask the user what to run rather than guessing.

Run everything with `CI=1` set, so test runners do not start in watch mode or prompt.

## 2. Run it

Run each command and capture its output. Keep going after a failure so the report covers every
check, not just the first one that broke.

If the test runner cannot start at all because native dependencies are missing in a sandboxed
Linux environment, report the tests as skipped, not failed, and say they need to run locally.

## Reporting

- If everything passes: report a short "all clear", naming the checks that ran
- If anything fails: show the specific errors grouped by file, suggest fixes, and offer to apply them
- If anything was skipped, mention it briefly and say how to run it
- Don't reprint the full output if everything is clean - keep it concise
