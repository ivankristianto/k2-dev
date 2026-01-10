---
name: Beads Integration
description: This skill should be used when the user asks to "work with beads tasks", "create a beads ticket", "update beads status", "read task comments", "manage dependencies", "sync beads", "use bd commands", or needs guidance on beads task management workflows. Provides comprehensive beads CLI usage and workflow patterns.
version: 0.2.0
---

# Beads Integration Skill

## Overview

Beads is a git-backed issue tracker for multi-session work with dependencies and persistent memory. All tasks stored in `.beads/` directory as markdown files, committed to git, surviving conversation compaction.

**Key Capabilities:** Task creation/management, status transitions, comments, dependencies, git worktree integration, synchronization.

**Reference:** See k2-dev-reference.md#beads-cli-commands for complete command syntax.

## Core Concepts

### Task Structure
- **ID**: beads-{number}
- **Title**: Short description
- **Description**: Detailed requirements and context
- **Status**: open, in_progress, blocked, closed
- **Priority**: P0 (critical), P1 (high), P2 (medium), P3 (low)
- **Comments**: Discussion and updates
- **Dependencies**: Relationships between tasks

### Git-Backed Storage
- Stored in `.beads/` as markdown
- Committed to git for persistence
- Survives conversation compaction
- Shareable across team
- Full history via git log

## K2-Dev Workflow Integration

### Reading Task Context (Before Implementation)

```bash
bd show beads-123              # Task details
bd comments beads-123          # All comments for context
bd dep list beads-123          # Check dependencies
```

**Parse output for:**
- Title and description (requirements)
- Current status and priority
- Plan and implementation notes in comments
- Blocking dependencies

### Creating Tasks from Plans (Planner Agent)

```bash
# Epic
bd create "User Authentication System" -d "Epic description" -p P1

# Stories under epic
bd create "JWT Token Implementation" -d "Story" -p P1 --parent=beads-{epic}

# Subtasks
bd create "Auth middleware" -d "Subtask" -p P2 --parent=beads-{story}

# Dependencies (B blocks on A - A must complete before B)
bd dep add beads-{B} --blocks-on=beads-{A}
```

### Adding Implementation Context (Technical Lead)

```bash
bd comments beads-123 add "Implementation Plan:
1. Create middleware in src/auth/
2. Use existing JWT library
3. Follow patterns in AGENTS.md
4. Add tests in tests/auth/"
```

### Status Transitions

**Proper workflow:**
```bash
# Technical Lead assigns to Engineer
bd update beads-123 --status in_progress --assignee engineer

# After implementation
bd update beads-123 --status closed

# If blocked
bd update beads-123 --status blocked
bd comments beads-123 add "Blocked by: beads-120 needs merge first"
```

### Creating Follow-Up Tickets (After Review)

When review identifies issues beyond 2 iterations:

```bash
# Critical issues
bd create "Fix authentication bypass vulnerability" \
  -d "Critical security issue from PR #456 review. Details: [...]" \
  -p P0

# Link to original
bd comments beads-789 add "Follow-up ticket created: beads-790 (P0)"
bd comments beads-790 add "Follow-up from: beads-789"
```

## Best Practices

### Task Descriptions

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

### Synchronization

Always sync after:
- Creating or closing tasks
- Session end (automated via hook)
- Before generating reports
- After significant updates

```bash
bd sync
```

## Task Hierarchies

**Epic** (1-2 weeks): `bd create "User Management System" -d "Complete user CRUD with roles" -p P1`
**Story** (2-5 days): `bd create "User Profile Editing" -d "Users can edit profile" -p P1 --parent=beads-{epic}`
**Subtask** (0.5-2 days): `bd create "Profile API endpoint" -d "Create PUT /api/users/:id" -p P2 --parent=beads-{story}`

Link with dependencies to show hierarchy.

## Troubleshooting

**Task Not Found:**
```bash
bd sync                    # Sync first
bd list                    # List all
bd search "keyword"        # Search
```

**Status Not Updating:**
- Check for unsaved changes in editor
- Check git conflicts in .beads/
- Run bd sync

**Dependency Cycles:**
```bash
# Check before adding to avoid cycles
bd dep list beads-123
bd dep list beads-456
```

## Git Worktree Integration

```bash
bd worktree create beads-123    # Creates ../beads-123/, branch feature/beads-123
bd worktree list                # List worktrees
bd worktree remove beads-123    # Remove worktree
```

**Reference:** See git-worktree-workflow skill for detailed patterns.

## Agent-Specific Usage

**Technical Lead:**
- Validates tickets exist and are open
- Reads task context for planning (bd show, bd comments)
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

**Reference:** See k2-dev-reference.md for complete command syntax, priority levels, and task granularity guidelines.
