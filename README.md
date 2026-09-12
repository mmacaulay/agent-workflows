# claude-skills

Shared development workflows for the Maplesmith app family, used by
[perchhome](https://github.com/maplesmith/perchhome),
[aviary](https://github.com/maplesmith/aviary), and whatever comes next.

Same idea as [perch-core](https://github.com/maplesmith/perch-core), applied to process rather
than code: the workflows were duplicated per repository and drifting, so they moved somewhere
versioned that every app consumes.

**Start at [AGENTS.md](AGENTS.md)** for the playbook index and usage.

## Layout

```
.claude-plugin/     Claude Code plugin and marketplace manifests
skills/<name>/      the playbooks, as plain Markdown
AGENTS.md           index, readable by any coding agent
```

The playbooks are the substance. The manifests are a thin wrapper that lets Claude Code install
them as slash commands; the same files work when read directly by another agent, which is why
they carry no absolute paths and no assumptions about a particular app.

## Naming

The repository is called `claude-skills` for historical reasons. The content is not
Claude-specific and `skills/` is deliberately agent-neutral.
