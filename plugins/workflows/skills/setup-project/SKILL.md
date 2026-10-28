---
name: setup-project
description: Create or update this repository's personal workflow config - which tracker it uses (GitHub issues or Jira), which plan doc is current, any non-default check commands, and extra shipping steps. Stored under ~/.config/agent-workflows/, outside the repository, so nothing shared is touched. Use when setting up a project for the other playbooks, or when orient suggests it. Invoke as /setup-project.
---

The other playbooks read a small per-repository config to learn what they would otherwise have
to guess. This creates or updates it.

The config lives outside the repository, keyed by its GitHub `owner/repo`, so it is never
committed, works in every worktree and clone, and never touches a shared `AGENTS.md` or
`CLAUDE.md`.

## 1. Find the config file

```bash
f="$HOME/.config/agent-workflows/$(git remote get-url origin | sed -E 's#\.git$##; s#^.*[:/]([^/]+/[^/]+)$#\1#').md"; echo "$f"; cat "$f" 2>/dev/null || echo "(no config)"
```

If the repository has no `origin` remote, stop: the playbooks assume a GitHub repository.

The key follows the remote. If a repository moves to a new owner, its old config is orphaned;
if one exists under the old `owner/repo`, offer to move it to the new path.

If a config already exists, show it and ask what should change. Only re-detect the parts the
user wants to revisit.

## 2. Detect what you can

Read the project's `AGENTS.md` and `CLAUDE.md` first. Anything they already state, the
playbooks will find there, so the config only needs what they leave out or what the user wants
to override.

**Tracker.** Look at recent branch names and merged PR titles:

```bash
git branch -a --sort=-committerdate | head -30
gh pr list --state merged --limit 15 --json title,headRefName
```

Keys like `PROJ-123` mean Jira, with `PROJ` as the project key. Trailing numbers or
`Closes #N` mean GitHub issues. If they show a branch or PR title convention that differs from
the playbooks' defaults, note it.

**Plan doc.** Look for a plan or roadmap the work is tracked against:

```bash
git ls-files 'docs/*' | grep -i -E 'roadmap|plan|milestone' | head -20
```

If there is more than one candidate, ask which one is current. If there is none, the plan is
`none`.

**Checks.** Work out what the `check` playbook would run by default (its step 1). Record check
commands only if the user wants something different; otherwise leave them out so `check` keeps
detecting them and follows any changes to the project.

**Ship extras.** Ask whether anything should happen on every ship that the `ship` playbook
doesn't already do, such as updating a status doc or tagging a reviewer.

## 3. Confirm

Show the proposed config and ask the user to confirm or correct it in one pass. Don't ask about
fields that were detected unambiguously; show them and let the user object.

## 4. Write it

Create the directory if needed, then write the file. Always write `Tracker` and `Plan`, since
you always work them out. Leave out any other field that would just restate a default:

```markdown
# <owner>/<repo>

Tracker: GitHub issues | Jira, project <KEY>
Plan: <path to plan doc> | none
Branches: <naming convention, if not the playbook default>
PR titles: <title convention, if not the playbook default>
Check:
- <command>
Ship extras:
- <step>
```

Write it with the Write tool, never inside the repository.

## 5. Report

Print the file path and its contents, and say which playbooks will now use which fields:
`Tracker` and `Plan` for orient, start-work, ship, and release-notes; `Branches` for
start-work; `PR titles` and `Ship extras` for ship; `Check` for check.
