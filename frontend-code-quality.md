---
name: frontend-quality-review
description: Review frontend code for accessibility, TypeScript, performance, error handling, and stack-specific patterns (Next.js, Sanity, GraphQL). Use when reviewing .tsx/.ts/.jsx/.js/.css files.
---

# Frontend Quality Review

Review frontend code for accessibility, TypeScript, performance, error handling, and stack-specific patterns (Next.js, Sanity, GraphQL).

**Scope:** `.tsx`, `.ts`, `.jsx`, `.js`, `.html`, `.css`/`.scss` (skip tests, mocks, `*.generated.ts`, `node_modules/`, `.next/`, `dist/`)

**Trigger:** "review my branch/PR", "audit frontend", "check performance/accessibility"

## Review Passes

### Pass 1 — Accessibility

- **Semantics**: Landmark elements (`<main>`, `<nav>`, etc.), native button for `onClick`, proper heading hierarchy, `<ul>`/`<ol>` for lists
- **ARIA & Labels**: `aria-label` on unlabeled buttons; `<label htmlFor>` + `id` on inputs (not placeholder); no redundant aria; `aria-required`/`aria-invalid` on forms
- **Images**: `alt` on all `<img>`, `alt=""` for decorative; SVG: `aria-hidden="true"` if decorative, `aria-label` if meaningful
- **Keyboard**: `tabIndex` only 0 or -1; `onKeyDown` if `onClick` on div; focus management on modals

### Pass 2 — TypeScript Hygiene

- **`any` overuse**: Explicit `any`, `as any` casts, `any[]` with knowable shape, `Record<string, any>` — especially in shared utils, hooks, API layer
- **Unsafe patterns**: `@ts-ignore`/`@ts-expect-error` without comments; `as` casts on unverified data (APIs, `JSON.parse`); missing return types on exports; `unknown` cast without type guard
- **Skip**: Local script with `// TODO: type this` comment

### Pass 3 — Error & Loading States

- **Missing states**: Data fetch with no loading state; API with no error handling; no form submission feedback; silent failures
- **Retry logic**: Missing retry on failures; no exponential backoff; no max retry limit
- **Error UX**: Renders nothing on error (blank); exposes raw errors/stack traces; no ErrorBoundary on dynamic components
- **Cleanup**: `useEffect` fetch without abort controller (memory leak); optimistic UI without rollback

### Pass 4 — Performance (CWV)

- **CLS**: Img/video missing `width`/`height` or aspect-ratio; Next.js `<Image>` missing `width`/`height`/`fill`; async content (ads, embeds) no placeholder; dynamic content injected above fold; fonts missing `font-display: swap`
- **LCP**: Hero images not preloaded; `<Image>` missing `priority`; non-modern formats (WebP/AVIF); LCP in lazy component
- **INP**: Expensive sync work in handlers; re-renders on input; unthrottled scroll/input handlers; sync third-party scripts
- **Bundle**: Full imports (`import _ from 'lodash'` vs. `import debounce from 'lodash/debounce'`); no dynamic imports on large components; missing `prefetch` on `<Link>`; sync third-party script loading

### Pass 5 — Lodestone Stack

**Next.js patterns:**
- `next/dynamic` not used for route-level or large components (should lazy-load above-the-fold)
- `getStaticProps`/`getServerSideProps` doing async work without proper error handling (propagates to page render)
- Revalidation tokens (`SANITY_REVALIDATE_SECRET`) hardcoded or logged; ISR `revalidate` time too high or too low

**Sanity (GROQ) queries:**
- Fetching entire documents when only specific fields needed — always use field projection: `{ title, slug }` not `{ ... }`
- No field projection on nested objects (e.g. `author` returned with all fields instead of `author->{name, _id}`)
- Recursive projection (`...`) fetching unneeded nested data — be explicit about depth
- Queries against `production` dataset without proper access control in client-side code

**GraphQL patterns:**
- **Mutations**: Accepting user input without server-side validation; trusting client selection set/variables as safe (P1)
- **Over-fetching**: Selection set doesn't match what component renders; fetching convenience fields not used (P2)
- **N+1 queries**: Fetching a list then per-item queries in a loop instead of batched query or DataLoader (P1)
- **Untyped responses**: GraphQL query lacks generated types (`src/generated/`) — use `pnpm codegen` (P2)

## Track Findings

ID format: `A11Y-001`, `TS-002`, `ERR-003`, `PERF-004`, `STACK-005`

For each issue: file, line, domain, priority (P1/P2/P3), title, detail, recommendation, handoff notes

**Priorities:**
- **P1:** User/security impact (broken keyboard nav, unvalidated mutations, N+1 queries, silent loss, CLS > 0.1, LCP blocker)
- **P2:** Quality debt (`any` in shared code, missing retry, over-fetching, no ErrorBoundary, INP issues)
- **P3:** Polish (decorative alt, bundle optimization, GROQ cleanup)

## Output Format

Generate report grouped by priority (P1/P2/P3), then domain. Include:
- Summary table by domain
- Findings with file:line + detail + recommendation
- Handoff summary + planning groupings (Accessibility Sprint, TypeScript Cleanup, Resilience Pass, Performance Sprint, Stack-Specific)

## Behavior Notes

- **Be specific:** Always include file + line
- **Consolidate patterns:** 10× same issue in one file = one finding with count
- **No hallucinations:** Only flag visible code
- **Skip legitimate `any`:** Type utilities genuinely requiring `any` are OK
- **Tone:** Clinical, actionable for devs + leads + PMs