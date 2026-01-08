---
name: Quality Gates
description: This skill should be used when the user asks to "check quality gates", "validate against standards", "read AGENTS.md", "enforce CLAUDE.md patterns", "follow constitution.md", "validate code quality", or needs guidance on quality standards enforcement in k2-dev workflows.
version: 0.1.0
---

# Quality Gates Skill

## Overview

Quality gates are project-defined standards that all code changes must meet. In k2-dev, quality gates are defined in three configuration files at the project root and enforced throughout the development workflow.

**Configuration Files:**
- **AGENTS.md** - Agent behavior, quality standards, file validation patterns
- **CLAUDE.md** - Claude-specific project standards and coding patterns
- **constitution.md** - Core project principles and immutable constraints

## Core Concepts

### What are Quality Gates?

Quality gates are checkpoints that code must pass before progressing:
- **Type checking:** No TypeScript errors
- **Test coverage:** Minimum percentage coverage
- **Linting:** Code style compliance
- **Security:** No known vulnerabilities
- **Performance:** Acceptable metrics
- **Documentation:** Required comments/docs

### Why Quality Gates?

**Consistency:** Maintain uniform code quality across team
**Standards:** Enforce project-specific patterns and practices
**Prevention:** Catch issues before review
**Automation:** Reduce manual review burden
**Documentation:** Codify team agreements

### Enforcement Points

Quality gates enforced at:
1. **Engineer self-review** before creating PR
2. **PreToolUse hook** before file changes
3. **Reviewer validation** during code review
4. **CI/CD pipeline** automated checks

## Configuration Files

### AGENTS.md

Located at project root, defines:
- Agent behavioral guidelines
- Quality standards and thresholds
- File validation patterns
- Coding standards
- Review criteria

**Example structure:**
```markdown
# Agent Guidelines

## Quality Gates

### Type Checking
- All TypeScript files must pass `tsc --noEmit`
- No `any` types without explicit justification
- Strict mode enabled

### Test Coverage
- Minimum 80% line coverage
- 100% coverage for critical paths
- Tests for all public APIs

### Code Quality
- No `console.log` in production code
- Maximum function complexity: 15
- Maximum file length: 500 lines

### Security
- Input validation on all external data
- No hardcoded secrets or credentials
- OWASP Top 10 compliance

## File Validation Patterns

Validate these file types before changes:
- `src/**/*.ts`
- `src/**/*.tsx`
- `lib/**/*.js`
- `api/**/*.py`

Skip validation for:
- `**/*.test.ts`
- `**/*.spec.js`
- `docs/**/*`
- `*.md`

## Code Review Standards

### Blocking Issues
- Security vulnerabilities
- Logic errors
- Performance regressions
- Breaking changes without migration

### Non-Blocking Issues
- Style preferences
- Naming suggestions
- Refactoring opportunities
```

### CLAUDE.md

Located at project root, defines Claude-specific patterns:
- Project architecture patterns
- Preferred libraries and tools
- File organization conventions
- Coding style preferences
- Integration patterns

**Example structure:**
```markdown
# Claude Project Standards

## Architecture

### API Layer
- Use Express.js for REST APIs
- OpenAPI spec in `api/openapi.yaml`
- Route handlers in `api/routes/`
- Middleware in `api/middleware/`

### Data Layer
- TypeORM for database access
- Entities in `src/entities/`
- Repositories in `src/repositories/`
- Migrations in `migrations/`

### Testing
- Jest for unit and integration tests
- Supertest for API tests
- Test files colocated: `feature.ts` + `feature.test.ts`

## Coding Patterns

### Error Handling
```typescript
// Preferred pattern
try {
  const result = await operation();
  return { success: true, data: result };
} catch (error) {
  logger.error('Operation failed', { error });
  return { success: false, error: error.message };
}
```

### API Responses
```typescript
// Standard response format
interface APIResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}
```

## File Organization

```
src/
├── entities/      # TypeORM entities
├── repositories/  # Data access
├── services/      # Business logic
├── controllers/   # Request handlers
├── middleware/    # Express middleware
└── utils/         # Shared utilities
```
```

### constitution.md

Located at project root, defines immutable principles:
- Core project values
- Non-negotiable constraints
- Architectural decisions
- Security policies
- Compliance requirements

**Example structure:**
```markdown
# Project Constitution

## Core Principles

### Security First
- Never compromise security for convenience
- All external input must be validated
- Follow OWASP guidelines strictly
- Regular security audits required

### User Privacy
- Minimal data collection
- Explicit consent required
- GDPR compliance mandatory
- Data retention limits enforced

### Performance
- API response time <200ms (p95)
- Database queries optimized
- Caching strategy required
- Load testing before major releases

### Maintainability
- Code must be self-documenting
- Complex logic requires comments
- Public APIs must have docs
- Breaking changes need migration guides

## Non-Negotiable Constraints

1. No third-party analytics without approval
2. All passwords hashed with bcrypt (cost factor ≥12)
3. HTTPS only in production
4. Rate limiting on all public endpoints
5. Automated backups every 6 hours
```

## Reading Configuration Files

### Before Starting Work

All agents must read configuration:

```bash
# Check files exist
ls -la AGENTS.md CLAUDE.md constitution.md

# Read files
cat AGENTS.md
cat CLAUDE.md
cat constitution.md
```

**Parse for:**
- Quality gate thresholds
- File validation patterns
- Required checks
- Architectural patterns
- Security requirements
- Testing standards

### During Implementation

Reference configuration throughout:
- Check patterns before implementing
- Validate approach against architecture
- Ensure compliance with constraints
- Follow coding standards

### Before Self-Review

Engineer validates against all three files:
- AGENTS.md quality gates
- CLAUDE.md patterns
- constitution.md principles

## Validation Workflow

### Engineer Self-Review Checklist

**From AGENTS.md:**
- [ ] Type checking passes
- [ ] Test coverage meets threshold
- [ ] Linting passes
- [ ] Security checks pass
- [ ] File patterns match

**From CLAUDE.md:**
- [ ] Follows project architecture
- [ ] Uses preferred libraries
- [ ] Matches coding patterns
- [ ] Proper file organization

**From constitution.md:**
- [ ] Upholds core principles
- [ ] Respects constraints
- [ ] Security policies followed
- [ ] Performance requirements met

### Reviewer Validation

Reviewer checks compliance:

```bash
# Read configuration
cat AGENTS.md CLAUDE.md constitution.md

# Review changes against standards
git diff main...feature/beads-123

# Validate specific checks
npm run type-check
npm run lint
npm test -- --coverage
```

**Validation categories:**
- **P0 (Critical):** Security, data loss, breaking changes
- **P1 (Important):** Quality gates, architecture violations
- **P2 (Minor):** Style issues, optimization opportunities

### PreToolUse Hook Validation

Hook validates before Write/Edit operations:

```bash
# Check if file matches validation patterns from AGENTS.md
# If match, run quality checks
# Block operation if checks fail
```

Validates:
- File matches pattern (e.g., `src/**/*.ts`)
- Type checking passes
- Linting passes
- No security violations

## Quality Gate Examples

### Type Checking

**Standard:**
```typescript
// AGENTS.md: "All TypeScript must type-check"
npm run type-check  # Must pass
```

**Validation:**
```bash
tsc --noEmit
```

**Pass:** No errors
**Fail:** Any type errors

### Test Coverage

**Standard:**
```
// AGENTS.md: "Minimum 80% coverage"
```

**Validation:**
```bash
npm test -- --coverage
```

**Pass:** ≥80% line coverage
**Fail:** <80% line coverage

### Linting

**Standard:**
```
// AGENTS.md: "ESLint must pass"
```

**Validation:**
```bash
npm run lint
```

**Pass:** No errors
**Fail:** Any linting errors

### Security

**Standard:**
```
// constitution.md: "No hardcoded secrets"
// AGENTS.md: "Input validation required"
```

**Validation:**
```bash
# Check for secrets
git diff | grep -E '(api_key|password|secret)'

# Review for input validation
grep -r "req.body" src/api/
```

**Pass:** No secrets, all inputs validated
**Fail:** Secrets found, missing validation

## Handling Quality Gate Failures

### During Self-Review

If quality gates fail:

1. **Identify failure:** Which check failed?
2. **Fix issue:** Address root cause
3. **Re-validate:** Run checks again
4. **Repeat:** Until all gates pass
5. **Document:** Note any challenges in PR

### During Review

If Reviewer finds issues:

**Iteration 1:**
- Reviewer provides specific feedback
- Engineer fixes issues
- Re-review

**Iteration 2:**
- Reviewer re-checks fixes
- If passes, approve
- If fails, create follow-up tickets

**Follow-up tickets:**
- P0: Critical issues blocking merge
- P1: Important issues for next sprint
- P2: Minor improvements for backlog

### PreToolUse Hook Blocks

If hook blocks operation:

1. **Read error message:** Understand what failed
2. **Check AGENTS.md:** Review validation patterns
3. **Fix issue:** Address root cause
4. **Retry operation:** Should now pass

**Example:**
```bash
# Hook blocks Write operation
Error: Type checking failed for src/auth.ts

# Fix type errors
vi src/auth.ts

# Retry
# Should succeed now
```

## Best Practices

### DO

✅ **Read configuration first**
```bash
cat AGENTS.md CLAUDE.md constitution.md
```

✅ **Validate early and often**
- Check during implementation
- Validate before PR
- Re-check after changes

✅ **Understand rationale**
- Why does this standard exist?
- What problem does it prevent?
- How does it help the project?

✅ **Ask for clarification**
- Use AskUserQuestion if standard is unclear
- Request examples if needed
- Propose alternatives if justified

✅ **Document exceptions**
- Explain why standard can't be met
- Propose mitigation
- Get explicit approval

### DON'T

❌ **Don't skip configuration reading**
- Always read AGENTS.md, CLAUDE.md, constitution.md
- Never assume you know the standards

❌ **Don't bypass quality gates**
- Don't disable checks
- Don't commit with `--no-verify`
- Don't merge failing PRs

❌ **Don't ignore warnings**
- Address all linting warnings
- Fix all test failures
- Resolve all type errors

❌ **Don't negotiate immutable constraints**
- constitution.md defines non-negotiables
- These cannot be bypassed
- Raise concerns with user if needed

## Project Configuration Templates

If AGENTS.md doesn't exist, suggest creating:

```markdown
# AGENTS.md Template

## Quality Gates

### Type Checking
- Command: `npm run type-check`
- Must pass: Yes

### Test Coverage
- Command: `npm test -- --coverage`
- Threshold: 80%

### Linting
- Command: `npm run lint`
- Must pass: Yes

## File Validation Patterns

Validate before changes:
- `src/**/*.ts`
- `src/**/*.tsx`

Skip validation:
- `**/*.test.ts`
- `docs/**/*`

## Review Standards

### Blocking
- Security vulnerabilities
- Logic errors

### Non-Blocking
- Style preferences
```

## Integration with K2-Dev Workflow

**Technical Lead:**
- Reads configuration before planning
- Validates approach against architecture
- Ensures quality gates are clear

**Engineer:**
- Reads configuration at start
- Validates during implementation
- Self-reviews against all three files
- Only creates PR when all gates pass

**Reviewer:**
- Validates against AGENTS.md standards
- Checks CLAUDE.md pattern compliance
- Ensures constitution.md principles upheld
- Categorizes issues by severity (P0/P1/P2)

**Planner:**
- References configuration during planning
- Ensures plans comply with architecture
- Considers quality gate requirements

Follow quality gates rigorously to maintain code quality and project standards throughout the k2-dev development lifecycle.
