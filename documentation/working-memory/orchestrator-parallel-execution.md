# Orchestrator Parallel Execution Coordination

**Status**: Specification v0.1  
**Last Updated**: 2025-10-31  
**Purpose**: Document how Orchestrator coordinates parallel tool execution and background turns

## Overview

The Orchestrator manages parallel execution of tools and background turns. It analyzes tool dependencies, allocates resources, schedules execution, and coordinates results.

This document specifies the mechanisms for parallel execution coordination.

## Parallel Execution Model

### Execution Modes

The Orchestrator supports three execution modes:

1. **Sequential**: Tools execute one after another
   - Used for: dependent tools, resource-constrained environments
   - Latency: sum of all tool latencies

2. **Parallel**: Independent tools execute simultaneously
   - Used for: independent tools, resource-available environments
   - Latency: max of all tool latencies

3. **Hybrid**: Mix of sequential and parallel execution
   - Used for: complex plans with dependencies
   - Latency: optimized based on dependency graph

### Execution Scheduling

```
Executor provides plan with tools
  ↓
Orchestrator analyzes dependencies
  ↓
Orchestrator creates execution schedule
  ↓
Orchestrator allocates resources
  ↓
Orchestrator executes according to schedule
  ↓
Orchestrator collects results
  ↓
Orchestrator returns results to Executor
```

## Dependency Analysis

### Dependency Types

Tools can have dependencies on:

1. **Data Dependencies**: Tool B needs output from Tool A
2. **Resource Dependencies**: Tools compete for limited resources
3. **Ordering Dependencies**: Tool B must run after Tool A (for side effects)
4. **Conditional Dependencies**: Tool B runs only if Tool A succeeds

### Dependency Graph Construction

```
Input: Plan with tools
  ↓
1. Extract tool list from plan
   - tool_id, tool_name, parameters, expected_output
   ↓
2. Analyze each tool's inputs
   - Identify which inputs come from other tools
   - Identify which inputs come from context
   ↓
3. Build dependency graph
   - Nodes: tools
   - Edges: dependencies
   - Edge labels: dependency type
   ↓
4. Detect cycles
   - If cycle detected: error, cannot execute
   - If no cycle: proceed to scheduling
   ↓
Output: Dependency graph
```

### Dependency Detection Algorithm

```
for each tool T in plan:
  for each input I of T:
    if I is output of another tool T':
      add edge: T' → T (data dependency)
    if I requires resource R:
      check if other tools also need R:
        if yes: add edge: T' → T (resource dependency)
    if tool manifest specifies ordering:
      add edge: T' → T (ordering dependency)
```

## Execution Scheduling

### Schedule Generation

```
Input: Dependency graph
  ↓
1. Topological sort
   - Order tools respecting dependencies
   - Identify independent tools
   ↓
2. Group independent tools
   - Group 1: tools with no dependencies
   - Group 2: tools depending only on Group 1
   - Group N: tools depending on Group N-1
   ↓
3. Create execution schedule
   - Phase 1: Execute Group 1 in parallel
   - Phase 2: Execute Group 2 in parallel
   - Phase N: Execute Group N in parallel
   ↓
Output: Execution schedule with phases
```

### Schedule Example

```
Plan tools:
  - search_flights (no dependencies)
  - search_hotels (no dependencies)
  - search_activities (no dependencies)
  - compare_prices (depends on search_flights, search_hotels)
  - create_itinerary (depends on compare_prices, search_activities)

Dependency graph:
  search_flights ──┐
                  ├─→ compare_prices ──┐
  search_hotels ──┘                    ├─→ create_itinerary
                                       │
  search_activities ────────────────────┘

Execution schedule:
  Phase 1 (parallel): search_flights, search_hotels, search_activities
  Phase 2 (parallel): compare_prices
  Phase 3 (parallel): create_itinerary
```

## Resource Allocation

### Resource Types

The Orchestrator manages:

1. **Concurrent Tool Slots**: Max N tools running simultaneously
2. **Token Budget**: Total tokens available for tool execution
3. **API Rate Limits**: Per-tool rate limits
4. **Timeout Budget**: Total time available

### Resource Allocation Strategy

```
Input: Execution schedule, available resources
  ↓
1. Calculate resource requirements per tool
   - Estimated tokens: from tool manifest
   - Estimated duration: from tool manifest
   - API calls: from tool manifest
   ↓
2. Allocate concurrent slots
   - Max concurrent: min(available_slots, independent_tools_in_phase)
   - Assign tools to slots
   ↓
3. Allocate token budget
   - Per-tool budget: total_budget / num_tools
   - Adjust for tool importance/priority
   ↓
4. Set rate limits
   - Per-tool rate limit: from tool manifest
   - Enforce across all concurrent executions
   ↓
5. Set timeouts
   - Per-tool timeout: from tool manifest
   - Total timeout: sum of phase timeouts
   ↓
Output: Resource allocation plan
```

### Budget Allocation Example

```
Total token budget: 2000 tokens
Tools in Phase 1: 3 (search_flights, search_hotels, search_activities)

Allocation:
  search_flights: 600 tokens (30%)
  search_hotels: 600 tokens (30%)
  search_activities: 400 tokens (20%)
  buffer: 400 tokens (20%)

Rationale:
  - search_flights and search_hotels are equally important
  - search_activities is less critical
  - buffer for unexpected needs
```

## Execution Coordination

### Execution Loop

```
for each phase in schedule:
  1. Prepare tools
     - Resolve parameters
     - Inject context
     - Set resource limits
  ↓
  2. Start tools
     - Launch all tools in phase simultaneously
     - Track execution state
  ↓
  3. Monitor execution
     - Poll for completion
     - Monitor resource usage
     - Detect failures
  ↓
  4. Handle completion
     - Collect results
     - Validate outputs
     - Update context
  ↓
  5. Handle failures
     - Retry if applicable
     - Escalate if not retryable
     - Continue with available results
  ↓
  6. Prepare next phase
     - Use outputs from current phase as inputs
     - Proceed to next phase
```

### Execution State Machine

```
Tool states:
  PENDING → RUNNING → COMPLETED
                   ↓
                 FAILED → RETRYING → COMPLETED
                   ↓
                 ESCALATED

Orchestrator tracks:
  - Current state of each tool
  - Start time, end time, duration
  - Resource usage
  - Errors and warnings
```

## Background Turn Coordination

### Background Turn Lifecycle

```
Turn 1: Send immediate response
  ↓
Background Turn: Continue work
  ↓
1. Initialize background turn
   - Create turn_id for background work
   - Link to parent turn_id
   - Set urgency: LOW (background)
   ↓
2. Execute background work
   - Run tools not completed in Turn 1
   - Update working memory
   - Add observations to Scratch Page
   ↓
3. Collect results
   - Gather all findings
   - Prepare follow-up response
   ↓
4. Finalize background turn
   - Commit state to memory
   - Prepare follow-up message
   ↓
Turn 2: Send follow-up response
```

### Background Turn Scheduling

```
Executor determines:
  - Which tools are critical (needed for immediate response)
  - Which tools are background (can continue after response)
  ↓
Orchestrator schedules:
  - Phase 1: Critical tools (synchronous)
  - Response sent
  - Phase 2: Background tools (asynchronous)
  ↓
Background turn:
  - Executes Phase 2 tools
  - Collects results
  - Prepares follow-up
```

## Error Handling

### Tool Failure Handling

```
Tool fails during execution
  ↓
1. Classify failure
   - Transient (retry-able): network timeout, rate limit
   - Permanent (not retry-able): invalid input, auth failure
   ↓
2. Decide action
   - If transient: retry (up to max_retries)
   - If permanent: escalate or skip
   ↓
3. Update schedule
   - If tool skipped: adjust dependent tools
   - If tool retried: reschedule
   ↓
4. Continue execution
   - Proceed with available results
   - Notify Executor of failures
```

### Cascading Failures

```
Tool A fails
  ↓
Tool B depends on Tool A
  ↓
1. Detect dependency failure
   - Tool B cannot run without Tool A output
   ↓
2. Decide action
   - Skip Tool B (if optional)
   - Escalate (if required)
   - Use default value (if available)
   ↓
3. Continue execution
   - Proceed with remaining tools
```

## Result Aggregation

### Result Collection

```
All tools in phase complete
  ↓
1. Collect results
   - Gather outputs from all tools
   - Gather errors/warnings
   - Gather resource usage
   ↓
2. Validate results
   - Check output schema
   - Check for errors
   - Check completeness
   ↓
3. Aggregate results
   - Combine into single result object
   - Include metadata (timing, resources)
   ↓
4. Return to Executor
   - Pass aggregated results
   - Pass any errors/warnings
```

### Result Format

```
ExecutionResult {
  phase: integer,
  tools_executed: string[],
  results: {
    [tool_id]: {
      status: "success" | "failure" | "partial",
      output: object,
      error: string | null,
      duration_ms: integer,
      tokens_used: integer
    }
  },
  total_duration_ms: integer,
  total_tokens_used: integer,
  failures: {
    [tool_id]: {
      error: string,
      retryable: boolean,
      retry_count: integer
    }
  }
}
```

## API Contracts

### Orchestrator.execute_parallel()

```
execute_parallel(
  plan: Plan,
  context: ExecutionContext,
  resource_limits: ResourceLimits
) → ExecutionResult

where:
  Plan: { tools: Tool[], dependencies: Dependency[] }
  ExecutionContext: { working_memory, consciousness, fc_context }
  ResourceLimits: { max_concurrent, token_budget, timeout_ms }
```

### Orchestrator.schedule_background_turn()

```
schedule_background_turn(
  parent_turn_id: string,
  background_tools: Tool[],
  context: ExecutionContext
) → BackgroundTurnId

Returns: ID for tracking background turn
```

## Testing Requirements

### Test Suite 1: Dependency Analysis

**TC-1.1: Simple linear dependency**
- Tools: A → B → C
- Expected: Schedule has 3 phases
- Verify: Correct ordering

**TC-1.2: Parallel independent tools**
- Tools: A, B, C (no dependencies)
- Expected: Schedule has 1 phase with 3 tools
- Verify: All run in parallel

**TC-1.3: Complex dependency graph**
- Tools: A, B, C, D with mixed dependencies
- Expected: Correct topological sort
- Verify: No tool runs before dependencies

**TC-1.4: Circular dependency detection**
- Tools: A → B → C → A
- Expected: Error detected
- Verify: Execution prevented

### Test Suite 2: Resource Allocation

**TC-2.1: Token budget allocation**
- Budget: 2000 tokens, 3 tools
- Expected: Budget distributed fairly
- Verify: Total ≤ 2000 tokens

**TC-2.2: Concurrent slot allocation**
- Slots: 2, Tools: 4
- Expected: Tools scheduled in 2 concurrent groups
- Verify: Max 2 concurrent

### Test Suite 3: Execution

**TC-3.1: Parallel execution**
- Tools: A, B, C (independent)
- Expected: All start simultaneously
- Verify: Total time ≈ max(A, B, C)

**TC-3.2: Sequential execution**
- Tools: A → B → C
- Expected: Sequential execution
- Verify: Total time ≈ A + B + C

**TC-3.3: Hybrid execution**
- Tools: A, B (parallel) → C
- Expected: A and B parallel, then C
- Verify: Correct ordering and timing

### Test Suite 4: Error Handling

**TC-4.1: Tool failure**
- Tool A fails
- Expected: Error handled, execution continues
- Verify: Dependent tools handled correctly

**TC-4.2: Cascading failure**
- Tool A fails, Tool B depends on A
- Expected: Tool B skipped or escalated
- Verify: Correct handling

### Test Suite 5: Background Turns

**TC-5.1: Background turn scheduling**
- Critical tools complete, background tools scheduled
- Expected: Background turn created
- Verify: Background tools execute asynchronously

## Performance Targets

- **Dependency analysis**: < 100ms
- **Schedule generation**: < 100ms
- **Resource allocation**: < 50ms
- **Execution overhead**: < 10% of tool execution time
- **Result aggregation**: < 50ms

## Next Steps

See related documentation:
- `documentation/turn/hobgoblins/executor.md` - Executor hobgoblin
- `documentation/turn/parallelism.md` - Parallelism patterns
- `documentation/tools/si_tools_architecture.md` - Tool architecture

