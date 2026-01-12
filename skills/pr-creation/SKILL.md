---
name: PR Creation
description: Quick PR creation reference
version: 0.1.0
---

# PR Creation

## Title Format

```
<type>: <summary> (<ticket-id>)
```

Types: feat, fix, refactor, docs, test, chore, perf, security

### Description Template

**IMPORTANT:** Always follow the Pull Request template in `.github/pull_request_template.md` if it exists.

**If no template exists, use:**

```markdown
## Summary

Brief description of changes

## Changes

- List of changes

## Testing

- How validated

## References

- Implements: <ticket>
```

## Create PR

```bash
gh pr create --title "feat: Add feature (beads-123)" \
             --body "$(cat <<'EOF'
## Summary
...

## Changes
- ...

## Testing
- ...

## References
- Implements: beads-123
EOF
)"
```

## Get PR URL

```bash
gh pr view --json url -q .url
```
