# K2-Dev Quick Reference

Common commands, patterns, and concepts referenced by k2-dev skills.

## Beads CLI Commands

### Task Management

```bash
bd show beads-123              # Show task details
bd list                        # List all tasks
bd list --filter=status:open   # Filter by status
bd list --filter=priority:P0   # Filter by priority
bd search "keyword"            # Search tasks
```

### Task Creation

```bash
bd create "Title" -d "Description" -p P1               # Create task
bd create --title="Title" --parent=beads-100           # Create with parent
```

### Task Updates

```bash
bd update beads-123 --status in_progress    # Update status
bd update beads-123 --priority P0           # Update priority
bd update beads-123 --assignee engineer     # Assign task
```

### Comments

```bash
bd comments beads-123                       # View comments
bd comments beads-123 add "Comment text"    # Add comment
```

### Dependencies

```bash
bd dep add beads-456 beads-123             # 456 depends on 123
bd dep list beads-123                      # List dependencies
bd dep remove beads-456 beads-123          # Remove dependency
```

### Synchronization

```bash
bd sync                                    # Sync with remote
```

### Worktrees

```bash
bd worktree create beads-123               # Create worktree
bd worktree list                           # List worktrees
bd worktree remove beads-123               # Remove worktree
```

## Git Worktree Basics

### Structure

```
project/               # Main worktree (main branch)
├── .git/
└── src/

../beads-123/         # Linked worktree (feature branch)
├── .git -> project/.git/worktrees/beads-123
└── src/
```

### Commands

```bash
git worktree add ../beads-123 -b feature/beads-123    # Create worktree
git worktree list                                      # List worktrees
git worktree remove ../beads-123                       # Remove worktree
git worktree prune                                     # Prune stale entries
```

## GitHub CLI (gh) Commands

### Pull Requests

```bash
gh pr create --title "Title" --body "Body"    # Create PR
gh pr view 123                                # View PR
gh pr view 123 --web                          # View in browser
gh pr diff 123                                # View diff
gh pr checks 123                              # Check CI status
```

## Quality Standards Files

### AGENTS.md

Defines:

- Quality gates (type checking, test coverage, linting)
- File validation patterns
- Agent behavior guidelines
- Code review standards
- Project architecture patterns
- Preferred libraries and tools
- File organization conventions
- Coding style preferences

### constitution.md

Defines:

- Core project principles
- Non-negotiable constraints
- Security policies
- Compliance requirements

## Test Coverage Levels

| Coverage Type  | Threshold    | Priority  |
| -------------- | ------------ | --------- |
| Critical Paths | 100%         | Must have |
| Unit Tests     | 80%+         | Standard  |
| Integration    | Major flows  | Important |
| E2E Tests      | Key journeys | Important |

## Priority Levels

| Level  | Meaning            | Use Cases                            |
| ------ | ------------------ | ------------------------------------ |
| **P0** | Critical, blocking | Security, data loss, production down |
| **P1** | High priority      | Key features, important bugs         |
| **P2** | Medium priority    | Improvements, non-critical bugs      |
| **P3** | Low priority       | Nice-to-have, backlog                |

## Task Granularity

| Type        | Duration   | Scope                       |
| ----------- | ---------- | --------------------------- |
| **Epic**    | 1-2 weeks  | Major feature or user value |
| **Story**   | 2-5 days   | User-facing capability      |
| **Subtask** | 0.5-2 days | Technical implementation    |

## Common File Patterns

### Finding Files

```bash
glob "**/*auth*"              # Find auth-related files
glob "**/routes/*"            # Find routes
glob "**/*.test.{js,ts}"      # Find test files
```

### Searching Content

```bash
grep "pattern" --output_mode=content           # Search with context
grep "TODO\|FIXME" --output_mode=files_with_matches    # Find TODOs
```

## Test Types

| Type            | Focus                  | When to Use                      |
| --------------- | ---------------------- | -------------------------------- |
| **Unit**        | Individual functions   | Business logic, utilities        |
| **Integration** | Component interactions | APIs, databases, services        |
| **E2E**         | User workflows         | Critical flows, UI               |
| **Performance** | Load and speed         | High-traffic features            |
| **Security**    | Security controls      | Auth, validation, sensitive data |

## Review Severity Levels

| Level          | Impact    | Examples                            |
| -------------- | --------- | ----------------------------------- |
| **P0**         | Critical  | Security vulnerabilities, data loss |
| **P1**         | Important | Logic errors, quality gates         |
| **P2**         | Minor     | Style issues, optimizations         |
| **Suggestion** | Optional  | Refactoring, alternatives           |

## Branch Naming

```
feature/beads-{id}      # Feature branches
```

## K2-Dev Workflow States

```
open → in_progress → (review) → closed
                   ↓
                 blocked
```

## Standard PR Template Structure

```markdown
## Summary

{Brief overview}

## Changes

- {Specific changes}

## Testing

- {Test coverage}

## Reviewer Notes

- {Important context}

## References

- Implements: beads-{id}
```

## Git Commit Format

```
<type>: <summary>

<type> = feat|fix|refactor|docs|test|chore|perf|security
```

---

This reference document reduces duplication across skill files. Skills reference sections here instead of repeating content.
