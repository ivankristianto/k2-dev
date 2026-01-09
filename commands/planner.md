---
name: k2:planner
description: Start planning workflow to analyze requirements and create actionable beads tasks
argument-hint: optional feature description
allowed-tools:
  - Read
  - Task
  - AskUserQuestion
---

# K2:Planner - Requirements Planning Workflow

Start the planning workflow to analyze user requirements, clarify details, and create structured beads tasks with dependencies.

## What This Command Does

This command invokes the Planning skill which executes in the main conversation context:

1. Planning skill analyzes requirements with full codebase access
2. Asks user clarifying questions directly in main conversation
3. Creates initial implementation plan
4. Technical Lead reviews and refines plan based on architecture
5. Planner skill and Tech Lead iterate until plan is solid
6. Planner skill converts plan to hierarchical beads tasks with dependencies

## Key Benefits Over Agent-Based Approach

- **Faster execution**: No agent spawning overhead
- **Easier debugging**: Everything happens in main conversation context
- **Direct context access**: Skill uses main conversation tools
- **Better UX**: Questions asked directly, not in isolated subagent context

## How to Use This Command

**Step 1: Capture User Requirements**

- If argument provided, use it as initial requirement
- If no argument, ask user: "What feature or change would you like to plan?"
- Get clear description of what user wants to build/design/add

**Step 2: Invoke Planning Skill**

- Use the Skill tool to invoke the "k2-dev:planning" skill
- Provide the user requirement
- The skill will execute in main conversation context

Example:

```
Skill tool with:
- skill: "k2-dev:planning"
- args: "User wants to: {user requirement}"

The skill will then:
1. Explore relevant code and architecture
2. Ask clarifying questions (in main conversation)
3. Create detailed implementation plan
4. Request Technical Lead review for architectural feedback (via Task tool)
5. Refine plan based on feedback
6. Convert to beads tasks with dependencies
```

**Step 3: Monitor Progress**

- Planning skill will explore codebase using main conversation tools
- Skill will ask user questions directly (not via AskUserQuestion in subagent)
- Skill will create initial plan
- Technical Lead will provide feedback (via Task tool invocation)
- Iteration continues until plan is finalized

**Step 4: Report Results**

- Planning skill will provide final plan and created beads task IDs
- Show the task hierarchy to user
- Explain dependencies and suggested order

## Planning Workflow Details

### Phase 1: Analysis

- Planning skill explores current codebase (Read, Grep, Glob, Bash)
- Identifies relevant files, patterns, architecture
- Understands existing implementation
- Uses main conversation tools for exploration

### Phase 2: Clarification

- Planning skill asks specific questions directly in main conversation
- Questions about:
  - Scope and boundaries
  - Technical approach preferences
  - Integration points
  - Constraints and requirements
- Iterates until requirements are clear

### Phase 3: Initial Planning

- Creates detailed implementation plan
- Breaks down into logical steps
- Identifies dependencies
- Considers architecture and patterns
- Documents assumptions

### Phase 4: Technical Review

- Planning skill uses Task tool to invoke Technical Lead agent
- Technical Lead reviews plan
- Provides architectural feedback
- Suggests improvements
- Validates feasibility
- Checks alignment with project standards

### Phase 5: Refinement

- Planning skill incorporates feedback
- Adjusts plan based on Tech Lead input
- Iterates until both agree
- Finalizes technical approach

### Phase 6: Task Creation

- Converts plan to beads tasks using `bd create`
- Creates hierarchical structure with epics if needed
- Sets up dependencies with `bd dep add`
- Adds detailed descriptions and comments
- Returns task IDs and hierarchy

### Phase 7: Sync Tasks and Generate Report

When all tickets are created, ALWAYS DO `bd sync`

And Generate structured report:

```markdown
## Feature: The name of the feature

### Summary
High level summary 2-3 sentences.

### List of Tickets in dependency graph
List the ticket in dependency graph

### Unblocked Tickets
- List of unblock tickets

### Blocked Tickets
- List of blocked tickets
```

## Important Notes

- **Main Context Execution**: Planning skill runs in main conversation, not isolated subagent
- **Direct Interaction**: User questions asked directly in main conversation
- **Tool Access**: Skill uses tools from main conversation context
- **Orchestration**: Skill can still invoke Technical Lead agent via Task tool when needed
- **Hierarchical**: Creates parent/child tasks for complex features
- **Dependencies**: Sets up proper task dependencies automatically

## Configuration Files Used

Planning skill and Technical Lead will reference:

- **AGENTS.md**: Project guidelines and standards
- **CLAUDE.md**: Claude-specific patterns and practices
- **(docs|specs)/constitution.md**: Project principles
- Any project documentation (README, docs/, specs/, etc.)

## Example Usage

With feature description:

```
User: /k2:planner Add user authentication with JWT
```

Interactive mode:

```
User: /k2:planner
Assistant: What feature or change would you like to plan?
User: I want to add dark mode to the entire application
```

## Success Indicators

The planning workflow is complete when:

- ✅ Requirements are fully understood and clarified
- ✅ Technical Lead has reviewed and approved approach
- ✅ Beads tasks created with clear descriptions
- ✅ Dependencies set up properly
- ✅ Task hierarchy makes logical sense
- ✅ User understands the plan and next steps

Present the final task structure to the user showing:

- Task IDs and titles
- Parent-child relationships
- Dependencies
- Recommended execution order
- How to start implementation (use `/k2:start beads-XXX`)

## Skill Aliases

This planning functionality can also be invoked via:
- `/k2:planner` (this command)
- `/planner` (direct skill invocation)

Both invoke the same Planning skill and execute in the main conversation context.
