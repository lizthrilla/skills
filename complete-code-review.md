---
name: complete-code-review
description: Full-stack code review combining backend and frontend skills in parallel. Use when asked for "complete review", "full code review", or "review everything".
---

# Complete Code Review

Run full-stack code review by executing **Backend Quality Review** and **Frontend Quality Review** in parallel on the same branch, PR, or code.

**Trigger:** "complete review", "full code review", "review everything", "run all reviews"

---

## Before Starting

1. **Gather the code:**
   - From a PR: `gh pr diff <PR-number>`
   - From a branch: `git diff main`
   - Or paste the code directly
2. **Identify what's present:** Check whether the diff contains frontend files, backend files, or both — and skip irrelevant skills (see [Single-Type Repos](#single-type-repos) below).
3. **Large PRs (500+ changed lines):** Run the Security pass across all files first, then complete remaining passes. Split into multiple sessions if needed.

---

## Execution

1. **Invoke both skills in parallel:**
   - `skill: backend-quality-review` — Security, backend performance, code quality, architecture, testing, documentation
   - `skill: frontend-quality-review` — Accessibility, security, TypeScript, component design, error handling, performance, stack-specific
2. **Consolidate findings** by domain and priority, deduplicating overlapping issues (see [Overlap Handling](#overlap-handling))
3. **Generate unified report** (see Output Format below)

---

## Single-Type Repos

If the diff contains **only backend files** (no `.tsx`/`.jsx`/`.css`): run only `skill: backend-quality-review`.

If the diff contains **only frontend files** (no server-side logic, resolvers, or infrastructure): run only `skill: frontend-quality-review`.

Run both only when the diff spans both concerns.

---

## Overlap Handling

Some concerns appear in both skills (e.g., GraphQL validation, error handling, performance). When both skills flag the same issue:

- Capture it **once** under the most relevant domain
- Note in the finding which layer it affects (frontend / backend / both)
- Do not duplicate the recommendation

---

## Track Findings

Unified ID format — IDs are assigned sequentially across both reviews (not restarted per skill):

`A11Y-001`, `SEC-002`, `TS-003`, `COMP-004`, `ERR-005`, `PERF-006`, `STACK-007`, `BPERF-008`, `CQ-009`, `ARCH-010`, `TEST-011`, `DOC-012`

For each issue: file, line, domain, priority (P1/P2/P3), detail, recommendation, handoff notes

**Priorities:**
- **P1:** User/security impact (unvalidated mutations, N+1 queries, broken keyboard nav, XSS, hardcoded secrets, silent loss, CLS > 0.1, LCP blocker)
- **P2:** Quality debt (`any` in shared code, missing retry, no ErrorBoundary, tight coupling, god components)
- **P3:** Polish (decorative alt, bundle optimization, doc improvements)

---

## Output Format

Generate unified report with:
- Summary table by domain (all 12 domains)
- P1/P2/P3 findings grouped by domain
- Handoff summary + planning groupings:
  - **Security Sprint** (mark urgent)
  - **Accessibility Sprint**
  - **TypeScript Cleanup**
  - **Frontend Performance**
  - **Resilience Pass**
  - **Stack-Specific**
  - **Backend Performance**
  - **Code Quality & Architecture**
  - **Testing & Documentation**

---

## Behavior Notes

- **Parallel execution:** Both skills run independently; findings merged by domain/priority
- **No duplication:** Overlapping concerns captured once (see Overlap Handling)
- **Security first:** P1 security findings flagged prominently, marked for urgent review
- **Be specific:** Always include file + line
- **Tone:** Clinical, actionable for devs + leads + PMs
