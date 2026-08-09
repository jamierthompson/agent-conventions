# Multi-agent workflow

How I run work across one or more AI agents. Repo-agnostic; a project's own documentation wins where
it speaks. The throughline: **agents are cheap, wrong answers are expensive** — so every claim is
grounded, every slice is independently broken before it ships, and the hand-offs are written down,
not held in a context window.

## The source-of-truth ladder — cite, don't remember

Raw model memory is the **last** resort, never the first. In order:

1. **Capability before knowledge.** If a tool, skill, subagent, or MCP server already does the
   thing, use it — and invoke it _before_ writing code or searching. Look before you assume; don't
   reinvent a capability you already have.
2. **The version-exact local source.** For any framework/library behavior, read the installed docs
   and types in _this_ repo — not your training memory, which is routinely wrong about specifics and
   silently out of date.
3. **The cited web** for external standards — and **cite the source that actually contains the
   fact**, not one that merely sounds authoritative.
4. **Memory, only when nothing above applies** — and flag it as such so it can be checked.

Make "cite, don't remember" _executable_: search the installed docs before you write, don't recall.
Accuracy over confidence, every time.

## The decision loop: research → drafts → debate → synthesis

For an architecturally significant decision (hard to reverse, crosses a boundary, locks an external
contract, rests on a version-dependent fact):

1. **Research** into a dense, cited digest — every claim pinned to a primary source.
2. **N independent drafts.** Spawn several drafters with _diverse role-lenses_, each producing a
   full proposal **before seeing the others'**. Independence is the value; don't let them converge
   prematurely. (See the [`architect`](../agents/architect.md) subagent.)
3. **Adversarial debate.** Critiques must be **fact-grounded** — cite the codebase, the local docs,
   or a standard — never vibes. The critic concedes when the rebuttal is sound. (See the
   [`devils-advocate`](../agents/devils-advocate.md) subagent.)
4. **Synthesis.** Two failure modes to refuse:
   - **Don't smooth a fake consensus.** Where the drafts genuinely disagree, resolve it explicitly
     and say why — don't paper the seam.
   - **Don't add agents to fix coordination.** When the work feels tangled, the fix is a sharper
     delegation prompt, not more agents.

### A debate produces a hypothesis, not a decision

Independent drafting plus adversarial cross-examination enumerates options and kills weak arguments.
It is **powerless against a false premise every participant shares** — the lenses inherit it and
sharpen their reasoning on top of it. So: **if a load-bearing claim is testable, stop the debate and
measure it**, then feed the result back. The lead's highest-value move is finding the claim everyone
is standing on and going to check it.

When a measurement overturns a verdict, **keep the losing reasoning** rather than rewriting the
record to match the outcome. It was sound on its premises, and the next person to ask needs to know
it was already argued well and still came out wrong.

### What this loop is _not_ for

**Product and interface design stays owner-led.** Agents drafting blind to owner intent optimize
confidently for the wrong thing — there, agents _verify_ an emerging design (cited fact digests,
clickable spikes to react to, adversarial review of the chosen direction) rather than propose one.
Reserve `architect` × N + `devils-advocate` for calls that are **expensive to undo**: the data
model, a module boundary, a public signature, an external contract.

## The self-contained subagent brief

A subagent has none of your context. Every brief carries:

- **Objective** — one crisp task, not a grab-bag.
- **Source-of-truth files by path** — the exact docs/files to read; don't make it hunt.
- **Boundaries** — what's out of scope, and any binding decisions it must respect.
- **Output format** — a _dense, cited, skimmable digest_, not a raw dump. You want the conclusion
  back, not the transcript.
- **The cite-don't-remember rule, restated** — and the model tier, as an explicit cost/quality call.

> Rule of thumb: a **skill** teaches the _how_, a **hook** enforces the _rule_, a **subagent**
> isolates the _work_. Reach for the one that fits.

## Hand-offs run on external memory, not context

- **Persist progress to the repo before re-spawning.** A prompt that says "continue what we were
  doing" fails; a pointer to a written summary doesn't. External memory beats context-window
  stuffing every time.
- **One task ≈ one commit ≈ one clean hand-off.** Keep the unit of work small enough that the
  written record fully captures it.

## Every coding session: a lead + independent adversarial QA

This is the non-negotiable. Gate-green is _developer-done_, not _review-done_.

- **Staffing scales, the rule doesn't.** The floor for coding work is **lead + one coder + one fresh
  QA** — the lead briefs and curates, it does not author (that separation is what keeps it
  interruptible). Team: the lead manages N coding agents and spawns **one fresh QA per coding
  agent**. The QA pass is mandatory at every size; only the head-count moves.
- **The QA agent is fresh with _no prior context_ of the work** — not merely "not the author". A
  teammate who helped design, debate, or debug the slice is disqualified; fresh eyes are the whole
  point. (See [`adversarial-qa`](../agents/adversarial-qa.md).)
- **QA tries to break it** — think like a product-team QA engineer: attack the edges the author
  optimized past, and **write the missing edge/error/boundary test cases**.
- **The reviewer reviews; the author fixes — don't collapse the two.** QA finds and writes the
  failing case; the owning author fixes; QA re-checks. Loop until the slice survives.
- **The owner is the gate for the PR itself.** A slice that survives QA is still not a pull request.
  The owner reads the code, and the PR opens only after that review — so the workflow holds
  unchanged on a human team, where teammates only ever see owner-reviewed PRs.

## Agent teams: own a slice, isolate it, curate the history

- **Each agent owns a complete slice and is accountable for it** — a gate-green unit, not WIP handed
  up half-done.
- **Isolate in an in-root git worktree** (`git worktree add`), _not_ an ephemeral isolation flag.
  Separate checkouts make file conflicts structurally impossible, and an edit-accepting permission
  scope then covers every edit with no per-edit prompts.
- **The lead's job is history, not rescue.** The lead rebases, squashes, reorders, and drops to tell
  the story once — but does **not** inherit responsibility for an unfinished slice. Deliver it done.
- **Shared-branch hygiene:** `--force-with-lease`, never plain `--force`.
- **Fresh agents every pass, on both sides of the loop** — a new reviewer each QA round _and_ a new
  coder each fix round. A coder resumed to fix QA's findings defends the design it already chose; a
  reviewer re-checking its own findings grades its own homework. Carry state between rounds through
  written reports and commits, never by resuming a context.

### A silent agent is working, not ignoring you

An agent in an edit-accepting mode reads its inbox **only between tool calls**. While it is mid-run
it cannot see a message you sent, and it will not act on one until it next comes to rest. Two
consequences:

- **Put everything in the opening brief.** A task added mid-flight lands unreliably — the agent may
  reach its own definition of done and stop before it ever reads the addition. If you must add work,
  expect to re-send it after the agent goes idle, and verify from git that it landed.
- **Verify state before acting on an idle signal.** Idle means "at rest," not "finished" —
  duplicates arrive, messages cross, and a buffered test run looks like silence. Check the branch
  for commits and `git status` the worktree before you nudge, respawn, or tear anything down.

**Before removing a worktree, run `git status` inside that specific worktree.** Merged commits are
no evidence the directory is safe: an agent's finished-but-uncommitted work is invisible to the
branch. Treat a `git worktree remove --force` that fails with _"Directory not empty"_ as the last
warning that something is in there, not as a dependency-directory nuisance to `rm -rf` past.

## Rendered surfaces get a real browser check before "done"

Committed tests in a headless DOM don't paint and can't render async server components. Any
user-facing surface gets an agent-driven browser pass before it's called done: focus is visible and
not obscured by sticky chrome, tap targets meet the size floor, nothing shifts on load or on an
intentional font swap, the themed surface is flash-free in the initial HTML, and the console is
clean. (Quality _bars_ live in [`web-quality.md`](./web-quality.md); this is the _discipline_ of
actually driving the browser before you sign off.)

## Closing the session

Before the merge, two writes are non-negotiable:

1. **Update the README** for anything that changed — scripts, structure, conventions, status.
2. **Put the QA log in the PR body** — per coding agent: what QA probed, what passed, each defect →
   fix → re-check, and the tests added. Record each slice's entry **as its loop closes**, not
   reconstructed at the end. The QA log is the durable evidence that the dev↔QA loop actually
   happened; the green gate is not that evidence.

   There is **no separate session record**. That practice existed and was retired — a second record
   drifts out of sync with the first. Git history and the PR body are the audit trail.
