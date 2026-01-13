---
name: k2:start
description: Start implementation workflow for one or more beads tickets
argument-hint: "[beads-123] [--skip-worktree] or beads-123,beads-234 (optional - auto-selects if omitted)"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - TodoWrite
  - Task
  - Skill
  - AskUserQuestion
---

# K2:Start - Implementation Workflow

Coordinate the full implementation lifecycle: validation → worktree/branch → implementation → review → merge → cleanup.

**CRITICAL:** Execute logic directly in this context.

## Performance: Beads Data Caching

**Speed optimization:** `bd show` and `bd comments` are called multiple times per workflow. Cache the results in P2 and reuse them:

| Phase | Without Cache | With Cache |
|-------|---------------|------------|
| P2 | `bd show` (validation) | `bd show` + cache full data |
| P3 | `bd show` + `bd comments` | Use cached TICKET_DATA + TICKET_COMMENTS |
| P5 | Engineer re-fetches | Pass cached data in prompt |
| P8 | Re-fetch comments | Use cached TICKET_COMMENTS |

**Estimated savings:** ~2-3 seconds per workflow (fewer shell command executions)

## Ticket Selection & Parsing

**No argument:** Run `bv --robot-next`, parse recommendation, inform user, continue with that ticket
**Parse argument:** Accept comma-separated `beads-123,beads-234` (multiple tickets share one worktree) or single `beads-123`
**Flag support:** `--skip-worktree` to create branch in main repository instead of creating a worktree

## Checkpoint Logging Pattern

Instead of logging every phase, use **3 checkpoint comments** for efficiency:

| Checkpoint | Phases | Content |
|------------|--------|---------|
| CP1 | P1-P4 | Setup: validated, standards read, worktree created |
| CP2 | P5-P7 | Execution: implementation, PR, initial review |
| CP3 | P9 | Complete: merged, closed, cleaned up |

**Resumption pattern:**
```bash
bd comments beads-{id} | grep "Checkpoint [1-3]:"
```

**Each checkpoint:**
```bash
bd comments add beads-{id} "$(cat <<'EOF'
## Checkpoint N: ✅ {status}

{summary of phases completed}
EOF
)"
```

**CRITICAL:** Always log to beads at checkpoints. This is NOT optional.

---

## Task Tool Usage (CRITICAL)

**IMPORTANT:** When using the Task tool to launch agents (Engineer, Reviewer, PR Writer):

1. **The Task tool BLOCKS by default** - it waits until the agent completes
2. **The result is returned automatically** in the Task tool response
3. **DO NOT call TaskOutput** after the Task tool completes - the output is already in the result
4. **TaskOutput is ONLY for background tasks** (run_in_background=true)

**Common mistake:** Calling `TaskOutput` with the agent ID after Task tool completes causes errors because the task has already been cleaned up.

**Correct pattern:**

```
Task tool → agent completes → read result from Task response → continue workflow
```

**Incorrect pattern:**

```
Task tool → agent completes → call TaskOutput → ERROR: task not found
```

## Workflow Phases

### P0: Parse Arguments

**Parse the command arguments:**

1. Extract ticket IDs (comma-separated or single)
2. Detect `--skip-worktree` flag
3. Store `use_worktree` boolean (true by default, false if `--skip-worktree` present)

**Example parsing:**

```
"beads-123 --skip-worktree" → ticket_ids: ["beads-123"], use_worktree: false
"beads-123,beads-234" → ticket_ids: ["beads-123", "beads-234"], use_worktree: true
"--skip-worktree beads-123" → ticket_ids: ["beads-123"], use_worktree: false
```

### P1: Initialize Todos

TodoWrite initial task list (conditional based on `use_worktree`):

**If use_worktree = true:**

```json
[
  {
    "content": "Validate tickets and identify project root",
    "status": "in_progress",
    "activeForm": "Validating tickets and locating project root"
  },
  {
    "content": "Read project context (standards, task details, comments)",
    "status": "pending",
    "activeForm": "Reading project context"
  },
  {
    "content": "Create git worktree for feature branch",
    "status": "pending",
    "activeForm": "Creating git worktree"
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
    "content": "Add approval comment to GitHub PR",
    "status": "pending",
    "activeForm": "Adding approval comment to PR"
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

**If use_worktree = false:**

```json
[
  {
    "content": "Validate tickets and identify project root",
    "status": "in_progress",
    "activeForm": "Validating tickets and locating project root"
  },
  {
    "content": "Read project context (standards, task details, comments)",
    "status": "pending",
    "activeForm": "Reading project context"
  },
  {
    "content": "Create new branch in main repository",
    "status": "pending",
    "activeForm": "Creating new branch"
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
    "content": "Add approval comment to GitHub PR",
    "status": "pending",
    "activeForm": "Adding approval comment to PR"
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
    "content": "Generate final report",
    "status": "pending",
    "activeForm": "Generating final report"
  }
]
```

Note: When `use_worktree = false`, there is no "Clean up git worktree" todo.

### P2: Validate Tickets & Identify Project Root

**Step 1: Quick validation (fail-fast) + Cache data**

For each ticket, execute in parallel:
- `bd show {ticket-id} --short` → verify exists & status is open/in_progress
- `bd show {ticket-id}` → full task details (CACHE as TICKET_DATA)
- `bd comments {ticket-id}` → task comments (CACHE as TICKET_COMMENTS)

**CRITICAL:** Cache these in your context - you'll use them in P3, P5, and P8 instead of re-fetching.

If any ticket missing/closed: exit with error.

Update status: `bd update {ticket-id} --status=in_progress`

**Step 2: Identify project root**

Ask user if unclear. Verify: git repo with `.beads/` directory.

**No individual logging - checkpoint at P4.**

### P3: Read Project Context (Parallel)

**PERFORMANCE OPTIMIZATION: Execute ALL reads in parallel using multiple tool calls in a single message.**

Read in parallel:

1. `AGENTS.md` - Quality gates, validation patterns (from project root)
2. `(docs|specs)/constitution.md` - Project principles (optional, from project root)
4. **USE CACHED `TICKET_DATA`** - Task details (from P2, no re-fetch)
5. **USE CACHED `TICKET_COMMENTS`** - Task comments (from P2, no re-fetch)

**No individual logging - checkpoint at P4.**

### P4: Create Git Worktree or Branch (Conditional)

**Mode Selection:** Based on `use_worktree` flag from P0.

**If use_worktree = true (default):**

```bash
cd {project_root}
bd worktree create ../worktrees/feature/beads-{first_ticket_id}
```

Naming: `feature/beads-{id}` (first ticket ID for multiple). Record worktree path.

Set: `work_path = ../worktrees/feature/beads-{first_ticket_id}`

**If use_worktree = false:**

```bash
cd {project_root}
git checkout -b feature/beads-{first_ticket_id}
```

Naming: `feature/beads-{id}` (first ticket ID for multiple). Work happens in main repository.

Set: `work_path = {project_root}`

**Checkpoint 1 - Log setup completion:**

```bash
bd comments add beads-{first_ticket_id} "$(cat <<'EOF'
## Checkpoint 1: ✅ Setup Complete

**Tickets:** {ticket_ids}
**Project root:** {project_root}
**Branch:** feature/beads-{first_ticket_id}
**Path:** {work_path}
**Standards:** AGENTS.md, constitution.md loaded

Ready for implementation.
EOF
)"
```

### P5: Launch Engineer

```
Task tool → k2-dev:engineer
Model: sonnet  # Complex implementation requires deep reasoning
Prompt: "Implement tickets: {ticket_ids}
Work path: {work_path}
Project root: {project_root}
Standards: AGENTS.md, constitution.md

CACHED DATA (from command - no need to re-fetch):
- TICKET_DATA: {ticket_details_from_cache}
- TICKET_COMMENTS: {comments_from_cache}

CRITICAL - First:
cd {work_path}
pwd && git branch  # verify location

Then: implement following plan, self-review, push changes.
IMPORTANT: Stay in work path for all file ops. Do NOT create PR."
```

**Agent completion:** The Task tool blocks and returns the result automatically when the engineer completes. Read the result from the Task tool response directly. DO NOT call TaskOutput - the output is already in the result.

**No individual logging - checkpoint at P7.**

### P6: Create Pull Request

**Change to work path:** `cd {work_path}`

**Launch pr-writer:**

```
Task tool → k2-dev:pr-writer
Model: haiku  # Formulaic PR creation task, cost optimization
Description: "Create PR for implementation"
Prompt: "Create PR. Work path: {work_path}, Ticket: {ticket-id}, Branch: feature/beads-{id}

Change to work path, analyze changes/commits (git log, git diff), read PR templates,
generate description, create GitHub PR with proper formatting, link tickets, return URL."
```

**Agent completion:** The Task tool returns the result (including PR URL) automatically. DO NOT call TaskOutput. Extract PR URL from the Task tool response.

**No individual logging - checkpoint at P7.**

### P7: Launch Reviewer

**PERFORMANCE OPTIMIZATION:** Reviewer uses local git diff/log instead of GitHub API during review for faster execution.

```
Task tool → k2-dev:reviewer
Model: sonnet  # Deep code analysis and security validation required
Prompt: "Review implementation for ticket {ticket-id}

**PERFORMANCE MODE - Use local git operations:**
Work path: {work_path}
Branch: feature/beads-{first_ticket_id}
Base branch: {main_or_default}
PR number: {pr_number} (for final comment only)
Ticket ID: {ticket-id}

**Review process:**
1. cd {work_path}
2. Use local git diff {base_branch}...HEAD (NO GitHub API during review)
3. Use git log {base_branch}..HEAD for commit context
4. Validate against AGENTS.md, constitution.md
5. Post final review results (approval or feedback) to GitHub PR as comment
6. Add review summary to beads task comments

Quality gates: AGENTS.md, constitution.md"
```

**Agent completion:** The Task tool blocks and returns the result automatically when the reviewer completes. Read the approval status from the Task tool response directly. DO NOT call TaskOutput - the output is already in the result.

**Parse result:**

- **Approved:** → P9 (merge)
- **Changes requested:** → P8 (iterations)

**If approved, add comment to GitHub PR:**

```bash
cd {work_path}
gh pr comment {pr_number} --body "✅ Code review approved by k2-dev Reviewer agent.

The code has been reviewed and validated against project quality gates (AGENTS.md). Ready for merge."
```

**Why comment instead of approve?** GitHub doesn't allow approving your own PR. We add a comment to document the Reviewer agent's approval, then proceed to merge.

**Checkpoint 2 - Log execution completion:**

```bash
bd comments add beads-{first_ticket_id} "$(cat <<'EOF'
## Checkpoint 2: ✅ Execution Complete

**Implementation:** Engineer completed, changes pushed
**PR:** {pr_url}
**Review:** {approved/changes_requested}

{if_approved: 'Ready for merge' | if_changes: 'Iterations pending'}
EOF
)"
```

### P8: Review Iterations (max 2)

Check status: `bd comments beads-{id} | grep "Iteration"`

#### Iteration 1

Launch Engineer to address feedback:

```
Task tool → k2-dev:engineer
Model: sonnet  # Complex fixes require deep reasoning
Prompt: "Address review feedback for PR {pr_url}
Work path: {work_path}

Read feedback:
- gh pr view {pr_number} --comments
- USE CACHED TICKET_COMMENTS (from command)

Fix issues, respond to comments, run quality gates, push changes."
```

**Agent completion:** The Task tool returns the result automatically. DO NOT call TaskOutput.

**Log iteration 1:**

```bash
bd comments add beads-{first_ticket_id} "$(cat <<'EOF'
## Iteration 1: ✅ Feedback Addressed

Changes pushed, re-review requested
EOF
)"
```

**Re-launch Reviewer** (same prompt as P7). The Task tool returns the result automatically. DO NOT call TaskOutput.

**Parse result:**
- **Approved:** → Add approval comment to PR, then → P9
- **Changes:** → Iteration 2

**If approved, add comment to GitHub PR:**

```bash
cd {work_path}
gh pr comment {pr_number} --body "✅ Re-review approved (Iteration 1).

Feedback addressed. Ready for merge."
```

#### Iteration 2

Same process as Iteration 1.

**Log iteration 2:**

```bash
bd comments add beads-{first_ticket_id} "$(cat <<'EOF'
## Iteration 2: ✅ Final Review Passed

All feedback addressed
EOF
)"
```

**Final Reviewer launch.** The Task tool returns the result automatically. DO NOT call TaskOutput.

**Parse result:**
- **Approved:** → Add approval comment to PR, then → P9
- **Issues remain:** → Create follow-up tickets

**If approved, add comment to GitHub PR:**

```bash
cd {work_path}
gh pr comment {pr_number} --body "✅ Final review approved (Iteration 2).

All feedback addressed. Ready for merge."
```

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

**Log follow-ups:**

```bash
bd comments add beads-{first_ticket_id} "$(cat <<'EOF'
## Follow-Ups Created

Tickets: {ids}
Decision: {merge_now_or_wait}
EOF
)"
```

### P9: Merge & Cleanup (Checkpoint 3)

**Merge:**

```bash
cd {work_path}
gh pr merge {pr_number} --squash --delete-branch
```

**Close tickets:**

```bash
bd update beads-{id} --status=closed  # for each
bd sync
```

**Cleanup (conditional):**

**If use_worktree = true:**

```bash
cd {project_root}
bd worktree remove {worktree_path}
git worktree prune
```

**If use_worktree = false:** Branch deleted automatically during merge.

**Checkpoint 3 - Final summary:**

```bash
bd comments add beads-{first_ticket_id} "$(cat <<'EOF'
## Checkpoint 3: ✅ Complete

**PR:** {pr_url}
**Iterations:** {count}/2
**Status:** Merged and closed

## Follow-ups
{tickets_or_none}

---
_Implementation complete_
EOF
)"
```

**Generate reports:**

```
Skill: k2-dev:report
Args: {ticket-id}
```

**User summary:**

```markdown
## Implementation Complete: {ticket_ids}

✅ Merged: {pr_url}
✅ Iterations: {count}/2
✅ Follow-ups: {any_created}
```

---

## Error Handling

**Ticket missing:** `Error: Ticket {id} does not exist. Check ID and retry.` → exit
**Ticket closed:** `Error: Ticket {id} already closed. Use 'bd reopen {id}' if needed.` → exit
**Worktree exists (only when use_worktree = true):** `Error: Worktree exists at {path}. Cleanup: git worktree remove {path}` → exit
**Branch exists (only when use_worktree = false):** `Error: Branch feature/beads-{id} already exists. Delete with: git branch -D feature/beads-{id}` → exit

## Configuration

**Quality gates:** AGENTS.md, constitution.md
**Branch naming:** Single: `feature/beads-123`, Multiple: `feature/beads-123` (first ID)
**Review limit:** Max 2 iterations → then create follow-up tickets (P0/P1/P2)
**Workflow modes:**

- Default (worktree): Creates isolated worktree at `../worktrees/feature/beads-{id}`
- `--skip-worktree`: Creates branch in main repository, skips worktree creation/cleanup

## Reminders

- Execute logic directly in this context
- Use TodoWrite throughout
- Launch Engineer/Reviewer/PR-Writer as subagents (safe from command context)
- Log to beads at checkpoints (CP1, CP2, CP3) + iteration notes
