---
name: check
description: Run TypeScript type-checking and tests, report results concisely. Use when you want to verify the project builds cleanly and tests pass - after making changes, before committing, or as a quick health check.
---

Run the check suite and report results.

## 0. Locate the project

Node is managed via mise - no PATH prefix needed in either environment.

Run from the repository root, wherever that is. This also does the right thing inside a git
worktree, where a hardcoded path would silently check the wrong copy:

```bash
cd "$(git rev-parse --show-toplevel)"
```

## 1. TypeScript type-check (always)

```bash
npx tsc --noEmit 2>&1
```

## 2. Tests (local only - skip in Cowork sandbox)

Jest requires macOS-native binaries from node_modules that aren't available in the Cowork/scheduled-task Linux sandbox. Only run tests when executing in Claude Code on the Mac terminal:

```bash
npm test -- --selectProjects unit --ci --silent 2>&1
```

To detect the environment: if `uname -s` returns "Linux", skip tests and note they can only run locally.

## Reporting

- If type-check passes (and tests pass or were skipped): report a short "all clear"
- If anything fails: show the specific errors grouped by file, suggest fixes, and offer to apply them
- If tests were skipped, mention it briefly (e.g., "Tests skipped - run locally with `npm test`")
- Don't reprint the full output if everything is clean - keep it concise
