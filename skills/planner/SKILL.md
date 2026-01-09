---
name: Planning
description: This skill should be used when the user needs to plan features, break down requirements, create implementation tasks, or design software architecture. Use this skill for comprehensive requirements analysis, technical planning, and creating structured task hierarchies with dependencies. The skill executes in the main conversation context and can invoke the technical-lead agent for architectural review when needed.
version: 0.1.0
---

# Planning Skill

## Overview

The Planning skill transforms user requirements into actionable, well-structured implementation tasks with proper hierarchies and dependencies. It executes in the main conversation context for faster execution, easier debugging, and direct user interaction.

**Key Objectives:**

- Analyze requirements thoroughly through codebase exploration
- Clarify scope, constraints, and expectations through direct questioning
- Create detailed implementation plans with task hierarchies
- Set up proper dependencies for execution ordering
- Collaborate with Technical Lead for architectural validation

## When to Use This Skill

Invoke this skill when:

- User executes `/planner` or `/k2:planner` command
- Planning new features or major changes
- Breaking down user stories into actionable tasks
- Designing implementation approach for complex features
- Creating task hierarchies with dependencies
- Analyzing requirements before implementation

## Planning Workflow

### Phase 1: Context Gathering

**Step 1: Read Project Standards**

First, always read the project's quality standards:

```bash
# Check if standards files exist in project root
ls -la AGENTS.md CLAUDE.md docs/constitution.md

# Read available standards
cat AGENTS.md
cat CLAUDE.md
cat docs/constitution.md
```

What to look for:

- Quality gates and validation patterns
- Coding standards and conventions
- Testing requirements and coverage thresholds
- Architectural principles and constraints
- File organization patterns

**Step 2: Understand Requirements**

- Get feature description from command argument or user input
- Identify core problem/goal being addressed
- Note explicit constraints or requirements
- Recognize what's clear vs. what needs clarification

**Step 3: Explore Codebase**

Use exploration tools to understand context:

```bash
# Find relevant files
glob "**/*auth*"  # Find authentication-related files
glob "**/routes/*"  # Find route definitions
glob "**/*controller*"  # Find controllers

# Search for patterns
grep "app\.(get|post|put|delete)" --output_mode=content
grep "TODO\|FIXME" --output_mode=content

# Read key architecture files
read src/app.js
read src/index.js
read package.json
```

What to identify:

- File structure and module organization
- Similar features to maintain consistency
- Integration points and affected areas
- Testing patterns and frameworks
- Build and deployment processes

### Phase 2: Clarification

Ask targeted questions to gather critical information. Since this skill runs in main conversation context, ask questions directly:

**Scope and Boundaries:**

- What are the core features that MUST be included?
- What features can be deferred to future iterations?
- Are there existing systems this needs to integrate with?
- What's the expected timeline or urgency?

**Technical Approach:**

- Do you have preferences for technical approach (libraries, patterns)?
- Are there architectural constraints I should be aware of?
- Should this follow specific design patterns already in use?
- Are there performance or scalability requirements?

**User Experience:**

- What's the expected user workflow?
- Are there UI/UX requirements or designs to follow?
- What error handling and edge cases should be considered?
- Are there accessibility requirements?

**Testing and Quality:**

- What level of test coverage is expected?
- Are there specific test scenarios that must be covered?
- Are there security considerations or compliance requirements?
- What constitutes "done" for this feature?

**Questioning Strategy:**

- Ask 3-5 focused questions at a time (don't overwhelm)
- Prioritize questions that most impact the plan structure
- Use codebase analysis to ask informed, specific questions
- Accept reasonable defaults for non-critical decisions

### Phase 3: Create Implementation Plan

After gathering requirements, create a structured plan:

```markdown
# Implementation Plan: {Feature Name}

## Overview

{Brief summary of what's being built and why}

## Requirements Summary

{Consolidated requirements from user input and clarification}

## Architectural Approach

{High-level technical approach and key decisions}

### Integration Points

- Systems/modules that will be affected
- New components to be created

### Technology Choices

- Libraries, frameworks, or tools to be used
- Rationale for choices

## Implementation Phases

### Phase 1: {Phase Name}

**Goal**: {What this phase achieves}

**Tasks**:

1. {Specific task description}
   - Files: {list}
   - Dependencies: {what must exist first}
   - Acceptance criteria: {how to verify completion}

### Phase 2: {Phase Name}

...

## Task Hierarchy

### Epic: {Feature Name}

- Story 1: {User-facing capability}
  - Subtask 1.1: {Technical implementation}
  - Subtask 1.2: {Technical implementation}
- Story 2: {User-facing capability}

## Dependencies

{Task dependency relationships and execution order}

## Testing Strategy

{What needs to be tested and how}

## Risks and Mitigations

- **Risk**: {Identified risk}
  - **Mitigation**: {How to address it}
```

### Phase 4: Technical Lead Collaboration

**Step 1: Request Technical Lead Review**

Use the Task tool to invoke the technical-lead agent:

```
Task tool with:
- subagent_type: "technical-lead"
- prompt: "I've created an initial implementation plan for {feature name}.

Please review this plan and provide architectural feedback:

{paste full plan}

Specific areas I'd like your input on:
- Does the architectural approach align with project patterns?
- Are there any risks or concerns with this approach?
- Should any tasks be broken down differently?
- Are the dependencies and execution order sound?
- Does this meet the quality standards from AGENTS.md/CLAUDE.md?"
```

**Step 2: Incorporate Feedback**

- Receive and analyze Technical Lead's feedback
- Adjust plan based on architectural guidance
- Update task breakdown if needed
- Address identified risks and concerns
- Iterate until plan is approved (max 2-3 iterations)

### Phase 5: Convert to Beads Tasks

After plan is approved:

**Step 1: Determine Task Structure**

- **Simple feature** (1-3 days): Single story with subtasks
- **Medium feature** (3-7 days): Multiple related stories, consider epic
- **Complex feature** (1-2 weeks+): Epic with multiple stories and subtasks

**Step 2: Create Epic** (if needed)

```bash
bd create \
  --title="Epic: {Feature Name}" \
  --priority=P1 \
  --description="$(cat <<'EOF'
{Epic description from plan overview}

## Scope
{What's included in this epic}

## Goals
{High-level goals and objectives}

## Success Criteria
{How to measure epic completion}

## Related Documentation
- Planning document: {reference}
- Architecture notes: {key decisions}

## Stories
{Will be created as child tasks}
EOF
)"
```

Record the epic ID (e.g., beads-100).

**Step 3: Create Stories** (user-facing capabilities)

```bash
bd create \
  --title="{User story or capability name}" \
  --priority=P1 \
  --parent=beads-{epic_id} \
  --description="$(cat <<'EOF'
## Story Description
{What user-facing capability this provides}

## Requirements
{Specific requirements from plan}

## Implementation Approach
{High-level approach from plan}

## Acceptance Criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}

## Testing Requirements
{What needs to be tested}

## Subtasks
{Will be created as child tasks}
EOF
)"
```

Record story IDs.

**Step 4: Create Subtasks** (technical implementation)

```bash
bd create \
  --title="{Specific technical task}" \
  --priority=P1 \
  --parent=beads-{story_id} \
  --description="$(cat <<'EOF'
## Task Description
{Detailed description of what needs to be done}

## Files to Create/Modify
- {file1.js} - {what changes}
- {file2.js} - {what changes}

## Implementation Details
{Specific technical details from plan}
{Code patterns to follow}
{Integration points}

## Acceptance Criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}

## Testing
- {Test scenarios to cover}

## Dependencies
{What must be done before this}
EOF
)"
```

Record subtask IDs.

**Step 5: Set Up Dependencies**

```bash
# Task B blocks on Task A (A must complete before B)
bd dep add beads-{B} --blocks-on=beads-{A}

# Multiple dependencies
bd dep add beads-{C} --blocks-on=beads-{A}
bd dep add beads-{C} --blocks-on=beads-{B}
```

**Dependency Strategy:**

- Set up blocking relationships for sequential work
- Avoid creating dependencies where parallel work is possible
- Document why dependencies exist in task descriptions

**Step 6: Sync Tasks**

```bash
bd sync
```

### Phase 6: Generate Final Report

Create a comprehensive report for the user:

```markdown
## Planning Complete: {Feature Name}

### Summary

I've analyzed your requirements, explored the codebase, clarified details with you, collaborated with the Technical Lead on architecture, and created a comprehensive implementation plan with structured beads tasks.

### Tasks Created

- **1 Epic**: beads-{epic_id}
- **{N} Stories**: beads-{id}, beads-{id}, ...
- **{M} Subtasks**: beads-{id}, beads-{id}, ...
- **Total**: {N+M+1} tasks

### Task Hierarchy

#### Epic: {Epic Name} (beads-{epic_id})

Priority: P1 | Status: open

##### Story: {Story 1 Name} (beads-{story1_id})

Priority: P1 | Status: open | Parent: beads-{epic_id}

- Subtask: {Subtask 1.1} (beads-{id}) | Blocks: none
- Subtask: {Subtask 1.2} (beads-{id}) | Blocks: beads-{previous}

##### Story: {Story 2 Name} (beads-{story2_id})

Priority: P1 | Status: open | Parent: beads-{epic_id}

- Subtask: {Subtask 2.1} (beads-{id}) | Blocks: beads-{dependency}

### Execution Roadmap

#### Phase 1: Foundation (Parallel work possible)

1. Start with: beads-{id} - {task name}
2. Can work in parallel: beads-{id} - {task name}

#### Phase 2: Core Implementation (Sequential)

3. After Phase 1: beads-{id} - {task name}
4. Then: beads-{id} - {task name}

#### Critical Path

The longest dependency chain is:
beads-{A} → beads-{B} → beads-{C} → beads-{D}

### Architecture Decisions

Key architectural decisions made during planning:

1. {Decision 1 and rationale}
2. {Decision 2 and rationale}

### Next Steps

To start implementation, use:
```

/k2:start beads-{first_task_id}

```

This will:
1. Validate the task
2. Create a git worktree
3. Engage the Engineer agent
4. Handle review and merge process

All tasks have been synced to beads. View them with:
```

bd list --filter=parent:beads-{epic_id}

```

```

## Task Granularity Guidelines

### Epic

- **Duration**: 1-2 weeks of work
- **Scope**: High-level user value or major feature
- **Example**: "Epic: User Authentication System"

### Story

- **Duration**: 2-5 days of work
- **Scope**: Deliverable user-facing capability
- **Example**: "User Login with Email and Password"

### Subtask

- **Duration**: 0.5-2 days of work
- **Scope**: Specific technical implementation task
- **Example**: "Implement JWT token generation"

## Decision-Making Framework

When making planning decisions:

1. **Gather Context First**

   - Read all available standards and documentation
   - Explore codebase thoroughly
   - Ask clarifying questions
   - Understand project patterns

2. **Align with Project Standards**

   - AGENTS.md defines quality expectations
   - CLAUDE.md defines coding preferences
   - constitution.md defines constraints
   - Existing code demonstrates patterns

3. **Consider Multiple Approaches**

   - Identify 2-3 viable technical approaches
   - Evaluate tradeoffs
   - Choose approach that best balances concerns
   - Document rationale for chosen approach

4. **Validate with Technical Lead**

   - Present approach and reasoning
   - Be receptive to feedback
   - Adjust based on architectural guidance
   - Defer to Technical Lead's expertise

5. **Balance Detail and Flexibility**
   - Provide enough detail for clear implementation
   - Allow Engineer some implementation flexibility
   - Don't over-specify trivial details
   - Focus on architecture and approach

## Quality Standards

### Plan Quality Criteria

**Clarity and Specificity:**

- Tasks are concrete and actionable
- Technical approach is clearly explained
- File changes are identified
- No ambiguous language

**Completeness:**

- All user requirements addressed
- Testing strategy included
- Quality gates considered
- Edge cases identified

**Architectural Soundness:**

- Follows project patterns and conventions
- Technical Lead has reviewed and approved
- Aligns with CLAUDE.md and constitution.md
- Considers scalability and maintainability

**Realistic Estimation:**

- Task breakdown is appropriate for complexity
- Dependencies are logical and necessary
- Scope is achievable
- Risk assessment is realistic

## Common Planning Scenarios

### New Feature from Scratch

1. Explore codebase for similar features
2. Identify integration points
3. Plan data models and APIs
4. Design UI components if applicable
5. Plan testing strategy
6. Create epic → stories → subtasks hierarchy

### Refactoring Existing Code

1. Read and analyze existing implementation
2. Identify problems and improvement areas
3. Plan refactoring approach (incremental vs. complete)
4. Consider backward compatibility
5. Plan migration strategy if needed
6. Create tasks with clear acceptance criteria

### API Development

1. Identify endpoints needed
2. Design request/response schemas
3. Plan authentication/authorization
4. Consider error handling
5. Plan API versioning if applicable
6. Create tasks for endpoint implementation

### Database Changes

1. Understand current schema
2. Plan schema changes (migrations)
3. Consider data migration needs
4. Plan backward compatibility
5. Consider rollback strategy
6. Create tasks for migration scripts

## Tools and Commands

### File Exploration

```bash
# Find relevant files
glob "**/*{keyword}*"

# Search for patterns
grep "{pattern}" --output_mode=content

# Read files
read {file_path}
```

### Beads Commands

```bash
# Create epic
bd create --title="Epic: {name}" --priority=P1 --description="..."

# Create story with parent
bd create --title="{name}" --priority=P1 --parent=beads-{epic_id}

# Create subtask
bd create --title="{name}" --priority=P1 --parent=beads-{story_id}

# Add dependencies
bd dep add beads-{child} --blocks-on=beads-{parent}

# View task
bd show beads-{id}

# List tasks
bd list --filter=parent:beads-{epic_id}

# Sync to remote
bd sync
```

### Technical Lead Collaboration

```
Use Task tool to invoke Technical Lead:
- subagent_type: "technical-lead"
- prompt: "{request for architectural review and feedback}"
```

## Best Practices

### DO

✅ **Explore thoroughly**

- Read existing code patterns
- Understand architecture before planning
- Find similar features for consistency

✅ **Ask focused questions**

- 3-5 questions at a time
- Prioritize critical clarifications
- Use codebase analysis to inform questions

✅ **Create specific tasks**

- Concrete and actionable
- Clear acceptance criteria
- Files identified
- Approach specified

✅ **Collaborate with Technical Lead**

- Get architectural review
- Be receptive to feedback
- Iterate on plan based on guidance

✅ **Think about dependencies**

- Enable parallel work where possible
- Only create necessary blocking relationships
- Document dependency rationale

### DON'T

❌ **Don't plan without context**

- Assuming architecture without exploration
- Ignoring existing patterns
- Planning in isolation

❌ **Don't be vague**

- "Implement feature X"
- "Fix authentication"
- "Add tests"

❌ **Don't skip Technical Lead review**

- Plan without architectural validation
- Ignore technical constraints
- Make unilateral architectural decisions

❌ **Don't over-specify**

- Micromanage implementation details
- Specify every line of code
- Remove Engineer's judgment

❌ **Don't create unnecessary dependencies**

- Serializing parallelizable work
- Creating bottlenecks
- Over-constraining execution order

## Error Handling

### Missing Standards Files

- Ask user about project standards
- Use industry best practices as defaults
- Note absence in planning documentation
- Recommend creating AGENTS.md/CLAUDE.md

### Unclear Requirements

- Ask targeted clarifying questions
- Make reasonable assumptions and document them
- Present assumptions for validation
- Iterate until requirements are clear

### Conflicting Requirements

- Identify conflicts explicitly
- Present tradeoffs to user or Technical Lead
- Ask for prioritization or decision
- Document final decision and rationale

### Technical Lead Disagrees

- Listen carefully to feedback
- Understand architectural reasoning
- Adjust plan to incorporate guidance
- Defer to Technical Lead's expertise
- Document architectural decision

## Success Criteria

Planning is complete when:

- ✅ Requirements are fully understood and clarified
- ✅ Technical Lead has reviewed and approved approach
- ✅ Beads tasks created with clear descriptions
- ✅ Dependencies set up properly
- ✅ Task hierarchy makes logical sense
- ✅ User understands plan and next steps
- ✅ Tasks synced to beads
- ✅ Final report provided with all task IDs

---

Execute planning workflows in main conversation context for faster execution, easier debugging, and direct user interaction.
