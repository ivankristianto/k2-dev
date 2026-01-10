# k2-dev - Multiagent Development Orchestration

Enterprise-grade multiagent orchestration plugin for Claude Code that manages end-to-end software development workflows with specialized AI agents.

## Overview

k2-dev simulates a complete development team with specialized agents and skills:

**Agents:**
- **Technical Lead**: Orchestrates workflows, manages worktrees
- **Engineer**: Implements features following plans
- **Reviewer**: Performs code reviews and quality validation

**Skills (main conversation context):**
- **Planning**: Analyzes requirements, creates actionable plans
- **Test Planning**: Creates test plans and validates implementations

## Features

- 🎯 **Task-Driven Development**: Integrated with beads task management
- 🌲 **Git Worktree Workflow**: Isolated workspaces for each task
- ✅ **Quality Gates**: Enforces standards from AGENTS.md and CLAUDE.md
- 🔄 **Structured Workflows**: Planning → Implementation → Review → Merge
- 📋 **Automated Reporting**: Track progress and generate status reports
- 🧪 **Test Planning**: Automated test strategy and coverage planning

## Prerequisites

- [beads](https://github.com/steveyegge/beads) - Task management system
- [beads_viewer](https://github.com/steveyegge/beads) - Interactive task viewer for beads (`bv` command)
- [GitHub CLI](https://cli.github.com/) - Command-line tool for GitHub operations (`gh` command)
- Git with worktree support
- Project configuration files (optional but recommended):
  - `AGENTS.md` - Agent behavior guidelines
  - `CLAUDE.md` - Claude-specific project standards
  - `constitution.md` - Project principles and constraints

## Installation

Install via Claude Marketplace:

```bash
claude plugin install k2-dev@ivankristianto/k2-dev
```

**Alternative: Manual Installation**

```bash
# Clone or copy to Claude plugins directory
mkdir -p ~/.claude/plugins
cp -r k2-dev ~/.claude/plugins/

# Enable in Claude Code
claude --plugins k2-dev
```

## Commands

### `/k2:start`

Start implementation workflow for one or more tickets.

**Usage:**

```
/k2:start beads-123
/k2:start beads-123,beads-234,beads-345
/k2:start beads-123 --skip-worktree
/k2:start                              # No ticket - launches beads_viewer (bv) for interactive selection
```

**Options:**

- `--skip-worktree`: Create branch in main repository instead of creating an isolated worktree

**Interactive Mode:**

When called without a ticket ID, `/k2:start` will launch the `bv` (beads_viewer) command for interactive task selection. This provides a visual interface to browse and select tasks from your beads project.

**Workflow:**

1. Technical Lead validates tickets exist
2. Creates git worktree (branch: `feature/beads-{id}`) or creates branch in main repo if `--skip-worktree` is used
3. Reads task description and comments
4. Hands off to Engineer for implementation
5. Engineer implements with self-review
6. Creates GitHub PR automatically
7. Reviewer validates against quality gates
8. Iterates up to 2 times if needed
9. Creates follow-up tickets for remaining issues
10. Technical Lead merges and cleans up

### `/k2:planner`

Start planning workflow for new features using the Planning skill.

**Usage:**

```
/k2:planner
```

**Workflow:**

1. Planning skill analyzes requirement with full codebase access
2. Asks clarifying questions directly in main conversation
3. Creates initial plan
4. Invokes Technical Lead agent for architectural review
5. Planning skill incorporates feedback and refines plan
6. Converts to beads tasks with dependencies

**Benefits:**
- Faster execution (no agent spawning overhead)
- Easier debugging (everything in main conversation context)
- Direct user interaction (questions asked directly)

### `/k2:report`

Generate status report for a ticket.

**Usage:**

```
/k2:report beads-123
```

**Output:** Structured markdown report with:

- Task summary
- Current status
- Assignee
- Comments and history
- Related tickets

### `/k2:test`

Create test plan for a ticket using the Test Planning skill.

**Usage:**

```
/k2:test beads-123
```

**Workflow:**

1. Test Planning skill analyzes implementation
2. Creates comprehensive test plan
3. Defines test cases and coverage strategy
4. Adds test plan to beads comments
5. Coordinates with Engineer for implementation

### `/k2:doctor`

Check system prerequisites and project configuration.

**Usage:**

```
/k2:doctor
```

**What it checks:**

- **System Prerequisites**: Verifies `bd` (beads), `bv` (beads_viewer), `gh` (GitHub CLI), and `git` with worktree support are installed
- **Project Configuration**: Checks for `AGENTS.md`, `CLAUDE.md`, and `constitution.md` files
- **Beads Setup**: Validates `.beads/` directory and beads operational status
- **Beads Task Statistics**: Analyzes task health (open/closed/tombstone counts) and suggests maintenance with `bd compact` if needed

**Output:** Diagnostic report showing which requirements are met, missing, or need attention, plus actionable recommendations for beads maintenance.

## Agents

### Technical Lead

- **Role**: Workflow orchestrator and system architect
- **Responsibilities**:
  - Validate ticket status
  - Create and manage git worktrees
  - Coordinate agent handoffs (hub model)
  - Make architectural decisions
  - Merge PRs and cleanup
- **Tools**: All tools, full codebase access

### Engineer

- **Role**: Implementation specialist
- **Responsibilities**:
  - Execute implementation following plans
  - Perform self-review against quality gates
  - Create well-structured GitHub PRs
  - Respond to review feedback
  - Create follow-up tickets
- **Tools**: Read, Write, Edit, Bash, Grep, Glob

### Reviewer

- **Role**: Code quality validator
- **Responsibilities**:
  - Review code changes for quality and security
  - Validate against AGENTS.md/CLAUDE.md standards
  - Provide feedback on GitHub PRs
  - No code changes (review only)
- **Tools**: Read, Grep, Glob, Bash (read-only operations)

## Skills

The plugin provides specialized skills that execute in the main conversation context:

**Active Workflow Skills:**
1. **Planning** (`/k2:planner`): Requirements analysis, task creation, beads task management
2. **Test Planning** (`/k2:test`): Test strategy, test case definition, coverage planning

**Knowledge Domain Skills:**
3. **beads-integration**: Working with beads task management
4. **git-worktree-workflow**: Managing git worktrees for isolation
5. **quality-gates**: Reading and enforcing project standards
6. **pr-creation**: Creating well-structured pull requests
7. **test-planning**: Test strategy and planning (reference)
8. **code-review-standards**: Code review best practices

## Configuration

### Optional Settings

Create `.claude/k2-dev.local.md` to customize behavior:

```markdown
---
defaultBranch: main
prTemplateCustomization: true
qualityGateThresholds:
  codeComplexity: medium
  testCoverage: 80
---

# K2-Dev Configuration

Custom configuration for this project...
```

### Project Configuration Files

#### AGENTS.md (Project Root)

Define agent behavior, quality gates, and file validation patterns:

```markdown
# Agent Guidelines

## Quality Gates

- All TypeScript files must pass type checking
- Test coverage minimum: 80%
- No console.log in production code

## File Validation Patterns

Validate these file types before changes:

- src/\*_/_.ts
- src/\*_/_.tsx
- lib/\*_/_.js

## Code Review Standards

...
```

#### CLAUDE.md (Project Root)

Claude-specific project standards and patterns.

#### constitution.md (Project Root)

Project principles and constraints all agents must follow.

## Workflow Details

### Implementation Workflow

```
User: /k2:start beads-123,beads-234
    ↓
Technical Lead:
  - Validates tickets exist and are open
  - Creates worktree: feature/beads-123
  - Reads task descriptions
    ↓
Engineer:
  - Implements changes
  - Self-reviews against quality gates
  - Creates GitHub PR
    ↓
Reviewer:
  - Reviews code changes
  - Comments on GitHub PR
  - Validates quality gates
    ↓
[Iteration: max 2 times]
    ↓
Engineer (if needed):
  - Creates follow-up tickets (P0 for critical)
    ↓
Technical Lead:
  - Closes beads tasks
  - Syncs with bd sync
  - Removes worktree
  - Provides report to user
```

### Planning Workflow

```
User: /k2:planner "Add user authentication"
    ↓
Planning Skill (main conversation context):
  - Analyzes requirement
  - Explores codebase
  - Asks clarifying questions directly
  - Creates initial plan
    ↓
Planning Skill invokes Technical Lead agent:
  - Reviews plan
  - Provides architectural feedback
    ↓
Planning Skill:
  - Refines plan
  - Iterates with Tech Lead
  - Converts to beads tasks
  - Sets up dependencies
  - Adds detailed descriptions
```

## Hooks

### PreToolUse Hook

Validates file changes against quality gates before Write/Edit operations.

- Checks file patterns defined in AGENTS.md
- Enforces project standards from CLAUDE.md
- Blocks non-compliant changes

### Stop Hook

Runs cleanup on session end:

- Always executes `bd sync`
- Ensures task state is synchronized
- Logs session summary

## Best Practices

1. **Always define quality gates** in AGENTS.md for your project
2. **Keep beads tasks updated** with clear descriptions and plans
3. **Use hierarchical tasks** for complex features
4. **Set dependencies** between tasks to maintain order
5. **Review AGENTS.md/CLAUDE.md** regularly and keep them current
6. **Follow PR templates** for consistency
7. **Create follow-up tickets** rather than extending reviews beyond 2 iterations

## Troubleshooting

### Ticket Validation Fails

- Ensure ticket exists: `bd show beads-123`
- Check ticket is open: `bd list --filter=id:beads-123`
- Verify beads is synced: `bd sync`

### Worktree Creation Fails

- Check git worktree list: `git worktree list`
- Remove stale worktrees: `git worktree prune`
- Ensure branch name doesn't conflict

### Quality Gate Validation Issues

- Review AGENTS.md file patterns
- Check if files match validation patterns
- Verify standards in CLAUDE.md are achievable

### Agent Coordination Issues

- All coordination happens through Technical Lead (hub model)
- Check Technical Lead has access to required tools
- Verify agent descriptions are clear

## Development

### Adding New Agents

Add new agent `.md` files to `agents/` directory following the template.

### Adding New Skills

Create skill directories under `skills/` with `SKILL.md` files.

### Customizing Workflows

Modify agent system prompts and coordination logic in agent files.

## License

MIT

## Support

For issues and questions, please refer to your project's support channels.
