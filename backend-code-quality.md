---
name: backend-quality-review
description: Review backend, infrastructure, and architecture. Use when reviewing API, server-side, or infrastructure files.
---

# Backend Quality Review

Review backend, infrastructure, and architecture for security, performance, code quality, and design patterns.

**Scope:** Gateway/API, GraphQL resolvers, server-side logic, data layer, caching, error handling, type safety, testing, deployment. Skip `.tsx`/`.jsx` UI components, `*.generated.ts`, `node_modules/`, `.next/`, `dist/`, test files.

**Trigger:** "review backend", "architecture review", "review the API", "check security", "review server code", or when reviewing non-UI files

## Before Starting

1. **Gather the code:** Run `git diff main`, `gh pr diff <PR-number>`, or paste the code to review.
2. **Large PRs (500+ changed lines):** Run the Security pass across all files first, then complete remaining passes. Split into multiple sessions if needed.

## Review Passes

### Pass 1 — Security

- **Input validation**: Server-side validation on all mutations; no trust of client-provided values as safe
- **Auth**: Proper auth checks before data access; no public exposure of sensitive fields
- **Data exposure**: No PII/secrets in responses or logs; error messages don't leak stack traces
- **Injection**: SQL/query injection prevention; sanitized user inputs; parameterized queries
- **Rate limiting**: Missing rate limiting on mutations or expensive queries
- **Secrets**: No hardcoded API keys, credentials, or tokens in source; secrets loaded from environment variables only; nothing sensitive committed to version control

### Pass 2 — Backend Performance

- **N+1 queries**: Fetching a list then per-item queries in a loop instead of batched query or DataLoader (P1)
- **Algorithm complexity**: O(n²) or worse in hot paths; unnecessary re-computation
- **Caching**: Missing cache on expensive/repeated queries; cache invalidation correctness
- **Memory**: Unbounded data accumulation; large in-memory datasets
- **Duplicate work**: Redundant fetches or computations within a request
- **Concurrency**: Race conditions in async operations; unhandled promise rejections; missing `await`; `Promise.all` vs sequential `await` used incorrectly; shared mutable state accessed concurrently

### Pass 3 — Code Quality

- **Readability**: Unclear naming; functions doing too many things; deeply nested logic
- **SRP**: Single Responsibility — one function/module, one clear job
- **DRY**: Duplicated logic that should be abstracted (without premature abstraction)
- **Function size**: Functions longer than can be reasoned about at once
- **Error handling**: Errors caught and handled at appropriate layers; no silent failures; proper propagation; no swallowed exceptions

### Pass 4 — Architecture & Design

- **Separation of concerns**: Data layer not mixed with business logic; controllers/resolvers not doing too much; business logic not leaked into the transport layer
- **Coupling**: Tight coupling between modules that should be independent; changes ripple unexpectedly
- **Dependency injection**: Hard-coded dependencies making testing difficult; side effects not injectable
- **Anti-patterns**: Fat controllers, anemic domain models, god objects, missing abstractions for clearly repeated patterns
- **Async error chains**: Unhandled rejection in middleware chains; missing error boundary at service edges

### Pass 5 — Testing

- **Coverage**: Critical paths tested; API endpoints/resolvers have integration tests
- **Test quality**: Tests verify real behavior, not just mocks; edge cases covered; tests are not brittle
- **Test isolation**: Tests depend on shared mutable state; order-dependent tests; missing setup/teardown

### Pass 6 — Documentation

- **README**: Up to date with setup, environment variables, and architecture overview
- **API docs**: Endpoints/mutations documented with inputs, outputs, and error codes
- **Code comments**: Non-obvious logic explained; no commented-out dead code; no misleading comments

## Track Findings

ID format: `SEC-001`, `BPERF-002`, `CQ-003`, `ARCH-004`, `TEST-005`, `DOC-006`

For each issue: file, line, domain, priority (P1/P2/P3), title, detail, recommendation, handoff notes

**Priorities:**
- **P1:** User/security impact (unvalidated mutations, N+1 queries, auth bypass, data exposure, hardcoded secrets, silent data loss)
- **P2:** Quality debt (tight coupling, missing error handling, no integration tests, poor naming, async issues)
- **P3:** Polish (code style, optimization opportunities, doc improvements)

## Output Format

Generate report grouped by priority (P1/P2/P3), then domain. Include:
- Summary table by domain
- Findings with file:line + detail + recommendation
- Handoff summary + planning groupings:
  - **Security Sprint** (mark urgent)
  - **Backend Performance**
  - **Code Quality & Architecture**
  - **Testing & Documentation**

## Behavior Notes

- **Be specific:** Always include file + line
- **Consolidate patterns:** 10× same issue in one file = one finding with count
- **No hallucinations:** Only flag visible code
- **Tone:** Clinical, actionable for devs + leads + PMs

> For full-stack review, use the `complete-code-review` skill.
