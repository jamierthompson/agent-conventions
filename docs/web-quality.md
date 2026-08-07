# Web quality bars

The accessibility, performance, and hygiene targets every shipped surface clears. The repo's own
documentation owns the implementation specifics (how a given engine solves contrast, exact font
config, the build pipeline); this file is the _bars themselves_ — what "good" means, portably.

## Accessibility — WCAG 2.2 AA, with APCA as the quality layer

The conformance target is **[WCAG 2.2](https://www.w3.org/TR/WCAG22/) Level AA** — a W3C
Recommendation (Oct 2023) and the legal accessibility baseline in most regions. **APCA / Lc** is the
_perceptual quality_ target layered on top — sharper guidance, but **not legal cover** (it's a
candidate method for the unreleased WCAG 3). Cite the WCAG ratio as the standard; treat the Lc
figure as the quality goal.

### Contrast — measured, never eyeballed

- **Equal lightness difference ≠ equal contrast.** A fixed ΔL step that passes for blue can fail for
  yellow or cyan — contrast must be **measured per hue against the actual background**, not assumed
  from a uniform offset. Never hand-pick a contrast pair or a fixed ΔL and call it accessible.
- Where a system derives color (computing a contrast-solved pair against a mapped background, per
  color scheme), **consume the derived token** — don't override it with a hand-tuned value. If you
  _must_ author a static color (rare), it's then _your_ job to verify the ratio.

Targets — WCAG 2.2 AA floor / APCA Lc quality goal:

| Element                             | WCAG 2.2 AA | APCA quality  |
| ----------------------------------- | ----------- | ------------- |
| Body text                           | ≥ 4.5:1     | Lc 75         |
| Large text                          | ≥ 3:1       | Lc 45         |
| UI components, borders, focus rings | ≥ 3:1       | Lc 30         |
| Disabled / placeholder              | —           | Lc 30         |
| Incidental non-text                 | —           | Lc 15 (floor) |

> **"Large text" is measured in points, not pixels:** ≥ 18pt (≈ 24px), or 14pt bold (≈ 18.66px).
> 18*px* body text does **not** qualify — it needs the full 4.5:1.

### Focus & interaction (new in WCAG 2.2)

- Style **`:focus-visible`**, never bare `:focus`; never `outline: none` without an equivalent
  visible ring. The focus indicator needs ≥ 3:1 against adjacent colors.
- **Focus not obscured (2.4.11):** a focused element must not be fully hidden behind sticky headers,
  overlays, or other chrome.
- **Target size (2.5.8):** pointer targets are **≥ 24×24 CSS px**. Treat 24×24 as a firm floor — the
  spacing exception exists, but don't reach for it to dodge the floor.

## Performance — Core Web Vitals budgets

"Good" at the 75th percentile, mobile and desktop
([web.dev/vitals](https://web.dev/articles/vitals)):

- **LCP ≤ 2.5 s** — Largest Contentful Paint.
- **INP ≤ 200 ms** — Interaction to Next Paint (replaced FID in 2024).
- **CLS ≤ 0.1** — Cumulative Layout Shift.

Protect them at the architecture level and don't quietly undo the levers: keep the critical
(above-the-fold) query lean to protect LCP, keep themed/styled shell in the _prerendered_ HTML so it
lands flash-free, and don't introduce layout shift.

### The counter-intuitive font-preload discipline

Font preloading is a trap, because **more preloading is usually worse**. Preloading every face
floods the critical path and competes with the content that actually drives LCP. So:

- **Preload only the 1–2 shell faces** that appear above the fold on (nearly) every route.
- **Don't preload everything else.** Apply those faces, but let them load normally (tolerate
  `font-display: swap` for anything below the fold).
- **A font chosen at runtime can't be statically preloaded.** Build-time preload injection is a
  _static_ transform keyed to a statically-referenced font — if the face is selected by a runtime
  value, the toolchain bakes the resolved class into the HTML but emits _no_ preload link for a face
  it couldn't identify ahead of time. Don't fight this; design around it.
- If an above-the-fold face genuinely must preload, emit the link by hand —
  `<link rel="preload" as="font" crossorigin>` (the `crossorigin` attribute is **required** for
  fonts, even same-origin).
- **Verify empirically — check the response headers, not just the markup.** Build production, serve
  it, and inspect what the server actually sends:
  `curl -s -D - -o /dev/null <url> | grep -i '^link:'`. A toolchain may transport the preload hint
  as an HTTP `Link` response header instead of an HTML tag (Next 16 does — its served HTML carries
  _zero_ font link tags on a healthy build), so grepping the HTML alone reads a correct build as
  broken. Expect _only_ the shell faces, nothing more; **zero hints means broken** — either the face
  lost its preload flag or a proxy/CDN stripped the header in transit (a failure mode tags don't
  have).

## Secret hygiene

- **Never commit secrets** — not API keys, tokens, or credentials, not even "just to test". Secrets
  live in the environment, never in the repo.
- **The public/secret split is a tripwire.** Frameworks expose a _public_ env prefix (e.g.
  `NEXT_PUBLIC_*`, `VITE_*`) whose values are **inlined into the client bundle** — anything with
  that prefix is world-readable. Treat the prefix as a klaxon: a real secret behind a public prefix
  is a leak. Public config gets the prefix; secrets never do.
- **Commit a `.env.example`** listing every required variable with placeholder values, and confirm
  `.env*` is gitignored _before_ the first commit. The example is the contract; the real values stay
  out of git.

## Dependency hygiene

- **Commit the lockfile** on every dependency change — reproducible installs are non-negotiable.
- **Audit dependencies** for known vulnerabilities as part of the routine, not as a fire-drill.
- **Pin CI actions** to a specific version/SHA rather than a floating tag — a mutable tag is a
  supply-chain hole in the build.
