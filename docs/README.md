# Conventions

Portable, repo-agnostic conventions — personal defaults and cross-repo discipline, not overrides for
a project that has documented its own way of working.

**The rule that governs the rest: a repository's own documentation always wins.** When a repo
carries an `AGENTS.md`, its own `docs/`, or a `CONTRIBUTING.md`, that documentation is the source of
truth for everything it covers. These docs hold only what generalizes; stack-specific rules —
framework caching models, CSS-layer mechanics, build config, the exact gate command — belong to each
project.

| Doc                                      | What it owns                                                                                                                                       |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`agent-workflow.md`](agent-workflow.md) | **The core.** The source-of-truth ladder, the decision loop, the self-contained subagent brief, handoffs, and the independent adversarial QA loop. |
| [`preferences.md`](preferences.md)       | Standing directives: industry-standard first, design-token architecture, how work is explained and handed back, no decision-log ceremony.          |
| [`engineering.md`](engineering.md)       | TypeScript, React, code organization, and testing philosophy.                                                                                      |
| [`git-and-pr.md`](git-and-pr.md)         | Branching, Conventional Commits, the local-gate-mirrors-CI rule, and the curate-then-merge workflow.                                               |
| [`web-quality.md`](web-quality.md)       | WCAG 2.2 AA + APCA contrast, Core Web Vitals budgets, the font-preload discipline, secret and dependency hygiene.                                  |

Start with **[`agent-workflow.md`](agent-workflow.md)** — everything else is downstream of it. The
two sections that carry the most weight are _the decision loop_ and _every coding session: a lead +
independent adversarial QA_.

The worked failure cases that produced these rules live outside this repo — these docs carry only
the rules themselves.

These docs are deliberately short. State a rule once, point at the gate that enforces it, and never
restate what CI already checks — a restated rule is a second copy that can drift from the thing
enforcing it. Every editing pass should be net-negative in lines unless it's adding a rule that came
from a real failure.
