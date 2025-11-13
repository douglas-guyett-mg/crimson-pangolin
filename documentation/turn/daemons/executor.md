# Executor Daemon

## Purpose

The Executor is Si's **autonomous LLM-based decision agent**. It is **always spawned** (regardless of Router's decision) and makes adaptive decisions about what to do next in a turn.

Executor's job is to:
1. Query Working Memory to understand the current situation
2. Receive comprehensive context (consciousness, goals, memories, available tools/daemons)
3. Use an LLM to decide what action to take next
4. Execute the decision (call tool, query daemon, spawn sub-turn, respond, etc.)
5. Loop with results until the turn is complete
6. Update Working Memory at commit

Executor is the "autonomous decision-maker" of Si's cognitive architecture - it uses LLM reasoning to adaptively decide what to do based on the current situation, not a fixed sequence of steps.

## Responsibilities

1. **Query Working Memory**: Get current state, context, and situation assessment
2. **Receive Context**: Get comprehensive information:
   - Consciousness (mandates, capabilities, modes, governance)
   - Frontal Cortex (goals, pending actions, constraints)
   - Conversation history and recent observations
   - Episodic and semantic memory (relevant to current task)
   - Available tools and daemons
   - Budget status (tokens, tool calls, time)
3. **LLM Decision Making**: Use an LLM to decide what to do next:
   - Call a tool to get information or take action
   - Query a daemon (FC, Consciousness, Memory, etc.)
   - Spawn a sub-turn for parallel work
   - Respond to the user
   - End the turn
4. **Execute Decision**: Perform the action decided by the LLM
5. **Handle Errors**: Detect failures and use Error Handler capability
6. **Check Constraints**: Verify constraints are respected during execution
7. **Loop**: Feed results back into next LLM call for adaptive decision-making
8. **Update Memory**: Update Working Memory with results and outcomes
9. **Log Decisions**: Record all decisions and reasoning to Turn Trace for auditability and learning

## What Executor Does

### LLM-Based Decision Making
- Receives comprehensive context from Working Memory
- Uses LLM to reason about what to do next
- Decides between: tool calls, daemon queries, sub-turns, responses, or end-turn
- Explains reasoning for each decision (logged to Turn Trace)

### Adaptive Execution
- Executes the LLM's decision
- Collects results
- Feeds results back into next LLM call
- Adapts based on intermediate outcomes (not locked into pre-made plan)

### Tool Invocation
- Calls appropriate tools based on LLM decision
- Passes correct parameters
- Manages tool execution
- Collects and validates results

### Daemon Queries
- Queries Frontal Cortex for planning assistance
- Queries Consciousness for mandates and capabilities
- Queries Memory daemons for relevant information
- Collects results and feeds back to LLM

### Parallelism
- Can spawn sub-turns for parallel work
- Manages resource constraints
- Synchronizes results
- Collects outcomes

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

## Turn Orchestration Process

```
Router decides: "Proceed with Executor"
    ↓
Executor spawned
    ↓
Query Working Memory: "What's the current situation?"
    ↓
WM returns: assembled context (consciousness, goals, memories, tools, etc.)
    ↓
[Async] Gut monitors execution
    ↓
DECISION LOOP:
  ↓
  Executor LLM receives: [System Prompt] + [Context] + [Available Actions]
    ↓
  LLM decides: "I should [action]"
    ↓
  Execute action:
    - If call_tool: invoke tool, collect results
    - If query_daemon: query daemon, collect results
    - If spawn_subturn: create sub-turn, collect results
    - If respond: generate response to user
    - If end_turn: complete turn
    ↓
  Check constraints (built-in capability):
    - Verify budgets respected
    - Verify consciousness mandates followed
    ↓
  Monitor Gut concerns:
    - If concern severity > threshold:
      - Interrupt flow
      - Query FC: "Should we update approach?"
      - Decide: continue, redirect, or escalate
    ↓
  Handle errors (built-in capability):
    - Detect failures
    - Assess severity
    - Execute recovery strategy
    ↓
  Update Working Memory with results
    ↓
  Log decision to Turn Trace (reasoning + action + outcome)
    ↓
  Is turn complete? (response sent + all goals met)
    - If NO: Loop back to LLM with new results
    - If YES: Proceed to commit
    ↓
Update Working Memory at commit
    ↓
Execution complete
```

## Lifecycle State Execution

When Executor decides to execute the **Action(s)** state, it:

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
  - Make available to other daemons
    ↓
If error occurs:
  - Alert Error Handler
  - Wait for recovery decision
  - Resume or abort
    ↓
Action(s) state complete
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

## Key Principles

**Executor is the turn orchestrator.** It:
1. Queries other daemons to understand what work is needed
2. Decides which lifecycle states to execute (not all states occur in every turn)
3. Executes only the necessary states
4. Logs all decisions to Turn Trace for auditability and learning

**Executor is also the action-taker.** When the Action(s) lifecycle state is needed, Executor transforms plans into results by calling tools and managing their execution.

**Executor enables learning.** By logging which states were executed and why, the system can learn which decisions work well and improve daemon instructions over time.

