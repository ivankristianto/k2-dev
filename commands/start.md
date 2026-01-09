---
name: k2:start
description: Start implementation workflow for one or more beads tickets (runs Technical Lead orchestration logic directly)
argument-hint: "[beads-123] or beads-123,beads-234 (optional - auto-selects if omitted)"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - TodoWrite
  - Task
  - AskUserQuestion
---

# K2:Start - Implementation Workflow

Run Technical Lead orchestration logic directly to coordinate the full implementation lifecycle: validation → worktree → implementation → review → merge → cleanup.

**CRITICAL:** Execute logic directly in this context. DO NOT launch "technical-lead" subagent (causes recursion).

## Ticket Selection & Parsing

**No argument:** Run `bv --robot-triage`, parse recommendation, inform user, continue with that ticket
**Parse argument:** Accept comma-separated `beads-123,beads-234` (multiple tickets share one worktree) or single `beads-123`

## Phase Execution Pattern

Each phase follows this pattern to support stateless resumption:

**1. Check completion:**

```bash
bd comments beads-{id} | grep "Phase {N}: ✅"
```

If found → skip to next phase. If not found → execute phase.

**2. Execute phase work** (see phase details below)

**3. Update todos:** Mark current as completed, next as in_progress

**4. Log completion:**

```bash
bd comments add beads-{id} "## Phase {N}: ✅ Completed

{phase_name}

{relevant_details}"
```

---

## Workflow Phases

### P1: Initialize Todos

TodoWrite initial task list:

```json
[
  {
    "content": "Validate tickets exist and are open",
    "status": "in_progress",
    "activeForm": "Validating tickets"
  },
  {
    "content": "Read project standards (AGENTS.md, CLAUDE.md)",
    "status": "pending",
    "activeForm": "Reading project standards"
  },
  {
    "content": "Identify project root directory",
    "status": "pending",
    "activeForm": "Identifying project root"
  },
  {
    "content": "Create git worktree for feature branch",
    "status": "pending",
    "activeForm": "Creating git worktree"
  },
  {
    "content": "Read task details and comments from beads",
    "status": "pending",
    "activeForm": "Reading task details"
  },
  {
    "content": "Analyze task and add initial comments",
    "status": "pending",
    "activeForm": "Analyzing task"
  },
  {
    "content": "Launch Engineer agent for implementation",
    "status": "pending",
    "activeForm": "Launching Engineer agent"
  },
  {
    "content": "Create pull request using pr-writer agent",
    "status": "pending",
    "activeForm": "Creating pull request"
  },
  {
    "content": "Launch Reviewer agent for code review",
    "status": "pending",
    "activeForm": "Launching Reviewer agent"
  },
  {
    "content": "Review iteration 1: Address feedback (if needed)",
    "status": "pending",
    "activeForm": "Review iteration 1"
  },
  {
    "content": "Review iteration 2: Address final feedback (if needed)",
    "status": "pending",
    "activeForm": "Review iteration 2"
  },
  {
    "content": "Merge approved pull request",
    "status": "pending",
    "activeForm": "Merging pull request"
  },
  {
    "content": "Close tickets and sync beads",
    "status": "pending",
    "activeForm": "Closing tickets"
  },
  {
    "content": "Clean up git worktree",
    "status": "pending",
    "activeForm": "Cleaning up worktree"
  },
  {
    "content": "Generate final report",
    "status": "pending",
    "activeForm": "Generating final report"
  }
]
```

### P2: Validate Tickets

For each ticket: `bd show {ticket-id}` → verify exists & status is open/in_progress. If any ticket missing/closed: exit with error.

Update status: `bd update {ticket-id} --status=in_progress`

Log: `Tickets validated and status updated to in_progress: {ticket_ids}`

### P3: Read Project Standards

Read from project root (where .beads/ directory is):

- `AGENTS.md` - Quality gates, validation patterns
- `CLAUDE.md` - Project standards
- `(docs|specs)/constitution.md` - Project principles (optional)

Log: `AGENTS.md: {read/not_found}, CLAUDE.md: {read/not_found}, constitution.md: {read/not_found}`

### P4: Identify Project Root

Ask user if unclear. Verify: git repo with `.beads/` directory.

Log: `Path: {project_root}`

### P5: Create Git Worktree

```bash
cd {project_root}
bd worktree create ../worktrees/feature/beads-{first_ticket_id}
```

Naming: `feature/beads-{id}` (first ticket ID for multiple). Record worktree path.

Log: `Branch: feature/beads-{first_ticket_id}, Path: {worktree_path}`

### P6: Read Task Details

```bash
bd show beads-{id}  # for each ticket
```

Read description, comments, requirements, dependencies.

Log: `Tickets analyzed: {ticket_ids}`

### P7: Analyze & Document

Add Technical Lead analysis to each ticket:

```bash
bd comments add beads-{id} "## Technical Lead Analysis

### Task Assessment
{complexity_assessment}
{estimated_scope}
{dependencies_noted}

### Implementation Approach
{suggested_approach}
{technical_considerations}

### Next Steps
Ready for Engineer agent to implement"
```

Log: `Complexity: {complexity}, Scope: {scope}, Dependencies: {dependencies}`

### P8: Launch Engineer

```
Task tool → k2-dev:engineer
Prompt: "Implement tickets: {ticket_ids}
Worktree: {worktree_path}
Project root: {project_root}
Standards: AGENTS.md, CLAUDE.md, constitution.md

CRITICAL - First:
cd {worktree_path}
pwd && git branch  # verify location
bd show beads-{id} && bd comments beads-{id}  # read context

Then: implement following plan, self-review, push changes.
IMPORTANT: Stay in worktree for all file ops. Do NOT create PR."
```

Wait for completion.

Log: `Implementation completed and changes pushed`

### P9: Create Pull Request

**Change to worktree:** `cd {worktree_path}`

**Launch pr-writer:**

```
Task tool → k2-dev:pr-writer
Description: "Create PR for implementation"
Prompt: "Create PR. Worktree: {worktree_path}, Ticket: {ticket-id}, Branch: feature/beads-{id}

Change to worktree, analyze changes/commits (git log, git diff), read PR templates,
generate description, create GitHub PR with proper formatting, link tickets, return URL."
```

Record PR URL.

Log: `PR: {pr_url}, Branch: feature/beads-{id}`

### P10: Launch Reviewer

```
Task tool → k2-dev:reviewer
Prompt: "Review PR {pr_url}
Worktree: {worktree_path}
Quality gates: AGENTS.md, CLAUDE.md, constitution.md

Validate standards, add GitHub comments, add summary to beads."
```

Wait for result:

- **Approved:** → P12 (merge)
- **Changes requested:** → P11 (iterations)

Log: `Review result: {approved/changes_requested}. {if_approved: "Ready for merge"} {if_changes: "Feedback received, iterations needed"}`

### P11: Review Iterations (max 2)

Check status: `bd comments beads-{id} | grep "Phase 11, Iteration"`

#### Iteration 1

Launch Engineer to address feedback:

```
Task tool → k2-dev:engineer
Prompt: "Address review feedback for PR {pr_url}
Worktree: {worktree_path}

Read feedback:
- gh pr view {pr_number} --comments
- bd comments beads-{id}

Fix issues, respond to comments, run quality gates, push changes."
```

Log iteration 1: `Review Feedback Addressed. Changes pushed. Re-review requested.`

Re-launch Reviewer (same prompt as P10).

- **Approved:** → P12
- **Changes:** → Iteration 2

#### Iteration 2

Same process as Iteration 1.

Log iteration 2: `Second Review Feedback Addressed. Changes pushed. Final review requested.`

Final Reviewer launch.

- **Approved:** → P12
- **Issues remain:** → Create follow-up tickets

#### Follow-Up Tickets (after 2 iterations with issues)

```bash
bd create --title="{title}" --priority=P0|P1|P2 --description="$(cat <<'EOF'
Issue from beads-{original_id} review requiring follow-up.

## Issue Description
{description}

## Context
Related to: beads-{original_id}
Review PR: {pr_url}

## Acceptance Criteria
{criteria}
EOF
)"
```

**Prioritization:** P0=critical/security/breaking, P1=significant quality, P2=minor improvements

**Decision:** P0 → ask user (merge now or wait?). P1/P2 → can merge with follow-ups.

Log: `Follow-Up Tickets Created. Tickets: {ids}. Decision: {merge_now_or_wait}`

### P12: Merge & Cleanup

**Merge:**

```bash
cd {worktree_path}
gh pr merge {pr_number} --squash
```

**Close tickets:**

```bash
bd update beads-{id} --status=closed  # for each
bd comments add beads-{id} "Implementation complete!

## Pull Request
PR: {pr_url}
Branch: feature/beads-{id}
Review iterations: {count}/2

## Changes
{summary}

## Follow-up Tickets
{tickets_or_none}"
```

**Sync:** `bd sync`

**Cleanup:**

```bash
cd {project_root}
bd worktree remove {worktree_path}
git worktree prune
```

**Generate reports:**

```
Skill tool → k2-dev:report (for each ticket-id)
```

Log: `PR merged: {pr_url}, Tickets closed: {ticket_ids}, Worktree cleaned up, Reports generated`

**User summary:**

```markdown
## Implementation Complete: {ticket_ids}

### Workflow Summary

✅ Validated → ✅ Worktree → ✅ Implemented → ✅ Reviewed → ✅ Merged → ✅ Closed → ✅ Cleaned

### Pull Request

URL: {pr_url}
Branch: feature/beads-{id}
Review iterations: {count}/2

### Follow-up Tickets

{any_created}
```

---

## Error Handling

**Ticket missing:** `Error: Ticket {id} does not exist. Check ID and retry.` → exit
**Ticket closed:** `Error: Ticket {id} already closed. Use 'bd reopen {id}' if needed.` → exit
**Worktree exists:** `Error: Worktree exists at {path}. Cleanup: git worktree remove {path}` → exit

## Configuration

**Quality gates:** AGENTS.md, CLAUDE.md, constitution.md
**Branch naming:** Single: `feature/beads-123`, Multiple: `feature/beads-123` (first ID)
**Review limit:** Max 2 iterations → then create follow-up tickets (P0/P1/P2)

## Reminders

- Run Technical Lead logic directly (not as subagent)
- Use TodoWrite throughout
- Launch Engineer/Reviewer/PR-Writer as subagents (safe from command context)
- Update beads comments at key milestones
