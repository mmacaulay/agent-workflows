# agent-workflows

Personal development workflows for any kind of project, plus a few specific to Expo apps. They
were duplicated per repository and drifting, so they moved somewhere versioned that every
project consumes.

**Start at [AGENTS.md](AGENTS.md)** for the playbook index and usage.

## Layout

```
.claude-plugin/marketplace.json   Claude Code marketplace manifest, listing both plugins
plugins/workflows/                general-purpose playbooks
plugins/expo/                     Expo-only playbooks, which build on workflows
plugins/<name>/skills/<skill>/    the playbooks, as plain Markdown
AGENTS.md                         index, readable by any coding agent
```

The playbooks are the substance. The manifests are a thin wrapper that lets Claude Code install
them as slash commands; the same files work when read directly by another agent, which is why
they carry no absolute paths and no assumptions about a particular project.

## Naming

Nothing here is Claude-specific. The playbooks are plain Markdown and the manifests under
`.claude-plugin/` exist only because that is where Claude Code looks; any other agent reads
`plugins/*/skills/` directly.
