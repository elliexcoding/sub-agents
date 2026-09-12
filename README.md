# Sub-agents

Reusable software-engineering subagents for Claude Code, Codex, and adjacent
agentic coding tools.

Keep the main agent focused on planning, architectural decisions, integration,
and verification. Specialised subagents gather evidence, implement bounded
tasks, and independently review the result, returning concise evidence to the
parent. Delegate when the task justifies the handoff; complete small, clear
changes directly.

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
| PR and diff review | `reviewer` | `reviewer` | Independent read-only checks against requirements and code |
| Test failure diagnosis | `test-diagnostician` | `test_diagnostician` | Reproduce, isolate, and report root cause |
| Security review | `security-auditor` | `security_auditor` | Read-only source-to-sink analysis |
| Documentation research | `docs-researcher` | `docs_researcher` | Verify APIs and version-sensitive behaviour |
| Scoped implementation | `implementation-worker` | `implementation_worker` | Implement a written brief, validate, and escalate design surprises |
| Git housekeeping | - | `git_housekeeper` | Explicit fast-forward merges and safe worktree clean-up |

## Recommended workflow

Run the main session on Astra for Codex or Opus for Claude Code. The main agent
owns the plan and coordinates implementation; a separate planner subagent is
unnecessary when the main session already runs on that model.

1. Gather evidence with `explorer` (`Explore` in Claude Code) or
   `docs_researcher` (`docs-researcher` in Claude Code) when a separate lookup is
   useful. The main agent checks decision-critical evidence in the source.
2. Have the main agent write the intended behaviour, design decisions,
   constraints, task boundaries, dependencies, and acceptance criteria. Keep a
   multi-step plan in the target project's usual documentation location and
   update it when decisions change. A short message is enough for a small task.
3. Assign bounded tasks to `implementation_worker` (`implementation-worker` in
   Claude Code). Pass the brief below with each assignment. Parallelise only
   independent tasks with separate ownership; sequence shared interfaces and
   dependent changes. Workers retain control of routine implementation details.
4. Return architectural surprises to the main agent with evidence and the
   decision needed. The main agent revises the plan, narrows the next task, or
   handles the difficult implementation itself. Repeated unexplained failures
   need diagnosis, not more identical attempts.
5. Give a separate `reviewer` the requirements, current plan, exact diff or
   files, and validation evidence. It checks the actual code and design
   independently. The main agent triages findings and assigns corrections.
6. The main agent checks the combined changes and completes the relevant
   integration checks. Record unmet acceptance criteria and validation blockers
   explicitly before reporting the outcome.

### Worker brief

Include only the context needed for the assigned task, with references to the
current source and plan. Do not assume the worker inherited the conversation.

```text
Task and intended behaviour:
Plan reference and decisions relevant to this task:
Owned files or modules; areas outside scope:
Existing code, interfaces, and dependency evidence:
Constraints, invariants, and important edge cases:
Acceptance criteria:
Validation commands and expected outcomes:
Dependencies on other work:
Escalate to the parent if:
```

The worker reports changed files and behaviour, acceptance criteria met or still
outstanding, validation commands and results, deviations, and decisions needed.
The parent supplies a revised brief after resolving an escalation.

### Starting a session

Installing these files selects subagent behaviour and models. Select the main
session's model separately in the app, or launch a CLI session with:

```sh
codex --model gpt-6-astra --config 'model_reasoning_effort="high"'
```

```sh
claude --model opus --effort high
```

Use this prompt with your task, or adapt it into the consuming project's
`AGENTS.md` or `CLAUDE.md`:

```text
Own the planning, design decisions, integration, and final verification.
For work that benefits from delegation, gather focused evidence, write a plan,
and give bounded implementation tasks to the implementation worker using the
installed tool's agent name. Include ownership, constraints, acceptance
criteria, and validation commands. Resolve design surprises yourself and keep
the written plan current. Have a separate reviewer check the requirements and
actual changes, triage its findings, and verify the combined result. Complete
small, clear tasks directly when delegation would add unnecessary overhead.
```

## Claude Code model routing

Claude Code agents use family aliases so each provider can select its recommended
current model while preserving the intended cost and capability tier:

| Workload | Agents | Model alias | Effort |
| --- | --- | --- | --- |
| Planning, coordination, and difficult decisions | Main session | `opus` | High; raise for difficult tasks |
| High-volume exploration and documentation research | `Explore`, `docs-researcher` | `haiku` | Not set (unsupported) |
| Balanced diagnosis and scoped implementation | `test-diagnostician`, `implementation-worker` | `sonnet` | Medium |
| Quality-first review and security analysis | `reviewer`, `security-auditor` | `opus` | High |

Checked against [Claude Code model guidance](https://code.claude.com/docs/en/model-config)
on 2026-09-12: the Anthropic API aliases select Opus 5 and Sonnet 5; Haiku remains
4.5. Provider defaults and account restrictions can resolve aliases differently.
Keep aliases for portability; choose a supported full model ID when an exact
version is required. Opus 5 needs Claude Code 2.1.219 or later, and Sonnet 5
needs 2.1.197 or later.

For especially difficult coordination, Fable 5.1 is an optional main-session
upgrade where available (`claude --model claude-fable-5-1 --effort high`, requiring
Claude Code 2.1.257 or later).

The explicit `Explore` definition keeps exploration on Haiku even when the
built-in Explore would inherit a more expensive model. See the
[subagent documentation](https://code.claude.com/docs/en/sub-agents) for model
precedence and frontmatter support.

## Codex model routing

Codex uses Astra in the main session and GPT-5.6 specialists according to the
cost and judgement required by each workload:

| Workload | Agents | Model | Reasoning effort |
| --- | --- | --- | --- |
| Planning, coordination, and difficult decisions | Main session | `gpt-6-astra` | High; extra high for difficult tasks |
| High-volume exploration and documentation research | `explorer`, `docs_researcher` | `gpt-5.6-luna` | Medium |
| Mechanical Git merges and worktree clean-up | `git_housekeeper` | `gpt-5.6-luna` | Low |
| Balanced diagnosis and scoped implementation | `test_diagnostician`, `implementation_worker` | `gpt-5.6-terra` | Medium |
| Quality-first review and security analysis | `reviewer`, `security_auditor` | `gpt-5.6-sol` | High |

Checked against [OpenAI model guidance](https://learn.chatgpt.com/docs/models)
on 2026-09-12. The existing specialist model IDs remain current. Start with the
listed efforts and raise them when complexity warrants it; maximum effort is
not a blanket default. Astra can also handle difficult implementation or an
additional review when the specialist's remit is insufficient.

Each Codex role pins both `model` and `model_reasoning_effort`. Values in its
TOML take precedence over spawn overrides, so requesting Astra while selecting
the Sol `reviewer` role does not upgrade that role. Have the Astra main agent
handle the escalation, or use a separately configured role. See
[custom agent configuration](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents).

## Design principles

- Prefer read-only agents for exploration, review, security, and docs research.
- Use cheaper models for high-volume scanning and log triage.
- Use higher-reasoning models only when judgement materially affects correctness,
  security, or architecture.
- Require agents to cite file paths, commands, assumptions, and uncertainty.
- Give workers clear ownership and preserve other contributors' edits.
- Keep plans current and return architectural decisions to the main agent.
- Review against requirements and actual code, including flaws in the plan.
- Keep each agent narrow enough that the parent can predict its output shape.
- Avoid nested delegation unless the parent explicitly asks for it.

## Compatibility notes

Claude Code subagents are Markdown files with YAML frontmatter. Codex custom
agents are standalone TOML files. The two sets intentionally use matching names
where each tool permits it, but their formats are kept separate so each tool can
load them natively.
