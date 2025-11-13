# Router-Frontal Cortex Integration

**Status**: Specification v0.1  
**Last Updated**: 2025-10-31  
**Purpose**: Document how Router checks for existing plans and requests plan creation from Frontal Cortex

## Overview

The Router makes critical decisions about how to process each turn. A key part of this decision-making is determining whether an execution plan exists and, if not, requesting that Frontal Cortex create one.

This document specifies the interaction between Router and Frontal Cortex for plan management.

## Router Decision Points

### Decision 1: Is There an Existing Plan?

When Router receives input, it first checks if there's an active plan in Frontal Cortex:

```
Router receives input
  ↓
Router queries FC: "Is there an active plan for this goal?"
  ↓
FC responds: Plan exists OR Plan doesn't exist
  ↓
Router proceeds to Decision 2
```

### Decision 2: Should We Use the Existing Plan?

If a plan exists, Router decides whether to use it:

```
Plan exists
  ↓
Router evaluates:
  - Is plan still relevant to current input?
  - Has context changed significantly?
  - Is plan still valid?
  ↓
Router decides: Use plan OR Create new plan
```

### Decision 3: Should We Create a New Plan?

If no plan exists or existing plan is invalid, Router decides whether to create one:

```
No plan OR plan invalid
  ↓
Router evaluates:
  - Is this a simple query (no plan needed)?
  - Is this a complex task (plan needed)?
  - Do we have time to create a plan?
  ↓
Router decides: Create plan OR Proceed without plan
```

## API Contracts

### Query: Check for Existing Plan

**Request**:
```
Router → FC: check_plan_for_goal(
  agent_id: string,
  goal_id: string,
  context: {
    user_input: string,
    current_mode: string,
    urgency: "HIGH" | "MEDIUM" | "LOW"
  }
)
```

**Response**:
```
FC → Router: {
  plan_exists: boolean,
  plan: Plan | null,
  plan_status: "active" | "paused" | "completed" | "failed",
  plan_relevance: number (0-1),
  plan_age_seconds: number,
  recommendation: "use_existing" | "create_new" | "modify_existing"
}
```

**Plan Data Structure**:
```
Plan {
  id: string (ULID),
  goal_id: string,
  title: string,
  description: string,
  steps: Step[],
  status: "active" | "paused" | "completed" | "failed",
  priority: number (1-10),
  created_at: ISO-8601,
  updated_at: ISO-8601,
  estimated_duration: duration,
  progress: {
    completed_steps: number,
    total_steps: number,
    percent_complete: number
  },
  constraints: Constraint[],
  dependencies: string[] (other plan ids),
  metadata: object
}
```

### Request: Create New Plan

**Request**:
```
Router → FC: create_plan(
  agent_id: string,
  goal_id: string,
  context: {
    user_input: string,
    current_mode: string,
    urgency: "HIGH" | "MEDIUM" | "LOW",
    consciousness_mandates: Mandate[],
    active_goals: Goal[],
    pending_actions: PendingAction[]
  }
)
```

**Response**:
```
FC → Router: {
  plan_created: boolean,
  plan: Plan,
  plan_id: string,
  estimated_duration: duration,
  estimated_steps: number,
  status: "ready" | "pending_approval" | "error",
  error: string | null
}
```

### Request: Update Plan Progress

**Request**:
```
Router → FC: update_plan_progress(
  agent_id: string,
  plan_id: string,
  updates: {
    completed_steps: number,
    current_step: number,
    status: "active" | "paused" | "completed" | "failed",
    notes: string
  }
)
```

**Response**:
```
FC → Router: {
  updated: boolean,
  plan: Plan,
  next_step: Step | null,
  status: "active" | "paused" | "completed" | "failed"
}
```

## Router Decision Flow

### Complete Decision Sequence

```
1. Router receives input
   ↓
2. Extract goal from input
   - If explicit goal: use it
   - If implicit goal: infer from context
   - If no goal: create ad-hoc goal
   ↓
3. Query FC: check_plan_for_goal(goal_id)
   ↓
4. Evaluate response:
   - plan_exists = false?
     → Go to Step 5 (Create Plan)
   - plan_exists = true?
     → Go to Step 6 (Evaluate Existing Plan)
   ↓
5. CREATE PLAN PATH:
   - Assess urgency
   - If HIGH urgency: skip plan creation, proceed without plan
   - If MEDIUM/LOW urgency: request plan creation
   - FC creates plan
   - Router receives plan
   - Go to Step 7 (Activate Daemons)
   ↓
6. EVALUATE EXISTING PLAN PATH:
   - Check plan_relevance score
   - If relevance > 0.8: use existing plan
   - If relevance < 0.5: create new plan (go to Step 5)
   - If 0.5 ≤ relevance ≤ 0.8: modify existing plan
   - Go to Step 7 (Activate Daemons)
   ↓
7. ACTIVATE DAEMONS:
   - Use plan to determine which daemons to activate
   - Set urgency level
   - Set expected flow
   - Activate daemons
   ↓
8. COORDINATE EXECUTION:
   - Pass plan to Executor
   - Executor uses plan to orchestrate tool execution
   - Executor updates plan progress
```

## Plan Relevance Scoring

Router uses plan relevance score to decide whether to reuse an existing plan:

```
relevance_score = (
  0.3 * goal_match +
  0.2 * context_match +
  0.2 * recency_score +
  0.15 * constraint_satisfaction +
  0.15 * dependency_satisfaction
)

where:
  goal_match: 0-1 (does plan match current goal?)
  context_match: 0-1 (does plan match current context?)
  recency_score: 0-1 (how recent is the plan?)
  constraint_satisfaction: 0-1 (are constraints still satisfied?)
  dependency_satisfaction: 0-1 (are dependencies still valid?)
```

**Thresholds**:
- relevance_score > 0.8: Use existing plan
- 0.5 ≤ relevance_score ≤ 0.8: Modify existing plan
- relevance_score < 0.5: Create new plan

## Plan Modification

If existing plan is partially relevant, Router can request modification:

```
Router → FC: modify_plan(
  agent_id: string,
  plan_id: string,
  modifications: {
    add_steps: Step[],
    remove_steps: string[] (step ids),
    reorder_steps: string[] (step ids in new order),
    update_constraints: Constraint[],
    update_priority: number
  }
)
```

## Integration with Daemon Activation

Router uses the plan to determine which daemons to activate:

```
Plan received
  ↓
Analyze plan steps
  ↓
Determine required daemons:
  - If plan has ambiguities: activate Clarifier
  - If plan has tool execution: activate Executor
  - If plan has evaluation: activate Evaluator
  - If plan has learning: activate Reflector
  ↓
Activate determined daemons
```

## Error Handling

### Plan Creation Fails

```
Router requests plan creation
  ↓
FC returns error
  ↓
Router decides:
  - If HIGH urgency: proceed without plan
  - If MEDIUM urgency: proceed without plan, log warning
  - If LOW urgency: escalate error, ask for clarification
```

### Plan Becomes Invalid

```
During execution:
  - Executor detects plan step failure
  - Executor notifies Router
  - Router queries FC: is plan still valid?
  - FC responds: plan invalid
  - Router requests new plan
  - Executor continues with new plan
```

### Circular Dependencies

```
Plan has circular dependencies
  ↓
FC detects during creation
  ↓
FC returns error with details
  ↓
Router requests modified plan without circular deps
```

## Testing Requirements

### Test Suite 1: Plan Checking

**TC-1.1: Check existing plan**
- Setup: Active plan in FC
- Router queries: check_plan_for_goal(goal_id)
- Expected: Plan returned with relevance score
- Verify: Plan data is complete and valid

**TC-1.2: Check non-existent plan**
- Setup: No plan in FC
- Router queries: check_plan_for_goal(goal_id)
- Expected: plan_exists = false
- Verify: No error, clean response

### Test Suite 2: Plan Creation

**TC-2.1: Create simple plan**
- Router requests: create_plan(goal_id, context)
- Expected: Plan created with 3-5 steps
- Verify: Plan is valid and executable

**TC-2.2: Create complex plan**
- Router requests: create_plan(goal_id, context) with complex goal
- Expected: Plan created with 10+ steps
- Verify: Plan has proper dependencies

**TC-2.3: Create plan with constraints**
- Router requests: create_plan(goal_id, context) with constraints
- Expected: Plan respects all constraints
- Verify: Constraint satisfaction score > 0.9

### Test Suite 3: Plan Relevance

**TC-3.1: High relevance plan**
- Setup: Plan matches current goal and context
- Router evaluates: relevance_score
- Expected: relevance_score > 0.8
- Verify: Router uses existing plan

**TC-3.2: Low relevance plan**
- Setup: Plan doesn't match current goal
- Router evaluates: relevance_score
- Expected: relevance_score < 0.5
- Verify: Router creates new plan

**TC-3.3: Medium relevance plan**
- Setup: Plan partially matches
- Router evaluates: relevance_score
- Expected: 0.5 ≤ relevance_score ≤ 0.8
- Verify: Router modifies plan

### Test Suite 4: Error Handling

**TC-4.1: Plan creation fails**
- Setup: FC unable to create plan
- Router requests: create_plan(...)
- Expected: Error returned
- Verify: Router handles gracefully

**TC-4.2: Plan becomes invalid**
- Setup: Plan exists but becomes invalid
- Executor detects: step failure
- Router requests: new plan
- Expected: New plan created
- Verify: Execution continues

## Performance Targets

- **Plan check**: < 50ms
- **Plan creation**: < 500ms
- **Plan modification**: < 200ms
- **Relevance scoring**: < 100ms

## Next Steps

See related documentation:
- `documentation/frontal-cortex/README.md` - Frontal Cortex specification
- `documentation/turn/daemons/router.md` - Router daemon
- `documentation/turn/daemons/executor.md` - Executor daemon

