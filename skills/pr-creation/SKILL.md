---
name: PR Creation
description: This skill should be used when the user asks to "create a pull request", "create a PR", "submit changes for review", "use gh pr create", "write PR description", or needs guidance on creating well-structured GitHub pull requests in k2-dev workflows.
version: 0.2.0
---

# PR Creation Skill

## Overview

Creating high-quality pull requests is essential for effective code review and team collaboration. This skill provides guidance for creating well-structured, informative PRs using GitHub CLI.

**Key Objectives:** Clear communication of changes, context for reviewers, traceability to requirements, professional presentation.

**Reference:** See k2-dev-reference.md#github-cli-gh-commands for complete gh command syntax.

## PR Structure

### Essential Components

Every PR should include:
1. **Title** - Clear, concise summary
2. **Description** - What and why
3. **Changes** - What was modified
4. **Testing** - How it was validated
5. **Reviewer Notes** - Guidance for reviewers
6. **References** - Links to tickets, docs

### Title Format

**Pattern:** `<type>: <summary> (<ticket-id>)`

**Types:** feat, fix, refactor, docs, test, chore, perf, security

**Examples:**
```
feat: Add JWT authentication middleware (beads-123)
fix: Resolve memory leak in image processor (beads-456)
security: Fix SQL injection vulnerability in search (beads-567)
```

**Reference:** See k2-dev-reference.md#git-commit-format

### Description Template

**IMPORTANT:** Always follow the Pull Request template in `.github/pull_request_template.md` if it exists.

**If no template exists, use:**

```markdown
## Summary
Brief overview of what this PR does and why it's needed.

## Changes
- Bullet point list of specific changes
- File modifications
- New dependencies
- Configuration updates

## Testing
- Test scenarios covered
- Manual testing performed
- Automated test additions

## Reviewer Notes
- Areas requiring special attention
- Design decisions made
- Known limitations or trade-offs
- Follow-up work planned

## References
- Fixes beads-123
- Related to beads-456
- Documentation: [link]
```

## Creating PRs with GitHub CLI

**CRITICAL**: Always use heredoc (EOF) pattern for PR descriptions to prevent escaping issues. Never use inline strings with special characters or multi-line content.

### Using Heredoc for PR Descriptions (RECOMMENDED)

```bash
gh pr create --title "feat: Add user authentication (beads-123)" \
             --body "$(cat <<'EOF'
## Summary
Implements JWT-based authentication for API endpoints.

## Changes
- Added auth middleware in src/auth/middleware.ts
- Created token generation endpoint POST /api/auth/login
- Added authentication tests
- Updated API documentation

## Testing
- Unit tests for middleware (100% coverage)
- Integration tests for login flow
- Manual testing with Postman
- Security testing with OWASP ZAP

## Reviewer Notes
- Used jsonwebtoken library (already in dependencies)
- Token expiry set to 24h (configurable via env var)
- Refresh token flow deferred to beads-124

## References
- Implements beads-123
- Follows patterns from AGENTS.md section 4.2
- API docs updated in docs/api/authentication.md
EOF
)"
```

### Additional Options

```bash
# With reviewers
gh pr create --title "..." --body "..." --reviewer @teammate

# Draft PR (work in progress)
gh pr create --title "..." --body "..." --draft

# With labels
gh pr create --title "..." --body "..." --label security
```

**Reference:** See k2-dev-reference.md#github-cli-gh-commands for all gh pr commands.

## K2-Dev PR Workflow

### Engineer Creates PR (After Implementation and Self-Review)

```bash
# Ensure all changes committed and pushed
git status  # Should be clean
git push

# Create PR
gh pr create --title "feat: Implement authentication (beads-123)" \
             --body "$(cat <<'EOF'
## Summary
Implements JWT authentication as specified in beads-123.

## Changes
- JWT middleware with token validation
- Login endpoint: POST /api/auth/login
- Logout endpoint: POST /api/auth/logout
- Auth tests with 95% coverage
- Updated API documentation

## Testing
- Unit tests: src/auth/*.test.ts (95% coverage)
- Integration tests: tests/integration/auth.test.ts
- Manual testing: All scenarios in test plan (bd comments beads-123)
- Security: No hardcoded secrets, input validation added

## Quality Gates
✅ Type checking passed (npm run type-check)
✅ Linting passed (npm run lint)
✅ Tests passed (npm test)
✅ Coverage: 95% (exceeds 80% requirement)
✅ Security scan: No vulnerabilities

## Reviewer Notes
- Token secret read from environment variable JWT_SECRET
- Used bcrypt for password hashing (cost factor 12 per constitution.md)
- Session management deferred to beads-124 (logged as P1 follow-up)

## References
- Implements: beads-123
- Follows: AGENTS.md authentication standards
- Architecture: CLAUDE.md section on API middleware
- Security: constitution.md password handling requirements
EOF
)"

# Get PR URL and add to beads
PR_URL=$(gh pr view --json url -q .url)
bd comments beads-123 add "PR created: ${PR_URL}"
```

## PR Description Best Practices

### DO

✅ **Be specific about changes:**
```markdown
## Changes
- Added JWT middleware in src/auth/middleware.ts
- Implemented token validation with expiry checking
- Added refresh token rotation
```

✅ **Include test coverage:**
```markdown
## Testing
- Unit tests: 95% coverage (src/auth/*.test.ts)
- Integration tests: Login flow, token refresh, logout
- Manual: Tested with Postman collection (attached)
- Security: OWASP ZAP scan - no vulnerabilities
```

✅ **Explain design decisions:**
```markdown
## Reviewer Notes
- Used RS256 for JWT signing (more secure than HS256)
- Token expiry: 15min access + 7day refresh (industry standard)
- Chose jsonwebtoken over jose (better TypeScript support)
```

✅ **Link to requirements:**
```markdown
## References
- Implements: beads-123
- Blocked by: beads-120 (now merged)
- Follows: AGENTS.md authentication patterns
- Architecture: Matches existing OAuth flow in src/auth/oauth.ts
```

✅ **Highlight quality gates:**
```markdown
## Quality Gates
✅ Type checking passed
✅ Linting passed
✅ Tests passed (95% coverage)
✅ Security scan clean
```

### DON'T

❌ **Don't be vague:**
```markdown
## Changes
- Updated some files
- Fixed bugs
- Added tests
```

❌ **Don't skip testing details:**
```markdown
## Testing
- Tested it
```

❌ **Don't omit reviewer context:**
```markdown
## Reviewer Notes
- Please review
```

❌ **Don't forget references:**
```markdown
## References
(empty)
```

## Special Cases

### Multiple Tickets in One PR

```bash
gh pr create --title "feat: Implement user profile features (beads-123, beads-234)" \
             --body "$(cat <<'EOF'
## Summary
Implements user profile viewing and editing features.

## Tickets
- beads-123: Profile viewing page
- beads-234: Profile editing functionality

## Changes
[Combined changes for both tickets]

## Testing
[Tests covering both tickets]

## References
- Implements: beads-123, beads-234
EOF
)"
```

### Breaking Changes

Highlight prominently:

```markdown
## ⚠️ Breaking Changes

- API endpoint renamed: `/users/me` → `/api/v2/profile`
- Response format changed: `user` field → `profile` field
- Migration guide: docs/migrations/v2.md

## Migration Steps
1. Update API calls to use new endpoint
2. Update response parsing for new field names
3. Update tests
```

### Security Fixes

```bash
gh pr create --title "security: Fix SQL injection in search (beads-567)" \
             --body "..." \
             --label security
```

Add sensitivity note:
```markdown
## Security Note
This PR fixes a critical SQL injection vulnerability. Details in private security advisory GHSA-xxxx.

Do not merge until security advisory is prepared.
```

## After PR Creation

### Update Beads Task

```bash
PR_NUM=$(gh pr view --json number -q .number)
bd comments beads-123 add "PR created: #${PR_NUM}"
bd update beads-123 --status in_progress
```

### Monitor PR

```bash
gh pr status              # Check status
gh pr checks              # View CI checks
gh pr view 123 --json reviews   # View reviews
```

## PR Template Example

Create `.github/pull_request_template.md`:

```markdown
## Summary
<!-- Brief overview of changes and motivation -->

## Changes
<!-- Bullet list of specific changes -->
-
-

## Testing
<!-- How were these changes tested? -->
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed
- [ ] Security testing completed

## Quality Gates
<!-- Check all that apply -->
- [ ] Type checking passes
- [ ] Linting passes
- [ ] Tests pass (coverage ≥80%)
- [ ] Security scan clean
- [ ] Self-review completed

## Reviewer Notes
<!-- Important context for reviewers -->
-

## References
<!-- Related tickets, docs, PRs -->
- Implements: beads-XXX
- Related:
- Docs:
```

## Integration with Review Process

After PR creation:
1. **Engineer:** PR created, URL added to beads
2. **Reviewer:** Reviews PR, adds comments on GitHub
3. **Engineer:** Addresses feedback, updates PR
4. **Reviewer:** Re-reviews, approves or requests more changes
5. **Maximum 2 iterations:** Then create follow-up tickets
6. **Technical Lead:** Merges approved PR

All review feedback happens on GitHub PR, not in beads. Beads only tracks PR URL and status.

Create clear, comprehensive PRs to facilitate efficient reviews and maintain high code quality standards in k2-dev workflows.

**Reference:** See k2-dev-reference.md for gh commands, PR template structure, and common patterns.
