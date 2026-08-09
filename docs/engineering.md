# Engineering discipline

Portable code conventions I carry between projects. The repo's own documentation wins where it
speaks — stack-specific rules (framework caching models, CSS-layer mechanics, build config) live in
each project, not here. This file is the cross-repo discipline: TypeScript, React, code
organization, and how I test.

## TypeScript

- **No `any`.** When a type is genuinely unknown, use `unknown` and _narrow_ — `any` switches off
  the checker exactly where you most need it. (A defensive `unknown`-then- narrow is how you catch
  the bug class where bad external data flows straight through.)
- **Annotate every function signature explicitly** at module boundaries. Don't lean on inference
  where another file consumes the result — the boundary is the contract, so write it down.
- **`interface` vs `type`:** `interface` for object shapes meant to be implemented or extended;
  `type` for unions, intersections, and mapped types. Pick by intent, be consistent.
- **Type external data explicitly.** Anything crossing the wire — API responses, parsed input, env —
  gets an explicit type at the boundary; never let implicit inference define the shape of data you
  don't control.
- **Use path aliases** for imports. They survive file moves and give lint/import-boundary rules a
  stable thing to match on.

## React

- **Server Component is the default.** Reach for a Client Component (`'use client'`) only when you
  need a genuinely browser-only capability: state, effects, event handlers, browser APIs, or
  context.
- **Keep the client boundary as low in the tree as possible** — a leaf, not a layout. The
  server-rendered shell stays large and the client bundle stays small. A `'use client'` high in the
  tree drags everything below it to the client.
- **Literal dynamic imports only.** A templated `import(\`…/${x}\`)` defeats the bundler's static
  analysis and kills code-splitting; enumerate the literal imports instead.

## Code organization — establish the pattern early, instantiate late

The discipline that prevents both premature abstraction and big-ball-of-mud:

- **Name where each kind of code _will_ live — and shared-shaped code goes to its shared home on
  FIRST use.** The first button lands in the shared `ui/` directory, not beside its first consumer;
  there is no "wait for a second use" rule. What "I'll need it later" does _not_ justify is
  speculative _abstraction_ — generalizing, parameterizing, or building machinery for a hypothetical
  future caller. Put the thing in the right place immediately; design it only for the callers that
  exist.
- **Keep concerns separable before you split them.** Separation of concerns is a property you
  maintain continuously; the _file split_ is just the moment it pays off.
- **Lift state to the lowest common owner — and not before.** The trigger to lift is the moment
  you're prop-drilling the same state through two or more components.
- **One file, one concern.** One component per file, with its styles and tests co-located. A
  shared-shaped type lives in its shared home from the start — don't strand it in one module and
  make the next importer go digging.
- **Keep routing thin.** Route/entry files wire things together and mount components from the source
  tree; real logic lives in modules, not in the routing layer.
- **No magic values.** Extract a named constant for anything meaningful or used more than once.
- **Avoid broad barrel re-exports.** A catch-all `index.ts` that re-exports everything defeats
  per-route code-splitting; import from the real path.

### Naming

Components and their files PascalCase; plain modules camelCase; URL slugs, CSS selectors, and
HTML/DOM props kebab-case; types and interfaces PascalCase. Consistency over cleverness.

### Comments — rare and load-bearing

A comment earns its place only when it carries a _non-obvious why_, a real gotcha, or a citable
reason (a spec, a standard, a deliberate decision). Never restate what the code plainly does; never
leave historical or aspirational notes — that's what git history is for. If a comment would rot the
moment the code changes, it shouldn't exist.

**The test to apply, literally: would this trip up a junior dev reading the _current_ code?** If
not, delete it. Code and tests are self-documenting for the most part, so the bar is high and the
expected delete rate is too. Two patterns to cut on sight: a comment narrating what the code _used
to_ do or why it changed, and an essay-length docstring where a name would have done.

**Audit comments with a separate pass, not the author.** An author defends their own comments — the
same reason review needs fresh eyes. A read-only agent over the finished diff catches what the
writer cannot see.

## Testing philosophy

- **Vitest + React Testing Library** as the default unit/component stack.
- **Co-locate tests** next to their subject — a test file sits beside the thing it tests, one test
  file ≈ one unit of change.
- **Meaningful over exhaustive.** Test the behavior and the contract, the edges and the error paths
  — not every trivial getter for a coverage number. Query by accessible role and text the way a user
  would, not by test-ids or implementation details.
- **Dual-environment for isomorphic code.** Code meant to run on both server and client (or in both
  Node and the browser) gets tested in _both_ environments — a single-env pass hides the
  environment-specific break.
- **Playwright for the primary end-to-end flow.** One real-browser test of the main user journey
  catches what unit tests in a headless DOM structurally cannot — the things that only break once
  the whole stack is wired together and actually painting.
- **A test name says what it asserts, never who wrote it or why it exists.** No `QA edges`, no
  `defects:`, no round or reviewer in a `describe`. A reader months later should not be able to tell
  which tests arrived in a review pass — and a name describing the behavior is usually a better
  explanation than a comment above the test.
- **Don't pin what never shipped.** A review that surfaces a defect through an exotic input has done
  its job; the suite then keeps the _invariant_, not the exotica. If the input can't actually occur
  — the encoding forbids it, no call site exists, the type makes it unreachable — the input belongs
  in the commit message and the invariant belongs in the test. A suite full of impossible cases
  reads as coverage and is really archaeology.
- **Mutation-test a new suite before trusting it.** Break the implementation deliberately and
  confirm the tests scream. A suite never seen to fail is trusted on faith — and this is how you
  catch the vacuous assertion (a negative check over output never confirmed to exist) and the layer
  that can't see the bug it was written for.
