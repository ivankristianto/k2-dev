---
name: Quality Gates
description: This skill should be used when the user asks to "check quality gates", "validate against standards", "read AGENTS.md", "enforce project patterns", "follow constitution.md", "validate code quality", or needs guidance on quality standards enforcement in k2-dev workflows.
version: 0.2.0
---

# Quality Gates Skill

## Overview

Quality gates are project-defined standards that all code changes must meet. In k2-dev, gates are defined in three configuration files at project root and enforced throughout the development workflow.

**Configuration Files:**
- **AGENTS.md** - Quality standards, file validation patterns, agent guidelines
- **constitution.md** - Core principles and immutable constraints

**Reference:** See k2-dev-reference.md#quality-standards-files for overview.

## Core Concepts

### What are Quality Gates?

Checkpoints code must pass before progressing:
- **Type checking:** No TypeScript errors
- **Test coverage:** Minimum percentage coverage
- **Linting:** Code style compliance
- **Security:** No known vulnerabilities
- **Performance:** Acceptable metrics
- **Documentation:** Required comments/docs

### Why Quality Gates?

**Consistency:** Uniform code quality across team
**Standards:** Enforce project-specific patterns
**Prevention:** Catch issues before review
**Automation:** Reduce manual review burden
**Documentation:** Codify team agreements

### Enforcement Points

1. **Engineer self-review** before creating PR
2. **PreToolUse hook** before file changes
3. **Reviewer validation** during code review
4. **CI/CD pipeline** automated checks

## Configuration Files

### AGENTS.md Structure

Located at project root, defines:
- Quality gate thresholds (type checking, test coverage, linting)
- File validation patterns (which files trigger validation)
- Coding standards and review criteria
- Agent behavioral guidelines

**Example sections:**
```markdown
## Quality Gates
- Type Checking: All TypeScript files must pass `tsc --noEmit`
- Test Coverage: Minimum 80% line coverage
- Linting: ESLint must pass
- Security: OWASP Top 10 compliance

## File Validation Patterns
Validate: `src/**/*.ts`, `src/**/*.tsx`, `lib/**/*.js`
Skip: `**/*.test.ts`, `docs/**/*`, `*.md`

## Code Review Standards
Blocking: Security vulnerabilities, logic errors, performance regressions
Non-Blocking: Style preferences, naming suggestions, refactoring opportunities
```

### AGENTS.md Structure

Located at project root, defines all quality standards:
- Quality gate thresholds (type checking, test coverage, linting)
- File validation patterns (which files trigger validation)
- Coding standards and review criteria
- Agent behavioral guidelines
- Project-specific patterns and conventions

**Example sections:**

Defines immutable principles:
- Core project values
- Non-negotiable constraints
- Architectural decisions
- Security policies
- Compliance requirements

**Example:**
```markdown
## Core Principles
- Security First: Never compromise security for convenience
- User Privacy: GDPR compliance mandatory
- Performance: API response time <200ms (p95)

## Non-Negotiable Constraints
1. All passwords hashed with bcrypt (cost factor ≥12)
2. HTTPS only in production
3. Rate limiting on all public endpoints
```

## Reading Configuration Files

### Before Starting Work

All agents must read configuration:

```bash
# Check files exist
ls -la AGENTS.md constitution.md

# Read files
cat AGENTS.md
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

Engineer validates against both files:
- AGENTS.md quality gates and patterns
- constitution.md principles

## Validation Workflow

### Engineer Self-Review Checklist

**From AGENTS.md:**
- [ ] Type checking passes
- [ ] Test coverage meets threshold
- [ ] Linting passes
- [ ] Security checks pass
- [ ] File patterns match
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
cat AGENTS.md constitution.md

# Review changes
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

**Reference:** See k2-dev-reference.md#review-severity-levels

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

**Standard:** All TypeScript must type-check
**Validation:** `tsc --noEmit`
**Pass:** No errors | **Fail:** Any type errors

### Test Coverage

**Standard:** Minimum 80% coverage (from AGENTS.md)
**Validation:** `npm test -- --coverage`
**Pass:** ≥80% line coverage | **Fail:** <80%

### Linting

**Standard:** ESLint must pass
**Validation:** `npm run lint`
**Pass:** No errors | **Fail:** Any linting errors

### Security

**Standard:** No hardcoded secrets (constitution.md), input validation required (AGENTS.md)
**Validation:**
```bash
git diff | grep -E '(api_key|password|secret)'  # Check for secrets
grep -r "req.body" src/api/                     # Review for input validation
```
**Pass:** No secrets, all inputs validated | **Fail:** Secrets found, missing validation

## Handling Quality Gate Failures

### During Self-Review

If quality gates fail:
1. Identify which check failed
2. Fix root cause
3. Re-validate
4. Repeat until all gates pass
5. Document any challenges in PR

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
1. Read error message (understand what failed)
2. Check AGENTS.md (review validation patterns)
3. Fix issue (address root cause)
4. Retry operation (should now pass)

**Example:**
```bash
# Hook blocks Write operation
Error: Type checking failed for src/auth.ts

# Fix type errors
vi src/auth.ts

# Retry - should succeed now
```

## Best Practices

### DO
✅ Read configuration first: `cat AGENTS.md constitution.md`
✅ Validate early and often during implementation
✅ Understand rationale for each standard
✅ Ask for clarification if standard is unclear
✅ Document exceptions with rationale and mitigation

### DON'T
❌ Skip reading AGENTS.md, constitution.md
❌ Bypass quality gates (disable checks, commit --no-verify, merge failing PRs)
❌ Ignore warnings (address linting warnings, test failures, type errors)
❌ Negotiate immutable constraints (constitution.md is non-negotiable)

## Project Configuration Templates

If AGENTS.md doesn't exist, suggest creating:

```markdown
# AGENTS.md Template

## Quality Gates
- Type Checking: `npm run type-check` - Must pass
- Test Coverage: `npm test -- --coverage` - Threshold: 80%
- Linting: `npm run lint` - Must pass

## File Validation Patterns
Validate: `src/**/*.ts`, `src/**/*.tsx`
Skip: `**/*.test.ts`, `docs/**/*`

## Review Standards
Blocking: Security vulnerabilities, logic errors
Non-Blocking: Style preferences
```

## Integration with K2-Dev Workflow

**Technical Lead:**
- Reads configuration before planning
- Validates approach against architecture
- Ensures quality gates are clear

**Engineer:**
- Reads configuration at start
- Validates during implementation
- Self-reviews against both files
- Only creates PR when all gates pass

**Reviewer:**
- Validates against AGENTS.md standards
- Ensures constitution.md principles upheld
- Categorizes issues by severity (P0/P1/P2)

**Planner:**
- References configuration during planning
- Ensures plans comply with architecture
- Considers quality gate requirements

Follow quality gates rigorously to maintain code quality and project standards throughout the k2-dev development lifecycle.

**Reference:** See k2-dev-reference.md for priority levels and common patterns.
