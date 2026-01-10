---
name: k2:doctor
description: Check system prerequisites and project configuration
allowed-tools:
  - Bash
  - Read
  - Glob
---

# K2-Dev Doctor

Check that all prerequisites and project configuration files are properly set up for k2-dev workflows.

## What to Check

### 1. System Prerequisites

Check that these CLI tools are installed and accessible:

**Required:**
- `bd` (beads) - Task management system
- `bv` (beads_viewer) - Interactive task viewer
- `gh` (GitHub CLI) - GitHub operations
- `git` - Version control with worktree support

**Checks to perform:**
- Run `command -v <tool>` to verify each tool is in PATH
- Run `<tool> --version` to verify it's executable
- For git, check if `git worktree` command is available

### 2. Project Configuration Files

Check for these files in the project root:

**Recommended (but optional):**
- `AGENTS.md` - Agent behavior guidelines and quality gates
- `CLAUDE.md` - Claude-specific project standards
- `constitution.md` - Project principles and constraints

**Checks to perform:**
- Check if each file exists in current working directory
- If file exists, read first few lines to verify it's not empty
- Report which files are present and which are missing

### 3. Beads Initialization

Check if beads is properly initialized:

**Checks to perform:**
- Check if `.beads/` directory exists
- Run `bd list` to verify beads is working
- Report beads status

### 4. Beads Task Statistics

Analyze beads task health and suggest maintenance:

**Checks to perform:**
- Run `bd stats` to get task statistics
- Count closed and tombstone tasks
- Calculate percentage of closed/tombstone vs total tasks
- Suggest pruning if there are many closed/tombstone tasks

**Pruning recommendations:**
- If closed tasks > 50: Suggest `bd compact` to prune old closed tasks
- If tombstone tasks > 20: Suggest cleanup of tombstone tasks
- Explain benefits of compaction (reduced git repo size, faster operations)

## Output Format

Provide a clear diagnostic report:

```
🏥 K2-Dev System Check
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Prerequisites
  ✓ beads (bd) - v1.2.3
  ✓ beads_viewer (bv) - v0.5.0
  ✓ GitHub CLI (gh) - v2.40.0
  ✓ Git - v2.39.0
  ✓ Git worktree support - Available

✅ Project Configuration
  ✓ AGENTS.md - Present (1,234 bytes)
  ✓ CLAUDE.md - Present (5,678 bytes)
  ⚠ constitution.md - Missing (optional)

✅ Beads Setup
  ✓ .beads/ directory - Present
  ✓ Beads operational - 15 total issues

📊 Beads Task Statistics
  • Total tasks: 150
  • Open: 15 (10%)
  • In Progress: 5 (3%)
  • Closed: 85 (57%)
  • Tombstone: 45 (30%)

  ⚠ Maintenance Recommended:
    → Run `bd compact` to prune 85 closed tasks
    → This will reduce git repo size and improve performance
    → Closed tasks will be semantically summarized and archived

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: All critical requirements met ✓
Note: Consider running beads maintenance (see recommendations above)
```

## Error Handling

If any required prerequisite is missing:
1. Clearly mark it with ❌
2. Provide installation instructions
3. Mark status as "Action required"

If optional files are missing:
1. Mark with ⚠️
2. Note that they are optional but recommended
3. Explain what each file is used for

## Implementation Notes

- Use `Bash` tool to check for CLI tools and beads statistics
- Use `Read` and `Glob` tools to check for configuration files
- Run `bd stats` to get beads task statistics
- Parse the stats output to count closed and tombstone tasks
- Do NOT use `Task` tool - this should be fast and synchronous
- Provide actionable feedback if anything is missing
- Keep output clean and easy to read
- Be specific about maintenance commands (e.g., `bd compact --help` for options)
