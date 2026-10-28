---
name: ship
description: Complete the shipping workflow for a finished feature - run checks, sync docs, write a conventional commit, push, and open a well-formed PR with acceptance criteria and a link to its GitHub issue or Jira ticket. Use when implementation is done and ready to ship. Invoke as /ship, or /ship #187 or /ship PROJ-123 if the work item isn't obvious from the branch name.
---

Run this when implementation is complete. Covers everything from "done coding" to "PR open."

## 1. Confirm we're on a feature branch

Find the default branch, rather than assuming `main`, and use it wherever `<default>` appears
below:

```bash
git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null \
  || gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

Drop any `origin/` prefix from the result. If both fail, use `main`.

```bash
git branch --show-current
```

If on the default branch, stop and warn: "You're on <default> - run /start-work first to create a feature branch."

Check for uncommitted changes:
```bash
git status --short
git diff <default>...HEAD --stat
```

Report what's changed so far.

## 2. Determine the related work item

Derive it from the branch name: a trailing number is a GitHub issue (`feat/dark-mode-187` → `#187`),
a key like `PROJ-123` is a Jira ticket. If the project's `AGENTS.md` or `CLAUDE.md` names the
tracker, trust that over the branch name. If it's still not obvious, ask the user.

Fetch it to get AC for the PR description.

**GitHub issue:**
```bash
gh issue view <number> --json number,title,body,labels
```

**Jira ticket:** use whatever Jira access is available (an Atlassian MCP connector, `acli`, or
the `jira` CLI). If there is none, ask the user to paste the AC.

## 3. Run checks

Follow the `check` playbook. It works out what this project's type-check, lint, and tests are.

**If checks fail, stop here.** Report the failures and fix them before proceeding - do not open a PR with a broken build.

## 4. Sync docs

Bring project docs in line with the work:

- If the project keeps a plan or roadmap doc, does this work complete any unchecked `- [ ]` items? If yes, update them to `- [x]` with the issue or ticket reference.
- Does the issue or ticket have complete AC?
- Did this change make anything in the README, `AGENTS.md`, or other docs out of date?
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
feat(settings): add a dark mode toggle

Follows the system appearance by default. The toggle overrides it, and
the choice persists across launches.
```

## 6. Push

```bash
git push -u origin <branch-name>
```

## 7. Open the PR

Write the description for a reviewer who wasn't in the session. Anything that was only explained in chat needs explaining here, or leaving out.

Writing rules:
- Plain language first. Define a term the first time it appears, or avoid it.
- One idea per bullet. If a bullet needs a semicolon or a parenthetical, split it.
- Say what results mean, not just the numbers.
- Deviations from the issue go first under Decisions, and are flagged in the AC too.

```
## What this does
<A short plain-language paragraph: what the change lets the product or user do, and where it fits (what it unblocks next). If there's a flow, follow with a numbered list of what now happens.>

## Decisions worth reviewing
<One entry per decision a reviewer might question. Deviations from the issue or spec come first.>

**<The decision, in one sentence.>**
<A sentence or two on why, including what the alternative would have broken.>

## Acceptance criteria
<Copy AC from the issue and check off each satisfied item. Keep items short; explanations belong under Decisions, not in parentheses here. Leave unmet items unchecked and say why.>
- [x] ...
- [x] ...

## How it was tested
- **Automated:** <commands run, and what the tests cover>
- **Manual or real data:** <what was run, what happened, and what that shows>

## Screenshots
<If UI changed: attach screenshots or simulator captures. Omit this section otherwise.>

## Where to look
- `<path>`: <what it holds>

<Link line: see below>
```

Leave out "Decisions worth reviewing" when there were no real decisions (a typo fix, a dependency bump) rather than padding it.

End with the link to the work item:
- **GitHub issue:** `Closes #<number>`, so merging closes the issue.
- **Jira ticket:** `Jira: <KEY>`, and put the key in the PR title too, following the project's
  title convention if its instructions set one, otherwise as a prefix: `PROJ-123 feat(scope): ...`.
  Merging does not close a Jira ticket. Tell the user it will need moving unless their Jira
  automation does it.

Pass the body through a heredoc so backticks and quotes survive the shell:

```bash
gh pr create --title "<commit subject>" --body-file - <<'EOF'
<description>
EOF
```

## 8. Report

Print the PR URL and a one-line summary of what shipped.

If any new issues were identified while syncing docs (step 4), list them and ask if the user wants to file them now.
