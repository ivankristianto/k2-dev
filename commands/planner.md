---
name: k2:planner
description: Start planning workflow to analyze requirements and create actionable beads tasks
argument-hint: optional feature description
allowed-tools:
  - Read
  - Task
  - AskUserQuestion
---

# K2:Planner - Requirements Planning

**ALWAYS use skill `k2-dev:planner` with reference to `skills/planner/SKILL.md` for comprehensive planning patterns and workflow steps.**

Invoke Planning skill to analyze requirements, clarify details, and create structured beads tasks with dependencies.

## Benefits of Skill-Based Approach

- **Faster:** No agent spawning overhead - executes in main conversation context
- **Direct interaction:** Questions asked directly, not via isolated subagent
- **Full context access:** Uses main conversation tools (Read/Grep/Glob/Bash)
- **Referenced patterns:** Always consult `skills/planner/SKILL.md` for detailed workflow guidance

## How to Use

**1. Capture requirement:**

- If argument provided → use it
- If no argument → ask: "What feature or change would you like to plan?"

**2. Invoke Planning skill:**

```
Skill tool with:
- skill: "k2-dev:planner"
- args: "User wants to: {user requirement}"

**IMPORTANT:** Always reference skills/planner/SKILL.md for detailed planning patterns and workflow steps.
```

**3. Skill executes workflow in main context:**

- **Analysis:** Explore codebase (Read/Grep/Glob/Bash), understand architecture
- **Clarification:** Ask questions directly in main conversation (scope, approach, constraints, integration points)
- **Planning:** Create detailed implementation plan with logical steps, dependencies, patterns
- **Tech Review:** Invoke Technical Lead agent (via Task tool) for architectural feedback, validation, standards check
- **Refinement:** Incorporate feedback, iterate until solid approach finalized
- **Task Creation:** Convert to beads tasks using `bd create`, set up hierarchy (epics/stories/subtasks), add dependencies with `bd dep add`
- **Sync & Report:** Run `bd sync`, generate structured report with task graph and unblocked/blocked tickets

## Configuration Files

Planning skill references: AGENTS.md, CLAUDE.md, (docs|specs)/constitution.md, project documentation

## Usage Examples

```bash
/k2:planner Add user authentication with JWT  # with description
/k2:planner  # interactive - asks for requirement
/planner     # alias (direct skill invocation)
```

## Final Report Structure

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

✅ Requirements clarified → ✅ Tech Lead approved → ✅ Beads tasks created → ✅ Dependencies set → ✅ Execution order clear

Present: task IDs/titles, parent-child relationships, dependencies, execution order, next steps (`/k2:start beads-XXX`)
