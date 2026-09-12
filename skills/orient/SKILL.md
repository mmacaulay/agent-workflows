---
name: orient
description: Get a full situational briefing at the start of a session — open PRs, what's next in the issue backlog by phase, recent commits, and any in-progress branch work. Use when picking up after a break or at the start of a new session. Invoke as /orient.
---

Give a concise "where things stand" briefing. The goal is to answer: what's in flight, what's done, and what's next — in under 2 minutes.

## 1. Current branch state

```bash
git branch --show-current
git status --short
git log main...HEAD --oneline 2>/dev/null
```

If on a feature branch with uncommitted or unpushed work, surface it prominently — that's the most actionable thing.

## 2. Open pull requests

```bash
gh pr list --state open --json number,title,headRefName,isDraft,reviewDecision,statusCheckRollup \
  | jq '.[] | {number, title, branch: .headRefName, draft: .isDraft, review: .reviewDecision}'
```

For each open PR, note: title, whether it's a draft, and review status. Flag any that are approved and ready to merge.

## 3. Recent activity

```bash
git log --oneline --since="7 days ago" origin/main
```

Summarise in 2–3 lines what was shipped recently. This gives context for what phase the project is currently in.

## 4. What's next — open issues by phase

```bash
gh issue list --state open --json number,title,labels --limit 60 \
  | jq 'sort_by(.labels[0].name) | .[] | {number, title, phase: (.labels[] | select(.name | startswith("phase:")) | .name)}'
```

Group by phase and show only the current and next phase. Don't list Phase 4/5 items unless there's nothing else — those aren't actionable yet.

For each open issue in the current phase, note whether it has a linked PR or is unstarted.

## 5. Any doc/issue drift to flag

Quick check:
```bash
# Skip this block if the project has no docs/roadmap.md
[ -f docs/roadmap.md ] && grep -c "\- \[ \]" docs/roadmap.md
```

Count unchecked roadmap items in Phase 2 and Phase 3. If there are open items with no GitHub issue, flag them — they're work with no owner.

Check if any open PR is missing a `Closes #N` reference (PRs that won't auto-close their issue on merge).

## 6. Briefing

Deliver a structured summary:

**In flight:** [open PRs with status]
**Recently shipped:** [last few commits in plain language]
**Up next:** [top 3–5 open issues in the current phase, in priority order]
**Flags:** [anything that needs attention — drifted docs, untracked roadmap items, stale PRs]

Keep it tight. If everything is clean, say so — a clear runway is good news.
