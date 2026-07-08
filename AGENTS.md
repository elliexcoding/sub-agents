# Agent Guidance

Speak in British English.

## Project Purpose

This repository stores reusable coding subagents that can be symlinked into
tool-specific agent directories. Keep agent definitions small, specialised, and
safe to use across many software projects.

## Editing Rules

- Keep Claude Code agents in `claude-code/*.md`.
- Keep Codex agents in `codex/*.toml`.
- Prefer ASCII unless a target platform requires otherwise.
- Keep names unique within each tool family.
- Do not add project-specific paths, secrets, access tokens, or personal machine
  assumptions to agent definitions.
- Make read-only agents genuinely read-only where the platform supports it.
- When adding a new agent, update the catalogue in `README.md`.

## Quality Checks

Before handing off changes, run:

```sh
git diff --check
```

Also inspect new agent frontmatter or TOML for parseable syntax and consistent
naming.
