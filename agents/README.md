# Subagents

Reusable, **repo-agnostic** subagent definitions in Claude Code's format: YAML frontmatter (`name`,
`description`, optional `tools`) over a focused system-prompt body. They encode _process_, not
domain knowledge — so they travel across every project.

| Agent                                   | Use it when                                                                                                                                                                                                                          |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`adversarial-qa`](adversarial-qa.md)   | A coding slice is gate-green ("developer-done") but not yet reviewed. Spawn a **fresh** instance with **no prior context of the work** — it tries to break the slice and writes the missing edge/error/boundary tests before the PR. |
| [`architect`](architect.md)             | An **architecture-class** decision needs more than one mind. Spawn several in **parallel** — each drafts its own independent end-to-end proposal for the same problem, so the debate compares genuinely different designs.           |
| [`devils-advocate`](devils-advocate.md) | A front-runner proposal needs pressure-testing. It attacks the leading design — but every challenge must cite a source, and it concedes when the rebuttal is sound.                                                                  |

These compose into two loops:

- **Delivery:** build → independent QA (`adversarial-qa`) → fix → re-check → PR.
- **Decision:** research → independent drafts (`architect` × N) → adversarial debate
  (`devils-advocate`) → synthesis.

## Two things that are easy to get wrong

**`adversarial-qa` is disqualified by context, not by authorship.** "Not the agent that wrote it" is
the floor. The standard is **no prior context of the work** — a teammate that helped design, debate,
or debug the slice inherits the author's mental model _including the author's blind spot_, and will
review the code against intent it already agreed with. Brief it on the requirements and the diff
only, never on the author's reasoning. This distinction was learned the hard way.

**The decision loop is for calls that are expensive to undo** — the data model, a module boundary, a
public signature, an external contract. It runs several times the token cost of a single session,
and it is the wrong tool for product and interface design, where owner intent is the primary input
and agents drafting blind to it optimize confidently for the wrong thing. There, agents _verify_ an
emerging design rather than propose one.

## Where they live

Drop them in `~/.claude/agents/` to make them available in every repo, or in a project's
`.claude/agents/` to scope them to it. Nearest file wins, so a project's agents take precedence.

> **Repo-specific domain reviewers belong in each repo's own `.claude/agents/`**, not here. This
> folder is only for agents that encode process. If a lens turns out to be portable — no
> project-specific rules in it — it belongs here instead.
