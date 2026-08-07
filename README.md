# agent-conventions

The conventions, subagent definitions, and skills I load into every coding-agent session. This is
the operational core of how I direct agents — portable across repos and, for the documents
themselves, across providers.

## What's here

| Directory            | Contents                                                                                                                                                           |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`docs/`](docs/)     | Five convention docs — the workflow, standing directives, engineering discipline, git/PR workflow, and web quality bars. Tool-neutral markdown; the portable core. |
| [`agents/`](agents/) | Three process subagents in Claude Code's format: `adversarial-qa`, `architect`, `devils-advocate`.                                                                 |
| [`skills/`](skills/) | The `agent-team` orchestration skill — how a lead runs multiple coding agents against one task.                                                                    |

Start with [`docs/agent-workflow.md`](docs/agent-workflow.md); everything else is downstream of it.

## How it's wired in

This checkout is the single source of truth — referenced, never copied.

**Claude Code.** `~/.claude/CLAUDE.md` imports the convention docs by path
(`@~/dev/agent-conventions/docs/…`), and `~/.claude/agents` and `~/.claude/skills` are symlinks into
this repo, which makes the subagents and skills available in every session.

**Other tools.** Any tool that reads `AGENTS.md` (Codex, Cursor, Gemini CLI, Zed, …) can consume the
same docs: point a project's `AGENTS.md` — or the tool's global config file — at `docs/`. The
`agents/` and `skills/` formats are Claude Code adapters; nothing in `docs/` assumes a provider.

## The rule that governs the rest

**A repository's own documentation always wins over these defaults.** These files hold only what
generalizes; anything stack- or project-specific belongs to each project.

## How it changes

Edited in place as rules earn their way in or out — a rule enters when a real failure produces it,
and git history is the audit trail. There is no decision log.

Two known gaps, in the open: every rule here is held by discipline rather than enforced by hooks,
and the system measures itself only ad hoc (one skill has been validated by eval; the rest haven't).

## License

[MIT](LICENSE)
