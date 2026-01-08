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

This command initiates a collaborative planning workflow:

1. Planner agent analyzes requirements with full codebase access
2. Asks user clarifying questions
3. Creates initial implementation plan
4. Technical Lead reviews and refines plan based on architecture
5. Planner and Tech Lead iterate until plan is solid
6. Planner converts plan to hierarchical beads tasks with dependencies

## How to Use This Command

**Step 1: Capture User Requirements**
- If argument provided, use it as initial requirement
- If no argument, ask user: "What feature or change would you like to plan?"
- Get clear description of what user wants to build/design/add

**Step 2: Launch Planner Agent**
- Use the Task tool to launch the "planner" agent
- Provide the user requirement
- Give full context about the planning workflow

Example:
```
Task tool with:
- subagent_type: "planner"
- prompt: "User wants to: {user requirement}.

          Analyze this requirement with full codebase access:
          1. Explore relevant code and architecture
          2. Ask clarifying questions to understand scope and constraints
          3. Create detailed implementation plan
          4. Request Technical Lead review for architectural feedback
          5. Refine plan based on feedback
          6. Convert to beads tasks with dependencies

          Follow the k2-dev planning workflow."
```

**Step 3: Monitor Progress**
- Planner will explore codebase
- Planner will ask user questions (via AskUserQuestion tool)
- Planner will create initial plan
- Technical Lead will provide feedback
- Iteration continues until plan is finalized

**Step 4: Report Results**
- Planner will provide final plan and created beads task IDs
- Show the task hierarchy to user
- Explain dependencies and suggested order

## Planning Workflow Details

### Phase 1: Analysis
- Planner explores current codebase
- Identifies relevant files, patterns, architecture
- Understands existing implementation
- Has access to: Read, Grep, Glob, Bash (for exploration)

### Phase 2: Clarification
- Planner asks specific questions about:
  - Scope and boundaries
  - Technical approach preferences
  - Integration points
  - Constraints and requirements
- Uses AskUserQuestion tool for interactive clarification
- Iterates until requirements are clear

### Phase 3: Initial Planning
- Creates detailed implementation plan
- Breaks down into logical steps
- Identifies dependencies
- Considers architecture and patterns
- Documents assumptions

### Phase 4: Technical Review
- Technical Lead reviews plan
- Provides architectural feedback
- Suggests improvements
- Validates feasibility
- Checks alignment with project standards (AGENTS.md, CLAUDE.md)

### Phase 5: Refinement
- Planner incorporates feedback
- Adjusts plan based on Tech Lead input
- Iterates until both agree
- Finalizes technical approach

### Phase 6: Task Creation
- Converts plan to beads tasks using `bd create`
- Creates hierarchical structure with epics if needed
- Sets up dependencies with `bd dep add`
- Adds detailed descriptions and comments
- Returns task IDs and hierarchy

## Important Notes

- **Full Codebase Access**: Planner can read any file to understand architecture
- **Interactive**: User will be asked questions - be prepared to provide details
- **Collaborative**: Planner works WITH Technical Lead for best results
- **Hierarchical**: Creates parent/child tasks for complex features
- **Dependencies**: Sets up proper task dependencies automatically

## Configuration Files Used

Planner and Technical Lead will reference:
- **AGENTS.md**: Project guidelines and standards
- **CLAUDE.md**: Claude-specific patterns and practices
- **constitution.md**: Project principles
- Any project documentation (README, docs/, etc.)

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
