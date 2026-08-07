# Git & PR workflow

Portable VCS discipline. The repo's own documentation wins on the specifics (exact gate commands, CI
job names, merge consequences on the host platform); this is the shape I keep everywhere.

## Never commit to `main` — in repos with a gate

Branch first. On any platform where merge-to-`main` is a production deploy, `main` must stay green
and shippable at all times — so feature work never touches it directly.

**This discipline is scoped to repos where something checks or ships `main`** — an app, a library,
anything with CI or a deploy watching. A docs-only personal repo edited in place (the conventions
repo itself is one) commits small, focused, Conventional-Commit-shaped changes directly to `main`: a
PR with no reviewer and no CI check is ceremony, and the no-ceremony rule wins. The moment such a
repo grows a gate, it graduates into the full workflow.

- **Branch name = commit type + kebab description:** `feat/short-description`,
  `fix/short-description`, `chore/short-description`. The `type` is the _same token_ as the
  Conventional-Commit type the work will land as.
- **Sync `main` into the branch right before merge** — that's the freshness that counts; conflicts
  get resolved locally, not sprung on the PR.

## Commits — Conventional Commits 1.0.0

Follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/):

```
<type>[optional scope]: <description>

[optional body — the WHY, not the what]

[optional footer(s)]
```

- **Small and focused** — each commit is _one logical change_, complete and gate-green on its own.
- **Description** in the imperative mood, lower-case, no trailing period.
- **Scope** is optional but useful in a multi-package workspace (e.g. `chore(api):`).
- **Breaking changes** signalled with `!` before the colon or a `BREAKING CHANGE:` footer.
- The **body explains why**, not what — the diff already shows what.

Even with no commitlint installed, hold the convention by discipline — it's what makes the history
(and any release automation) legible.

## The local gate mirrors CI — exactly

- Run the **full gate locally before pushing**. It runs the _same commands in the same order_ as CI;
  green locally means green in CI. **Never push "just to see if CI passes"** — the chain is
  identical, so find out locally.
- The gate must be green at **two moments**: every slice hand-off, and on the curated tip before the
  PR merges.

Before committing, review your own diff: the code runs, it's auto-formatted (never by hand), there
are no debug logs / dead code / unrelated changes, no secrets, the lockfile is committed if
dependencies changed, and imports use the project's path aliases.

## Independent adversarial QA, then the owner, before the PR

A slice does not get a PR until a **fresh agent with no prior context** has tried to break it and
written the missing tests, and the author has fixed what it found. Gate-green is _developer-done_,
not _review-done_. (Full loop in [`agent-workflow.md`](./agent-workflow.md).)

QA-passed is still not PR-ready: **the owner reads the code next, and the PR opens only after that
review.** Opening the PR is the owner's call, not the end of the agent's loop — on a human team,
teammates only ever see owner-reviewed PRs.

## Opening the PR

- **One PR = one purpose** — a coherent slice, typically one issue.
- **The title is Conventional-Commit-shaped** because it titles the merge on `main` (and lands
  _verbatim_ as the squash-commit subject when a squash is warranted).
- **The description is the durable story:** what changed, why, and how it was tested. Write it for
  the teammate who'll review it _and_ the future agent who, reading `git log`, will find only this.
  Don't use `--fill`; author the title and body deliberately.
- No checklists with unfinished items — everything's done before the PR opens.

## The lead curates history and merges

Agents deliver complete, gate-green slices; the **lead curates the history** rather than fixing the
slices:

- Rebase onto the latest `main`; optionally squash, fixup, reorder, or drop commits to tell the
  story cleanly.
- On a shared branch, push with **`--force-with-lease`, never plain `--force`** — the lease refuses
  to clobber a teammate's work you haven't seen.
- **Merge with a regular merge commit:** the curated branch lands on `main` intact — its commits
  _are_ the story. Squash only when a branch's history isn't worth keeping; then the PR's title and
  body become the single commit.
- **Delete the branch** after merge — keep the repo tidy.

## Branch protection

Where the PR workflow applies, configure the host so it can't be bypassed: require a pull request
before merging, require the CI status check to pass, and require branches to be up to date before
merge. Repos outside the PR workflow's scope (see above) stay unprotected on purpose.
