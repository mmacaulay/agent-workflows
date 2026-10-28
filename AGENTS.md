# Agent workflows

Personal development playbooks, for any kind of project, plus a few specific to Expo apps. Every
playbook is plain Markdown, so any coding agent can read one directly. Claude Code additionally
installs this repository as a plugin marketplace and surfaces them as slash commands; nothing
about the content depends on that.

## The playbooks

They come in two plugins. `workflows` is general-purpose. `expo` is for Expo apps only, and
builds on `workflows`: its playbooks call `check` and `release-notes`.

### workflows

| Playbook | Use it when |
| --- | --- |
| [`setup-project`](plugins/workflows/skills/setup-project/SKILL.md) | Setting up a project for the others. Records its tracker, plan doc, and any non-default checks in a personal config file. |
| [`orient`](plugins/workflows/skills/orient/SKILL.md) | Starting a session, or picking up after a break. Your PRs and reviews, backlog, in-flight branches. |
| [`start-work`](plugins/workflows/skills/start-work/SKILL.md) | Beginning a tracked work item. Fetches the issue or ticket, gathers context, proposes an approach, creates the branch. |
| [`check`](plugins/workflows/skills/check/SKILL.md) | Verifying the project type-checks, lints, and tests cleanly, in Node, Python, or Rails. |
| [`ship`](plugins/workflows/skills/ship/SKILL.md) | Implementation is done. Runs checks, writes the commit, opens a PR with acceptance criteria. |
| [`release-notes`](plugins/workflows/skills/release-notes/SKILL.md) | Turning merged work into a CHANGELOG entry, plus App Store "What's New" copy for apps. |

### expo

| Playbook | Use it when |
| --- | --- |
| [`release-prep`](plugins/expo/skills/release-prep/SKILL.md) | Preparing a build for TestFlight or the App Store. Version bump, tag, checks. |
| [`app-store-audit`](plugins/expo/skills/app-store-audit/SKILL.md) | Before a submission. Checks for common rejection causes. |

## What they assume

- The working directory is the repository root
- Code and pull requests live on GitHub, and `git` and the `gh` CLI are available and authenticated
- Work is tracked in GitHub issues or Jira. For Jira, the agent needs some way to read tickets,
  such as an Atlassian MCP connector or a Jira CLI, or the playbooks ask you to paste them
- For the `expo` playbooks, an Expo project with `app.json`

No absolute paths, no machine-specific configuration in the playbooks themselves. Where a
project has its own conventions (tracker, plan doc, check commands, branch names), the playbooks
look in this order:

1. A personal per-repository config at `~/.config/agent-workflows/<owner>/<repo>.md`, written by
   `setup-project`. It lives outside the repository, so it is never committed, it works in every
   worktree, and it never touches a shared `AGENTS.md`.
2. The project's own `AGENTS.md` or `CLAUDE.md`.
3. The playbook's own detection and defaults.

Where a playbook wants a project document such as a roadmap, it checks whether one exists first
and skips that step when it does not.

## Using these outside Claude Code

Read the relevant playbook and follow it. To make them visible to an agent that reads a
repository's own `AGENTS.md`, vendor this repo as a submodule and point at it:

```bash
git submodule add https://github.com/mmacaulay/agent-workflows .agent/workflows
```

Then add a line to the consuming repository's `AGENTS.md`:

```markdown
Shared workflows live in `.agent/workflows/plugins/*/skills/`. Read the relevant `SKILL.md`
before starting work, shipping a PR, or preparing a release.
```

## Using these in Claude Code

```
/plugin marketplace add mmacaulay/agent-workflows
/plugin install workflows@mmacaulay
/plugin install expo@mmacaulay
```

Install `workflows` everywhere. `expo` only matters in Expo projects, so it can be installed at
project scope there instead of globally. The playbooks then appear as slash commands (`/orient`,
`/ship`, `/release-prep`, and so on).

## What is not here

Repository-specific workflows stay in their own repositories, because they encode knowledge that
does not generalise: a particular asset pipeline, a particular backend's deploy, or a `docs/`
tree with a roadmap, a PRD, and a decisions log.

The test for belonging here is whether a playbook would still make sense in a project that has
none of those things. A playbook that only makes sense for one stack goes in that stack's plugin,
the way `expo` does.

## Publishing a change

Consumers install by version, and skills are loaded from a per-version copy in the install
cache, not from the marketplace clone. Updating the marketplace refreshes the clone but **not**
the installed copy, so no change reaches consumers until the version moves, including an edit
to a playbook's body.

Each plugin has its own version, in `plugins/<plugin>/.claude-plugin/plugin.json`. Bump the
plugin a change touches, then:

```bash
claude plugin marketplace update mmacaulay
claude plugin update <plugin>@mmacaulay   # install is a no-op once installed
```

Restart the session to load the new version. `claude plugin details` reports the marketplace's
version, not the installed one, so check `installPath` in `~/.claude/plugins/installed_plugins.json`
to confirm the upgrade landed.

## Changing a playbook

These are consumed by more than one project, so treat a change the way you would treat a change
to shared code. If an edit only makes sense for one project, it belongs in that project's own
skills directory instead.
