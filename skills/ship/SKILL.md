---
name: ship
description: Complete the shipping workflow for a finished feature - run checks, sync product docs, write a conventional commit, push, and open a well-formed PR with acceptance criteria and Closes reference. Use when implementation is done and ready to ship. Invoke as /ship or /ship #187 if the issue number isn't obvious from the branch name.
---

Run this when implementation is complete. Covers everything from "done coding" to "PR open."

## 1. Confirm we're on a feature branch

```bash
git branch --show-current
```

If on `main`, stop and warn: "You're on main - run /start-work first to create a feature branch."

Check for uncommitted changes:
```bash
git status --short
git diff main...HEAD --stat
```

Report what's changed so far.

## 2. Determine the related issue

Derive the issue number from the branch name (e.g., `feat/google-calendar-187` → `#187`). If not obvious, ask the user.

Fetch the issue to get AC for the PR description:
```bash
gh issue view <number> --json number,title,body,labels
```

## 3. Run checks

Run the check suite:
```bash
npx tsc --noEmit 2>&1
npm test -- --selectProjects unit --ci --silent 2>&1
```

**If checks fail, stop here.** Report the failures and fix them before proceeding - do not open a PR with a broken build.

## 4. Sync docs

Run the sync-docs workflow inline:

- If `docs/roadmap.md` exists, does this work check off any unchecked `- [ ]` items? If yes, update them to `- [x]` with the issue reference.
- Check the related GitHub issue - are the labels and phase correct? Does the issue have complete AC?
- If the project keeps a PRD, did any P0/P1 requirement get satisfied by this work?
- Note any new issues surfaced that should be filed.

Make any doc updates now, before the commit, so they go in the same PR.

## 5. Stage and commit

Review what should be staged:
```bash
git status --short
```

Stage relevant files (be explicit - don't blanket `git add .`):
```bash
git add <file1> <file2> ...
```

Write a conventional commit message:
- Format: `<type>(<scope>): <imperative subject>`
- Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`
- Subject: imperative, lowercase, ≤72 chars, no trailing period
- Body (optional): explain *why* if non-obvious

Example:
```
feat(calendar): integrate Google Calendar via GoogleSignIn SDK

On-device PKCE OAuth flow; tokens stored in SecureStore. Calendar widget
now renders Apple Calendar and Google Calendar events simultaneously.
```

## 6. Push

```bash
git push -u origin <branch-name>
```

## 7. Open the PR

Write a PR description with these sections:

```
## Summary
<2-4 bullet points: what was built and why>

## Acceptance criteria
<Copy AC from the issue, check off each item that is satisfied>
- [x] ...
- [x] ...

## Test notes
<What was tested, how to verify, any device/config requirements>

## Screenshots
<If UI changed: note that screenshots or simulator captures should be attached>

Closes #<number>
```

```bash
gh pr create --title "<commit subject>" --body "<description>"
```

## 8. Report

Print the PR URL and a one-line summary of what shipped.

If any new issues were identified during sync-docs (step 4), list them and ask if the user wants to file them now.
