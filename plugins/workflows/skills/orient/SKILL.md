---
name: orient
description: Get a full situational briefing at the start of a session - your open PRs and reviews, what's next in the backlog, recent commits, and any in-progress branch work. Works with GitHub issues or Jira. Use when picking up after a break or at the start of a new session. Invoke as /orient.
---

Give a concise "where things stand" briefing. The goal is to answer: what's in flight, what's done, and what's next - in under 2 minutes.

## 0. Context

Find the default branch, rather than assuming `main`, and use it wherever `<default>` appears
below:

```bash
git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null \
  || gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

Drop any `origin/` prefix from the result. If both fail, use `main`.

Work out the tracker. If the project's `AGENTS.md` or `CLAUDE.md` names one, use it. Otherwise
look at recent branch names: keys like `PROJ-123` mean Jira, trailing numbers mean GitHub issues.
If neither tells you, assume GitHub issues and say so in the briefing.

## 1. Current branch state

```bash
git branch --show-current
git status --short
git log origin/<default>...HEAD --oneline 2>/dev/null
```

If on a feature branch with uncommitted or unpushed work, surface it prominently - that's the most actionable thing.

## 2. Pull requests

Your own open PRs, and PRs waiting on your review:

```bash
gh pr list --author @me --state open --json number,title,headRefName,isDraft,reviewDecision
gh pr list --search "review-requested:@me" --state open --json number,title,author
```

For each of your PRs, note: title, whether it's a draft, and review status. Flag any that are approved and ready to merge. List review requests separately; they are someone else's work waiting on you.

## 3. Recent activity

```bash
git log --oneline --since="7 days ago" origin/<default>
```

Summarise in 2-3 lines what was shipped recently.

## 4. What's next

**GitHub issues:**

```bash
gh issue list --state open --json number,title,labels,milestone,assignees --limit 60
```

Order by milestone, then by any priority labels the project uses. If issues are assigned, show the user's first.

**Jira:** list the user's open tickets in the current sprint, using whatever Jira access is available (an Atlassian MCP connector, `acli`, or the `jira` CLI). If there is none, say so and skip this step rather than guessing.

If the project keeps a plan or roadmap doc (for example under `docs/`), use it to decide what is current: find the section being worked through and list the open issues for it first.

For each item you list, note whether it has a linked PR or is unstarted.

## 5. Drift to flag

- If there is a plan doc, flag unchecked items in its current section that have no issue or ticket. That's work with no owner.
- Flag any open PR of yours that doesn't reference its issue: `Closes #N` for GitHub issues, or the ticket key in the branch or title for Jira.

## 6. Briefing

Deliver a structured summary:

**In flight:** [your open PRs with status]
**Waiting on you:** [review requests, if any]
**Recently shipped:** [last few commits in plain language]
**Up next:** [top 3-5 open items, in priority order]
**Flags:** [anything that needs attention - unowned plan items, unlinked PRs, stale PRs]

Keep it tight. If everything is clean, say so - a clear runway is good news.
