---
name: backend-quality-review
description: Review backend, infrastructure, and architecture for security, performance, code quality, and design patterns. Use when reviewing API, GraphQL, server-side, or infrastructure files.
---

# Backend Quality Review

Review backend, infrastructure, and architecture for security, performance, code quality, and design patterns.

**Scope:** Gateway/API, GraphQL resolvers, server-side logic, data layer, caching, error handling (server-side), type safety, testing, deployment. Skip `.tsx`/`.jsx` UI components, `*.generated.ts`, `node_modules/`, `.next/`, `dist/`.

**Trigger:** "review backend", "architecture review", "review the API", "check security", or when reviewing non-UI files

## Review Passes

### Pass 1 — Security

- **Input validation**: Server-side validation on all mutations; no trust of client selection set or variables as safe
- **Auth**: Proper auth checks before data access; no public exposure of sensitive fields
- **Data exposure**: No PII/secrets in responses or logs; error messages don't leak stack traces
- **Injection**: GROQ/SQL injection prevention; sanitized user inputs
- **Rate limiting**: Missing rate limiting on mutations or expensive queries

### Pass 2 — Backend Performance

- **N+1 queries**: Fetching a list then per-item queries in a loop instead of batched query or DataLoader (P1)
- **Algorithm complexity**: O(n²) or worse in hot paths; unnecessary re-computation
- **Caching**: Missing cache on expensive/repeated queries; cache invalidation correctness
- **Memory**: Unbounded data accumulation; large in-memory datasets
- **Duplicate work**: Redundant fetches or computations within a request

### Pass 3 — Code Quality

- **Readability**: Unclear naming; functions doing too many things; deeply nested logic
- **SRP**: Single Responsibility — one function/module, one clear job
- **DRY**: Duplicated logic that should be abstracted (without premature abstraction)
- **Function size**: Functions longer than can be reasoned about at once

### Pass 4 — Architecture & Design

- **Design patterns**: Appropriate use of patterns; no anti-patterns
- **Separation of concerns**: Data layer not mixed with business logic; resolvers not doing too much
- **Dependency injection**: Hard-coded dependencies making testing difficult
- **Error handling**: Errors caught and handled; no silent failures; proper propagation

### Pass 5 — Testing & Documentation

- **Coverage**: Critical paths tested; resolvers have integration tests
- **Test quality**: Tests verify real behavior, not mocks; edge cases covered
- **Documentation**: README up to date; API docs; meaningful comments on non-obvious code

## Track Findings

ID format: `SEC-001`, `BPERF-002`, `CQ-003`, `ARCH-004`, `TEST-005`

For each issue: file, line, domain, priority (P1/P2/P3), title, detail, recommendation, handoff notes

**Priorities:**
- **P1:** User/security impact (unvalidated mutations, N+1 queries, auth bypass, data exposure, silent data loss)
- **P2:** Quality debt (tight coupling, missing error handling, no integration tests, poor naming)
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