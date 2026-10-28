---
name: ship
description: Complete the shipping workflow for a finished feature - run checks, prove the change works, rate its risk, review the diff with fresh context, sync docs, write a conventional commit, push, and open a well-formed PR with acceptance criteria, risk, revert notes, and a link to its GitHub issue or Jira ticket. Use when implementation is done and ready to ship. Invoke as /ship, or /ship #187 or /ship PROJ-123 if the work item isn't obvious from the branch name.
---

Run this when implementation is complete. Covers everything from "done coding" to "PR open."

## 1. Confirm we're on a feature branch

Read the project config, which the `setup-project` playbook writes:

```bash
f="$HOME/.config/agent-workflows/$(git remote get-url origin | sed -E 's#\.git$##; s#^.*[:/]([^/]+/[^/]+)$#\1#').md"; cat "$f" 2>/dev/null || echo "(no config)"
```

Its `Tracker`, `Plan`, `PR titles`, and `Ship extras` fields override the detection and defaults
below. If its `Plan` file no longer exists, say so, suggest re-running `setup-project`, and skip the
plan step rather than searching for another.

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
a key like `PROJ-123` is a Jira ticket. If the config's `Tracker` or the project's `AGENTS.md`
or `CLAUDE.md` names the tracker, trust that over the branch name. If it's still not obvious, ask the user.

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

## 4. Prove it works

Passing checks show nothing broke. They don't show the change does what the work item asks.
Produce evidence that it does, and show it rather than asserting it:

- A test that exercises the new behaviour, and its output
- A command run against the real thing (a script, an API request to a dev server), and its output
- For UI, a screenshot or simulator capture of the changed screen

If `start-work` recorded how this would be proven, do that. Keep the evidence; it goes in the PR
under "How it was tested".

If something can't be verified here (it needs production data, a device, or credentials you
don't have), say so plainly. The PR will list it as unverified rather than implying it was
checked.

## 5. Assess risk

Rate the change by what it touches, not by how big it is:

- **High:** authentication or permissions, payments or billing, data migrations or anything that
  deletes data, CI or release workflows, installers and auto-updaters, production config or
  secrets
- **Medium:** behaviour changes in shared code, public APIs, dependency upgrades
- **Low:** docs, copy, tests, isolated UI, internal refactors with test coverage

Then work out how to revert it. Usually that is reverting the merge commit. Note anything that
makes it harder: a migration that needs a reverse step, data that can't be restored, a mobile
build already in users' hands, a feature flag that can switch it off instead.

## 6. Review the diff with fresh eyes

Have the full branch diff reviewed by a reviewer that hasn't seen this session, so it reads the
code rather than remembering intent:

```bash
git diff $(git merge-base <default> HEAD)
```

- **In Claude Code:** run the `code-review` skill at `low` for a low-risk change, `medium` for
  medium, and `high` for high. For a change touching authentication, user input, or secrets, also
  run `security-review`.
- **Elsewhere:** give a separate agent session, or a subagent, only the diff, the acceptance
  criteria, and the rules below.

Rules for the reviewer and for acting on what it finds:

- Every finding cites a file and line. Treat one without a location as a guess.
- Issues the branch didn't introduce are marked **pre-existing**. Don't fix them here; they become
  follow-ups in the report.
- Fix the whole class of a confirmed bug, not just the line that was flagged.
- For a finding you reject, note why. It goes in the PR so a human reviewer doesn't raise it again.
- After fixing, re-run `check`, then re-review **only the new changes**. Don't re-raise findings
  that were fixed or rejected.
- **Stop after three rounds.** If the reviewer is still finding real problems, the approach is
  probably wrong. Stop and discuss it with the user rather than starting a fourth round.

For a high-risk change, the automated review doesn't replace a human. The PR says so (step 10).

## 7. Sync docs

Bring project docs in line with the work:

- If there is a plan doc (the config's `Plan`, or if the config has no `Plan` field, one under `docs/`), does this work complete any unchecked `- [ ]` items? If yes, update them to `- [x]` with the issue or ticket reference.
- Does the issue or ticket have complete AC?
- Did this change make anything in the README, `AGENTS.md`, or other docs out of date?
- Note any new issues surfaced that should be filed.
- Carry out any `Ship extras` the config lists.

Make any doc updates now, before the commit, so they go in the same PR.

## 8. Stage and commit

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

## 9. Push

```bash
git push -u origin <branch-name>
```

## 10. Open the PR

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
- **Proof:** <the evidence from step 4: what was run, what happened, and what that shows>
- **Unverified:** <anything that couldn't be checked, and why. Omit if none.>
- **Review:** <rounds run, what was fixed, and any finding rejected with the reason>

## Risk
<Low, Medium, or High, and what puts it there. For High: "Needs human review:" and the specific part a person should read.>

## How to revert
<Usually "Revert the merge commit." Add anything that makes it harder: a migration's reverse step, data that can't be restored, a shipped mobile build.>

## Screenshots
<If UI changed: attach screenshots or simulator captures. Omit this section otherwise.>

## Where to look
- `<path>`: <what it holds>

<Link line: see below>
```

Leave out "Decisions worth reviewing" when there were no real decisions (a typo fix, a dependency bump) rather than padding it.

End with the link to the work item:
- **GitHub issue:** `Closes #<number>`, so merging closes the issue.
- **Jira ticket:** `Jira: <KEY>`, and put the key in the PR title too, following the config's
  `PR titles` or the project's own convention if either sets one, otherwise as a prefix: `PROJ-123 feat(scope): ...`.
  Merging does not close a Jira ticket. Tell the user it will need moving unless their Jira
  automation does it.

Pass the body through a heredoc so backticks and quotes survive the shell:

```bash
gh pr create --title "<commit subject>" --body-file - <<'EOF'
<description>
EOF
```

## 11. Report

Print the PR URL, a one-line summary of what shipped, and its risk rating. For a high-risk change,
remind the user it needs a human review before merging.

If any new issues were identified while syncing docs (step 7), or the review found pre-existing
problems (step 6), list them and ask if the user wants to file them now.
