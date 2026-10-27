---
name: start-work
description: Begin a new work item end-to-end - fetch the GitHub issue, surface relevant roadmap and PRD context, orient to the affected code, propose an implementation approach for confirmation, then pull main and create a feature branch. Use at the start of any session where a commit or PR will result. Invoke as /start-work #187 or /start-work with a task description.
---

Run this before writing any code or making any commits. Never commit directly to main.

## 1. Understand the work item

If an issue number was provided, fetch it:
```bash
gh issue view <number> --json number,title,body,labels,milestone
```

If only a task description was provided, search for a matching issue first:
```bash
gh issue list --state open --search "<key terms>" --json number,title,labels
```

Extract and note:
- The issue title and full body
- Acceptance criteria (every `- [ ]` item)
- Phase label (determines where this sits in the roadmap)
- Any referenced issues (blockers, related work)

## 2. Surface roadmap and PRD context

If the project has `docs/roadmap.md`, find the section for this issue's phase. Identify:
- Which roadmap checklist item(s) this issue addresses
- Whether there are dependencies (other unchecked items that should come first)
- Any notes in the roadmap about implementation approach or constraints

If the project keeps a PRD (`docs/prd-v1.md` or similar) and the issue touches a core feature, scan the relevant section for requirements and decisions that apply. Skip when there is no such doc; not every project keeps one.

Report what you found: "This is a Phase X item. The roadmap says [X]. The PRD says [Y]."

## 3. Orient to the affected code

Based on the issue title, labels, and AC, identify the likely affected areas. Use targeted searches - don't read everything:

```bash
# Find relevant types and components
grep -rn "<key type or feature name>" src/ --include="*.ts" --include="*.tsx" -l
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
git checkout main && git pull origin main
```

Derive the branch name from the issue title and number using this convention:

| Work type | Pattern | Example |
|-----------|---------|---------|
| New feature | `feat/<short-description>-<issue>` | `feat/dark-mode-187` |
| Bug fix | `fix/<short-description>-<issue>` | `fix/login-crash-185` |
| Doc/planning | `docs/<short-description>` | `docs/roadmap-sync` |
| Chore/config | `chore/<short-description>` | `chore/workflow-skills` |
| Refactor | `refactor/<short-description>` | `refactor/settings-store` |

```bash
git checkout -b <branch-name>
```

## 6. Confirm ready

Report:
- Branch name and base commit
- One-sentence summary of the plan
- The first concrete thing to implement

Example: "On `feat/dark-mode-187` branched from `b8a05ab`. Plan confirmed. Starting with the theme toggle in the settings screen."
