# Plan Generator Hobgoblin

## Purpose

The Plan Generator creates execution plans when a turn requires multiple steps or complex work. It decides what steps to take, in what order, and with what resources.

## Responsibilities

1. **Analyze Requirements**: Understand what needs to be done
2. **Generate Plan**: Create step-by-step execution plan
3. **Optimize Plan**: Order steps efficiently
4. **Validate Plan**: Ensure plan is feasible and compliant
5. **Provide to Executor**: Hand off plan for execution

## When Plan Generator is Activated

Router activates Plan Generator when:
- Multiple steps are needed
- Complex work is required
- Tool execution is needed
- Coordination is needed

**Examples**:
- "Analyze this data and create a report" (multiple steps)
- "Book a flight and hotel" (multiple tools)
- "Plan my vacation" (complex coordination)

## Decision Inputs

Plan Generator makes decisions using:

### Consciousness
- **Mandates**: What priorities should guide the plan?
- **Values**: What matters in planning?
- **Governance**: What constraints apply?
- **Capabilities**: What tools are available?

### Working Memory
- **Context**: What's the situation?
- **Available Resources**: What tools/information are available?
- **Constraints**: What limitations exist?

### Episodic Memory
- **Similar Plans**: What plans worked for similar situations?
- **Learned Patterns**: What planning patterns are effective?
- **Outcomes**: What plan structures succeeded?
- **Skills**: What planning skills has Si developed?

### Input
- **Requirements**: What needs to be done?
- **Constraints**: What limitations exist?
- **Urgency**: How urgent is this?

## Plan Structure

A plan consists of:

### Steps
- **What**: What action to take
- **Why**: Why this step is needed
- **How**: How to execute it
- **Dependencies**: What must happen first
- **Success Criteria**: How to know if step succeeded

### Example Plan
```
Goal: Book a flight and hotel for Paris trip

Step 1: Search flights
  - What: Find flights to Paris for dates X-Y
  - Why: Need to know available options
  - How: Use flight search tool
  - Dependencies: None
  - Success: Found at least 3 options

Step 2: Search hotels
  - What: Find hotels near airport
  - Why: Need accommodation
  - How: Use hotel search tool
  - Dependencies: None (can run parallel with Step 1)
  - Success: Found at least 3 options

Step 3: Compare options
  - What: Compare flights and hotels by price/quality
  - Why: Need to help user decide
  - How: Analyze search results
  - Dependencies: Steps 1 and 2 complete
  - Success: Created comparison matrix

Step 4: Present options
  - What: Show user the best options
  - Why: User needs to make decision
  - How: Generate response with options
  - Dependencies: Step 3 complete
  - Success: User can make informed decision
```

## Plan Generation Process

```
Plan Generator receives requirements
    ↓
Analyze requirements:
  - What needs to be done?
  - What constraints exist?
  - What resources are available?
    ↓
Generate initial plan:
  - Identify steps
  - Determine dependencies
  - Estimate effort
    ↓
Optimize plan:
  - Reorder for efficiency
  - Identify parallelizable steps
  - Minimize resource usage
    ↓
Validate plan:
  - Check against constraints
  - Verify feasibility
  - Confirm resource availability
    ↓
Provide to Executor
```

## Plan Optimization

Plan Generator optimizes for:

### Speed
- Minimize total execution time
- Parallelize independent steps
- Prioritize high-impact steps

### Resource Efficiency
- Minimize tool calls
- Reuse information
- Avoid redundant work

### Reliability
- Minimize failure points
- Include error handling
- Plan for contingencies

### User Experience
- Provide early feedback
- Show progress
- Deliver value incrementally

## Learning Through Plan Generator

Plan Generator improves through the learning loop:

**Feedback Sources**:
- Did the plan work as expected?
- Were steps in the right order?
- Were there unnecessary steps?
- Could steps have been parallelized?
- Did the plan achieve the goal efficiently?

**Learning Updates**:
- Episodic Memory: Store successful plan patterns
- Skills: Improve planning for similar situations
- Consciousness: Refine planning priorities if needed

**Example**:
- Turn 1: Plan Generator creates plan for travel booking
- Executor executes plan successfully
- Evaluator: Plan was efficient and effective → GOOD
- Reflector: Learn "This plan structure works well for travel"

## Key Principle

**Plan Generator enables efficient execution.** By creating a clear plan upfront, Plan Generator ensures that Executor can execute efficiently without wasting time on decisions or wrong approaches.

