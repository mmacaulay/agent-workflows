---
name: start-work
description: Begin a new work item end-to-end - fetch the GitHub issue or Jira ticket, surface relevant plan context, orient to the affected code, propose an implementation approach for confirmation, then pull the default branch and create a feature branch. Use at the start of any session where a commit or PR will result. Invoke as /start-work #187, /start-work PROJ-123, or /start-work with a task description.
---

Run this before writing any code or making any commits. Never commit directly to the default branch.

## 0. Context

Find the default branch, rather than assuming `main`, and use it wherever `<default>` appears
below:

```bash
git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null \
  || gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

Drop any `origin/` prefix from the result. If both fail, use `main`.

Work out the tracker. If the project's `AGENTS.md` or `CLAUDE.md` names one, use it. Otherwise
infer it from the work item: a key like `PROJ-123` is a Jira ticket, `#187` or a bare number is
a GitHub issue. If neither tells you, ask.

## 1. Understand the work item

**GitHub issue:** fetch it.
```bash
gh issue view <number> --json number,title,body,labels,milestone
```

If only a task description was provided, search for a matching issue first:
```bash
gh issue list --state open --search "<key terms>" --json number,title,labels
```

**Jira ticket:** fetch it with whatever Jira access is available (an Atlassian MCP connector,
`acli`, or the `jira` CLI). If there is none, ask the user to paste the ticket.

Extract and note:
- The title and full description
- Acceptance criteria (every `- [ ]` item, or the ticket's acceptance criteria field)
- Any referenced issues or tickets (blockers, related work)

## 2. Surface plan context

If the project keeps a plan or roadmap doc (for example under `docs/`), find the item(s) this
work addresses. Identify:
- Whether there are dependencies (other unchecked items that should come first)
- Any notes about implementation approach or constraints

Skip this step when there is no such doc. Report what you found in a sentence or two.

## 3. Orient to the affected code

Based on the title, labels, and AC, identify the likely affected areas. Use targeted searches - don't read everything:

```bash
git grep -n -l "<key type or feature name>"
```

Read the 2-3 most directly relevant files to understand current shape. Note:
- What exists that this work builds on
- Any patterns to follow (look at similar already-done work)
- Any gotchas or constraints visible in the code

Keep this section tight - the goal is orientation, not a full codebase analysis. If you're unsure which files are relevant, say so rather than reading broadly.

## 4. Propose an approach

Based on steps 1-3, write a brief implementation plan:
- What needs to be built (in order)
- Which files will be created or modified
- Any key decisions or trade-offs to flag before starting
- Rough test strategy (what to verify)

**Stop here and wait for the user to confirm the approach** before proceeding to step 5. If the plan needs adjusting, iterate before touching any code.

## 5. Set up the branch

Once the approach is confirmed:

```bash
git checkout <default> && git pull origin <default>
```

Derive the branch name from the title and issue number or ticket key. If the project's
instructions set a branch convention, follow that. Otherwise:

| Work type | Pattern | Example |
|-----------|---------|---------|
| New feature | `feat/<short-description>-<issue>` | `feat/dark-mode-187` |
| Bug fix | `fix/<short-description>-<issue>` | `fix/login-crash-185` |
| Doc/planning | `docs/<short-description>` | `docs/roadmap-sync` |
| Chore/config | `chore/<short-description>` | `chore/workflow-skills` |
| Refactor | `refactor/<short-description>` | `refactor/settings-store` |

For a Jira ticket, put the key first instead: `feat/PROJ-123-dark-mode`. Jira links branches,
commits, and PRs to the ticket by finding the key in their names.

```bash
git checkout -b <branch-name>
```

## 6. Confirm ready

Report:
- Branch name and base commit
- One-sentence summary of the plan
- The first concrete thing to implement

Example: "On `feat/dark-mode-187` branched from `b8a05ab`. Plan confirmed. Starting with the theme toggle in the settings screen."
