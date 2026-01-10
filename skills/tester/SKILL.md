---
name: Test Planning
description: This skill should be used when the user needs to create comprehensive test plans, define test cases and coverage strategies, validate test completeness, analyze ticket requirements for testing needs, or document test strategies. The skill executes in the main conversation context and provides thorough test planning guidance without implementing tests.
version: 0.2.0
---

# Test Planning Skill

## Overview

Creates structured test plans with specific test cases ensuring thorough coverage of functionality, edge cases, and error conditions. Executes in main conversation context for comprehensive, actionable test strategies.

**Key Objectives:**
- Analyze ticket requirements and implementation for testing needs
- Define appropriate test strategy (unit, integration, e2e, performance, security)
- Create specific, implementable test cases with full details
- Ensure comprehensive coverage analysis
- Document test plans in beads for implementation reference

**Invoked when:** `/tester` or `/k2:test` command executed with ticket ID

## Seven-Phase Test Planning Workflow

### Phase 1: Context Gathering

**1. Read Project Standards**
```bash
ls -la AGENTS.md CLAUDE.md docs/constitution.md
cat AGENTS.md CLAUDE.md
```
Extract: Testing standards, coverage requirements, quality gates, test patterns, frameworks, security requirements.

**2. Read Beads Task Context**
```bash
bd show beads-{id}
bd comments beads-{id}
```
Identify: Task description, acceptance criteria (become test scenarios), implementation approach, special testing instructions, related tasks for integration testing.

**3. Analyze Implementation** (if code exists)
```bash
git log --oneline feature/beads-{id}
git diff main...feature/beads-{id}
glob "**/*.test.{js,ts,py}"
glob "**/tests/**"
read {implementation_files}
read {test_files}
```
Understand: Code structure, logic flow, integration points, existing test patterns, complex logic, error handling.

### Phase 2: Define Test Strategy

**Test Type Selection** (Reference: k2-dev-reference.md#test-types)

| Test Type | When Required |
|-----------|---------------|
| **Unit** | Business logic, algorithms, data transformations, utilities |
| **Integration** | API endpoints, database interactions, service integrations |
| **E2E** | Critical user workflows, UI changes, multi-step journeys |
| **Performance** | High-traffic features, resource-intensive operations |
| **Security** | Auth, input validation, data handling, sensitive operations |
| **Accessibility** | UI components (keyboard nav, screen readers, ARIA, contrast) |

**Coverage Goals:**
- Unit Test Coverage: {from AGENTS.md or 80% default}
- Requirements Coverage: 100%
- Critical Path Coverage: All critical paths
- Edge Case Coverage: All identified edge cases
- Error Condition Coverage: All error paths

**Testing Tools:** Identify from existing project setup (Jest, pytest, Cypress, etc.)

### Phase 3: Design Test Cases

**Test Case Structure:**
```markdown
### TC-XXX: {Clear Scenario Name}
**Type**: Unit|Integration|E2E|Performance|Security|Accessibility
**Priority**: Critical|High|Medium|Low
**Preconditions**: {Setup required}
**Test Steps**:
1. {Specific action}
2. {Specific action}
**Expected Result**: {Specific, measurable outcome}
**Test Data**: {Input values}
**Notes**: {Implementation guidance}
```

**Required Test Categories:**

1. **Happy Path** (CRITICAL - Always include)
   - Primary successful user flow
   - All main functionality from requirements
   - Verify acceptance criteria met
   - Realistic, valid input data
   - Priority: Critical or High

2. **Edge Cases** (IMPORTANT)
   - Boundary values (min, max, zero, empty)
   - Unusual but valid inputs
   - Extreme data volumes
   - Concurrent operations
   - Timing-dependent scenarios
   - Priority: High or Medium

3. **Error Conditions** (IMPORTANT)
   - Invalid inputs (wrong type, format, range)
   - Missing required data
   - Auth/authorization failures
   - Network failures/timeouts
   - Resource exhaustion
   - Malformed data
   - Priority: High or Medium

4. **Security Tests** (if applicable)
   - Input validation/sanitization
   - SQL injection, XSS, CSRF prevention
   - Auth bypass attempts
   - Authorization boundary testing
   - Sensitive data exposure checks
   - Priority: Critical or High

5. **Performance Tests** (if applicable)
   - Load testing (expected load)
   - Stress testing (peak load)
   - Response time validation
   - Resource usage monitoring
   - Priority: Medium or High

6. **Accessibility Tests** (for UI)
   - Keyboard navigation (Tab, Enter, Esc)
   - Screen reader compatibility
   - ARIA labels correctness
   - Color contrast validation
   - Focus management
   - Priority: Medium or High

### Phase 4: Coverage Analysis

**1. Requirements Coverage Matrix**
```markdown
| Requirement/Acceptance Criteria | Test Cases | Status |
|---------------------------------|------------|--------|
| User can log in with email | TC-001, TC-002 | ✅ Covered |
| Token expires after 24h | TC-003, TC-004 | ✅ Covered |
| Invalid credentials rejected | TC-005, TC-006 | ✅ Covered |
| Rate limiting applied | TC-007 | ⚠️ Partial - Need TC-008 |
```

**2. Code Coverage Analysis** (if implementation exists)
- Identify uncovered code paths
- Note complex functions without sufficient tests
- Flag error handling paths without tests
- Ensure all public APIs have coverage

**3. Gap Documentation**
```markdown
## Coverage Gaps
- **Gap**: {Description}
  - **Risk**: High|Medium|Low
  - **Recommendation**: {Action}
```

### Phase 5: Create Test Plan Document

```markdown
# Test Plan: beads-{id} - {Title}

## Executive Summary
{Brief overview}

## Test Strategy

### Scope
**In Scope**: {What will be tested}
**Out of Scope**: {What won't be tested + rationale}

### Test Types and Approach
- **Unit Tests**: {approach + coverage goal}
- **Integration Tests**: {approach + coverage goal}
- **E2E Tests**: {approach + coverage goal}
- **Performance/Security**: {if applicable}

### Coverage Goals
- Unit: {percentage}, Requirements: 100%, Edge Cases: {description}, Errors: {description}

### Tools
- Framework: {e.g., Jest, pytest}
- Mocking: {library}
- Utilities: {tools}

## Test Cases

### Unit Tests
#### TC-001: {Scenario}
{Full test case details}

### Integration Tests
#### TC-010: {Scenario}
{Full test case details}

### Coverage Matrix
{Requirements coverage table}

## Test Data Requirements
{Test data, setup/teardown, fixtures}

## Risk Assessment
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| {risk} | H/M/L | H/M/L | {strategy} |

## Coverage Gaps and Recommendations
{Identified gaps with risk}

## Success Criteria
- [ ] All Critical/High tests pass
- [ ] Coverage goals met
- [ ] No P0 security vulnerabilities
- [ ] Performance requirements met (if applicable)
```

### Phase 6: Document in Beads

```bash
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

```markdown
## Test Planning Complete: beads-{id}

### Test Plan Summary
- **Total Test Cases**: {count}
  - Critical: {count}, High: {count}, Medium: {count}, Low: {count}

### Test Types
- Unit: {count}, Integration: {count}, E2E: {count}, Performance: {count}, Security: {count}

### Coverage Analysis
- Requirements: {percentage or "100%"}
- Happy Path: {count}, Edge Cases: {count}, Error Conditions: {count}

### Test Plan Location
Added as comment to beads-{id}

### Key Testing Considerations
- {Important note 1}
- {Important note 2}

### Coverage Gaps (if any)
- {Gap with risk + recommendation}

### Recommended Tools
- Framework: {name}
- Additional: {tools}

### Next Steps
Test plan documented in beads comments. Engineer can implement tests from this plan.
```

## Priority Guidelines

Reference: k2-dev-reference.md#priority-levels

**Critical:** Core functionality, security, data integrity (auth flows, payment processing, data validation)
**High:** Important features, major edge cases, error handling (API endpoints, user flows, error conditions)
**Medium:** Secondary features, less common edge cases (optional features, rare edge cases)
**Low:** Cosmetic issues, very rare edge cases (UI polish, future enhancements)

## Best Practices

### DO
✅ Be comprehensive - happy path, edge cases, error conditions
✅ Be specific - exact steps, concrete results, actual test data
✅ Prioritize tests - critical first, risk-based approach
✅ Make tests implementable - clear enough to code directly

### DON'T
❌ Be vague ("TC-001: Test login. Test the login. It should work.")
❌ Skip error cases - only testing happy path
❌ Ignore security - no injection/auth bypass/validation tests
❌ Forget edge cases - boundaries, empty/null, concurrent operations

## Error Handling

**Missing Standards:** Use industry best practices, default 80% coverage, note absence, recommend AGENTS.md/CLAUDE.md
**Unclear Requirements:** Review thoroughly, analyze implementation, make documented assumptions
**No Existing Tests:** Search thoroughly, note absence, recommend framework, create fresh plan
**Incomplete Implementation:** Focus on planned functionality, note gaps, ensure plan covers full requirements

## Success Criteria

Test planning complete when:
- ✅ Test strategy documented
- ✅ Test cases defined with full details
- ✅ Coverage analysis complete
- ✅ Test plan added as beads comment
- ✅ Summary report provided
- ✅ Gaps identified and assessed
- ✅ Recommended tools documented
- ✅ Implementation guidance provided

**Reference:** See k2-dev-reference.md for commands, coverage levels, test types, and common patterns.
