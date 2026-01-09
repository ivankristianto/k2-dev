---
name: Test Planning
description: This skill should be used when the user needs to create comprehensive test plans, define test cases and coverage strategies, validate test completeness, analyze ticket requirements for testing needs, or document test strategies. The skill executes in the main conversation context and provides thorough test planning guidance without implementing tests.
version: 0.1.0
---

# Test Planning Skill

## Overview

Comprehensive test planning ensures code quality, catches bugs early, and provides confidence in implementations. This skill creates structured test plans with specific test cases that ensure thorough coverage of functionality, edge cases, and error conditions.

**Key Objectives:**
- Analyze ticket requirements and implementation for testing needs
- Define appropriate test strategy (unit, integration, e2e, performance, security)
- Create specific, implementable test cases with full details
- Ensure comprehensive coverage analysis
- Document test plans in beads for implementation reference

## When to Use This Skill

Invoke this skill when:
- User executes `/tester` or `/k2:test` command with a ticket ID
- Test planning is needed for a beads ticket
- Validating existing test coverage
- Creating test strategies for features
- Analyzing requirements for testing needs
- Documenting test cases for implementation

## Test Planning Workflow

### Phase 1: Context Gathering

**Step 1: Read Project Standards**

First, always read the project's quality standards:

```bash
# Check if standards files exist in project root
ls -la AGENTS.md CLAUDE.md docs/constitution.md

# Read available standards
cat AGENTS.md
cat CLAUDE.md
```

What to look for:
- Testing standards and coverage requirements
- Quality gates for test completeness
- Test patterns and frameworks in use
- Security testing requirements
- Performance testing requirements

**Step 2: Read Beads Task Context**

```bash
# Get complete task details
bd show beads-{id}

# Read all comments for context
bd comments beads-{id}
```

What to identify:
- Task description and requirements
- Acceptance criteria (become test scenarios)
- Implementation approach and plan
- Special testing instructions or constraints
- Related tasks for integration testing

**Step 3: Analyze Implementation** (if code exists)

```bash
# View implementation changes
git log --oneline feature/beads-{id}
git diff main...feature/beads-{id}

# Find implementation files
glob "**/src/**"
glob "**/lib/**"

# Locate existing tests
glob "**/*.test.{js,ts}"
glob "**/*.spec.{js,ts}"
glob "**/tests/**"
glob "**/__tests__/**"

# Understand test patterns
read {test_file}
```

What to understand:
- Code structure and logic flow
- Integration points and dependencies
- Existing test frameworks and patterns
- Complex logic requiring special attention
- Error handling and edge cases in code

### Phase 2: Define Test Strategy

Create a comprehensive test strategy appropriate to the change:

**1. Determine Test Types Needed**

| Test Type | When to Use | Coverage Focus |
|-----------|-------------|----------------|
| **Unit Tests** | Individual functions, classes, modules | Logic correctness in isolation |
| **Integration Tests** | Component interactions, APIs, databases | Component integration |
| **E2E Tests** | User workflows, UI changes | Full user flows |
| **Performance Tests** | High-traffic features, scalability | Load and response time |
| **Security Tests** | Authentication, data handling | Security controls |
| **Accessibility Tests** | UI components | WCAG compliance |

**2. Define Coverage Goals**

```markdown
Coverage Goals:
- Unit Test Coverage: {percentage from AGENTS.md or 80% default}
- Requirements Coverage: 100%
- Critical Path Coverage: All critical paths
- Edge Case Coverage: All identified edge cases
- Error Condition Coverage: All error paths
```

**3. Identify Testing Tools**

From existing project setup:
- Test framework (Jest, pytest, etc.)
- Mocking libraries
- Test utilities
- Coverage tools

### Phase 3: Design Test Cases

Create specific, implementable test cases:

**Test Case Structure:**

```markdown
### TC-XXX: {Clear, Descriptive Scenario Name}

**Type**: Unit | Integration | E2E | Performance | Security | Accessibility
**Priority**: Critical | High | Medium | Low
**Preconditions**: {Setup required before test}

**Test Steps**:
1. {Clear, specific action}
2. {Clear, specific action}
3. {Clear, specific action}

**Expected Result**: {Specific, measurable outcome}
**Test Data**: {Input values, test accounts, etc.}
**Notes**: {Any special considerations or implementation guidance}
```

**1. Happy Path Test Cases** (CRITICAL - Always include)

- Test the primary, successful user flow
- Cover all main functionality from requirements
- Verify acceptance criteria are met
- Use realistic, valid input data
- Priority: Critical or High

**2. Edge Case Test Cases** (IMPORTANT)

- Boundary values (min, max, zero, empty)
- Unusual but valid inputs
- Extreme data volumes
- Concurrent operations
- Timing-dependent scenarios
- Priority: High or Medium

**3. Error Condition Test Cases** (IMPORTANT)

- Invalid inputs (wrong type, format, range)
- Missing required data
- Authentication/authorization failures
- Network failures or timeouts
- Resource exhaustion
- Malformed data
- Priority: High or Medium

**4. Security Test Cases** (if applicable)

- Input validation and sanitization
- SQL injection attempts
- XSS attempts
- CSRF protection
- Authentication bypass attempts
- Authorization boundary testing
- Sensitive data exposure checks
- Priority: Critical or High

**5. Performance Test Cases** (if applicable)

- Load testing (expected load)
- Stress testing (peak load)
- Response time validation
- Resource usage monitoring
- Scalability verification
- Priority: Medium or High

**6. Accessibility Test Cases** (for UI)

- Keyboard navigation (Tab, Enter, Esc)
- Screen reader compatibility
- ARIA label presence and correctness
- Color contrast validation
- Focus management
- Priority: Medium or High

### Phase 4: Coverage Analysis

**1. Requirements Coverage Matrix**

Map every requirement to test cases:

```markdown
| Requirement/Acceptance Criteria | Test Cases             | Status                  |
| ------------------------------- | ---------------------- | ----------------------- |
| User can log in with email      | TC-001, TC-002         | ✅ Covered               |
| Token expires after 24h         | TC-003, TC-004         | ✅ Covered               |
| Invalid credentials rejected    | TC-005, TC-006         | ✅ Covered               |
| Rate limiting applied           | TC-007                 | ⚠️ Partial - Need more   |
```

**2. Code Coverage Analysis** (if implementation exists)

- Identify code paths not covered
- Note complex functions without sufficient tests
- Flag error handling paths without tests
- Ensure all public APIs have coverage

**3. Gap Documentation**

```markdown
## Coverage Gaps Identified

- **Gap**: Rate limiting not thoroughly tested
  - **Risk**: Medium
  - **Recommendation**: Add TC-008, TC-009 for rate limit scenarios
```

### Phase 5: Create Test Plan Document

Generate a structured test plan:

```markdown
# Test Plan: beads-{id} - {Title}

## Executive Summary

{Brief overview of what's being tested and the approach}

## Test Strategy

### Scope

**In Scope**:
- {What will be tested}
- {Feature boundaries}

**Out of Scope**:
- {What won't be tested}
- {Rationale}

### Test Types and Approach

- **Unit Tests**: {approach and coverage goal}
- **Integration Tests**: {approach and coverage goal}
- **E2E Tests**: {approach and coverage goal}
- **Performance Tests**: {if applicable}
- **Security Tests**: {if applicable}

### Coverage Goals

- Unit Test Coverage: {percentage}
- Requirements Coverage: 100%
- Edge Case Coverage: {description}
- Error Condition Coverage: {description}

### Testing Tools and Frameworks

- Test Framework: {e.g., Jest, pytest, Cypress}
- Mocking Library: {e.g., jest.mock, unittest.mock}
- Test Utilities: {e.g., React Testing Library, Supertest}

## Test Cases

### Unit Tests

#### TC-001: {Scenario Name}
**Type**: Unit
**Priority**: Critical
**Preconditions**: {setup}
**Test Steps**:
1. {action}
2. {action}
**Expected Result**: {outcome}

#### TC-002: {Scenario Name}
...

### Integration Tests

#### TC-010: {Scenario Name}
**Type**: Integration
**Priority**: Critical
...

### Coverage Matrix

| Requirement | Test Cases | Status |
| ----------- | ---------- | ------ |
| {req-1} | TC-001, TC-002 | Covered |
| {req-2} | TC-003 | Covered |

## Test Data Requirements

- {Test data needed}
- {Setup/teardown approach}
- {Fixtures or factories required}

## Testing Environment

- {Environment setup requirements}
- {Test database or services needed}
- {Mock services or stubs needed}

## Risk Assessment

| Risk     | Likelihood | Impact  | Mitigation            |
| -------- | ---------- | ------- | --------------------- |
| {risk-1} | {H/M/L}    | {H/M/L} | {mitigation strategy} |

## Coverage Gaps and Recommendations

{Any identified gaps with risk assessment}

## Testing Notes

- {Special considerations}
- {Implementation guidance}
- {Known limitations}

## Success Criteria

- [ ] All Critical and High priority tests pass
- [ ] Coverage goals met
- [ ] No P0 security vulnerabilities
- [ ] Performance requirements met (if applicable)
```

### Phase 6: Document Test Plan in Beads

Add the test plan to the beads task:

```bash
# Add test plan as comment
bd comments beads-{id} add "$(cat <<'EOF'
# Test Plan Created - {date}

{Full test plan content}

---

**Created by**: Test Planning skill
**Date**: {timestamp}
**Implementation Status**: Pending
EOF
)"
```

### Phase 7: Generate Final Report

Create a summary report for the user:

```markdown
## Test Planning Complete: beads-{id}

### Test Plan Summary

- **Total Test Cases**: {count}
  - Critical: {count}
  - High: {count}
  - Medium: {count}
  - Low: {count}

### Test Types

- Unit Tests: {count} test cases
- Integration Tests: {count} test cases
- E2E Tests: {count} test cases
- Performance Tests: {count} test cases (if applicable)
- Security Tests: {count} test cases (if applicable)

### Coverage Analysis

- Requirements Coverage: {percentage or "100%"}
- Happy Path Scenarios: {count}
- Edge Cases: {count}
- Error Conditions: {count}

### Test Plan Location

- Added as comment to beads-{id}
- {word count} words, {test case count} test cases

### Key Testing Considerations

- {Important note 1}
- {Important note 2}
- {Risk or special consideration}

### Coverage Gaps (if any)

- {Gap 1 with risk assessment}
- {Gap 2 with recommendation}

### Recommended Tools

- Test Framework: {name}
- Additional Tools: {list}

### Next Steps

- Test plan documented in beads comments
- Engineer can implement tests from this plan
- Or: Tests validated and complete
```

## Priority Guidelines

| Priority | When to Use | Examples |
|----------|-------------|----------|
| **Critical** | Core functionality, security, data integrity | Auth flows, payment processing, data validation |
| **High** | Important features, major edge cases, error handling | API endpoints, user flows, error conditions |
| **Medium** | Secondary features, less common edge cases | Optional features, rare edge cases |
| **Low** | Cosmetic issues, very rare edge cases | UI polish, future enhancements |

## Test Type Selection Guide

**Always include unit tests for:**
- Business logic and algorithms
- Data transformations
- Validation functions
- Utility functions

**Add integration tests for:**
- API endpoints
- Database interactions
- Service integrations
- Component interactions

**Include e2e tests for:**
- Critical user workflows
- UI changes
- Multi-step user journeys
- Key business flows

**Add performance tests for:**
- High-traffic features
- Resource-intensive operations
- Scalability concerns

**Include security tests for:**
- Authentication/authorization
- Input validation
- Data handling
- Sensitive operations

## Common Testing Scenarios

### API Development

**Unit Tests:**
- Request validation
- Response formatting
- Error handling
- Business logic

**Integration Tests:**
- Endpoint contracts
- Database operations
- External service calls
- Authentication/authorization

**E2E Tests:**
- Complete request/response cycles
- Error scenarios
- Authentication flows

### Database Changes

**Unit Tests:**
- Query logic
- Data transformations
- Validation rules

**Integration Tests:**
- Database operations
- Migration scripts
- Data integrity

### UI Components

**Unit Tests:**
- Component logic
- State management
- Event handlers

**Integration Tests:**
- Component interactions
- API integrations
- State updates

**E2E Tests:**
- User workflows
- Form submissions
- Navigation flows

**Accessibility Tests:**
- Keyboard navigation
- Screen reader support
- ARIA attributes
- Color contrast

### Authentication/Authorization

**Unit Tests:**
- Token generation/validation
- Permission checks
- Password hashing

**Integration Tests:**
- Login/logout flows
- Protected route access
- Token refresh

**Security Tests:**
- SQL injection prevention
- XSS prevention
- CSRF protection
- Auth bypass attempts

## Tools and Commands

### Beads Operations

```bash
# View task
bd show beads-{id}

# Read comments
bd comments beads-{id}

# Add test plan comment
bd comments beads-{id} add "{test plan content}"

# Create follow-up ticket for test implementation
bd create --title="Implement tests for beads-{id}" --priority=P1
```

### Test Exploration

```bash
# Find test files
glob "**/*.test.{js,ts,py}"
glob "**/tests/**"
glob "**/__tests__/**"

# Find test patterns
grep "describe\|test\|it" --output_mode=content

# Run tests (if testing existing coverage)
npm test
pytest

# Coverage report
npm test -- --coverage
pytest --cov
```

### Git Operations

```bash
# View implementation changes
git diff main...feature/beads-{id}

# View commit history
git log --oneline feature/beads-{id}

# Check specific file changes
git show feature/beads-{id}:path/to/file
```

## Best Practices

### DO

✅ **Be comprehensive**
- Cover happy path, edge cases, and error conditions
- Ensure all requirements have test coverage
- Identify and document gaps

✅ **Be specific**
- Exact test steps
- Concrete expected results
- Actual test data

✅ **Prioritize tests**
- Critical functionality first
- Risk-based approach
- Document priority rationale

✅ **Make tests implementable**
- Clear enough to code directly
- Include all necessary data
- Specify tools and frameworks

### DON'T

❌ **Don't be vague**
```markdown
TC-001: Test login
- Test the login
- It should work
```

❌ **Don't skip error cases**
- Only testing happy path
- Ignoring validation failures
- Missing error handling

❌ **Don't ignore security**
- No injection tests
- No auth bypass tests
- No input validation tests

❌ **Don't forget edge cases**
- Boundary values
- Empty/null inputs
- Concurrent operations
- Timing issues

## Error Handling

### Missing Standards Files

- Use industry best practices
- Default to 80% unit test coverage
- Note absence in test plan
- Recommend creating AGENTS.md/CLAUDE.md

### Unclear Requirements

- Review beads task and comments thoroughly
- Analyze implementation if available
- Make reasonable assumptions and document them
- Note assumptions in test plan

### Existing Tests Not Found

- Search thoroughly (test/, tests/, **tests**/, spec/)
- Note absence in test plan
- Recommend test framework and structure
- Create plan assuming fresh implementation

### Incomplete Implementation

- Focus test plan on planned functionality
- Note what's not yet implemented
- Ensure plan covers full requirements
- Make plan flexible for changes

## Success Criteria

Test planning is complete when:

- ✅ Test strategy documented
- ✅ Test cases defined with full details
- ✅ Coverage analysis complete
- ✅ Test plan added as beads comment
- ✅ Summary report provided to user
- ✅ Gaps identified and assessed
- ✅ Recommended tools documented
- ✅ Implementation guidance provided

---

Execute test planning in main conversation context for comprehensive, actionable test strategies.
