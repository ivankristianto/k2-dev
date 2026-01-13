---
name: k2:refine
description: Analyze and update a beads ticket with technical analysis and refined requirements
argument-hint: ticket-id
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
  - TodoWrite
  - Task
---

# K2:Refine - Ticket Refinement

Invoke Technical Lead agent to analyze a beads ticket and update its description with technical analysis and refined requirements.

## Workflow

**Launch Technical Lead agent:**

```
Task tool → k2-dev:technical-lead
Model: opus
Prompt: "Refine ticket: {ticket-id}

Read the ticket via 'bd show {ticket-id}' and analyze:
1. Technical approach and implementation strategy
2. Key requirements and acceptance criteria
3. Dependencies and integration points
4. Risks and considerations

Update the ticket description with:
- Technical analysis summary
- Refined requirements
- Implementation guidance

Use 'bd update {ticket-id} --description="..."' to set the new description."
```

## Usage

```bash
/k2:refine beads-123
```

## Result

Ticket description is updated with technical analysis and refined requirements.
