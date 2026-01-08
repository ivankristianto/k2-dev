---
name: Beads Integration
description: This skill should be used when the user asks to "work with beads tasks", "create a beads ticket", "update beads status", "read task comments", "manage dependencies", "sync beads", "use bd commands", or needs guidance on beads task management workflows. Provides comprehensive beads CLI usage and workflow patterns.
version: 0.1.0
---

# Beads Integration Skill

## Overview

Beads is a git-backed issue tracker designed for multi-session work with dependencies and persistent memory. This skill provides guidance for working with beads task management within k2-dev workflows.

**Key Capabilities:**
- Task creation and management
- Status transitions and updates
- Comment and context management
- Dependency tracking
- Git worktree integration
- Synchronization workflows

## Core Concepts

### Task Structure

Beads tasks consist of:
- **ID**: Unique identifier (e.g., beads-123)
- **Title**: Short description
- **Description**: Detailed requirements and context
- **Status**: open, in_progress, blocked, closed
- **Priority**: P0 (critical), P1 (high), P2 (medium), P3 (low)
- **Comments**: Discussion and updates
- **Dependencies**: Relationships between tasks

### Git-Backed Storage

All tasks stored in `.beads/` directory as markdown files:
- Committed to git for persistence
- Survives conversation compaction
- Shareable across team members
- Full history via git log

## Essential Commands

### Viewing Tasks

**List all tasks:**
```bash
bd list
```

**List with filters:**
```bash
bd list --filter=status:open
bd list --filter=priority:P0
bd list --filter=assignee:engineer
```

**Show task details:**
```bash
bd show beads-123
```

**Search tasks:**
```bash
bd search "authentication"
```

### Creating Tasks

**Create new task:**
```bash
bd create "Task title" -d "Detailed description" -p P1
```

**Create with dependencies:**
```bash
bd create "Task B" -d "Depends on Task A"
bd dep add beads-456 beads-123  # beads-456 depends on beads-123
```

### Updating Tasks

**Change status:**
```bash
bd update beads-123 --status in_progress
bd update beads-123 --status closed
```

**Update priority:**
```bash
bd update beads-123 --priority P0
```

**Assign task:**
```bash
bd update beads-123 --assignee engineer
```

### Comments

**Add comment:**
```bash
bd comments beads-123 add "Implementation notes..."
```

**View comments:**
```bash
bd comments beads-123
```

### Dependencies

**Add dependency:**
```bash
bd dep add beads-456 beads-123  # 456 depends on 123
```

**List dependencies:**
```bash
bd dep list beads-123
```

**Remove dependency:**
```bash
bd dep remove beads-456 beads-123
```

### Synchronization

**Sync with remote:**
```bash
bd sync
```

Run this:
- After closing tasks
- Before starting new work
- At session end (automated via Stop hook)

## K2-Dev Workflow Integration

### Reading Task Context

Before starting implementation, gather full context:

```bash
# Get task details
bd show beads-123

# Read all comments for additional context
bd comments beads-123

# Check dependencies
bd dep list beads-123
```

Parse the output to extract:
- Title and description (requirements)
- Current status
- Priority
- Plan and implementation notes in comments
- Blocking dependencies

### Creating Tasks from Plans

When Planner converts plans to tasks:

```bash
# Create epic
bd create "User Authentication System" -d "Epic description" -p P1

# Create stories under epic
bd create "JWT Token Implementation" -d "Story description" -p P1

# Create subtasks
bd create "Auth middleware" -d "Subtask description" -p P2

# Set up dependencies
bd dep add beads-103 beads-102  # Middleware depends on token implementation
```

### Adding Implementation Context

Technical Lead adds implementation notes:

```bash
bd comments beads-123 add "Implementation Plan:
1. Create middleware in src/auth/
2. Use existing JWT library
3. Follow patterns in AGENTS.md
4. Add tests in tests/auth/
"
```

### Status Transitions

Follow proper workflow:

```bash
# Technical Lead assigns to Engineer
bd update beads-123 --status in_progress --assignee engineer

# After implementation
bd update beads-123 --status closed

# If blocked
bd update beads-123 --status blocked
bd comments beads-123 add "Blocked by: beads-120 needs merge first"
```

### Creating Follow-Up Tickets

When review identifies issues beyond 2 iterations:

```bash
# Create P0 for critical issues
bd create "Fix authentication bypass vulnerability" -d "Critical security issue from PR #456 review" -p P0

# Link to original task
bd comments beads-789 add "Follow-up ticket created: beads-790 (P0)"
bd comments beads-790 add "Follow-up from: beads-789"
```

## Best Practices

### Task Descriptions

Write clear, actionable descriptions:
- Include acceptance criteria
- Reference relevant files or components
- Link to related tickets
- Specify constraints or requirements

**Good:**
```
Implement JWT authentication middleware

Acceptance Criteria:
- Validates JWT tokens on protected routes
- Returns 401 for invalid/expired tokens
- Follows patterns in src/auth/oauth.ts
- Test coverage >80%

Related: beads-120 (API refactor)
```

**Bad:**
```
Add auth stuff
```

### Comments

Use comments for:
- Implementation plans and notes
- Progress updates
- Blockers and resolutions
- Review feedback summary
- Links to PRs and commits

### Dependencies

Set dependencies to:
- Prevent starting work too early
- Show task relationships
- Enable parallel work where possible
- Communicate workflow order

### Priority Levels

Use priorities consistently:
- **P0**: Critical, blocking production, security issues
- **P1**: High priority, needed soon, key features
- **P2**: Medium priority, planned work, improvements
- **P3**: Low priority, nice-to-have, backlog

### Synchronization

Always sync after:
- Creating or closing tasks
- Session end (automated via hook)
- Before generating reports
- After significant updates

## Task Hierarchies

### Epics

Large features spanning multiple sessions:
```bash
bd create "User Management System" -d "Epic: Complete user CRUD with roles" -p P1
```

### Stories

User-facing capabilities (2-5 days):
```bash
bd create "User Profile Editing" -d "Story: Users can edit their profile" -p P1
```

### Subtasks

Technical implementation units (0.5-2 days):
```bash
bd create "Profile API endpoint" -d "Subtask: Create PUT /api/users/:id endpoint" -p P2
```

Link them with dependencies to show hierarchy.

## Troubleshooting

### Task Not Found

```bash
# Sync first
bd sync

# List all tasks
bd list

# Search for it
bd search "keyword"
```

### Status Not Updating

Check for:
- Unsaved changes in editor
- Git conflicts in .beads/
- Need to sync

### Dependency Cycles

Avoid circular dependencies:
```bash
# Check dependencies before adding
bd dep list beads-123
bd dep list beads-456

# Ensure no cycles exist
```

## Integration with Git Worktree

When using bd worktree commands:

**Create worktree for task:**
```bash
bd worktree create beads-123
```

This creates:
- Git worktree at ../beads-123/
- Branch feature/beads-123
- Switches context to new worktree

**List worktrees:**
```bash
bd worktree list
```

**Remove worktree:**
```bash
bd worktree remove beads-123
```

## Command Reference Quick Sheet

| Action | Command |
|--------|---------|
| List tasks | `bd list` |
| Show task | `bd show beads-123` |
| Create task | `bd create "Title" -d "Description" -p P1` |
| Update status | `bd update beads-123 --status in_progress` |
| Add comment | `bd comments beads-123 add "Text"` |
| View comments | `bd comments beads-123` |
| Add dependency | `bd dep add beads-456 beads-123` |
| List dependencies | `bd dep list beads-123` |
| Search | `bd search "keyword"` |
| Sync | `bd sync` |
| Close | `bd update beads-123 --status closed` |

## Additional Resources

For complete beads CLI reference, see:
- Official CLI documentation: https://github.com/steveyegge/beads/blob/main/docs/CLI_REFERENCE.md
- Beads repository: https://github.com/steveyegge/beads

## Usage in K2-Dev Agents

**Technical Lead:**
- Validates tickets exist and are open
- Reads task context for planning
- Adds architectural notes as comments
- Closes tasks after merge
- Syncs beads at end

**Engineer:**
- Reads task description and comments for requirements
- Checks dependencies before starting
- Updates status during implementation
- Creates follow-up tickets when needed

**Planner:**
- Creates hierarchical task structures
- Sets up dependencies
- Writes detailed descriptions
- Links related tasks

**Reviewer:**
- Reads task context for review
- Does NOT add comments to beads (uses GitHub PR instead)
- May recommend follow-up tickets

**Tester:**
- Reads requirements from beads
- Adds test plan as comment
- References beads ticket in test documentation

Follow these patterns for consistent beads integration across all k2-dev workflows.
