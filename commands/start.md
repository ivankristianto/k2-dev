---
name: k2:start
description: Start implementation workflow for one or more beads tickets
argument-hint: beads-123 or beads-123,beads-234
allowed-tools:
  - Read
  - Bash
  - Task
---

# K2:Start - Implementation Workflow

Start the implementation workflow for one or more beads tickets using the k2-dev multiagent orchestration system.

## What This Command Does

This command initiates a structured implementation workflow orchestrated by the Technical Lead agent:

1. Validates that all specified ticket(s) exist and are open
2. Creates a git worktree with branch `feature/beads-{first-ticket-id}`
3. Reads task descriptions and comments from beads
4. Hands off to Engineer agent for implementation
5. Engineer implements, self-reviews, and creates GitHub PR
6. Reviewer agent validates against quality gates
7. Iterates up to 2 times if needed, creates follow-up tickets
8. Technical Lead merges PR, closes tickets, syncs, and cleans up worktree

## How to Use This Command

**Parse the ticket argument:**
- Accepts comma-separated ticket IDs: `beads-123,beads-234,beads-345`
- Multiple tickets work together in same session and worktree
- Single ticket: `beads-123`

**Step 1: Validate Tickets**
- Use `bd show {ticket-id}` to verify each ticket exists
- Check status is "open" or "in_progress"
- If any ticket doesn't exist or is closed: show error and exit immediately
- Do NOT continue if validation fails

**Step 2: Launch Technical Lead Agent**
- Use the Task tool to launch the "technical-lead" agent
- Provide the validated ticket IDs
- Pass full context about the workflow

Example:
```
Task tool with:
- subagent_type: "technical-lead"
- prompt: "Start implementation workflow for tickets: beads-123, beads-234.
          Tickets are validated and open. Create worktree, read task details,
          and coordinate Engineer → Reviewer workflow per k2-dev process."
```

**Step 3: Report Results**
- Technical Lead will provide a final report when workflow completes
- Show the report to the user
- Include: tickets closed, PR URL, worktree status, any follow-up tickets created

## Important Notes

- **Hub Model**: Technical Lead coordinates ALL agent handoffs
- **Quality Gates**: Enforced via AGENTS.md and CLAUDE.md in project root
- **Review Limit**: Maximum 2 iterations, then create follow-up tickets (P0 for critical)
- **Branch Naming**: `feature/beads-{first-ticket-id}` (e.g., feature/beads-123)
- **Review Location**: All review comments go in GitHub PR, not beads
- **Single Worktree**: Multiple tickets use one shared worktree

## Configuration Files Required

The Technical Lead and other agents will read these files from project root:
- **AGENTS.md**: Agent guidelines, quality gates, file validation patterns
- **CLAUDE.md**: Claude-specific project standards
- **constitution.md**: Project principles and constraints

These files are REQUIRED for agents to function properly.

## Example Usage

Single ticket:
```
User: /k2:start beads-123
```

Multiple tickets:
```
User: /k2:start beads-123,beads-234,beads-345
```

## Error Handling

**If ticket doesn't exist:**
- Show: "Error: Ticket beads-123 does not exist. Please check ticket ID and try again."
- Exit immediately

**If ticket is closed:**
- Show: "Error: Ticket beads-123 is already closed. Use 'bd reopen beads-123' if you need to work on it."
- Exit immediately

**If worktree already exists:**
- Check: `git worktree list`
- If branch exists, show error and suggest cleanup: `git worktree remove path`

## Success Indicators

The workflow is complete when Technical Lead reports:
- ✅ Implementation completed
- ✅ Review passed (or follow-up tickets created)
- ✅ PR created and merged
- ✅ Beads tickets closed and synced
- ✅ Worktree removed
- ✅ Report generated

Present the final report to the user in a clear, structured format.
