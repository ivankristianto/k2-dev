---
name: Code Review Standards
description: This skill should be used when the user asks to "review code", "perform code review", "check code quality", "review PR", "provide code feedback", or needs guidance on code review best practices and standards in k2-dev workflows.
version: 0.1.0
---

# Code Review Standards Skill

## Overview

Code review is a critical quality control process that catches bugs, ensures standards compliance, and facilitates knowledge sharing. This skill provides comprehensive guidance for conducting effective code reviews.

**Review Objectives:**
- Ensure code quality and maintainability
- Catch bugs and logic errors
- Validate security practices
- Enforce project standards
- Share knowledge

## Review Process

### Four-Pass Review Method

**Pass 1: High-Level Understanding**
- Read PR description and ticket context
- Understand what the code is supposed to do
- Review architectural approach
- Identify scope and boundaries

**Pass 2: Line-by-Line Detailed Review**
- Read every changed line
- Check logic correctness
- Identify potential bugs
- Note style issues

**Pass 3: Standards Validation**
- Validate against AGENTS.md quality gates
- Check CLAUDE.md pattern compliance
- Verify constitution.md principles
- Ensure project standards followed

**Pass 4: Architectural Assessment**
- Evaluate design decisions
- Check for code smells
- Assess maintainability
- Consider alternatives

## Review Checklist

### Code Quality

**Readability:**
- [ ] Code is self-documenting
- [ ] Variable names are descriptive
- [ ] Functions are appropriately sized (<50 lines ideal)
- [ ] Complex logic has explanatory comments
- [ ] No commented-out code

**Structure:**
- [ ] Proper separation of concerns
- [ ] DRY principle followed (no duplication)
- [ ] Single Responsibility Principle
- [ ] Appropriate abstraction levels
- [ ] Consistent code style

**Error Handling:**
- [ ] All errors handled appropriately
- [ ] Error messages are informative
- [ ] No silent failures
- [ ] Edge cases considered
- [ ] Graceful degradation

### Logic and Correctness

**Functionality:**
- [ ] Implements requirements correctly
- [ ] Edge cases handled
- [ ] Boundary conditions tested
- [ ] No off-by-one errors
- [ ] Correct algorithm complexity

**Data Handling:**
- [ ] Null/undefined checks
- [ ] Type safety maintained
- [ ] Data validation present
- [ ] Proper data transformations
- [ ] No data loss scenarios

**Concurrency (if applicable):**
- [ ] Race conditions prevented
- [ ] Proper locking/synchronization
- [ ] Thread-safe operations
- [ ] Deadlock prevention

### Security

**OWASP Top 10 Check:**
- [ ] No SQL injection vulnerabilities
- [ ] No XSS vulnerabilities
- [ ] No authentication bypass
- [ ] No broken access control
- [ ] No security misconfiguration
- [ ] No sensitive data exposure
- [ ] No insufficient logging
- [ ] No insecure deserialization
- [ ] No using components with known vulnerabilities
- [ ] No unvalidated redirects

**Input Validation:**
- [ ] All external input validated
- [ ] Proper sanitization
- [ ] Type checking
- [ ] Length/size limits enforced
- [ ] Whitelist validation where possible

**Authentication & Authorization:**
- [ ] Proper authentication checks
- [ ] Authorization verified
- [ ] Session management secure
- [ ] No hardcoded credentials
- [ ] Secure password handling

**Data Protection:**
- [ ] Sensitive data encrypted
- [ ] No secrets in code
- [ ] Proper key management
- [ ] HTTPS enforced
- [ ] Secure headers present

### Performance

**Efficiency:**
- [ ] No unnecessary computations
- [ ] Appropriate data structures
- [ ] Efficient algorithms
- [ ] No N+1 query problems
- [ ] Proper indexing

**Resource Usage:**
- [ ] No memory leaks
- [ ] File handles closed
- [ ] Database connections managed
- [ ] Caching implemented where beneficial
- [ ] Batch operations used appropriately

**Scalability:**
- [ ] Handles expected load
- [ ] Scales horizontally if needed
- [ ] No single points of contention
- [ ] Asynchronous where appropriate

### Testing

**Coverage:**
- [ ] Meets minimum coverage requirement (typically 80%)
- [ ] Critical paths fully tested
- [ ] Edge cases covered
- [ ] Error conditions tested
- [ ] Integration tests present

**Test Quality:**
- [ ] Tests are clear and maintainable
- [ ] Tests are deterministic (no flakiness)
- [ ] Good test data
- [ ] Appropriate mocking
- [ ] Tests run fast

**Test Types:**
- [ ] Unit tests for business logic
- [ ] Integration tests for APIs
- [ ] E2E tests for critical flows (if applicable)
- [ ] Security tests for sensitive operations

### Documentation

**Code Documentation:**
- [ ] Public APIs documented
- [ ] Complex logic explained
- [ ] Assumptions stated
- [ ] TODOs tracked
- [ ] Examples provided where helpful

**External Documentation:**
- [ ] README updated if needed
- [ ] API docs updated
- [ ] Migration guide if breaking changes
- [ ] Changelog entry
- [ ] Configuration documented

### Project Standards

**AGENTS.md Compliance:**
- [ ] Quality gates pass
- [ ] File patterns follow conventions
- [ ] Code review standards met
- [ ] Testing requirements satisfied

**CLAUDE.md Compliance:**
- [ ] Architectural patterns followed
- [ ] Preferred libraries used
- [ ] File organization correct
- [ ] Coding style consistent

**constitution.md Compliance:**
- [ ] Core principles upheld
- [ ] Constraints respected
- [ ] Security policies followed
- [ ] Performance requirements met

## Providing Feedback

### Feedback Structure

**Use standard format:**
```markdown
**[Severity]** [Category]: [Issue]

[Explanation]

[Suggestion]

[Example or reference]
```

**Severity levels:**
- **P0 (Critical):** Security vulnerabilities, data loss, breaking changes
- **P1 (Important):** Logic errors, quality gate violations, architecture issues
- **P2 (Minor):** Style issues, optimization opportunities, refactoring suggestions
- **Suggestion:** Nice-to-haves, alternative approaches, learning opportunities

### Example Feedback

**P0 - Security Issue:**
```markdown
**P0** Security: SQL Injection Vulnerability

The search query is directly concatenated into the SQL statement, allowing injection attacks.

```typescript
// Current (vulnerable)
const query = `SELECT * FROM users WHERE name = '${searchTerm}'`;

// Fix: Use parameterized queries
const query = 'SELECT * FROM users WHERE name = ?';
const results = await db.query(query, [searchTerm]);
```

Reference: OWASP SQL Injection Prevention Cheat Sheet
```

**P1 - Logic Error:**
```markdown
**P1** Logic: Off-by-One Error in Pagination

The pagination logic will skip the last item on each page due to incorrect boundary condition.

```typescript
// Current (incorrect)
const start = page * pageSize;
const end = start + pageSize;  // Should be exclusive

// Fix
const start = page * pageSize;
const end = Math.min(start + pageSize, totalItems);
```

Add test case TC-045 to catch this.
```

**P2 - Style Issue:**
```markdown
**P2** Style: Inconsistent Error Handling

Some functions return error objects while others throw exceptions. This is inconsistent with the pattern in CLAUDE.md section 3.2.

Consider refactoring to use consistent error handling:
```typescript
// Preferred pattern from CLAUDE.md
return { success: false, error: 'Invalid input' };
```

This is non-blocking but worth addressing for consistency.
```

**Suggestion:**
```markdown
**Suggestion** Refactoring: Extract Common Logic

The validation logic in lines 45-60 is similar to the validation in users.ts:123-138. Consider extracting to a shared validation utility.

```typescript
// Potential refactoring
import { validateEmail, validatePassword } from './utils/validation';

// Benefits: DRY, easier testing, consistent validation
```

Not required for this PR, but could be a good follow-up task.
```

### Tone and Approach

**DO:**
- ✅ Be specific and objective
- ✅ Explain the reasoning
- ✅ Provide examples and references
- ✅ Suggest solutions, not just problems
- ✅ Distinguish between must-fix and nice-to-have
- ✅ Acknowledge good practices
- ✅ Ask questions if unclear

**DON'T:**
- ❌ Be vague ("this looks wrong")
- ❌ Be condescending or dismissive
- ❌ Focus only on negatives
- ❌ Nitpick style in isolation (P2 or suggestion)
- ❌ Demand specific implementations (suggest)
- ❌ Leave feedback without explanation

## Review Iterations

### Iteration 1

**Reviewer:**
1. Conducts four-pass review
2. Categorizes findings by severity
3. Provides comprehensive feedback on GitHub PR
4. Requests changes if issues found
5. Approves if all checks pass

**Engineer:**
1. Reads all feedback carefully
2. Addresses all P0 and P1 issues
3. Considers P2 and suggestions
4. Pushes fixes
5. Responds to comments
6. Requests re-review

### Iteration 2

**Reviewer:**
1. Reviews changes since last review
2. Verifies P0/P1 issues resolved
3. Checks for new issues introduced
4. Approves if satisfied
5. If still issues, prepare for follow-up tickets

**Engineer:**
1. Addresses remaining feedback
2. If can't resolve in reasonable time:
   - Create follow-up tickets (P0 for critical)
   - Document in PR
   - Request approval to merge with tickets

### After 2 Iterations

If issues remain after 2 iterations:

**Create follow-up tickets:**
```bash
# Critical issues
bd create "Fix authentication bypass vulnerability" \
  -d "Critical issue from PR #456 review. Details: [...]" \
  -p P0

# Important issues
bd create "Refactor validation logic" \
  -d "Code quality issue from PR #456. Details: [...]" \
  -p P1

# Minor improvements
bd create "Extract common validation utilities" \
  -d "Refactoring opportunity from PR #456. Details: [...]" \
  -p P2
```

**Document in PR:**
```markdown
## Outstanding Issues

After 2 review iterations, the following issues remain:

### Created Follow-Up Tickets
- **P0:** beads-789 - Fix authentication bypass (blocking next release)
- **P1:** beads-790 - Refactor validation logic
- **P2:** beads-791 - Extract validation utilities

### Decision
Merging this PR to maintain forward progress. Critical issues tracked in P0 ticket and will be addressed immediately.
```

**Reviewer approves with conditions:**
```markdown
Approving with understanding that:
- P0 follow-up ticket beads-789 created
- Will be addressed before next release
- Current implementation is functional but needs security hardening
```

## GitHub PR Review Interface

### Inline Comments

```markdown
src/auth/middleware.ts:45

**P1** Security: Missing Input Validation

The email parameter should be validated before use.

Suggestion:
```typescript
if (!isValidEmail(email)) {
  return res.status(400).json({ error: 'Invalid email format' });
}
```
```

### Summary Comment

```markdown
## Review Summary

Thank you for this implementation! Overall structure is good. Found a few issues that need addressing:

### Critical Issues (P0)
- SQL injection vulnerability in search function (line 123)

### Important Issues (P1)
- Missing input validation (lines 45, 67)
- Off-by-one error in pagination (line 234)

### Minor Issues (P2)
- Inconsistent error handling pattern (throughout)
- Magic numbers should be constants (lines 89, 91)

### Positive Notes
✅ Excellent test coverage (95%)
✅ Clear variable names
✅ Good separation of concerns

### Next Steps
Please address P0 and P1 issues. P2 issues can be addressed or deferred to follow-up tickets. Let me know if you have questions!

### Standards Validation
✅ AGENTS.md quality gates pass
✅ CLAUDE.md patterns followed
⚠️ constitution.md: Missing rate limiting (see line 156)
```

## K2-Dev Review Workflow

### Reviewer Agent Process

1. **Read context:**
```bash
bd show beads-123
bd comments beads-123
cat AGENTS.md CLAUDE.md constitution.md
```

2. **Review code:**
```bash
gh pr view 456 --web
gh pr diff 456
```

3. **Provide feedback on GitHub:**
- Inline comments on specific lines
- Summary comment with categorized issues
- Use severity labels (P0/P1/P2/Suggestion)

4. **Report to Technical Lead:**
```
Review complete for beads-123 (PR #456):
- P0 issues: 1 (security)
- P1 issues: 2 (logic errors)
- P2 issues: 3 (style)
- Requesting changes
```

5. **Re-review after changes:**
- Check if issues addressed
- Verify no new issues introduced
- Approve or continue iteration

## Best Practices

### Review Timing

**DO:**
- ✅ Review within 24 hours of PR creation
- ✅ Block time for focused review
- ✅ Review when alert and focused
- ✅ Take breaks for large PRs

**DON'T:**
- ❌ Rush through review
- ❌ Review when tired or distracted
- ❌ Delay reviews indefinitely

### Review Scope

**DO:**
- ✅ Review all changed files
- ✅ Check test coverage
- ✅ Validate documentation updates
- ✅ Consider broader impact

**DON'T:**
- ❌ Only review main files
- ❌ Skip test files
- ❌ Ignore documentation
- ❌ Focus only on syntax

### Communication

**DO:**
- ✅ Be constructive and helpful
- ✅ Explain reasoning clearly
- ✅ Provide examples and references
- ✅ Acknowledge good work
- ✅ Ask for clarification if needed

**DON'T:**
- ❌ Be dismissive or condescending
- ❌ Make personal attacks
- ❌ Demand without explaining
- ❌ Ignore engineer's responses

## Command Reference

| Action | Command |
|--------|---------|
| View PR | `gh pr view 456` |
| View on web | `gh pr view 456 --web` |
| View diff | `gh pr diff 456` |
| Check CI | `gh pr checks 456` |
| Add review comment | Use GitHub web interface |
| Approve | `gh pr review 456 --approve` |
| Request changes | `gh pr review 456 --request-changes` |
| Comment | `gh pr review 456 --comment` |

## Review Templates

### Initial Review Comment

```markdown
## Review Started

Beginning review of PR #456 for beads-123.

Will be checking:
✓ Code quality and logic
✓ Security vulnerabilities
✓ Standards compliance (AGENTS.md, CLAUDE.md, constitution.md)
✓ Test coverage
✓ Documentation

Feedback coming soon.
```

### Approval Comment

```markdown
## ✅ Approved

Great work on this implementation! All quality gates pass.

### Highlights
- Excellent test coverage (95%)
- Clean, maintainable code
- Good security practices
- Clear documentation

### Standards Validation
✅ AGENTS.md quality gates pass
✅ CLAUDE.md patterns followed
✅ constitution.md principles upheld

Ready to merge!
```

### Request Changes Comment

```markdown
## 🔄 Changes Requested

Found some issues that need addressing before merge. Overall approach is solid!

### Must Fix (P0/P1)
1. [P0] SQL injection vulnerability (line 123)
2. [P1] Logic error in pagination (line 234)

### Should Consider (P2)
3. Inconsistent error handling (throughout)

### Suggestions
- Consider extracting validation logic to utility

Please address P0/P1 issues and push updates. Let me know if you have questions!
```

Follow these code review standards to maintain high code quality and catch issues early in k2-dev development workflows.
