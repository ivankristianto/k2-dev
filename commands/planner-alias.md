---
name: planner
description: Start planning workflow to analyze requirements and create actionable beads tasks (alias for /k2:planner)
argument-hint: optional feature description
allowed-tools:
  - Read
  - Task
  - AskUserQuestion
---

# /planner - Planning Skill Alias

This is an alias for `/k2:planner`. Both commands invoke the same Planning skill which executes in the main conversation context.

## Usage

```
/planner Add user authentication with JWT
```

Or:

```
/planner
```

See `/k2:planner` command documentation for full details on the planning workflow.
