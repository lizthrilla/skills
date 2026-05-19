---
name: complete-code-review
description: Run full coverage code review by executing Backend Quality Review and Frontend Quality Review in parallel. Use when asked for "complete review", "full code review", or "review everything".
---

# Complete Code Review

Run full coverage code review by executing **Backend Quality Review** and **Frontend Quality Review** in parallel on the same branch, PR, or code.

**Trigger:** "complete review", "full code review", "review everything", "run all reviews"

---

## Overview

This wrapper orchestrates two skills in parallel:
1. **Backend Quality Review** → Security, backend performance, code quality, architecture, testing
2. **Frontend Quality Review** → Accessibility, TypeScript, performance, stack-specific (Next.js, Sanity, GraphQL)

Findings consolidated into unified report with 10 domains: Accessibility, TypeScript, Error Handling, Performance, Stack-Specific, Security, Backend Performance, Code Quality, Architecture, Testing.

---

## Execution

1. **Gather code:** `git diff`, `gh pr diff`, or pasted code
2. **Run both skills in parallel** on the same code
3. **Consolidate findings** by domain and priority
4. **Generate unified report** with ID format: `A11Y-001`, `TS-002`, `ERR-003`, `PERF-004`, `STACK-005`, `SEC-006`, `BPERF-007`, `CQ-008`, `ARCH-009`, `TEST-010`

---

## Track Findings

For each issue: file, line, domain, priority (P1/P2/P3), detail, recommendation, handoff notes

**Priorities:**
- **P1:** User/security impact (unvalidated mutations, N+1 queries, broken keyboard nav, silent loss, CLS > 0.1, LCP blocker)
- **P2:** Quality debt (`any` in shared code, missing retry, no ErrorBoundary, tight coupling)
- **P3:** Polish (decorative alt, bundle optimization, GROQ cleanup)

---

## Output Format

Generate unified report with:
- Summary table by domain (all 10)
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
- **No duplication:** Overlapping concerns (e.g., GraphQL validation, error states) captured once
- **Security first:** P1 security findings flagged prominently, marked for urgent review
- **Be specific:** Always include file + line
- **Tone:** Clinical, actionable for devs + leads + PMs