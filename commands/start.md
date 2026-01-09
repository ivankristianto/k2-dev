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
 {"content": "Launch Reviewer agent for code review", "status": "pending", "activeForm": "Launching Reviewer agent"}]
```

### Phase 2: Ticket Validation

For each ticket ID:

```bash
bd show {ticket-id}
```

- Verify ticket exists in beads
- Confirm status is "open" or "in_progress"
- If any ticket doesn't exist or is closed: **show error and exit immediately**
- Update todo: Mark validation as completed

### Phase 3: Read Project Standards

Read from PROJECT root (where .beads/ directory is):

- Read `AGENTS.md` - Quality gates, validation patterns
- Read `CLAUDE.md` - Project standards and patterns
- Read `(docs|specs)/constitution.md` - Project principles (if exists)
- Note: These are optional but highly recommended

Update todo: Mark "Read project standards" as completed

### Phase 4: Identify Project Root

Ask user for project root if not clear:

- Verify it's a git repository
- Confirm presence of `.beads/` directory
- This is where all work will happen

Update todo: Mark "Identify project root" as completed

### Phase 5: Create Git Worktree

```bash
cd {project_root}
bd worktree create ../worktrees/feature/beads-{first_ticket_id}
```

- Use consistent naming: `feature/beads-{id}` (use first ticket ID for multiple)
- Create worktree in sibling directory
- Verify worktree creation succeeded
- Record worktree path for later use

Update todo: Mark "Create git worktree" as completed

### Phase 6: Read Task Details

```bash
bd show beads-{id}  # for each ticket
```

- Read full task description and all comments
- Understand requirements, context, special instructions
- Check dependencies if any

Update todo: Mark "Read task details" as completed

### Phase 7: Analyze and Document

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

### Phase 8: Launch Engineer Agent

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

### Phase 9: Create Pull Request

When Engineer returns with implementation complete:

1. **Change to worktree directory**:

   ```bash
   cd {worktree_path}
   ```

2. **Create PR using pr-creation skill**:

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

3. **Record PR URL** for next steps

### Phase 10: Launch Reviewer Agent

After PR is created:

1. Update todo: Add "Launch Reviewer agent"
2. Use Task tool to launch Reviewer:

```
Task tool with:
- subagent_type: "k2-dev:reviewer"
- prompt: "Review PR {pr_url}
          Worktree: {worktree_path}
          Quality gates: AGENTS.md, CLAUDE.md, constitution.md

          Validate against all quality standards and provide feedback."
```

### Phase 11: After Reviewer Completes

Track review iterations (max 2):

- Iteration 1: If changes needed, launch Engineer again
- Iteration 2: Final review, if issues remain → create follow-up tickets

When approved:

- Merge PR: `gh pr merge {pr_number} --squash --delete-branch`
- Close tickets: `bd update beads-{id} --status=closed`
- Sync beads: `bd sync`
- Cleanup worktree: `bd worktree remove {worktree_path}`

### Phase 12: Final Report

Use the Skill tool to generate comprehensive reports for each completed ticket:

```
For each ticket ID:
  Skill tool with:
  - skill: "k2-dev:report"
  - args: "{ticket-id}"
```

Then provide a workflow summary:

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
