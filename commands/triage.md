---
name: k2:triage
description: Analyze ready tickets and recommend the best next ticket to work on
argument-hint: "[--json]"
allowed-tools:
  - Read
  - Bash
---

# K2:Triage - Next Ticket Recommendation

Analyze unblocked tickets and recommend the single best next ticket to work on.

**Read-only:** This command only analyzes and recommends. It does NOT modify any beads task status or any other state.

## Usage

```
k2:triage              # Analyze ready tickets and recommend the best one(s) to work on
k2:triage --json       # Output ticket IDs in JSON format
```

## Output

**Default (no --json):** Human-readable markdown with ticket details.

**With --json:** Machine-readable JSON output:

```json
{
  "ticket": ["beads-123", "beads-234"]
}
```

**Outputs one or more tickets that can be worked on together.**

## Logic Flow

### P1: Get Ready Tickets

```bash
bd ready --sort=priority
```

### P2: Get Ticket Details

For each ticket, fetch:

```bash
bd show {ticket-id}  # Full details
```

Parse:

- Type: bug, epic, story, task
- Priority: P0, P1, P2
- Status: should be open
- Dependencies: any blocking tickets

### P3: Apply Priority Rules

**Bug handling:** If type is `bug` and priority is P2, auto-upgrade to P1.

**Epic handling:** If type is `epic`:

1. Get child tickets: `bd list --parent={parent_id}`
2. If children exist, analyze the first unblocked child instead
3. If no children, epic itself is the ticket

### P4: Determine Best Ticket(s)

Sort by:

1. Priority (P0 > P1 > P2)
2. Type (bugs first, then stories, then tasks)
3. Dependencies (prefer tickets with no blockers)

**Group related tickets:** If the top ticket has a child ticket or a closely related dependency that is also unblocked and has the same priority, include them together.

**Limit:** Maximum 3 tickets in one recommendation.

### P5: Output (Conditional)

**If --json flag is present:** Output JSON:

```json
{
  "ticket": ["beads-123", "beads-234"]
}
```

**If no --json flag:** Output markdown template below.

### P6: Markdown Output Template

```
## Recommended Next Ticket(s)

### {ticket-1-id}: {title}
- Priority: {priority} {adjusted_flag}
- Type: {type}
- Status: {status}
- Summary: {2-3 line description}

### {ticket-2-id}: {title}
- Priority: {priority}
- Type: {type}
- Status: {status}
- Summary: {2-3 line description}

**Why these tickets:**
- {reason for selection}
- Related: {why they work together}
```

**If no ready tickets:**

```
No ready tickets found.

Run `bd blocked` to see blocked tickets that need attention first.
```

## Example

```markdown
## Recommended Next Ticket(s)

### beads-456: Fix memory leak in auth service
- Priority: P1 (upgraded from bug P2)
- Type: bug
- Status: open
- Summary: Memory grows unbounded when processing concurrent auth requests

### beads-457: Add metrics endpoint for auth service
- Priority: P1
- Type: task
- Status: open
- Summary: Add /metrics endpoint to expose auth service health and performance metrics

**Why these tickets:**

- P1 bug takes precedence, but closely related P1 task is part of the same service
- Both target the auth service - work together efficiently
- No blockers on either ticket
```

## Constraints

- Output ONE recommendation (can include multiple related tickets)
- Maximum 3 tickets per recommendation
- Handle epics by drilling into children
- Keep output concise and actionable
