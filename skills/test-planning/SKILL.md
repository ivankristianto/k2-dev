---
name: Test Planning
description: This skill should be used when the user asks to "create a test plan", "write test cases", "plan testing strategy", "define test coverage", "create test scenarios", or needs guidance on comprehensive test planning in k2-dev workflows.
version: 0.1.0
---

# Test Planning Skill

## Overview

Comprehensive test planning ensures code quality, catches bugs early, and provides confidence in implementations. This skill provides guidance for creating structured test plans with specific test cases.

**Key Objectives:**
- Define clear test strategy
- Create specific, implementable test cases
- Ensure comprehensive coverage
- Document testing approach

## Test Strategy Components

### Test Types

**Unit Tests:**
- Test individual functions/methods
- Fast execution
- Isolated from dependencies
- Mock external services

**Integration Tests:**
- Test component interactions
- Database access
- API endpoints
- Service integrations

**End-to-End Tests:**
- Test complete user flows
- Full system integration
- UI interactions
- Real or staging environment

**Performance Tests:**
- Load testing
- Stress testing
- Response time validation
- Resource usage

**Security Tests:**
- Input validation
- Authentication/authorization
- Injection vulnerabilities
- Security headers

### Coverage Goals

**Minimum viable:**
- Critical paths: 100%
- Unit tests: 80%+
- Integration tests: Major flows
- E2E tests: Key user journeys

**Project-specific:**
- Check AGENTS.md for requirements
- May specify higher thresholds
- May require specific test types

## Test Case Structure

### Standard Format

Each test case should define:

```markdown
### TC-XXX: Test Case Title

**Type:** Unit / Integration / E2E / Performance / Security
**Priority:** Critical / High / Medium / Low

**Preconditions:**
- Required setup
- Data state
- Environment configuration

**Test Steps:**
1. Step one
2. Step two
3. Step three

**Expected Result:**
- What should happen
- Expected output
- Success criteria

**Test Data:**
- Input values
- Mock data
- Sample payloads
```

### Priority Levels

**Critical:**
- Security validations
- Data integrity checks
- Core business logic
- Payment processing
- Authentication

**High:**
- Main user flows
- API endpoints
- Data transformations
- Error handling

**Medium:**
- Edge cases
- Secondary features
- Optional flows
- UI variations

**Low:**
- Nice-to-have features
- Rare edge cases
- Performance optimizations

## Test Plan Document Structure

```markdown
# Test Plan: {ticket-id} - {title}

## Scope
What will be tested in this plan.

## Test Strategy

### Approach
- Test types to be used
- Testing tools and frameworks
- Environment requirements

### Coverage Goals
- Target coverage percentages
- Critical areas
- Known gaps

## Test Cases

### Unit Tests

#### TC-001: Validate Token Generation
**Type:** Unit
**Priority:** Critical
**Preconditions:** JWT secret configured
**Test Steps:**
1. Call generateToken() with user ID
2. Verify token is returned
3. Decode token and check payload
**Expected Result:** Valid JWT token with correct user ID and expiration
**Test Data:** userID: 123

[More test cases...]

### Integration Tests

#### TC-010: Login Flow End-to-End
**Type:** Integration
**Priority:** Critical
**Preconditions:** Test user exists in database
**Test Steps:**
1. POST to /api/auth/login with credentials
2. Verify 200 response
3. Extract token from response
4. Use token to access protected route
**Expected Result:** Successful login with valid token
**Test Data:**
- username: "test@example.com"
- password: "Test123!"

[More test cases...]

## Coverage Matrix

| Requirement | Test Cases | Coverage |
|-------------|------------|----------|
| User can log in | TC-010, TC-011 | ✅ |
| Token expires after 24h | TC-012, TC-013 | ✅ |
| Invalid credentials rejected | TC-014, TC-015 | ✅ |

## Testing Notes

### Assumptions
- Test database available
- Environment variables configured
- External services mocked

### Risks
- Flaky tests in CI
- Timing-dependent tests

### Dependencies
- Jest testing framework
- Supertest for API tests
- Jest-extended matchers

## Recommended Tools

- **Framework:** Jest
- **API Testing:** Supertest
- **Mocking:** Jest mock functions
- **Coverage:** Jest --coverage
```

## Creating Test Cases

### Happy Path Tests

Test the expected, successful flow:

**Example:**
```markdown
### TC-001: Successful User Login

**Type:** Integration
**Priority:** Critical

**Preconditions:**
- User exists: test@example.com / Test123!
- Database seeded

**Test Steps:**
1. POST /api/auth/login
   Body: { email: "test@example.com", password: "Test123!" }
2. Verify response status 200
3. Verify response contains token field
4. Decode token and check user ID matches

**Expected Result:**
- Status: 200
- Response: { success: true, token: "eyJ...", user: {...} }
- Token valid and contains correct user ID

**Test Data:**
{
  "email": "test@example.com",
  "password": "Test123!"
}
```

### Edge Case Tests

Test boundary conditions and limits:

**Example:**
```markdown
### TC-015: Login with Maximum Length Password

**Type:** Integration
**Priority:** Medium

**Preconditions:**
- User with 100-character password exists

**Test Steps:**
1. POST /api/auth/login with 100-char password
2. Verify successful authentication

**Expected Result:**
- Status: 200
- Authentication succeeds

**Test Data:**
- password: "a".repeat(100)
```

### Error Condition Tests

Test failure scenarios:

**Example:**
```markdown
### TC-020: Login with Invalid Credentials

**Type:** Integration
**Priority:** High

**Preconditions:**
- User exists with different password

**Test Steps:**
1. POST /api/auth/login with wrong password
2. Verify response status 401
3. Verify error message

**Expected Result:**
- Status: 401
- Response: { success: false, error: "Invalid credentials" }
- No token returned

**Test Data:**
{
  "email": "test@example.com",
  "password": "WrongPassword123!"
}
```

### Security Tests

Test security requirements:

**Example:**
```markdown
### TC-030: Prevent SQL Injection in Login

**Type:** Security
**Priority:** Critical

**Preconditions:**
- Application running

**Test Steps:**
1. POST /api/auth/login with SQL injection payload
2. Verify injection is prevented
3. Verify appropriate error response

**Expected Result:**
- No SQL error exposed
- Request rejected safely
- Status: 400 or 401

**Test Data:**
{
  "email": "admin'--",
  "password": "' OR '1'='1"
}
```

## Coverage Analysis

### Building Coverage Matrix

Map requirements to test cases:

```markdown
| Requirement | Test Cases | Status | Notes |
|-------------|-----------|---------|-------|
| REQ-001: User authentication | TC-001, TC-010, TC-020 | ✅ Covered | Happy path + errors |
| REQ-002: Token expiration | TC-012, TC-013 | ✅ Covered | Unit + integration |
| REQ-003: Password validation | TC-025, TC-026 | ⚠️ Partial | Need edge cases |
| REQ-004: Rate limiting | - | ❌ Not covered | Add TC-040 |
```

### Identifying Gaps

Check for:
- Requirements without test cases
- Features without error case tests
- Security requirements without security tests
- Edge cases not covered
- Performance requirements not validated

**Document gaps:**
```markdown
## Coverage Gaps

### Critical Gaps
- Rate limiting not tested → Add TC-040, TC-041
- Token refresh flow missing → Add TC-050-053

### Known Limitations
- Load testing deferred to beads-456
- Browser compatibility manual testing only
```

## Test Tools and Frameworks

### Jest (JavaScript/TypeScript)

**Unit tests:**
```typescript
describe('generateToken', () => {
  it('should generate valid JWT token', () => {
    const token = generateToken(123);
    expect(token).toBeDefined();
    expect(jwt.decode(token).userId).toBe(123);
  });
});
```

**Integration tests with Supertest:**
```typescript
describe('POST /api/auth/login', () => {
  it('should return token on valid credentials', async () => {
    const response = await request(app)
      .post('/api/auth/login')
      .send({ email: 'test@example.com', password: 'Test123!' })
      .expect(200);

    expect(response.body.token).toBeDefined();
  });
});
```

### pytest (Python)

**Unit tests:**
```python
def test_generate_token():
    token = generate_token(user_id=123)
    assert token is not None
    assert decode_token(token)['user_id'] == 123
```

**Integration tests:**
```python
def test_login_success(client):
    response = client.post('/api/auth/login', json={
        'email': 'test@example.com',
        'password': 'Test123!'
    })
    assert response.status_code == 200
    assert 'token' in response.json()
```

## Documenting Test Plan

### In Beads Comments

Add test plan to task:

```bash
bd comments beads-123 add "$(cat <<'EOF'
# Test Plan: Authentication Implementation

## Test Strategy
- Unit tests for token generation and validation
- Integration tests for login/logout flows
- Security tests for injection and auth bypass

## Test Cases
[... test cases ...]

## Coverage
- Target: 90% (exceeds 80% requirement)
- Critical paths: 100%

## Tools
- Jest + Supertest
- Jest-extended matchers
EOF
)"
```

### In Test Files

Reference test plan in code:

```typescript
/**
 * Authentication Tests
 *
 * Test Plan: See beads-123 comments
 * Coverage Goal: 90%
 *
 * Test Cases:
 * - TC-001: Token generation (Unit)
 * - TC-010: Login flow (Integration)
 * - TC-020: Invalid credentials (Integration)
 * - TC-030: SQL injection prevention (Security)
 */

describe('Authentication', () => {
  // Tests here
});
```

## Test Implementation Order

### Priority-Based

1. **Critical tests first:**
   - Security validations
   - Core business logic
   - Data integrity

2. **High priority tests:**
   - Main user flows
   - API endpoints
   - Error handling

3. **Medium/Low priority:**
   - Edge cases
   - Optional features
   - Performance tests

### Coverage-Driven

1. **Establish baseline:**
   - Write tests for existing code
   - Reach minimum coverage threshold

2. **Test new features:**
   - TDD approach if preferred
   - Write tests alongside implementation

3. **Fill gaps:**
   - Add missing edge cases
   - Complete coverage matrix

## K2-Dev Test Planning Workflow

### Tester Agent Creates Plan

When /k2:test command executed:

```bash
# Tester reads task
bd show beads-123
bd comments beads-123

# Analyzes implementation
cat src/auth/middleware.ts
grep -r "auth" src/

# Creates comprehensive test plan
# Adds to beads as comment
bd comments beads-123 add "[Test plan content]"
```

### Engineer Implements Tests

```bash
# Read test plan
bd comments beads-123

# Implement test cases
vi src/auth/middleware.test.ts

# Run tests
npm test

# Verify coverage
npm test -- --coverage
```

### Reviewer Validates

```bash
# Check test plan exists
bd comments beads-123 | grep "Test Plan"

# Review test implementation
git diff main...feature/beads-123 -- '**/*.test.ts'

# Verify coverage
npm test -- --coverage
```

## Best Practices

### DO

✅ **Be specific**
- Exact test steps
- Concrete expected results
- Actual test data

✅ **Cover scenarios**
- Happy path
- Edge cases
- Error conditions
- Security

✅ **Prioritize tests**
- Critical first
- Ensure minimum coverage
- Document gaps

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
- Ignoring validation

❌ **Don't ignore security**
- No injection tests
- No auth bypass tests
- No input validation tests

❌ **Don't forget performance**
- No load tests for critical paths
- No response time validation

## Command Reference

| Action | Command |
|--------|---------|
| Run tests | `npm test` or `pytest` |
| Coverage report | `npm test -- --coverage` |
| Watch mode | `npm test -- --watch` |
| Specific file | `npm test -- auth.test.ts` |
| Update snapshots | `npm test -- -u` |

Create comprehensive, specific test plans to ensure quality and catch issues early in k2-dev development workflows.
