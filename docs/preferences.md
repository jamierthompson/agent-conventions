# Standing directives

The directives an agent carries into every project, independent of stack. They govern how work is
proposed, explained, and handed back — not what the code does. A repository's own documentation
always wins over this file.

## 1. Industry-standard first — and say so when the path is left

**Strongly prefer the conventional, widely-adopted approach.** Where there's an established way the
broader community solves a problem — a standard library, a documented pattern, the framework's own
recommended path — that is the default, and a departure has to _earn_ itself against it. Novelty is
a cost, not a feature.

The second half matters more: **when the owner drifts off the standard path, name it.** A
well-trodden road that the next engineer and the next agent will recognize beats a clever bespoke
thing only its author understands. If the request reaches for something idiosyncratic and a boring
standard solution exists, push back, name the standard, and state what would be given up. Flag the
drift rather than quietly following it.

## 2. Scoped-semantic design tokens

The token architecture to hold on any UI system — the one Radix, shadcn/ui, Material, and Adobe
Spectrum converged on:

- **Three layers, in order:** **foundation** (raw primitives — the palette, the scale) →
  **semantic** (role tokens components actually read — "surface", "text-muted", "accent", "danger")
  → **brand/theme** (a scoped override of the semantic layer for a bounded region). Components only
  ever read semantic tokens; they never reach down to a primitive.
- **Scope-based isolation, not naming conventions.** A theme is applied by _scope_ — a
  `[data-theme]` / `:where()` boundary that re-binds the semantic tokens inside it — not by minting
  prefixed token names. The public contract is the generic semantic layer; there are **no
  project-prefixed token names**, because the scope provides the isolation.
- **Derive, don't enumerate.** Prefer computing tokens (contrast-solved, gamut-aware, scheme-aware)
  over hand-maintaining parallel light/dark/status palettes.

## 3. How the work comes back

- **Explain before fixing — teach as you go.** A diff on its own isn't the deliverable. State what's
  wrong, _why_ it's wrong, the fix, and the trade-off accepted. Plain language first, then the
  technical term. Answer questions thoroughly rather than assuming prior knowledge.
- **No band-aid fixes.** Treat the cause, not the symptom. Where the real fix is bigger than the
  patch, say so and let the owner decide — don't paper over it with something that will rot. If
  there's a simpler way, mention it; if there's a gotcha, flag it.
- **Verify on production, not just dev.** "Works in dev" is not done. Behavior that depends on the
  build, the deploy, caching, or real data is checked against the deployed environment before it's
  called finished.
- **Step-by-step, with approval at the seams.** Break work into small, focused tasks and show the
  plan before writing code. Complete one fully before starting the next. Size each task to one
  logical change so it maps to a single commit.
- **No magic, no hand-waving.** Anything written can be explained.

## 4. No decision-log ceremony

**Docs are current truth, edited in place. Git history is the audit trail.** No ADR folder, no
decision log, no numbered decision citations — that machinery duplicates what version control
already records and rots the moment a decision changes. When a decision changes, **edit the doc that
states it** so it always reads as present-tense truth; the _why_ and the _when_ live in the commit
that made the change. No "(updated 2026-06-30, previously X)" trails and no `decisions.md` — that is
what `git log` and `git blame` are for.

## See also

- [`agent-workflow.md`](./agent-workflow.md) — how multi-agent work is orchestrated.
- [`engineering.md`](./engineering.md) — the code discipline these directives assume.
- [`git-and-pr.md`](./git-and-pr.md) — the VCS workflow that makes git the audit trail.
- [`web-quality.md`](./web-quality.md) — the quality bars every shipped surface clears.
