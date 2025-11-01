# Executor Hobgoblin

## Purpose

The Executor orchestrates the execution of plans. It calls tools, manages their execution, handles results, and coordinates with other hobgoblins. Executor is the "doer" of Si's cognitive architecture.

## Responsibilities

1. **Receive Plan**: Accept plan from Plan Generator
2. **Execute Steps**: Call tools and execute steps
3. **Manage Execution**: Handle sequencing and parallelism
4. **Handle Results**: Process tool outputs
5. **Handle Errors**: Detect and respond to failures
6. **Coordinate**: Work with Error Handler if needed

## What Executor Does

### Tool Execution
- Calls appropriate tools based on plan
- Passes correct parameters
- Manages tool execution
- Collects results

### Sequencing
- Executes steps in correct order
- Respects dependencies
- Waits for prerequisites
- Triggers dependent steps

### Parallelism
- Identifies parallelizable steps
- Executes independent steps concurrently
- Synchronizes when needed
- Manages resource constraints

### Result Management
- Collects tool outputs
- Validates results
- Stores results in working memory
- Makes results available to other hobgoblins

### Error Handling
- Detects execution failures
- Alerts Error Handler
- Waits for recovery decision
- Resumes or aborts based on decision

## Decision Inputs

Executor makes decisions using:

### Consciousness
- **Capabilities**: What tools are available?
- **Governance**: What constraints apply to tool use?
- **Values**: What matters in execution?

### Working Memory
- **Plan**: What steps to execute?
- **Context**: What's the current situation?
- **Results**: What have previous steps produced?

### Episodic Memory
- **Similar Executions**: How were similar plans executed?
- **Learned Patterns**: What execution patterns work well?
- **Outcomes**: What execution strategies succeeded?
- **Skills**: What execution skills has Si developed?

### Input
- **Plan**: What needs to be executed?
- **Constraints**: What limitations exist?

## Execution Process

```
Executor receives plan
    ↓
Analyze plan:
  - Identify steps
  - Identify dependencies
  - Identify parallelizable steps
    ↓
Execute steps:
  ├─ Sequential steps: Execute in order
  ├─ Parallel steps: Execute concurrently
  └─ Dependent steps: Wait for prerequisites
    ↓
For each step:
  - Call appropriate tool
  - Pass parameters
  - Collect results
  - Store in working memory
    ↓
Handle results:
  - Validate results
  - Check for errors
  - Make available to other hobgoblins
    ↓
If error occurs:
  - Alert Error Handler
  - Wait for recovery decision
  - Resume or abort
    ↓
Execution complete
```

## Tool Execution Example

**Plan Step**: Search flights to Paris

```
Executor receives step:
  - Tool: flight_search
  - Parameters: destination=Paris, dates=X-Y, passengers=2
    ↓
Executor calls tool:
  - flight_search(destination="Paris", dates="X-Y", passengers=2)
    ↓
Tool returns results:
  - List of flights with prices, times, airlines
    ↓
Executor validates results:
  - Are results in expected format?
  - Are results reasonable?
  - Are there any errors?
    ↓
Executor stores results:
  - Save to working memory
  - Make available to other steps
    ↓
Executor continues to next step
```

## Parallelism Example

**Plan with parallel steps**:

```
Step 1: Search flights (parallel)
Step 2: Search hotels (parallel)
Step 3: Compare options (depends on 1 and 2)

Executor execution:
  ├─ Call flight_search (parallel)
  ├─ Call hotel_search (parallel)
  ├─ Wait for both to complete
  └─ Call compare_options
```

## Error Handling

When a tool fails, Executor alerts Error Handler:

```
Executor calls tool
    ↓
Tool fails (returns error)
    ↓
Executor detects error
    ↓
Executor alerts Error Handler
    ↓
Error Handler decides:
  - Retry?
  - Use alternative tool?
  - Escalate?
  - Fail gracefully?
    ↓
Executor resumes based on decision
```

## Learning Through Executor

Executor improves through the learning loop:

**Feedback Sources**:
- Did tool execution succeed?
- Were steps executed in optimal order?
- Could more steps have been parallelized?
- Did error handling work well?
- Was execution efficient?

**Learning Updates**:
- Episodic Memory: Store successful execution patterns
- Skills: Improve execution for similar plans
- Consciousness: Refine tool usage if needed

**Example**:
- Turn 1: Executor executes travel booking plan
- All steps succeed, results are good
- Evaluator: Execution was efficient → GOOD
- Reflector: Learn "This execution pattern works well"

## Key Principle

**Executor is the action-taker.** It transforms plans into results by calling tools and managing their execution. Executor's efficiency directly impacts turn completion time and user satisfaction.

