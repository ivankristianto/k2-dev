---
name: PR Creation
description: This skill should be used when the user asks to "create a pull request", "create a PR", "submit changes for review", "use gh pr create", "write PR description", or needs guidance on creating well-structured GitHub pull requests in k2-dev workflows.
version: 0.1.0
---

# PR Creation Skill

## Overview

Creating high-quality pull requests is essential for effective code review and team collaboration. This skill provides guidance for creating well-structured, informative PRs using GitHub CLI.

**Key Objectives:**

- Clear communication of changes
- Context for reviewers
- Traceability to requirements
- Professional presentation

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

**Pattern:**

```
<type>: <summary> (<ticket-id>)
```

**Types:**

- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring
- `docs`: Documentation
- `test`: Test additions/fixes
- `chore`: Maintenance tasks
- `perf`: Performance improvements
- `security`: Security fixes

**Examples:**

```
feat: Add JWT authentication middleware (beads-123)
fix: Resolve memory leak in image processor (beads-456)
refactor: Extract validation logic to utils (beads-789)
docs: Update API documentation for v2 endpoints (beads-234)
security: Fix SQL injection vulnerability in search (beads-567)
```

### Description Template

**Always Follow the Pull Request template inside .github folder if exists**

If Pull Request template not exists then follow the following template:

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

### Basic PR Creation

```bash
# From worktree with pushed branch
gh pr create --title "feat: Add feature X (beads-123)" \
             --body "PR description here..."
```

### Using PR Template

If project has `.github/pull_request_template.md`:

```bash
# Auto-fills with template
gh pr create --title "feat: Add feature X (beads-123)"
```

Template prompts for structured information.

### From Heredoc

For complex descriptions:

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

### With Reviewers

```bash
gh pr create --title "feat: Add feature X (beads-123)" \
             --body "..." \
             --reviewer @teammate1,@teammate2
```

### Draft PR

For work in progress:

```bash
gh pr create --title "feat: Add feature X (beads-123)" \
             --body "..." \
             --draft
```

## K2-Dev PR Workflow

### Engineer Creates PR

After implementation and self-review:

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
```

### PR URL in Beads Comment

Link PR to task:

```bash
# Get PR URL
PR_URL=$(gh pr view --json url -q .url)

# Add to beads comment
bd comments beads-123 add "PR created: ${PR_URL}"
```

### Reviewer Access

Reviewer uses PR for feedback:

```bash
# View PR
gh pr view 123

# Review on GitHub web interface
gh pr view 123 --web

# Add comments via CLI or web
```

## PR Description Best Practices

### DO

✅ **Be specific about changes**

```markdown
## Changes

- Added JWT middleware in src/auth/middleware.ts
- Implemented token validation with expiry checking
- Added refresh token rotation
```

Not:

```markdown
## Changes

- Added authentication
```

✅ **Include test coverage**

```markdown
## Testing

- Unit tests: 95% coverage (src/auth/\*.test.ts)
- Integration tests: Login flow, token refresh, logout
- Manual: Tested with Postman collection (attached)
- Security: OWASP ZAP scan - no vulnerabilities
```

✅ **Explain design decisions**

```markdown
## Reviewer Notes

- Used RS256 for JWT signing (more secure than HS256)
- Token expiry: 15min access + 7day refresh (industry standard)
- Chose jsonwebtoken over jose (better TypeScript support)
```

✅ **Link to requirements**

```markdown
## References

- Implements: beads-123
- Blocked by: beads-120 (now merged)
- Follows: AGENTS.md authentication patterns
- Architecture: Matches existing OAuth flow in src/auth/oauth.ts
```

✅ **Highlight quality gates**

```markdown
## Quality Gates

✅ Type checking passed
✅ Linting passed
✅ Tests passed (95% coverage)
✅ Security scan clean
```

### DON'T

❌ **Don't be vague**

```markdown
## Changes

- Updated some files
- Fixed bugs
- Added tests
```

❌ **Don't skip testing details**

```markdown
## Testing

- Tested it
```

❌ **Don't omit reviewer context**

```markdown
## Reviewer Notes

- Please review
```

❌ **Don't forget references**

```markdown
## References

(empty)
```

## PR Title Best Practices

### Good Titles

✅ Clear and specific:

```
feat: Add JWT authentication with refresh tokens (beads-123)
fix: Resolve race condition in payment processor (beads-456)
refactor: Extract common validation logic to utils (beads-789)
perf: Optimize database queries with eager loading (beads-234)
security: Fix XSS vulnerability in comment rendering (beads-567)
```

### Bad Titles

❌ Vague or unclear:

```
Update files (beads-123)
Fix bug (beads-456)
Changes (beads-789)
beads-234
Improvements (beads-567)
```

## Special Cases

### Multiple Tickets in One PR

When working on multiple tickets in same worktree:

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

Highlight breaking changes:

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

Mark as security fix:

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
# Get PR number
PR_NUM=$(gh pr view --json number -q .number)

# Update beads
bd comments beads-123 add "PR created: #${PR_NUM}"
bd update beads-123 --status in_progress
```

### Monitor PR

```bash
# Check status
gh pr status

# View checks
gh pr checks

# View reviews
gh pr view 123 --json reviews
```

## PR Template Example

Create `.github/pull_request_template.md`:

```markdown
## Summary

<!-- Brief overview of changes and motivation -->

## Changes

## <!-- Bullet list of specific changes -->

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

## <!-- Important context for reviewers -->

-

## References

<!-- Related tickets, docs, PRs -->

- Implements: beads-XXX
- Related:
- Docs:
```

## Command Reference

| Action          | Command                                   |
| --------------- | ----------------------------------------- |
| Create PR       | `gh pr create --title "..." --body "..."` |
| Create draft PR | `gh pr create --draft`                    |
| Use template    | `gh pr create` (interactive)              |
| View PR         | `gh pr view 123`                          |
| View on web     | `gh pr view 123 --web`                    |
| Check status    | `gh pr status`                            |
| List my PRs     | `gh pr list --author @me`                 |
| Add reviewer    | `gh pr edit 123 --add-reviewer @user`     |
| Add label       | `gh pr edit 123 --add-label bug`          |

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
