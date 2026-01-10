# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

k2-dev is a multiagent orchestration plugin for Claude Code that simulates a complete development team. It coordinates specialized AI agents and skills (Technical Lead, Engineer, Reviewer, plus Planning and Test Planning skills) for enterprise-grade software development workflows with beads task management integration.

**Agent vs Skills:**

- **Agents** (Technical Lead, Engineer, Reviewer, PR Writer): Run as isolated subagents for complex, multi-step workflows with isolated context
- **Skills** (Planning, Test Planning): Execute in main conversation context for faster execution and direct user interaction

## Architecture

### Hub-and-Spoke Model

The plugin uses a hub-and-spoke coordination pattern:

- **Technical Lead** is the hub that orchestrates all workflows
- Other agents (Engineer, Reviewer, PR Writer) are spokes launched by Technical Lead and report back
- No direct agent-to-agent communication; all coordination flows through Technical Lead

### Plugin Structure

```
k2-dev/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── agents/                   # Agent definitions (system prompts)
│   ├── technical-lead.md    # Hub: orchestrates workflows
│   ├── engineer.md          # Implementation
│   ├── reviewer.md          # Code review (read-only)
│   └── pr-writer.md         # Pull request creation
├── commands/                 # User-invocable slash commands
│   ├── start.md             # /k2:start - begin implementation
│   ├── planner.md           # /k2:planner - create plans (invokes skill)
│   ├── report.md            # /k2:report - status reports
│   └── test.md              # /k2:test - test planning (invokes skill)
├── skills/                   # Knowledge domains and active workflows
│   ├── planner/             # Planning skill (main context execution)
│   ├── tester/              # Test Planning skill (main context execution)
│   ├── beads-integration/   # Beads CLI patterns
│   ├── git-worktree-workflow/ # Git worktree management
│   ├── quality-gates/       # Enforcing standards
│   ├── pr-creation/         # Pull request structure
│   ├── test-planning/       # Test strategy (reference)
│   └── code-review-standards/ # Review practices
└── hooks/
    └── hooks.json           # PreToolUse (quality gates) & Stop (sync)
```

### Agent Tool Access

| Agent          | Tools                               | Purpose                                |
| -------------- | ----------------------------------- | -------------------------------------- |
| Technical Lead | All tools                           | Orchestration, coordination, decisions |
| Engineer       | Read, Write, Edit, Bash, Grep, Glob | Implementation only                    |
| Reviewer       | Read, Grep, Glob, Bash              | Read-only review                       |
| PR Writer      | Read, Write, Bash, Grep, Glob, TodoWrite | Pull request creation                  |

**Skills** (execute in main conversation context):
| Skill | Access Method | Purpose |
| ------------------ | ------------- | -------------------------------------- |
| Planning | `/k2:planner` | Requirements analysis, task creation |
| Test Planning | `/k2:test` | Test strategy, test case definition |

## Key Workflows

### Implementation Flow (`/k2:start`)

1. Technical Lead validates tickets exist and are open
2. Creates git worktree: `bd worktree create ../worktrees/feature-beads-123` OR creates branch in main repo if `--skip-worktree` flag is used
3. Reads task details via `bd show` and `bd comments`
4. Launches Engineer agent for implementation
5. Engineer implements, self-reviews against AGENTS.md/CLAUDE.md, pushes changes
6. Technical Lead launches internal PR Writer agent to create PR
7. Technical Lead launches Reviewer agent to validate code (max 2 iterations)
8. Technical Lead merges, closes tickets, syncs beads, removes worktree (or deletes branch if using `--skip-worktree`)

**Why PR Creation Uses Internal Subagent:**

- **Self-contained plugin**: No dependency on external agents - pr-writer is part of k2-dev
- **Context isolation**: PR creation knowledge doesn't bloat subsequent review/merge phases
- **Token efficiency**: One-time spawn cost (~2-3K) vs carrying PR context through 3+ phases (~15-30K)
- **Self-sufficient**: PR Writer can analyze changes independently (git log, git diff)
- **Cleaner separation**: PR creation is a distinct, self-contained task

### Planning Flow (`/k2:planner`)

1. Planning skill analyzes requirements with full codebase access
2. Explores codebase using main conversation tools (Glob/Grep/Read)
3. Asks clarifying questions directly in main conversation
4. Creates structured implementation plan
5. Invokes Technical Lead agent via Task tool for architectural review
6. Planning skill incorporates feedback and refines plan
7. Converts to hierarchical beads tasks with dependencies

**Benefits of skill-based approach:**

- Faster execution (no agent spawning overhead)
- Easier debugging (everything in main conversation context)
- Direct user interaction (questions asked directly, not via AskUserQuestion)

### Review Iteration Limit

- Maximum 2 review iterations per PR
- After 2 iterations: create follow-up tickets (P0 for critical issues)
- Prevents endless review loops

## Beads Integration

The plugin integrates deeply with [beads](https://github.com/steveyegge/beads) task management:

- Tickets stored as markdown in `.beads/` directory
- Git-backed persistence across conversation compaction
- Task hierarchies: epics → stories → subtasks
- Dependencies between tasks via `bd dep add`
- Automatic sync via Stop hook: `bd sync`

### Essential Beads Commands

```bash
bd show beads-123              # Get task details and comments
bd list --filter=status:open   # List open tasks
bd update beads-123 --status in_progress  # Update status
bd comments beads-123 add "..." # Add implementation notes
bd dep add beads-456 beads-123 # Add dependency
bd sync                        # Sync with remote (auto via hook)
```

## Quality Gates Enforcement

### PreToolUse Hook

The `hooks/hooks.json` defines a PreToolUse hook that:

- Intercepts Write/Edit operations
- Checks if file matches validation patterns in AGENTS.md
- Validates changes against AGENTS.md, CLAUDE.md, constitution.md
- Blocks non-compliant changes

### Project Configuration Files (Required in project root)

- **AGENTS.md**: Quality gates, file validation patterns, agent guidelines
- **CLAUDE.md**: Claude-specific project standards
- **constitution.md**: Project principles and constraints

If these don't exist, quality gate validation is skipped.

## Git Worktree Workflow

Each implementation uses isolated worktrees by default, or can use main repository with `--skip-worktree`:

**Default (Worktree Mode):**

```bash
# Create worktree (Technical Lead)
bd worktree create ../worktrees/feature-beads-123

# Work in isolated directory
cd ../feature-beads-123

# After merge and cleanup
bd worktree remove ../worktrees/feature-beads-123
```

Benefits:

- Isolated workspaces per task
- No context switching in main tree
- Parallel work on multiple tickets
- Clean cleanup after completion

**Alternative (--skip-worktree Mode):**

```bash
# Create branch in main repo
git checkout -b feature/beads-123

# Work in main repository
# (current directory)

# After merge
git checkout main
# Branch deleted via PR merge --delete-branch
```

Benefits:

- Simpler workflow for single-task development
- No worktree management overhead
- Familiar git branch workflow

## Command Development

### Creating New Commands

Add markdown files to `commands/` with YAML frontmatter:

```markdown
---
name: k2:mycommand
description: What this command does
argument-hint: ticket-id
allowed-tools:
  - Read
  - Bash
  - Task
---

# Command Title

Command instructions...
```

### Agent System Prompts

Agent files in `agents/` are pure markdown system prompts:

- Define agent role, responsibilities, behaviors
- Specify workflow patterns
- Define tool usage constraints
- Include coordination instructions

## Skills System

Skills are reusable knowledge domains invoked via the Skill tool during agent execution:

- **Frontmatter**: name, description, version
- **SKILL.md**: Comprehensive documentation
- Used by agents when specific knowledge is needed
- Skills provide reference documentation and patterns (e.g., pr-creation skill documents PR best practices)

## Configuration

### Local Settings (Optional)

Create `.claude/k2-dev.local.md` in user's project:

```markdown
---
defaultBranch: main
maxReviewIterations: 2
---

# Project-specific k2-dev settings
```

### Marketplace Configuration

The plugin is published to `k2-dev-marketplace` and installed via:

```bash
claude plugin install k2-dev@k2-dev-marketplace
```

## Development Workflow for This Plugin

### Testing Agent Changes

1. Edit agent `.md` files in `agents/`
2. Test via `/k2:start beads-test-ticket`
3. Verify agent behavior matches expectations
4. Update documentation if workflow changes

### Adding New Skills

1. Create directory under `skills/`
2. Add `SKILL.md` with frontmatter
3. Reference in agent prompts where relevant
4. Document usage in README.md

### Modifying Workflows

1. Identify which agents participate in workflow
2. Update agent system prompts in `agents/`
3. Update command documentation in `commands/`
4. Update README.md workflow diagrams
5. Test end-to-end with sample ticket

## Important Constraints

- **No direct agent-to-agent communication**: All flows through Technical Lead
- **Reviewer is read-only**: Cannot use Write/Edit tools
- **Max 2 review iterations**: Prevents loops, creates follow-up tickets instead
- **Worktrees outside main tree**: Created at `../feature-beads-{id}` level
- **Stop hook always syncs**: Ensures beads state is persisted
- **Quality gates are project-specific**: Each project defines AGENTS.md/CLAUDE.md

## Plugin Versioning

Update `plugin.json` version field using semantic versioning:

- Patch: Bug fixes, documentation
- Minor: New features, agent improvements
- Major: Breaking changes, workflow restructuring

Current version: 0.2.0

**Version 0.2.0 Changes:**

- Converted Planner and Tester from agents to skills
- Skills now execute in main conversation context for faster execution
- Added `/planner` and `/tester` command aliases
- Improved user experience with direct interaction in main conversation

## Commit and Push Workflow

**IMPORTANT**: When the user asks to commit and push changes, you MUST bump the version number in BOTH files before executing git push:

1. **Version files to update:**

   - `.claude-plugin/plugin.json` (line 3: `"version"` field)
   - `.claude-plugin/marketplace.json` (line 12: `"version"` field inside plugins array)

2. **Version bumping rules:**

   - Use semantic versioning: MAJOR.MINOR.PATCH
   - Patch: Bug fixes, documentation updates, minor changes
   - Minor: New features, skills, commands, or agent improvements
   - Major: Breaking changes, workflow restructuring, incompatible changes

3. **Required workflow:**

   ```bash
   # 1. Update version in both files (use Edit tool)
   # 2. Stage all changes including version files
   git add .
   # 3. Commit with descriptive message
   git commit -m "..."
   # 4. Push to remote
   git push
   ```

4. **Example version bump:**
   - Current: `0.2.1`
   - For documentation fix: `0.2.2`
   - For new command: `0.3.0`
   - For breaking change: `1.0.0`

**Never push without bumping both version files first.** Always keep the versions synchronized between `plugin.json` and `marketplace.json`.
