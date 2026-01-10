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

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: All critical requirements met ✓
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

- Use `Bash` tool to check for CLI tools
- Use `Read` and `Glob` tools to check for configuration files
- Do NOT use `Task` tool - this should be fast and synchronous
- Provide actionable feedback if anything is missing
- Keep output clean and easy to read
