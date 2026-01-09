---
name: k2:report
description: Generate status report for a beads ticket with summary, progress, and details
argument-hint: beads-123
allowed-tools:
  - Bash
  - Read
---

# K2:Report - Ticket Status Report

Generate comprehensive structured report for a beads ticket.

## Process

**1. Parse ticket ID** (ask if not provided: "Which ticket would you like a report for?")

**2. Fetch data:**
```bash
bd show {ticket-id}        # title, description, status, priority, dates, assignee
bd comments {ticket-id}    # all comments with timestamps
bd dep list {ticket-id}    # dependencies (blockers/blocking/parent/child/epic)
git worktree list | grep "feature/beads-{ticket-id}"  # worktree exists?
gh pr list --search "beads-{ticket-id}"  # find PR (URL, status, review state)
```

**3. Generate report** using format below

## Report Format

```markdown
# Ticket Report: {ticket-id}

## Summary
**Title:** {title} | **Status:** {status} | **Priority:** {priority} | **Assignee:** {assignee or "Unassigned"}

## Description
{description}

## Progress
**Created:** {created_date} | **Last Updated:** {updated_date} | **Current Phase:** {infer from status/comments}

## Related Work
**Git Branch:** feature/beads-{ticket-id} {exists/does-not-exist}
**Pull Request:** {PR URL & status or "Not created yet"}

## Dependencies
{list: depends-on (blockers), blocks (blocking), parent/child relationships, epic membership}

## Comments & History
{chronological list with timestamps - include automated & manual comments}

## Next Steps
{status-based actions:
- open: Start implementation with /k2:start {ticket-id}
- in_progress: Check work branch, continue implementation, review progress
- blocked: Resolve blocking dependencies listed above
- closed: Review completed work, verify PR merged}
```

## Error Handling

**Ticket not found:** `Error: Ticket {ticket-id} not found. Use 'bd list' to see available tickets.` → exit
**Beads command fails:** `Error: Unable to fetch ticket information. Ensure beads is installed and synced.` → exit

Present complete markdown report to user in clear, well-formatted way.
