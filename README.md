# Skills Library

A personal collection of reusable agent skills for code review and quality analysis.

## Skills

| Skill | File | Description |
|-------|------|-------------|
| `backend-quality-review` | [backend-code-quality.md](./backend-code-quality.md) | Review backend, infrastructure, and architecture across 6 passes: Security, Performance, Code Quality, Architecture, Testing, and Documentation. |
| `frontend-quality-review` | [frontend-code-quality.md](./frontend-code-quality.md) | Review frontend code across 7 passes: Accessibility, Security, TypeScript, Component Design, Error Handling, Performance (CWV), and Stack-Specific Patterns. |
| `complete-code-review` | [complete-code-review.md](./complete-code-review.md) | Orchestrates both backend and frontend reviews in parallel for full-stack coverage across 12 domains. |

## Usage

These skills are designed for use with GitHub Copilot CLI and other compatible AI agent environments. Each skill file contains:

- A YAML frontmatter block with `name` and `description`
- A **Before Starting** section with instructions on how to gather code and handle large PRs
- Structured review passes with specific checklists
- Scope definitions (what to include/exclude)
- A **Track Findings** section with ID formats and priority definitions

### Invoking a Skill

In Copilot CLI, use the `skill` tool with the skill name:

```
skill: backend-quality-review
skill: frontend-quality-review
skill: complete-code-review
```

### Trigger Phrases

Each skill responds to natural language triggers:

- **Backend:** "review backend", "architecture review", "review the API", "check security", "review server code"
- **Frontend:** "review my branch/PR", "review frontend", "audit frontend", "check my components"
- **Complete:** "complete review", "full code review", "review everything"

## Skill File Format

Each skill file follows this structure:

```markdown
---
name: skill-name
description: Short invocation hint for the agent
---

# Skill Title

**Scope:** What to review / what to skip
**Trigger:** Natural language phrases that invoke this skill

## Before Starting
Steps to gather code and handle large PRs

## Review Passes
Structured checklists by domain

## Track Findings
ID format, priority definitions

## Output Format
How to structure the report

## Behavior Notes
Agent behavior rules
```
