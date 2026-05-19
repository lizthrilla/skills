---
name: frontend-quality-review
description: Review frontend code for accessibility, security, TypeScript, performance, and error handling. Use when reviewing .tsx/.ts/.jsx/.js/.css files.
---

# Frontend Quality Review

Review frontend code for accessibility, security, TypeScript, performance, error handling, and stack-specific patterns.

**Scope:** `.tsx`, `.ts`, `.jsx`, `.js`, `.html`, `.css`/`.scss` (skip tests, mocks, `*.generated.ts`, `node_modules/`, `.next/`, `dist/`)

**Trigger:** "review my branch/PR", "review frontend", "audit frontend", "check my components", "check performance/accessibility"

## Before Starting

1. **Gather the code:** Run `git diff main`, `gh pr diff <PR-number>`, or paste the code to review.
2. **Large PRs (500+ changed lines):** Run Security and Accessibility passes first, then complete remaining passes. Split into multiple sessions if needed.

## Review Passes

### Pass 1 — Accessibility

- **Semantics**: Landmark elements (`<main>`, `<nav>`, etc.), native button for `onClick`, proper heading hierarchy, `<ul>`/`<ol>` for lists
- **ARIA & Labels**: `aria-label` on unlabeled buttons; `<label htmlFor>` + `id` on inputs (not placeholder); no redundant aria; `aria-required`/`aria-invalid` on forms
- **Images**: `alt` on all `<img>`, `alt=""` for decorative; SVG: `aria-hidden="true"` if decorative, `aria-label` if meaningful
- **Keyboard**: `tabIndex` only 0 or -1; `onKeyDown` if `onClick` on div; focus management on modals

### Pass 2 — Security

- **XSS**: `dangerouslySetInnerHTML` used without sanitization; user-controlled content rendered as raw HTML
- **Secrets**: Client-side code containing API keys, tokens, or credentials; environment variables that should be server-only exposed to the browser
- **Auth**: Sensitive routes or UI sections protected only by client-side checks (bypassable); no server-side guard
- **Third-party scripts**: Unverified third-party scripts loaded without Subresource Integrity (SRI); sync blocking scripts
- **CSP**: No Content Security Policy; `unsafe-inline` in `script-src`

### Pass 3 — TypeScript Hygiene

- **`any` overuse**: Explicit `any`, `as any` casts, `any[]` with knowable shape, `Record<string, any>` — especially in shared utils, hooks, API layer
- **Unsafe patterns**: `@ts-ignore`/`@ts-expect-error` without comments; `as` casts on unverified data (APIs, `JSON.parse`); missing return types on exports; `unknown` cast without type guard
- **Skip**: Local script with `// TODO: type this` comment; type utilities that genuinely require `any`

### Pass 4 — Component Design

- **God components**: Components exceeding ~200 lines or accepting 8+ props; should be split into smaller pieces
- **Prop drilling**: Props passed more than 2–3 levels deep without a clear reason; consider Context, composition, or co-location
- **State colocation**: State lifted higher than necessary causing unnecessary re-renders; global state used for local concerns
- **Side effects in render**: Business logic inside component body instead of hooks or services
- **Naming**: Vague names (`Component`, `Handler`, `Wrapper`, `Util`) that obscure intent

### Pass 5 — Error & Loading States

- **Missing states**: Data fetch with no loading state; API with no error handling; no form submission feedback; silent failures
- **Retry logic**: Missing retry on failures; no exponential backoff; no max retry limit
- **Error UX**: Renders nothing on error (blank screen); exposes raw errors/stack traces to users; no `ErrorBoundary` on dynamic components
- **Cleanup**: `useEffect` fetch without abort controller (memory leak); optimistic UI without rollback

### Pass 6 — Performance (CWV)

- **CLS**: Img/video missing `width`/`height` or aspect-ratio; async content (ads, embeds) no placeholder; dynamic content injected above fold; fonts missing `font-display: swap`
- **LCP**: Hero images not preloaded; non-modern formats (WebP/AVIF); LCP element inside lazy-loaded component
- **INP**: Expensive sync work in event handlers; unthrottled scroll/input handlers; re-renders on every keystroke; sync third-party scripts
- **Bundle**: Full library imports (`import _ from 'lodash'` vs `import debounce from 'lodash/debounce'`); no dynamic imports on large components; sync third-party script loading

### Pass 7 — Stack-Specific Patterns

Apply checks relevant to your stack. Skip sections that don't apply.

**Next.js patterns:**
- `next/dynamic` not used for route-level or large components
- `getStaticProps`/`getServerSideProps` doing async work without proper error handling
- Revalidation secrets hardcoded or logged; ISR `revalidate` time not tuned to data freshness requirements

**GraphQL patterns:**
- **Mutations**: Accepting user input without server-side validation; trusting client selection set/variables as safe (P1)
- **Over-fetching**: Selection set doesn't match what component renders; fetching fields not used (P2)
- **N+1 queries**: Fetching a list then per-item queries in a loop instead of batched query or DataLoader (P1)
- **Untyped responses**: GraphQL query lacks generated types — run your schema codegen tool (P2)

**CMS / data fetching:**
- Fetching entire documents when only specific fields are needed — use field projection
- No projection on nested objects (returning full nested records when only a few fields are used)
- Queries against production data sources from client-side code without proper access control

## Track Findings

ID format: `A11Y-001`, `SEC-002`, `TS-003`, `COMP-004`, `ERR-005`, `PERF-006`, `STACK-007`

For each issue: file, line, domain, priority (P1/P2/P3), title, detail, recommendation, handoff notes

**Priorities:**
- **P1:** User/security impact (broken keyboard nav, XSS, unvalidated mutations, N+1 queries, silent loss, CLS > 0.1, LCP blocker)
- **P2:** Quality debt (`any` in shared code, missing retry, over-fetching, no ErrorBoundary, INP issues, god components)
- **P3:** Polish (decorative alt, bundle optimization, naming)

## Output Format

Generate report grouped by priority (P1/P2/P3), then domain. Include:
- Summary table by domain
- Findings with file:line + detail + recommendation
- Handoff summary + planning groupings (Security Sprint, Accessibility Sprint, TypeScript Cleanup, Resilience Pass, Performance Sprint, Stack-Specific)

## Behavior Notes

- **Be specific:** Always include file + line
- **Consolidate patterns:** 10× same issue in one file = one finding with count
- **No hallucinations:** Only flag visible code
- **Tone:** Clinical, actionable for devs + leads + PMs

> For full-stack review, use the `complete-code-review` skill.
