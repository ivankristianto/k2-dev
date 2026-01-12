---
name: k2:planner
description: Enhance beads task descriptions or create new implementation plans
argument-hint: beads-123 [beads-456 ...] or feature description
allowed-tools:
  - Read
  - Bash
  - Task
  - AskUserQuestion
---

# K2:Planner - Requirements Planning & Task Enhancement

Enhances existing beads task descriptions or creates new implementation plans from scratch.

## Two Modes

### Mode 1: Enhance Existing Tickets (when beads IDs provided)

1. **Parse ticket IDs** from argument (space-separated, e.g., `beads-123 beads-456`)
2. **Visit each ticket** one by one:
   ```bash
   bd show {ticket-id}              # Get task details
   ```
3. **Evaluate description quality:**
   - Check for: clear requirements, acceptance criteria, technical approach, file references
   - If adequate → note as "already descriptive"
   - If lacking → run Planning skill to enhance
4. **Enhance via Planning skill** (if needed):
   ```bash
   Skill tool with:
   - skill: "k2-dev:planning"
   - args: "Enhance description for task {ticket-id}: {title}\n\nCurrent description:\n{description}"
   ```
5. **Update existing ticket** instead of creating new:
   ```bash
   # Read the beads file directly
   read .beads/tickets/{ticket-id}.md

   # Update the description section
   # (replace content after the `---` frontmatter block)

   # Sync changes
   bd sync
   ```

### Mode 2: Create New Plan (when no beads IDs, feature description provided)

1. **Capture requirement:**
   - If argument provided → use it
   - If no argument → ask: "What feature or change would you like to plan?"

2. **Invoke Planning skill:**
   ```
   Skill tool with:
   - skill: "k2-dev:planning"
   - args: "User wants to: {user requirement}"
   ```

3. **Skill executes workflow:**
   - Analysis: Explore codebase (Read/Grep/Glob/Bash)
   - Clarification: Ask questions directly in main conversation
   - Planning: Create detailed implementation plan
   - Tech Review: Invoke Technical Lead agent for feedback
   - Refinement: Incorporate feedback
   - **Task Creation:** Convert to beads tasks using `bd create`
   - **Sync & Report:** Run `bd sync`, generate task graph

## Configuration Files

Planning skill references: AGENTS.md, CLAUDE.md, (docs|specs)/constitution.md

## Usage Examples

```bash
/k2:planner beads-123                     # Enhance single ticket
/k2:planner beads-123 beads-456 beads-789 # Enhance multiple tickets
/k2:planner Add user authentication       # Create new plan
/k2:planner                               # Interactive - asks for requirement
```

## Enhanced Description Format

When updating existing tickets, use this format:

```markdown
## Summary

{2-3 sentence high-level summary}

## Requirements

- {Specific requirement}
- {Specific requirement}

## Technical Approach

{How to implement, key files, patterns to follow}

## Acceptance Criteria

- [ ] {Verify requirement 1}
- [ ] {Verify requirement 2}

## Files to Modify

- `{path}` - {what changes}

## Dependencies

- {blocking tasks, if any}

## Testing Strategy

{How to verify the implementation}
```

## Final Report (Mode 1 - Enhancement)

```markdown
## Enhancement Complete

### Tickets Processed

| Ticket | Status | Notes |
|--------|--------|-------|
| beads-123 | Enhanced | Added acceptance criteria |
| beads-456 | Already Descriptive | No changes needed |
| beads-789 | Enhanced | Added technical approach |

### Summary

- Total: {count} tickets
- Enhanced: {count}
- Already adequate: {count}
```

## Final Report (Mode 2 - New Plan)

```markdown
## Feature: {name}

### Summary

{2-3 sentence high-level summary}

### Tickets in Dependency Graph

{list tickets showing relationships}

### Unblocked Tickets

- {list ready-to-start tickets}

### Blocked Tickets

- {list tickets with dependencies}
```

## Success Indicators

**Mode 1:** All tickets have clear, actionable descriptions with requirements, acceptance criteria, and technical approach.

**Mode 2:** Requirements clarified → ✅ Tech Lead approved → ✅ Beads tasks created → ✅ Dependencies set → ✅ Execution order clear
