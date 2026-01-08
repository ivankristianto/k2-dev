---
name: k2:report
description: Generate status report for a beads ticket with summary, progress, and details
argument-hint: beads-123
allowed-tools:
  - Bash
  - Read
---

# K2:Report - Ticket Status Report

Generate a comprehensive structured markdown report for a beads ticket showing status, progress, assignee, comments, and related information.

## What This Command Does

Creates a detailed status report by:
1. Fetching ticket information from beads
2. Reading task description, status, and metadata
3. Gathering comments and history
4. Finding related tickets (dependencies, blockers)
5. Formatting as structured markdown report

## How to Use This Command

**Step 1: Parse Ticket Argument**
- Expect single ticket ID: `beads-123`
- If no argument provided, ask user: "Which ticket would you like a report for?"

**Step 2: Fetch Ticket Information**
- Use `bd show {ticket-id}` to get full ticket details
- Parse the output to extract:
  - Title
  - Description
  - Status (open, in_progress, closed, blocked)
  - Priority
  - Created date
  - Updated date
  - Assignee (if any)

**Step 3: Get Comments**
- Use `bd comments {ticket-id}` to fetch all comments
- Include timestamp and content for each comment

**Step 4: Find Related Tickets**
- Use `bd dep list {ticket-id}` to get dependencies
- Check for blockers or blocking relationships
- List parent ticket if this is a subtask
- List child tickets if this has subtasks

**Step 5: Check Work Status**
- Check if worktree exists for this ticket
- Use `git worktree list | grep "feature/beads-{ticket-id}"`
- If exists, note it in report

**Step 6: Find Related PR**
- Use `gh pr list --search "beads-{ticket-id}"` to find associated PRs
- Include PR URL, status, and review state if found

**Step 7: Generate Markdown Report**
Format the report using this structure:

```markdown
# Ticket Report: {ticket-id}

## Summary
**Title:** {title}
**Status:** {status}
**Priority:** {priority}
**Assignee:** {assignee or "Unassigned"}

## Description
{description}

## Progress
**Created:** {created_date}
**Last Updated:** {updated_date}
**Current Phase:** {infer from status and comments}

## Related Work
**Git Branch:** feature/beads-{ticket-id} {exists/does-not-exist}
**Pull Request:** {PR URL and status, or "Not created yet"}

## Dependencies
{list dependencies, blockers, parent/child relationships}

## Comments & History
{list all comments chronologically with timestamps}

## Next Steps
{infer what needs to happen next based on status}
```

## Report Structure Details

### Summary Section
- Key facts at a glance
- Current state snapshot
- Easy to scan

### Description Section
- Full task description from beads
- Preserves original formatting
- Shows complete context

### Progress Section
- Timeline information
- Current phase/stage
- Velocity indicators if applicable

### Related Work Section
- Git worktree status
- Pull request information
- Branch state

### Dependencies Section
- What this ticket depends on (blockers)
- What depends on this ticket (blocking)
- Parent/child task hierarchy
- Epic membership

### Comments & History Section
- All comments in chronological order
- Includes automated and manual comments
- Shows collaboration and decisions

### Next Steps Section
- Actionable items based on current status
- Suggested commands to run
- Who needs to take action

## Status-Based Next Steps

**If status is "open":**
- Next: Start implementation with `/k2:start {ticket-id}`

**If status is "in_progress":**
- Next: Check work branch, continue implementation, or review progress

**If status is "blocked":**
- Next: Resolve blocking dependencies listed above

**If status is "closed":**
- Next: Review completed work, verify PR is merged

## Important Notes

- **Structured Format**: Always use consistent markdown formatting
- **Comprehensive**: Include all available information
- **Actionable**: Provide clear next steps
- **Readable**: Format for easy scanning and reading

## Example Usage

```
User: /k2:report beads-123
```

Output should look like:
```markdown
# Ticket Report: beads-123

## Summary
**Title:** Add user authentication
**Status:** in_progress
**Priority:** P1
**Assignee:** engineer-agent

## Description
Implement JWT-based authentication for the API...
[full description]

## Progress
**Created:** 2026-01-05
**Last Updated:** 2026-01-08
**Current Phase:** Implementation (PR in review)

## Related Work
**Git Branch:** feature/beads-123 (exists)
**Pull Request:** #42 - Open, 2 approvals needed

## Dependencies
- **Depends on:** beads-120 (closed) - API endpoint refactoring
- **Blocks:** beads-125 (open) - Add protected routes

## Comments & History
1. [2026-01-05 10:00] Technical Lead: Created worktree and assigned to Engineer
2. [2026-01-07 14:30] Engineer: Implementation complete, PR created
3. [2026-01-08 09:15] Reviewer: First review iteration, requested changes

## Next Steps
- Engineer to address review feedback
- Reviewer to perform second review
- Maximum 1 more iteration before follow-up tickets
```

## Error Handling

**If ticket doesn't exist:**
- Show: "Error: Ticket {ticket-id} not found. Use 'bd list' to see available tickets."
- Exit

**If beads command fails:**
- Show: "Error: Unable to fetch ticket information. Ensure beads is installed and synced."
- Exit

Present the complete markdown report to the user in a clear, well-formatted way.
