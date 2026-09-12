# Agent workflows

Shared development playbooks for the Maplesmith app family. Every playbook is plain Markdown
under `skills/`, so any coding agent can read one directly. Claude Code additionally installs
this repository as a plugin and surfaces them as slash commands; nothing about the content
depends on that.

## The playbooks

| Playbook | Use it when |
| --- | --- |
| [`skills/orient/SKILL.md`](skills/orient/SKILL.md) | Starting a session, or picking up after a break. Open PRs, backlog, in-flight branches. |
| [`skills/start-work/SKILL.md`](skills/start-work/SKILL.md) | Beginning a tracked work item. Fetches the issue, gathers context, proposes an approach, creates the branch. |
| [`skills/check/SKILL.md`](skills/check/SKILL.md) | Verifying the project builds and tests cleanly. |
| [`skills/ship/SKILL.md`](skills/ship/SKILL.md) | Implementation is done. Runs checks, writes the commit, opens a PR with acceptance criteria. |
| [`skills/release-prep/SKILL.md`](skills/release-prep/SKILL.md) | Preparing a build for TestFlight or the App Store. Version bump, tag, checks. |
| [`skills/release-notes/SKILL.md`](skills/release-notes/SKILL.md) | Turning merged work into a CHANGELOG entry and App Store "What's New" copy. |
| [`skills/app-store-audit/SKILL.md`](skills/app-store-audit/SKILL.md) | Before a submission. Checks for common rejection causes. |
| [`skills/widget-scaffold/SKILL.md`](skills/widget-scaffold/SKILL.md) | Adding a new dashboard widget. Scaffolds the card, settings screen, types, and tests. |

## Using these outside Claude Code

Read the relevant playbook and follow it. They assume only:

- The working directory is the repository root
- `git` and the `gh` CLI are available and authenticated
- For the release playbooks, an Expo project with `app.json`

No absolute paths, no machine-specific configuration. Where a playbook wants a project document
such as `docs/roadmap.md`, it checks whether the file exists first and skips that step when it
does not, because not every project in this family keeps one.

To make them visible to an agent that reads a repository's own `AGENTS.md`, vendor this repo as a
submodule and point at it:

```bash
git submodule add https://github.com/maplesmith/agent-workflows .agent/workflows
```

Then add a line to the consuming repository's `AGENTS.md`:

```markdown
Shared workflows live in `.agent/workflows/skills/`. Read the relevant `SKILL.md` before
starting work, shipping a PR, or preparing a release.
```

## Using these in Claude Code

```
/plugin marketplace add maplesmith/agent-workflows
/plugin install maplesmith-workflows@maplesmith
```

The playbooks then appear as slash commands (`/orient`, `/ship`, `/release-notes`, and so on).

## What is not here

Repository-specific workflows stay in their own repositories, because they encode knowledge that
does not generalise:

- `generate-backgrounds` assumes Perch Home's asset layout and manifest generator
- `backend-deploy` assumes the Cloudflare Worker in `perch-backend`
- `issue-from-roadmap`, `sync-docs`, and `revisit-decision` assume a `docs/` tree with a roadmap,
  a PRD, and a decisions log

The test for belonging here is whether a playbook would still make sense in an app that has none
of those things.

## Changing a playbook

These are consumed by more than one project, so treat a change the way you would treat a change
to shared code. If an edit only makes sense for one app, it belongs in that app's own skills
directory instead.
