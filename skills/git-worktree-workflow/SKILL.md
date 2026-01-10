---
name: Git Worktree Workflow
description: This skill should be used when the user asks to "create a worktree", "manage git worktrees", "work on multiple branches simultaneously", "isolate task work", "use bd worktree", or needs guidance on git worktree patterns for task isolation in k2-dev workflows.
version: 0.2.0
---

# Git Worktree Workflow Skill

## Overview

Git worktrees enable working on multiple branches simultaneously by creating separate working directories. In k2-dev, each task gets its own worktree for complete isolation.

**Key Benefits:** Work on multiple tasks without branch switching, complete isolation prevents conflicts, clean separation of concerns, easy context switching, safe cleanup without losing work.

**Reference:** See k2-dev-reference.md#git-worktree-basics for core git worktree concepts and commands.

## K2-Dev Worktree Pattern

For each task:
1. Create worktree: `../beads-{id}/`
2. Branch: `feature/beads-{id}`
3. Work in isolation
4. PR and merge
5. Remove worktree

## Creating Worktrees

### Using bd worktree (Recommended)

Beads provides integrated worktree management:

```bash
bd worktree create beads-123
```

This creates:
- Directory: `../beads-123/`
- Branch: `feature/beads-123`
- Links to main repository
- Updates beads context

**Location:** Worktrees created at `../beads-{id}/` (sibling to main repo) for easy organization and management.

### Using Git Directly (Manual Control)

```bash
# Create worktree with new branch
git worktree add ../beads-123 -b feature/beads-123

# Create from existing branch
git worktree add ../beads-123 feature/beads-123
```

**Reference:** See k2-dev-reference.md#git-worktree-basics for complete git worktree commands.

### Multiple Tickets in One Worktree

When `/k2:start` receives multiple tickets:

```bash
# Use first ticket ID for worktree name
bd worktree create beads-123

# Work on beads-123, beads-234, beads-345 in same worktree
# All changes go to feature/beads-123 branch
```

## Working in Worktrees

### Switching Context

```bash
# Navigate to worktree
cd ../beads-123

# Verify location
pwd
git branch --show-current

# Check status
git status
```

### Making Changes

Work normally in worktree:

```bash
# Edit files
vi src/auth/middleware.ts

# Stage and commit
git add src/auth/middleware.ts
git commit -m "Add JWT middleware"

# Push to remote
git push -u origin feature/beads-123
```

### Returning to Main Repo

```bash
cd -  # Or use absolute path
cd ~/projects/my-project
```

## Managing Worktrees

### Listing Worktrees

```bash
# Using bd worktree
bd worktree list

# Using git directly
git worktree list
```

**Output example:**
```
/Users/dev/project           abc1234 [main]
/Users/dev/beads-123         def5678 [feature/beads-123]
/Users/dev/beads-456         ghi9012 [feature/beads-456]
```

### Removing Worktrees

**After PR is merged:**

```bash
# Using bd worktree (recommended)
bd worktree remove beads-123

# Using git directly
git worktree remove ../beads-123

# If worktree directory was manually deleted
git worktree prune
```

**Force remove (if needed):**
```bash
git worktree remove --force ../beads-123
```

## K2-Dev Workflow Integration

### Technical Lead Responsibilities

**Creating worktree:**
```bash
# After validating ticket
bd worktree create beads-123

# Navigate to worktree
cd ../beads-123

# Verify setup
pwd
git branch --show-current
bd show beads-123
```

**Cleanup after merge:**
```bash
# Return to main repo
cd ~/projects/my-project

# Remove worktree
bd worktree remove beads-123

# Verify removal
git worktree list
```

### Engineer Workflow

**Start working:**
```bash
# Should already be in worktree (Tech Lead created it)
pwd  # Verify: ../beads-123

# Read task context
bd show beads-123
bd comments beads-123

# Start implementation
vi src/feature.ts
```

**During implementation:**
```bash
# Regular git workflow
git add .
git commit -m "Implement feature X"
git push
```

**Create PR:**
```bash
# From worktree
gh pr create --title "feat: Add feature X (beads-123)" \
             --body "$(cat PR_TEMPLATE.md)"
```

### Multiple Worktrees Simultaneously

Engineer can work on multiple tasks:

```bash
# Terminal 1: Work on beads-123
cd ../beads-123
vi src/auth.ts

# Terminal 2: Work on beads-456
cd ../beads-456
vi src/profile.ts

# Each worktree is independent - no branch switching needed
```

## Branching Strategy

### Branch Naming

**Standard pattern:** `feature/beads-{id}`

**Examples:**
```
feature/beads-123
feature/beads-456
feature/beads-789
```

**Why this pattern:**
- Clear association with task
- Easy to identify in PR list
- Consistent and predictable
- Sortable and searchable

**Reference:** See k2-dev-reference.md#branch-naming

### Base Branch

Create worktrees from main/master:

```bash
# Ensure main is up to date first
cd ~/projects/my-project
git checkout main
git pull

# Now create worktree
bd worktree create beads-123
```

### Keeping Up to Date

**Rebase on main:**
```bash
cd ../beads-123
git fetch origin
git rebase origin/main
```

**Merge main (if preferred):**
```bash
cd ../beads-123
git merge origin/main
```

## Best Practices

### DO

✅ **Create worktree per task** - Each beads ticket gets its own worktree for complete isolation

✅ **Use consistent naming** - `../beads-{id}/` for location, `feature/beads-{id}` for branch

✅ **Clean up after merge** - Remove worktree after PR merged to keep workspace tidy

✅ **Verify before creating:**
```bash
git worktree list | grep beads-123  # Check doesn't already exist
```

✅ **Stay in worktree context** - Work entirely in worktree until PR merged

### DON'T

❌ **Don't nest worktrees** - Keep flat structure, don't create worktree inside another

❌ **Don't manually delete worktree directories:**
```bash
# Wrong:
rm -rf ../beads-123

# Right:
bd worktree remove beads-123
```

❌ **Don't reuse worktree for different tasks** - One worktree = one task, create new for new task

❌ **Don't forget to push:**
```bash
# Before creating PR, push changes
git push -u origin feature/beads-123
```

## Troubleshooting

### Worktree Already Exists

**Error:** `fatal: '../beads-123' already exists`

**Solution:**
```bash
git worktree list  # Check existing worktrees
git worktree remove ../beads-123  # Remove if stale
git worktree prune  # If directory deleted manually
```

### Branch Already Exists

**Error:** `fatal: a branch named 'feature/beads-123' already exists`

**Solution:**
```bash
git worktree list  # Check if worktree exists
git branch -D feature/beads-123  # Delete branch if no worktree
bd worktree create beads-123  # Recreate
```

### Can't Remove Worktree

**Error:** `fatal: validation failed, cannot remove working tree`

**Solution:**
```bash
cd ../beads-123
git status  # Check for uncommitted changes
git add . && git commit -m "WIP"  # Commit or stash changes

# Or force remove (caution: loses changes)
git worktree remove --force ../beads-123
```

### Lost Worktree Directory

**Scenario:** Worktree directory deleted but git still tracks it

**Solution:**
```bash
git worktree prune  # Prune stale entries
git worktree list  # Verify cleanup
```

## Integration with Beads

Beads tracks worktree association:

```bash
# Create worktree via beads
bd worktree create beads-123

# Beads knows about the worktree
bd show beads-123  # Shows worktree info

# List task worktrees
bd worktree list
```

When reading task context, `bd show beads-123` may include:
- Worktree location
- Branch name
- Work status

## Workflow Summary

**Standard k2-dev workflow:**

1. **Technical Lead:** Create worktree
   ```bash
   bd worktree create beads-123
   cd ../beads-123
   ```

2. **Engineer:** Work in worktree
   ```bash
   # Already in ../beads-123
   # ... implement feature ...
   git add . && git commit -m "feat: Add feature" && git push -u origin feature/beads-123
   ```

3. **Engineer:** Create PR
   ```bash
   gh pr create --title "feat: Feature (beads-123)"
   ```

4. **After merge, Technical Lead:** Clean up
   ```bash
   cd ~/projects/my-project
   bd worktree remove beads-123
   bd update beads-123 --status closed
   bd sync
   ```

Follow this pattern for consistent, isolated task development with clean separation of concerns.

**Reference:** See k2-dev-reference.md for git worktree commands, branch naming, and common patterns.
