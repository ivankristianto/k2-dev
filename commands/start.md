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

You are running the **Technical Lead orchestration logic directly** for the k2-dev multiagent development system. This command validates tickets, creates worktrees, analyzes tasks, and coordinates the implementation workflow.

## Critical Design Principle

**DO NOT use the Task tool to launch a "technical-lead" subagent** - that would cause recursion. Instead, execute the Technical Lead logic directly in this command context.

## Phase 0: Ticket Selection (if no argument provided)

If the user did NOT provide a ticket ID argument:

1. Run beads_viewer robot triage to get recommendation:

```bash
bv --robot-triage
```

2. Parse the output to extract the top recommended ticket ID
3. Inform the user: "No ticket specified. Using beads_viewer recommendation: {ticket-id}"
4. Continue with that ticket ID for the rest of the workflow

## Parse Tickets

Parse the argument (or auto-selected ticket) to extract ticket IDs:

- Accepts comma-separated: `beads-123,beads-234,beads-345`
- Multiple tickets share one worktree
- Single ticket: `beads-123`
- No argument: Auto-select via `bv --robot-triage`

## Workflow Execution

You will execute the following phases directly (not as a subagent):

### Stateless Workflow Pattern

Each phase logs its completion to beads comments for statelessness. This allows the workflow to resume if interrupted.

**At the start of each phase:**
```bash
# Check if phase is already completed
bd comments beads-{id} | grep "Phase {N}: ✅"
```
- If found: Skip to next phase (already completed)
- If not found: Execute the phase

**At the end of each phase:**
```bash
# Log phase completion
bd comments add beads-{id} "## Phase {N}: ✅ Completed

{phase_name}

{relevant_details}
"
```

### Phase 1: Initialization

Use `TodoWrite` to create initial todos:

```
[{"content": "Validate tickets exist and are open", "status": "in_progress", "activeForm": "Validating tickets"},
 {"content": "Read project standards (AGENTS.md, CLAUDE.md)", "status": "pending", "activeForm": "Reading project standards"},
 {"content": "Identify project root directory", "status": "pending", "activeForm": "Identifying project root"},
 {"content": "Create git worktree for feature branch", "status": "pending", "activeForm": "Creating git worktree"},
 {"content": "Read task details and comments from beads", "status": "pending", "activeForm": "Reading task details"},
 {"content": "Analyze task and add initial comments", "status": "pending", "activeForm": "Analyzing task"},
 {"content": "Launch Engineer agent for implementation", "status": "pending", "activeForm": "Launching Engineer agent"},
 {"content": "Create pull request using pr-creation skill", "status": "pending", "activeForm": "Creating pull request"},
 {"content": "Launch Reviewer agent for code review", "status": "pending", "activeForm": "Launching Reviewer agent"},
 {"content": "Review iteration 1: Address feedback (if needed)", "status": "pending", "activeForm": "Review iteration 1"},
 {"content": "Review iteration 2: Address final feedback (if needed)", "status": "pending", "activeForm": "Review iteration 2"},
 {"content": "Merge approved pull request", "status": "pending", "activeForm": "Merging pull request"},
 {"content": "Close tickets and sync beads", "status": "pending", "activeForm": "Closing tickets"},
 {"content": "Clean up git worktree", "status": "pending", "activeForm": "Cleaning up worktree"},
 {"content": "Generate final report", "status": "pending", "activeForm": "Generating final report"}]
```

### Phase 2: Ticket Validation

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 2: ✅"
```
- If found: Skip to Phase 3
- If not found: Continue below

For each ticket ID:

```bash
bd show {ticket-id}
```

- Verify ticket exists in beads
- Confirm status is "open" or "in_progress"
- If any ticket doesn't exist or is closed: **show error and exit immediately**
- Update ticket status to in_progress:

```bash
bd update {ticket-id} --status=in_progress
```

- Update todo: Mark validation as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 2: ✅ Completed

**Ticket Validation**

Tickets validated and status updated to in_progress:
{ticket_ids}"
```

### Phase 3: Read Project Standards

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 3: ✅"
```
- If found: Skip to Phase 4
- If not found: Continue below

Read from PROJECT root (where .beads/ directory is):

- Read `AGENTS.md` - Quality gates, validation patterns
- Read `CLAUDE.md` - Project standards and patterns
- Read `(docs|specs)/constitution.md` - Project principles (if exists)
- Note: These are optional but highly recommended

Update todo: Mark "Read project standards" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 3: ✅ Completed

**Project Standards Read**

- AGENTS.md: {read_or_not_found}
- CLAUDE.md: {read_or_not_found}
- constitution.md: {read_or_not_found}"
```

### Phase 4: Identify Project Root

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 4: ✅"
```
- If found: Skip to Phase 5
- If not found: Continue below

Ask user for project root if not clear:

- Verify it's a git repository
- Confirm presence of `.beads/` directory
- This is where all work will happen

Update todo: Mark "Identify project root" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 4: ✅ Completed

**Project Root Identified**

Path: {project_root}"
```

### Phase 5: Create Git Worktree

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 5: ✅"
```
- If found: Skip to Phase 6, extract worktree path from comments
- If not found: Continue below

```bash
cd {project_root}
bd worktree create ../worktrees/feature/beads-{first_ticket_id}
```

- Use consistent naming: `feature/beads-{id}` (use first ticket ID for multiple)
- Create worktree in sibling directory
- Verify worktree creation succeeded
- Record worktree path for later use

Update todo: Mark "Create git worktree" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 5: ✅ Completed

**Git Worktree Created**

Branch: feature/beads-{first_ticket_id}
Path: {worktree_path}"
```

### Phase 6: Read Task Details

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 6: ✅"
```
- If found: Skip to Phase 7
- If not found: Continue below

```bash
bd show beads-{id}  # for each ticket
```

- Read full task description and all comments
- Understand requirements, context, special instructions
- Check dependencies if any

Update todo: Mark "Read task details" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 6: ✅ Completed

**Task Details Read**

Tickets analyzed: {ticket_ids}"
```

### Phase 7: Analyze and Document

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 7: ✅"
```
- If found: Skip to Phase 8
- If not found: Continue below

Analyze the task(s) and add a comment to each ticket with:

```markdown
## Technical Lead Analysis

### Task Assessment

- {complexity_assessment}
- {estimated_scope}
- {dependencies_noted}

### Implementation Approach

- {suggested_approach}
- {technical_considerations}

### Next Steps

- Ready for Engineer agent to implement
- Run: Engineer agent will be launched to begin implementation
```

Use: `bd comments add beads-{id} "..."` or `bd update` if needed

Update todo: Mark "Analyze task" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 7: ✅ Completed

**Task Analysis Complete**

Complexity: {complexity}
Scope: {scope}
Dependencies: {dependencies}"
```

### Phase 8: Launch Engineer Agent

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 8: ✅"
```
- If found: Skip to Phase 9
- If not found: Continue below

After setup is complete, use Task tool to launch Engineer:

```
Task tool with:
- subagent_type: "k2-dev:engineer"
- prompt: "Implement feature for tickets: {ticket_ids}
          Worktree: {worktree_path}
          Project root: {project_root}
          Standards files read: AGENTS.md, CLAUDE.md, constitution.md

          Task context: {brief_summary}

          CRITICAL - First change to worktree directory:
          cd {worktree_path}
          pwd  # Verify you're in the correct location
          git branch  # Verify you're on feature/beads-{id} branch

          Then read task context:
          bd show beads-{id}
          bd comments beads-{id}

          Please implement following the plan in beads, perform self-review,
          and push your changes. Report back when implementation is complete.

          IMPORTANT:
          - Stay in the worktree directory for ALL file operations
          - All Read/Write/Edit operations must be done from within worktree
          - Use absolute paths when in doubt: {worktree_path}/
          - Do NOT create the PR. The Technical Lead will handle PR creation."
```

**IMPORTANT**: Engineer is a subagent, but that's OK because THIS command is running directly (not as a subagent).

Wait for Engineer to complete and report back.

Update todo: Mark "Launch Engineer agent" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 8: ✅ Completed

**Engineer Agent Launched**

Implementation completed and changes pushed."
```

### Phase 9: Create Pull Request

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 9: ✅"
```
- If found: Skip to Phase 10, extract PR URL from comments
- If not found: Continue below

When Engineer returns with implementation complete:

1. Update todo: Mark "Create pull request" as in_progress

2. **Change to worktree directory**:

   ```bash
   cd {worktree_path}
   ```

3. **Create PR using pr-creation skill**:

   Use the Skill tool to create a well-structured pull request:

   ```
   Skill tool with:
   - skill: "k2-dev:pr-creation"
   - args: "{ticket-id}"
   ```

   The pr-creation skill will:

   - Analyze the changes and commits in the worktree
   - Read PR templates if they exist
   - Generate comprehensive PR description
   - Create the GitHub PR with proper formatting
   - Link to beads tickets
   - Return the PR URL

4. **Record PR URL** for next steps

Update todo: Mark "Create pull request" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 9: ✅ Completed

**Pull Request Created**

PR: {pr_url}
Branch: feature/beads-{id}"
```

### Phase 10: Launch Reviewer Agent

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 10: ✅"
```
- If found: Skip to Phase 11 or Phase 12 (depending on review result)
- If not found: Continue below

After PR is created:

1. Update todo: Mark "Launch Reviewer agent" as in_progress

2. Use Task tool to launch Reviewer:

```
Task tool with:
- subagent_type: "k2-dev:reviewer"
- prompt: "Review PR {pr_url}
          Worktree: {worktree_path}
          Quality gates: AGENTS.md, CLAUDE.md, constitution.md

          Validate against all quality standards and provide feedback.
          Review PR on GitHub and add comments there. Also add review
          summary to beads task comments."
```

3. Wait for Reviewer to complete and report back

4. Review the feedback:

   - If PR is **approved**: Proceed to Phase 12 (Merge and Cleanup)
   - If **changes requested**: Proceed to Phase 11 (Review Iterations)

5. Update todo: Mark "Launch Reviewer agent" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 10: ✅ Completed

**Reviewer Agent Launched**

Review result: {approved_or_changes_requested}

{if_approved: PR approved, ready for merge}
{if_changes: Feedback received, iterations needed}"
```

### Phase 11: Review Iterations (if changes requested)

This phase handles the feedback loop between Reviewer and Engineer. Maximum 2 iterations.

**Check iteration status:**

```bash
bd comments beads-{id} | grep "Phase 11, Iteration"
```
- If "Iteration 2: ✅": Check final review result
- If "Iteration 1: ✅": Check if need iteration 2
- If neither found: Start iteration 1

#### Iteration 1

1. Update todo: Mark "Review iteration 1" as in_progress

2. Use Task tool to launch Engineer to address feedback:

```
Task tool with:
- subagent_type: "k2-dev:engineer"
- prompt: "Address review feedback for PR {pr_url}

  Worktree: {worktree_path}

  Read the review feedback from:
  - GitHub PR comments: gh pr view {pr_number} --comments
  - Beads task comments: bd comments beads-{id}

  Address all feedback items:
  - Fix code issues
  - Respond to review comments
  - Run quality gates: npm run typecheck && npm run lint && npm run format:fix
  - Push changes to PR branch

  Report back when all feedback is addressed."
```

3. When Engineer completes, update todo: Mark "Review iteration 1" as completed

**Log iteration 1 completion:**

```bash
bd comments add beads-{id} "## Phase 11, Iteration 1: ✅ Completed

**Review Feedback Addressed**

Changes pushed to PR branch.
Re-review requested."
```

4. Launch Reviewer again for re-review (use same prompt as Phase 10)

5. Check Reviewer's response:
   - If PR is **approved**: Proceed to Phase 12 (Merge and Cleanup)
   - If **changes requested**: Proceed to Iteration 2

#### Iteration 2

1. Update todo: Mark "Review iteration 2" as in_progress

2. Launch Engineer again to address remaining feedback (same process as Iteration 1)

3. When Engineer completes, update todo: Mark "Review iteration 2" as completed

**Log iteration 2 completion:**

```bash
bd comments add beads-{id} "## Phase 11, Iteration 2: ✅ Completed

**Second Review Feedback Addressed**

Changes pushed to PR branch.
Final review requested."
```

4. Launch Reviewer for final review

5. After final Reviewer response:
   - If PR is **approved**: Proceed to Phase 12 (Merge and Cleanup)
   - If **issues still remain**: Must create follow-up tickets

#### Follow-Up Ticket Creation (if issues remain after 2 iterations)

If Reviewer still has concerns after 2 iterations:

1. Use Bash to create follow-up tickets:

```bash
bd create --title="[Issue title]" --priority=P0|P1|P2 --description="$(cat <<'EOF'
Issue identified in beads-{original_id} review that requires follow-up.

## Issue Description
[Clear description of the issue]

## Context
Related to: beads-{original_id}
Review PR: {pr_url}

## Acceptance Criteria
[What needs to be done to resolve this]
EOF
)"
```

2. Prioritization guidelines:

   - **P0 (Critical)**: Security vulnerabilities, breaking bugs, data corruption risks
   - **P1 (Important)**: Significant code quality issues, major technical debt
   - **P2 (Nice-to-have)**: Minor refactoring, style improvements

3. Make decision:

   - If P0 issues: Ask user if they want to merge now or wait for fixes
   - If P1/P2 issues: Can merge with follow-ups

4. Document decision in beads task comments

**Log follow-up tickets:**

```bash
bd comments add beads-{id} "## Phase 11: Follow-Up Tickets Created

Tickets: {list_of_ticket_ids}
Decision: {merge_now_or_wait}
"

### Phase 12: Merge and Cleanup

**Check if already completed:**

```bash
bd comments beads-{id} | grep "Phase 12: ✅"
```
- If found: Skip to end (workflow already complete)
- If not found: Continue below

When PR is approved (or after follow-up tickets are created):

1. Update todo: Mark "Merge approved pull request" as in_progress

2. Merge the pull request:

```bash
cd {worktree_path}
gh pr merge {pr_number} --squash
```

- Use squash merge for clean history
- Ensure PR is approved before merging
- Delete remote branch automatically

3. Update todo: Mark "Merge approved pull request" as completed
4. Update todo: Mark "Close tickets and sync beads" as in_progress

5. Close all related tickets:

```bash
bd update beads-{id} --status=closed  # for each ticket
```

6. Add final comment to each ticket:

```bash
bd comments add beads-{id} "Implementation complete!

## Pull Request
- PR: {pr_url}
- Branch: feature/beads-{id}
- Review iterations: {count}/2

## Changes
{brief_summary_of_changes}

## Follow-up Tickets
{any_created_tickets_or_none}"
```

7. Sync beads with remote:

```bash
bd sync
```

8. Update todo: Mark "Close tickets and sync beads" as completed
9. Update todo: Mark "Clean up git worktree" as in_progress

10. Clean up worktree:

```bash
cd {project_root}
bd worktree remove {worktree_path}
git worktree prune
```

- Remove worktree directory
- Prune stale references
- Verify cleanup succeeded

11. Update todo: Mark "Clean up git worktree" as completed
12. Update todo: Mark "Generate final report" as in_progress

13. Use the Skill tool to generate comprehensive reports for each completed ticket:

```
For each ticket ID:
  Skill tool with:
  - skill: "k2-dev:report"
  - args: "{ticket-id}"
```

14. Update todo: Mark "Generate final report" as completed

**Log completion:**

```bash
bd comments add beads-{id} "## Phase 12: ✅ Completed

**Workflow Complete**

- PR merged: {pr_url}
- Tickets closed: {ticket_ids}
- Worktree cleaned up
- Reports generated"
```

15. Provide workflow summary to user:

```markdown
## Implementation Complete: {ticket_ids}

### Workflow Summary

- ✅ Tickets validated
- ✅ Worktree created: {path}
- ✅ Implementation completed by Engineer
- ✅ Code review passed
- ✅ PR merged: {pr_url}
- ✅ Tickets closed and synced
- ✅ Worktree cleaned up

### Pull Request

- URL: {pr_url}
- Branch: feature/beads-{id}
- Review iterations: {count}/2

### Follow-up Tickets

{any_created_tickets}
```

The detailed ticket reports will be generated by the k2:report skill above.

## Error Handling

**Ticket doesn't exist:**

```
Error: Ticket {id} does not exist.
Please check ticket ID and try again.
```

Exit immediately.

**Ticket is closed:**

```
Error: Ticket {id} is already closed.
Use 'bd reopen {id}' if you need to work on it.
```

Exit immediately.

**Worktree already exists:**

```
Error: Worktree already exists at {path}.
Cleanup: git worktree remove {path}
Then retry.
```

## Quality Gates

Enforce standards from:

- **AGENTS.md**: Quality gates, validation patterns
- **CLAUDE.md**: Project standards and patterns
- **constitution.md**: Project principles

## Branch Naming

- Single ticket: `feature/beads-123`
- Multiple tickets: `feature/beads-123` (use first ticket ID)

## Review Iteration Limit

- Maximum 2 review iterations
- After 2 iterations with issues: Create follow-up tickets
  - P0: Critical/blocking issues
  - P1: Important non-blocking issues
  - P2: Nice-to-have improvements

## Remember

- You ARE the Technical Lead logic - run directly, don't launch yourself as subagent
- Use TodoWrite to track progress throughout
- Update beads tasks with comments at key milestones
- Launch Engineer and Reviewer as subagents (that's safe from this context)
- Provide clear reports to user at each phase completion
