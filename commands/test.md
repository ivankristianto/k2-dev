---
name: k2:test
description: Create comprehensive test plan and test cases for a beads ticket
argument-hint: beads-123
allowed-tools:
  - Read
  - Bash
  - Task
---

# K2:Test - Test Planning Workflow

Create a comprehensive test plan with test cases for a beads ticket, ensuring thorough coverage and validation strategy.

## What This Command Does

This command invokes the Test Planning skill which executes in the main conversation context:

1. Test Planning skill reads ticket details and implementation
2. Analyzes requirements and acceptance criteria
3. Creates test strategy and coverage plan
4. Defines specific test cases
5. Documents test plan in beads task comments
6. Coordinates with Engineer if needed

## Key Benefits Over Agent-Based Approach

- **Faster execution**: No agent spawning overhead
- **Easier debugging**: Everything happens in main conversation context
- **Direct context access**: Skill uses main conversation tools
- **Better UX**: Test planning happens in main conversation, not isolated subagent

## How to Use This Command

**Step 1: Parse Ticket Argument**

- Expect single ticket ID: `beads-123`
- If no argument provided, ask user: "Which ticket would you like to create a test plan for?"

**Step 2: Validate Ticket**

- Use `bd show {ticket-id}` to verify ticket exists
- Check that ticket has implementation details (not just placeholder)
- If ticket is closed, show warning but continue (might be retrospective testing)

**Step 3: Invoke Test Planning Skill**

- Use the Skill tool to invoke the "k2-dev:test-planning" skill
- Provide ticket ID and context
- Skill will execute in main conversation context

Example:

```
Skill tool with:
- skill: "k2-dev:test-planning"
- args: "Create comprehensive test plan for ticket beads-123"

The skill will then:
1. Read ticket description and comments from beads
2. If implementation exists, analyze the code changes
3. Identify test scenarios (happy path, edge cases, errors)
4. Create test strategy (unit, integration, e2e)
5. Define specific test cases with inputs and expected outputs
6. Document coverage plan
7. Add test plan as comment to beads task
8. Coordinate with Technical Lead if clarification needed
```

**Step 4: Present Test Plan**

- Test Planning skill will create and document test plan
- Show summary to user
- Include:
  - Number of test cases
  - Coverage areas
  - Test types (unit, integration, e2e)
  - Any gaps or concerns

## Test Planning Workflow Details

### Phase 1: Analysis

- Read ticket description and acceptance criteria
- Review implementation if exists (code changes)
- Understand feature scope and boundaries
- Identify integration points
- Check quality gates in AGENTS.md

### Phase 2: Test Strategy

- Determine appropriate test types:
  - **Unit tests**: Individual function/module testing
  - **Integration tests**: Component interaction testing
  - **End-to-end tests**: Full user flow testing
  - **Performance tests**: If relevant
  - **Security tests**: If relevant
- Define coverage goals
- Consider test data requirements

### Phase 3: Test Case Definition

For each test case, define:

- **Test ID**: Unique identifier
- **Scenario**: What is being tested
- **Preconditions**: Required setup
- **Test Steps**: Actions to perform
- **Expected Result**: What should happen
- **Priority**: Critical, high, medium, low

### Phase 4: Coverage Analysis

- Identify coverage areas:
  - Happy path scenarios
  - Edge cases
  - Error conditions
  - Boundary values
  - Invalid inputs
- Map test cases to requirements
- Identify gaps

### Phase 5: Documentation

- Create structured test plan document
- Add as comment to beads task
- Include:
  - Test strategy overview
  - Test cases with full details
  - Coverage matrix
  - Testing notes and assumptions
  - Recommended test framework/tools

### Phase 6: Coordination

- If Engineer involvement needed, coordinate handoff
- If test implementation required, create follow-up task
- Ensure clarity on who implements tests

## Test Plan Format

The test plan should follow this structure:

```markdown
# Test Plan: {ticket-id} - {title}

## Test Strategy

**Scope:** {what will be tested}
**Approach:** {test types and methods}
**Coverage Goal:** {target coverage percentage or criteria}

## Test Cases

### TC-001: {Scenario Name}

**Type:** Unit / Integration / E2E
**Priority:** Critical / High / Medium / Low
**Preconditions:** {setup required}
**Steps:**

1. {action}
2. {action}
   **Expected Result:** {outcome}

### TC-002: {Scenario Name}

[repeat structure]

## Coverage Matrix

| Requirement     | Test Cases     | Status  |
| --------------- | -------------- | ------- |
| {requirement-1} | TC-001, TC-002 | Covered |
| {requirement-2} | TC-003         | Covered |

## Testing Notes

- {any special considerations}
- {test data requirements}
- {environment setup needs}

## Recommended Tools

- {test framework}
- {additional tools}
```

## Important Notes

- **Comprehensive**: Cover all scenarios including edge cases
- **Specific**: Each test case should be implementable
- **Prioritized**: Mark critical tests for minimum viable coverage
- **Documented**: Test plan stored in beads for reference
- **Actionable**: Should be clear enough for anyone to implement tests

## Configuration Files Used

Test Planning skill will reference:

- **AGENTS.md**: Testing standards and coverage requirements
- **CLAUDE.md**: Project-specific testing patterns
- **constitution.md**: Quality principles

## Example Usage

```
User: /k2:test beads-123
```

## Success Indicators

Test planning is complete when:

- ✅ Test strategy documented
- ✅ Test cases defined with full details
- ✅ Coverage analysis complete
- ✅ Test plan added as beads comment
- ✅ Any follow-up tasks created if needed

Present the test plan summary to the user showing:

- Number of test cases by type
- Coverage areas
- Priority breakdown
- Next steps for test implementation

## Skill Aliases

This test planning functionality can also be invoked via:
- `/k2:test` (this command)
- `/tester` (direct skill invocation)

Both invoke the same Test Planning skill and execute in the main conversation context.
