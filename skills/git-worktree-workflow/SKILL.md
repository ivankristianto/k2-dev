---
name: Git Worktree Workflow
description: This skill should be used when the user asks to "create a worktree", "manage git worktrees", "work on multiple branches simultaneously", "isolate task work", "use bd worktree", or needs guidance on git worktree patterns for task isolation in k2-dev workflows.
version: 0.1.0
---

# Git Worktree Workflow Skill

## Overview

Git worktrees enable working on multiple branches simultaneously by creating separate working directories. In k2-dev, each task gets its own worktree for complete isolation.

**Key Benefits:**

- Work on multiple tasks without branch switching
- Complete isolation prevents conflicts
- Clean separation of concerns
- Easy context switching
- Safe cleanup without losing work

## Core Concepts

### What is a Git Worktree?

A worktree is an additional working directory linked to the same repository:

- Shares git history and objects
- Has its own working directory
- Checks out a different branch
- Independent of other worktrees

**Example structure:**

```
project/               # Main worktree (main branch)
├── .git/
├── .beads/
└── src/

../beads-123/         # Linked worktree (feature/beads-123 branch)
├── .git -> project/.git/worktrees/beads-123
├── .beads/
└── src/
```

### K2-Dev Worktree Pattern

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
# Create worktree for task
bd worktree create beads-123
```

This:

- Creates `../beads-123/` directory
- Creates branch `feature/beads-123`
- Checks out the branch
- Links to main repository
- Updates beads context

**Location:**

- Default: `../beads-{id}/` (sibling to main repo)
- Keeps worktrees organized
- Easy to find and manage

### Using Git Directly

For manual control:

```bash
# Create worktree with specific branch
git worktree add ../beads-123 -b feature/beads-123

# Create from existing branch
git worktree add ../beads-123 feature/beads-123
```

### Multiple Tickets in One Worktree

When k2:start receives multiple tickets:

```bash
# Use first ticket ID for worktree name
bd worktree create beads-123

# Work on beads-123, beads-234, beads-345 in same worktree
# All changes go to feature/beads-123 branch
```

## Working in Worktrees

### Switching Context

**Navigate to worktree:**

```bash
cd ../beads-123
```

**Verify worktree:**

```bash
pwd
git branch --show-current
```

**Check status:**

```bash
git status
```

### Making Changes

Work normally in worktree:

```bash
# Edit files
vi src/auth/middleware.ts

# Stage changes
git add src/auth/middleware.ts

# Commit
git commit -m "Add JWT middleware"

# Push to remote
git push -u origin feature/beads-123
```

### Returning to Main Repo

```bash
cd - # Or use absolute path
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

### Checking Worktree Status

```bash
# From any worktree or main repo
git worktree list

# Check specific worktree
cd ../beads-123 && git status
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

# Each worktree is independent
# No branch switching needed
```

## Branching Strategy

### Branch Naming

**Standard pattern:**

```
feature/beads-{id}
```

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

### Base Branch

Create worktrees from main/master:

```bash
# Worktree automatically branches from current HEAD
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

**Merge main if preferred:**

```bash
cd ../beads-123
git merge origin/main
```

## Best Practices

### DO

✅ **Create worktree per task**

- Each beads ticket gets its own worktree
- Complete isolation
- Easy context switching

✅ **Use consistent naming**

- `../beads-{id}/` for location
- `feature/beads-{id}` for branch
- Predictable and organized

✅ **Clean up after merge**

- Remove worktree after PR merged
- Keeps workspace tidy
- Prevents confusion

✅ **Verify before creating**

```bash
# Check worktree doesn't already exist
git worktree list | grep beads-123
```

✅ **Stay in worktree context**

```bash
# Work entirely in worktree until PR merged
cd ../beads-123
# ... all work here ...
```

### DON'T

❌ **Don't nest worktrees**

- Don't create worktree inside another worktree
- Keep flat structure

❌ **Don't manually delete worktree directories**

```bash
# Wrong:
rm -rf ../beads-123

# Right:
bd worktree remove ../beads-123
```

❌ **Don't reuse worktree for different tasks**

- One worktree = one task
- Create new worktree for new task

❌ **Don't forget to push**

```bash
# Before creating PR, push changes
git push -u origin feature/beads-123
```

## Troubleshooting

### Worktree Already Exists

**Error:**

```
fatal: '../beads-123' already exists
```

**Solution:**

```bash
# Check existing worktrees
git worktree list

# Remove if stale
git worktree remove ../beads-123

# If directory deleted manually, prune
git worktree prune
```

### Branch Already Exists

**Error:**

```
fatal: a branch named 'feature/beads-123' already exists
```

**Solution:**

```bash
# Check if worktree exists
git worktree list

# If no worktree, delete branch and recreate
git branch -D feature/beads-123
bd worktree create beads-123
```

### Can't Remove Worktree

**Error:**

```
fatal: validation failed, cannot remove working tree
```

**Solution:**

```bash
# Check for uncommitted changes
cd ../beads-123
git status

# Commit or stash changes
git add .
git commit -m "WIP"

# Or force remove (caution: loses changes)
git worktree remove --force ../beads-123
```

### Lost Worktree Directory

**Scenario:** Worktree directory deleted but git still tracks it

**Solution:**

```bash
# Prune stale worktree entries
git worktree prune

# Verify cleanup
git worktree list
```

## Integration with Beads

### Task-Worktree Relationship

Beads tracks worktree association:

```bash
# Create worktree via beads
bd worktree create beads-123

# Beads knows about the worktree
bd show beads-123  # Shows worktree info

# List task worktrees
bd worktree list
```

### Worktree in Task Context

When reading task context:

```bash
bd show beads-123
```

Output may include:

- Worktree location
- Branch name
- Work status

## Command Reference

| Action          | BD Command                     | Git Command                                          |
| --------------- | ------------------------------ | ---------------------------------------------------- |
| Create worktree | `bd worktree create beads-123` | `git worktree add ../beads-123 -b feature/beads-123` |
| List worktrees  | `bd worktree list`             | `git worktree list`                                  |
| Remove worktree | `bd worktree remove beads-123` | `git worktree remove ../beads-123`                   |
| Prune stale     | N/A                            | `git worktree prune`                                 |
| Force remove    | N/A                            | `git worktree remove --force ../beads-123`           |

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
   git add .
   git commit -m "feat: Add feature"
   git push -u origin feature/beads-123
   ```

3. **Engineer:** Create PR

   ```bash
   gh pr create --title "feat: Feature (beads-123)"
   ```

4. **After merge**

5. **Technical Lead:** Clean up
   ```bash
   cd ~/projects/my-project
   bd worktree remove beads-123
   bd update beads-123 --status closed
   bd sync
   ```

Follow this pattern for consistent, isolated task development with clean separation of concerns.
