---
name: k2-dev:report
description: This skill should be used when you need to generate a comprehensive status report for a beads ticket, including summary, progress, dependencies, PR status, and next steps. Use this after workflow completion or when the user requests a ticket status report.
version: 1.0.0
---

# Report Skill - Ticket Status Report Generation

Generate comprehensive structured reports for beads tickets with complete context.

## When to Use This Skill

- After completing `/k2:start` workflow (Phase 9) to provide final status report
- When user requests ticket status via `/k2:report {ticket-id}`
- When Technical Lead needs comprehensive ticket overview
- For tracking progress across multiple tickets

## Execution Context

This skill executes in the **main conversation context**, not as an isolated agent. You have access to all tools (Bash, Read, etc.) to gather ticket information and generate reports.

## Report Generation Process

### Step 1: Parse Ticket ID

If ticket ID is provided as argument, use it. Otherwise, ask: "Which ticket would you like a report for?"

### Step 2: Fetch Data in Parallel

**PERFORMANCE OPTIMIZATION**: Run all commands simultaneously using multiple Bash tool calls in a single message:

```bash
bd show {ticket-id}        # title, description, status, priority, dates, assignee
bd comments {ticket-id}    # all comments with timestamps
bd dep list {ticket-id}    # dependencies (blockers/blocking/parent/child/epic)
git worktree list | grep -q "beads-{ticket-id}" && echo "worktree" || (git branch --list "feature/beads-{ticket-id}" | grep -q . && echo "branch" || echo "none")  # check worktree OR branch
gh pr view feature/beads-{ticket-id} --json url,state,reviewDecision 2>/dev/null || echo "No PR"  # direct PR lookup (faster than search)
```

**Performance notes:**
- All 5 commands are independent - execute in parallel for ~70-80% faster execution
- `gh pr view` is ~60% faster than `gh pr list --search` (direct lookup vs search)
- Combined git check handles both worktree and branch scenarios

### Step 3: Generate Structured Report

Format the collected data using this markdown template:

```markdown
# Ticket Report: {ticket-id}

## Summary
**Title:** {title} | **Status:** {status} | **Priority:** {priority} | **Assignee:** {assignee or "Unassigned"}

## Description
{description}

## Progress
**Created:** {created_date} | **Last Updated:** {updated_date} | **Current Phase:** {infer from status/comments}

## Related Work
**Git Branch:** feature/beads-{ticket-id} {worktree/branch/none - from combined check}
**Pull Request:** {PR URL & status from JSON or "Not created yet"}

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

### Step 4: Present Report

Present the complete markdown report to the user in a clear, well-formatted way.

## Error Handling

**Ticket not found:**
```
Error: Ticket {ticket-id} not found. Use 'bd list' to see available tickets.
```
Exit and inform user.

**Beads command fails:**
```
Error: Unable to fetch ticket information. Ensure beads is installed and synced.
```
Exit and inform user to check beads installation and run `bd sync`.

## Usage Examples

### Example 1: Report After Workflow Completion

```
Context: Technical Lead completed /k2:start workflow and reached Phase 9
Action: Invoke Skill tool with "k2-dev:report" and ticket-id
Result: Comprehensive report showing completed work, PR status, and follow-ups
```

### Example 2: User Requests Report via Command

```
Context: User runs /k2:report beads-123
Action: Command invokes this skill with ticket-id "beads-123"
Result: Full status report for the specified ticket
```

## Integration with K2-Dev Workflow

This skill is invoked:
1. **From `/k2:start` command**: At Phase 9 (after merge and cleanup) to provide final comprehensive report
2. **From `/k2:report` command**: When user explicitly requests a ticket status report
3. **From Technical Lead agent**: When generating status reports for ticket oversight

## Performance Characteristics

- **Parallel execution**: All data fetching done in parallel (~70-80% faster)
- **Fast PR lookup**: Direct `gh pr view` instead of search (~60% faster)
- **Single pass**: All data collected in one round-trip
- **Minimal overhead**: Executes in main context, no agent spawning cost

---

**Remember**: This is a skill, not an agent. Execute the logic directly in the current conversation context using available tools.
