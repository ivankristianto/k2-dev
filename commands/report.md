---
name: k2:report
description: Generate status report for a beads ticket with summary, progress, and details
argument-hint: beads-123
allowed-tools:
  - Skill
---

# K2:Report - Ticket Status Report

Generate comprehensive structured report for a beads ticket by invoking the report skill.

## Process

**1. Parse ticket ID from argument** (if not provided, ask: "Which ticket would you like a report for?")

**2. Invoke report skill:**

```
Use Skill tool:
skill: "k2-dev:report"
args: "{ticket-id}"
```

The report skill will:

- Fetch all ticket data in parallel (bd show, bd comments, bd dep list, git worktree/branch check, PR status)
- Generate a comprehensive structured markdown report
- Present it to the user with summary, progress, dependencies, comments, and next steps

## Note

This command is a thin wrapper that invokes the `k2-dev:report` skill. All report generation logic is implemented in `skills/report/SKILL.md`.
