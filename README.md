# Sub-agents

Reusable software-engineering subagents for Claude Code, Codex, and adjacent
agentic coding tools.

The goal of this repository is to keep expensive main-agent context clean. Broad
file discovery, log reading, framework lookups, review passes, and focused
debugging should happen in specialised subagents that return short, cited
summaries to the parent thread.

## Repository layout

```text
.
|-- claude-code/        # Markdown subagents for ~/.claude/agents/
|-- codex/              # TOML custom agents for ~/.codex/agents/
|-- AGENTS.md           # Maintenance guidance for this repository
|-- README.md           # Installation and agent catalogue
`-- .gitignore
```

## Install

Create the tool-specific agent directories, then symlink the files you want:

```sh
mkdir -p ~/.claude/agents ~/.codex/agents
ln -sf "$PWD"/claude-code/*.md ~/.claude/agents/
ln -sf "$PWD"/codex/*.toml ~/.codex/agents/
```

For project-scoped installs, symlink into the project instead:

```sh
mkdir -p /path/to/project/.claude/agents /path/to/project/.codex/agents
ln -sf "$PWD"/claude-code/*.md /path/to/project/.claude/agents/
ln -sf "$PWD"/codex/*.toml /path/to/project/.codex/agents/
```

After creating a new agents directory, restart the relevant coding tool if it
does not detect the files immediately.

## Agent catalogue

| Purpose | Claude Code | Codex | Default posture |
| --- | --- | --- | --- |
| Cheap codebase exploration | `Explore` | `explorer` | Read-only, concise evidence gathering |
| PR and diff review | `reviewer` | `reviewer` | Read-only findings with file references |
| Test failure diagnosis | `test-diagnostician` | `test_diagnostician` | Reproduce, isolate, and report root cause |
| Security review | `security-auditor` | `security_auditor` | Read-only source-to-sink analysis |
| Documentation research | `docs-researcher` | `docs_researcher` | Verify APIs and version-sensitive behaviour |
| Scoped implementation | `implementation-worker` | `implementation_worker` | Small targeted edits after exploration |

## Recommended usage

Ask the main agent to delegate noisy work explicitly:

```text
Use the explorer agent to map the project structure and return only the files,
entry points, and commands I need for this change.
```

```text
Review this branch against main. Have explorer map the affected paths, reviewer
look for correctness risks, and test_diagnostician inspect the relevant test
surface. Wait for all results and summarise them.
```

For implementation work, let a read-only agent gather evidence first, then hand a
small scoped change to `implementation-worker`. This keeps the costly reasoning
thread focused on judgement, not on trawling through files.

## Design principles

- Prefer read-only agents for exploration, review, security, and docs research.
- Use cheaper models for high-volume scanning and log triage.
- Use higher-reasoning models only when judgement materially affects correctness,
  security, or architecture.
- Require agents to cite file paths, commands, assumptions, and uncertainty.
- Keep each agent narrow enough that the parent can predict its output shape.
- Avoid nested delegation unless the parent explicitly asks for it.

## Compatibility notes

Claude Code subagents are Markdown files with YAML frontmatter. Codex custom
agents are standalone TOML files. The two sets intentionally use matching names
where each tool permits it, but their formats are kept separate so each tool can
load them natively.
