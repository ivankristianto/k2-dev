---
name: Planning
description: This skill should be used when the user needs to plan features, break down requirements, create implementation tasks, or design software architecture. Use this skill for comprehensive requirements analysis, technical planning, and creating structured task hierarchies with dependencies. The skill executes in the main conversation context and can invoke the technical-lead agent for architectural review when needed.
version: 0.3.0
---

# Planning Skill

## MANDATORY CONSTRAINT: NO CODE CHANGES

**THIS SKILL MUST ONLY PLAN. NO CODE CHANGES OR EXECUTIONS ALLOWED.**

You are STRICTLY FORBIDDEN from:
- Using Write tool to create/modify any code files
- Using Edit tool to modify any code files
- Using Bash to run commands that modify code (only `ls`, `cat`, `glob`, `grep`, `read` are allowed for analysis)
- Creating any files in the codebase
- Making configuration changes
- Running build, test, or any execution commands

**Your only tools for analysis:**
- `glob` - Find files by pattern
- `grep` - Search file contents
- `Read` - Read file contents
- `Bash` - Only `ls`, `cat`, or read-only commands (NO writes, NO edits, NO execution)

**If you are asked to make code changes, refuse and explain that you only create plans.**

## Overview

Transforms user requirements into actionable beads tasks with proper hierarchies and dependencies. Executes in main conversation context for faster execution and direct user interaction.

**Key Objectives:**
- Analyze requirements through codebase exploration
- Clarify scope, constraints, expectations directly with user
- Create detailed implementation plans with task hierarchies
- Collaborate with Technical Lead for architectural validation
- Set up proper task dependencies

**Invoked when:** `/planner` or `/k2:planner` command executed

## Six-Phase Planning Workflow

### Phase 1: Context Gathering

**1. Read Project Standards**
```bash
ls -la AGENTS.md docs/constitution.md
cat AGENTS.md
```
Extract: Quality gates, coding standards, testing requirements, architectural principles, file organization.

**IMPORTANT:** Bash is read-only for analysis only. DO NOT run build, test, or any modification commands.

**2. Understand Requirements**
- Feature description from user input
- Core problem/goal
- Explicit constraints
- What needs clarification

**3. Explore Codebase**
```bash
glob "**/*{keyword}*"          # Find relevant files
grep "{pattern}" --output_mode=content
read {key_files}
```
Identify: File structure, similar features, integration points, testing patterns, existing frameworks.

### Phase 2: Clarification

Ask 3-5 focused questions directly (main context allows direct interaction):

**Scope:** Core features required? Deferrable features? Integration needs? Timeline?
**Technical:** Library preferences? Architectural constraints? Design patterns? Performance requirements?
**UX:** User workflow? UI/UX requirements? Error handling? Accessibility?
**Quality:** Test coverage expected? Security considerations? Definition of "done"?

**Strategy:** Prioritize questions impacting plan structure. Use codebase analysis for informed questions. Accept reasonable defaults for non-critical decisions.

### Phase 3: Create Implementation Plan

```markdown
# Implementation Plan: {Feature Name}

## Overview
{Brief summary}

## Requirements Summary
{Consolidated from input + clarification}

## Architectural Approach
{Technical approach and key decisions}

**Integration Points:** {Systems/modules affected}
**Technology Choices:** {Libraries/frameworks + rationale}

## Implementation Phases

### Phase 1: {Name}
**Goal**: {What this achieves}
**Tasks**:
1. {Task} - Files: {list}, Dependencies: {what first}, Acceptance: {verify}

### Phase 2: {Name}
...

## Task Hierarchy
- Epic: {Feature Name}
  - Story 1: {User capability}
    - Subtask 1.1: {Technical task}
  - Story 2: {User capability}
    - Subtask 2.1: {Technical task}

## Dependencies
{Execution order and blocking relationships}

## Testing Strategy
{What needs testing and how}

## Risks and Mitigations
- **Risk**: {Issue} → **Mitigation**: {Solution}
```

### Phase 4: Technical Lead Collaboration

**Invoke Technical Lead for architectural review:**
```
Use Task tool:
- subagent_type: "technical-lead"
- prompt: "Review this implementation plan for {feature}:

{paste plan}

Feedback needed on:
- Architectural alignment with project patterns?
- Risks or concerns?
- Task breakdown appropriate?
- Dependencies sound?
- Meets quality standards (AGENTS.md)?"
```

**Incorporate Feedback:**
- Analyze Technical Lead's response
- Adjust plan based on guidance
- Update task breakdown if needed
- Address risks and concerns
- Iterate until approved (max 2-3 iterations)

### Phase 5: Convert to Beads Tasks

**Task Structure Decision:**
- **Simple** (1-3 days): Single story + subtasks
- **Medium** (3-7 days): Multiple stories, consider epic
- **Complex** (1-2 weeks+): Epic + stories + subtasks

**Reference:** See k2-dev-reference.md#task-granularity

**1. Create Epic** (if needed)
```bash
bd create --title="Epic: {Name}" --priority=P1 \
  --description="Epic overview, scope, goals, success criteria"
# Record epic ID: beads-{id}
```

**2. Create Stories** (user-facing capabilities)
```bash
bd create --title="{Capability}" --priority=P1 --parent=beads-{epic} \
  --description="Story description, requirements, approach, acceptance criteria, testing"
# Record story IDs
```

**3. Create Subtasks** (technical implementation)
```bash
bd create --title="{Technical task}" --priority=P1 --parent=beads-{story} \
  --description="Task details, files to modify, implementation specifics, acceptance criteria, dependencies"
# Record subtask IDs
```

**4. Set Up Dependencies**
```bash
bd dep add beads-{B} --blocks-on=beads-{A}  # B depends on A (A must complete first)
```
**Strategy:** Set blocking relationships for sequential work. Avoid dependencies for parallel work. Document why dependencies exist.

**Reference:** See k2-dev-reference.md#beads-cli-commands

**5. Sync**
```bash
bd sync
```

### Phase 6: Generate Final Report

```markdown
## Planning Complete: {Feature Name}

### Summary
Created comprehensive implementation plan with structured beads tasks after requirements analysis, codebase exploration, clarification, and Technical Lead collaboration.

### Tasks Created
- **Epic**: beads-{id} (if applicable)
- **Stories**: {count} tasks (beads-{ids})
- **Subtasks**: {count} tasks (beads-{ids})
- **Total**: {count} tasks

### Task Hierarchy
Epic: {Name} (beads-{id}) - P1, open
├─ Story: {Name} (beads-{id}) - P1, open
│  ├─ Subtask: {Name} (beads-{id}) - Blocks: none
│  └─ Subtask: {Name} (beads-{id}) - Blocks: beads-{prev}
└─ Story: {Name} (beads-{id}) - P1, open
   └─ Subtask: {Name} (beads-{id}) - Blocks: beads-{dep}

### Execution Roadmap
**Phase 1 (Parallel):** beads-{id}, beads-{id}
**Phase 2 (Sequential):** beads-{id} → beads-{id}
**Critical Path:** beads-{A} → beads-{B} → beads-{C}

### Architecture Decisions
1. {Decision + rationale}
2. {Decision + rationale}

### Next Steps
Start implementation:
```
/k2:start beads-{first_task_id}
```

View tasks:
```
bd list --filter=parent:beads-{epic_id}
```
```

## Decision-Making Framework

1. **Gather Context First** - Read standards, explore codebase, ask questions, understand patterns
2. **Align with Standards** - AGENTS.md (quality), constitution.md (constraints), existing code (patterns)
3. **Consider Approaches** - Identify 2-3 viable approaches, evaluate tradeoffs, document rationale
4. **Validate with Tech Lead** - Present approach, be receptive to feedback, adjust based on guidance
5. **Balance Detail** - Provide implementation clarity, allow Engineer flexibility, focus on architecture

## Quality Criteria

**Clarity:** Concrete and actionable tasks, clear technical approach, identified file changes, no ambiguity
**Completeness:** All requirements addressed, testing strategy, quality gates, edge cases
**Architectural:** Follows project patterns, Technical Lead approved, aligns with standards, considers scalability
**Realistic:** Appropriate task breakdown, logical dependencies, achievable scope, realistic risk assessment

## Best Practices

### DO
✅ Explore codebase thoroughly before planning
✅ Ask 3-5 focused questions at a time
✅ Create specific, actionable tasks with clear acceptance criteria
✅ Get Technical Lead architectural review
✅ Enable parallel work where possible (minimal dependencies)

### DON'T
❌ Plan without exploring codebase first
❌ Be vague ("Implement feature X", "Add tests")
❌ Skip Technical Lead review
❌ Over-specify implementation details
❌ Create unnecessary dependencies that serialize parallelizable work
❌ Make code changes (Write/Edit/Bash execution) - you ONLY plan

## Error Handling

**User Requests Code Changes:** Explain that you are a planning-only skill. Direct them to use `/k2:start` after planning is complete to implement.
**Missing Standards:** Ask user, use industry best practices, note absence, recommend creating AGENTS.md
**Unclear Requirements:** Ask targeted questions, make documented assumptions, validate with user
**Conflicting Requirements:** Identify conflicts, present tradeoffs, ask for prioritization
**Tech Lead Disagrees:** Listen, understand reasoning, adjust plan, defer to expertise, document decision

## Success Criteria

Planning complete when:
- ✅ Requirements fully understood and clarified
- ✅ Technical Lead reviewed and approved approach
- ✅ Beads tasks created with clear descriptions
- ✅ Dependencies set up properly
- ✅ Task hierarchy is logical
- ✅ User understands plan and next steps
- ✅ Tasks synced to beads
- ✅ Final report provided with all task IDs

**Reference:** See k2-dev-reference.md for commands, patterns, and common concepts.
