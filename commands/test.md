---
name: k2:test
description: Create comprehensive test plan and test cases for a beads ticket
argument-hint: beads-123
allowed-tools:
  - Read
  - Bash
  - Task
---

# K2:Test - Test Planning

Invoke Test Planning skill to create comprehensive test plan with test cases for a beads ticket.

## Benefits of Skill-Based Approach

- **Faster:** No agent spawning overhead - executes in main conversation context
- **Direct context:** Test planning happens in main conversation, not isolated subagent
- **Full access:** Uses main conversation tools for analysis

## How to Use

**1. Parse ticket ID** (ask if not provided: "Which ticket would you like to create a test plan for?")

**2. Validate:** `bd show {ticket-id}` → verify exists. Warn if closed but continue (retrospective testing OK).

**3. Invoke Test Planning skill:**
```
Skill tool with:
- skill: "k2-dev:test-planning"
- args: "Create comprehensive test plan for ticket beads-{ticket-id}"
```

**4. Skill executes workflow in main context:**
- **Analysis:** Read ticket (bd show/comments), review implementation (if exists), understand scope/boundaries, identify integration points, check AGENTS.md quality gates
- **Test Strategy:** Determine types (unit/integration/e2e/performance/security), define coverage goals, consider test data needs
- **Test Case Definition:** For each case define: ID, scenario, preconditions, steps, expected result, priority (critical/high/medium/low)
- **Coverage Analysis:** Identify areas (happy path, edge cases, errors, boundary values, invalid inputs), map to requirements, identify gaps
- **Documentation:** Create structured test plan, add as beads comment
- **Coordination:** Coordinate with Engineer if needed, create follow-up tasks for test implementation

## Test Plan Structure

```markdown
# Test Plan: {ticket-id} - {title}

## Test Strategy
**Scope:** {what tested} | **Approach:** {test types} | **Coverage Goal:** {target}

## Test Cases

### TC-001: {Scenario}
**Type:** Unit/Integration/E2E | **Priority:** Critical/High/Medium/Low
**Preconditions:** {setup}
**Steps:** 1. {action} 2. {action}
**Expected Result:** {outcome}

[repeat for each case]

## Coverage Matrix
| Requirement | Test Cases | Status |
|-------------|------------|--------|
| {req-1}     | TC-001,002 | ✓      |

## Testing Notes
{considerations, test data, environment needs}

## Recommended Tools
{framework, additional tools}
```

## Configuration Files

Test Planning skill references: AGENTS.md (testing standards), CLAUDE.md, constitution.md

## Usage Examples

```bash
/k2:test beads-123  # create test plan
/tester beads-123   # alias (direct skill invocation)
```

## Success Indicators

✅ Strategy documented → ✅ Test cases defined → ✅ Coverage analyzed → ✅ Plan in beads → ✅ Follow-ups created

Present summary: test count by type, coverage areas, priority breakdown, next steps for implementation
