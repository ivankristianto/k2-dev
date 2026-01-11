# Reviewer Agent Fix - Phase 1 Stopping Issue

## Problem

The Reviewer agent was stopping at Phase 1 and only announcing what it was going to do instead of actually executing tools:

```
Phase 1: Context Gathering
Reading Project Standards
Let me read the project standards files:
/Users/ivan/Works/AI/eval/AGENTS.md
```

The agent would show these messages but not actually use the Read tool or proceed to Phase 2.

## Root Cause

The Phase 1 instructions were **too descriptive and narrative**, giving the agent room to interpret them as "explain what you're going to do" rather than "just do it immediately":

**Before:**
```markdown
**STEP 1: Read Project Standards** (CRITICAL - Execute this FIRST):

Use the Read tool to read these files from the PROJECT root (not plugin root):

- AGENTS.md - Quality gates, file validation patterns, agent behavior guidelines
- CLAUDE.md - Claude-specific project standards, patterns, and preferences
- docs/constitution.md OR specs/constitution.md - Project principles
```

This language ("Use the Read tool to read these files") allowed the agent to narrate instead of execute.

## Solution

**1. Made Phase 1 instructions imperative and action-oriented:**

**After:**
```markdown
**CRITICAL: Execute tool calls IMMEDIATELY. NO narration, NO "let me do X", NO "I'll start by" - just execute tools.**

**STEP 1: Read project standards** (execute these Read tool calls first):

```
Read: {project_root}/AGENTS.md
Read: {project_root}/CLAUDE.md
Read: {project_root}/docs/constitution.md OR {project_root}/specs/constitution.md
```
```

**2. Added explicit anti-narration instruction:**
- "NO narration, NO 'let me do X', NO 'I'll start by' - just execute tools"

**3. Showed tool calls as code examples:**
- Changed from descriptive paragraphs to executable code blocks
- Makes it clear these are literal tool calls to execute

**4. Added transition guard:**
```markdown
**After completing Phase 1 context gathering, immediately proceed to Phase 2 analysis. Do not stop or wait for confirmation.**
```

This prevents the agent from pausing between phases.

## Testing

To test this fix:

1. Trigger a review workflow: `/k2:start beads-{id}`
2. Observe that Reviewer agent:
   - Immediately executes Read tool calls (no "Let me read..." narration)
   - Proceeds through all Phase 1 steps
   - Transitions directly to Phase 2 without stopping
   - Completes full review workflow

## Files Changed

- `agents/reviewer.md` - Lines 52-97 (Phase 1 rewrite)

## Version Bump Required

This is a bug fix, so version should be bumped from 0.9.1 → 0.9.2
