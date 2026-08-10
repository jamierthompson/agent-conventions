# Bootstrapping a JS/TS repo

The wiring a repo needs _before_ feature code — the setup every other doc here assumes already
happened. Only the choices that generalize and the gotchas that cost real time; a project's own
build config still belongs to the project.

## Package manager — pnpm

**pnpm**, not npm or yarn. `pnpm add <pkg>` for production deps, `pnpm add -D` for dev deps, and
`pnpm dlx` — never `npx`. Commit the lockfile on every dependency change (see
[`web-quality.md`](./web-quality.md)).

## Formatter — wired before the first feature commit

Prettier goes in first; formatting is never done by hand. Two pieces of wiring are easy to get
wrong:

- **Spread `eslint-config-prettier` last** in the flat config, so it switches off any ESLint rule
  that would fight the formatter. On a Next.js repo `eslint-config-next` doesn't enable formatting
  rules today, which makes this future-proofing rather than a live conflict — do it anyway, because
  the failure it prevents is silent and the fix is one line:

  ```typescript
  // eslint.config.mjs
  const eslintConfig = defineConfig([
    ...nextVitals,
    ...nextTs,
    // ...project rules
    prettier, // must come last
  ]);
  ```

- **Commit `.vscode/settings.json`** with `editor.formatOnSave` and `prettier.requireConfig: true`,
  so the editor formats with the _project's_ config rather than its own defaults. Without
  `requireConfig` the editor can wrap to a different print width than `pnpm format`, and the two
  reflow the same lines back and forth on every save — a diff that churns and never converges.

`.prettierignore` covers build output, the lockfile, and generated files. On Next.js that means
`.next/`, `out/`, and `next-env.d.ts`.

## Test runner — the Vitest wiring

The stack itself is in [`engineering.md`](./engineering.md); what belongs here is the config that
isn't obvious:

- **`resolve.tsconfigPaths: true`** honors the `@/*` alias from `tsconfig.json` natively in Vite 7+,
  so test files import exactly the way application code does. Older guides reach for
  `vite-tsconfig-paths` — it is now redundant and warns when detected.
- **A `tests/setup.ts`** importing `@testing-library/jest-dom/vitest`, wired through
  `test.setupFiles`, is what makes the Testing Library matchers available.
- Scripts expose both modes: `vitest run` for CI, `vitest` for watch.

## See also

- [`engineering.md`](./engineering.md) — the code discipline this setup serves.
- [`git-and-pr.md`](./git-and-pr.md) — the gate these scripts feed.
- [`web-quality.md`](./web-quality.md) — lockfile and dependency hygiene.

> For Next.js behavior — routing, Server vs. Client Components, caching, file conventions — read the
> version-exact docs the framework installs under `node_modules/next/dist/docs/`. That is the
> authority, and a hand-written copy drifts from it.
