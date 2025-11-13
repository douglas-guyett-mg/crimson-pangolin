# Parallelism: Concurrent Execution Within and Across Turns

## Overview

Si's architecture supports parallelism at two levels:
1. **Within a turn**: Multiple daemons can execute concurrently
2. **Across turns**: Multiple turns can execute concurrently

This enables efficient use of resources and faster response times.

## Parallelism Within a Turn

### Scenario: Multiple Daemons Working Together

When a turn requires complex decision-making, multiple daemons can work in parallel:

```
Turn starts
    ↓
Router: Decides urgency and which daemons to activate
    ↓
Parallel execution:
  ├─ Working Memory: Gathers context from consciousness, episodic memory, turn trace
  ├─ Frontal Cortex: Creates execution plan and verifies constraints
  └─ Executor: Prepares for tool orchestration
    ↓
Results merge
    ↓
Executor: Orchestrates tool execution
    ↓
Evaluator: Assesses outcome
```

### Example: Complex Query

**Input**: "I want to book a flight to Paris, but I'm concerned about my budget and carbon footprint"

**Parallel work**:
- **Clarifier** (parallel): Handles any ambiguous inputs
- **Gut** (parallel): Provides intuitive guidance on user intent

**Result**: All parallel daemons complete their work, then Executor orchestrates tool calls.

### Synchronization Points

Daemons can work in parallel until they need to synchronize:

```
Clarifier ──┐
            ├─→ Synchronization Point ──→ Executor
Gut ────────┘
```

At synchronization points, all parallel work must complete before proceeding.

## Parallelism Across Turns

### Scenario: Multiple Turns Running Simultaneously

Si can process multiple user inputs concurrently:

```
Turn 1: Suspicious email (HIGH URGENCY)
  ├─ Router: Detect urgency
  ├─ Responder: Send immediate response
  └─ Executor: Research in background

Turn 2: Travel planning (MEDIUM URGENCY)
  ├─ Router: Detect urgency
  ├─ Working Memory: Gather preferences
  └─ Frontal Cortex: Create plan

Turn 3: General question (LOW URGENCY)
  ├─ Router: Detect urgency
  └─ Frontal Cortex: Create comprehensive plan

All three turns active simultaneously
```

### Shared Working Memory

Parallel turns access and update shared working memory:

```
Working Memory (Shared)
├─ Conversation History
├─ Turn Trace
├─ Episodic Memory
├─ Consciousness
└─ Current Context

Turn 1 ──┐
         ├─→ Shared Working Memory ←─┬─ Turn 2
Turn 3 ──┘                           └─ Turn 4
```

### Coordination Across Turns

When turns run in parallel, they need to coordinate:

**Read Coordination**:
- Turn 1 reads conversation history
- Turn 2 reads same conversation history
- Both see consistent view

**Write Coordination**:
- Turn 1 updates turn trace with its findings
- Turn 2 updates turn trace with its findings
- Updates are merged (order may matter)

**Conflict Resolution**:
- If two turns try to update the same memory location, system must decide:
  - Last-write-wins?
  - Merge updates?
  - Queue updates?

This is a technology choice made when implementing the system.

## Parallelism Patterns

### Pattern 1: Independent Parallel Turns

Turns that don't interact can run fully in parallel:

```
Turn 1: "What's the weather?"
Turn 2: "Book me a restaurant"
Turn 3: "Translate this text"

All three are independent → full parallelism
```

### Pattern 2: Sequential Turns with Shared Context

Turns that depend on each other run sequentially:

```
Turn 1: "Find flights to Paris"
  ↓ (Turn 1 completes)
Turn 2: "Book the cheapest flight"
  ↓ (Turn 2 uses Turn 1's results)
Turn 3: "Book a hotel near the airport"
  ↓ (Turn 3 uses Turn 1's results)
```

### Pattern 3: Parallel Turns with Shared Context

Turns that can work in parallel but share context:

```
Turn 1: "Find flights to Paris"
  ├─ Executor: Search flights
  └─ Executor: Search hotels (parallel)
    ↓
Turn 2: "Compare prices"
  ├─ Uses Turn 1's flight results
  └─ Uses Turn 1's hotel results
```

## Parallelism Constraints

Not all work can be parallelized. Some constraints:

### Data Dependencies
- If Turn 2 needs results from Turn 1, they must be sequential
- If Turn 2 and Turn 3 are independent, they can be parallel

### Resource Constraints
- Limited number of concurrent tool calls
- Limited API rate limits
- Limited computational resources

### Ordering Constraints
- Some operations must happen in order (e.g., authentication before data access)
- Some operations have dependencies (e.g., plan must be created before execution)

## Parallelism Benefits

### Speed
- Multiple turns process simultaneously
- Multiple daemons work concurrently
- Faster overall response time

### Responsiveness
- High-urgency turns get immediate attention
- Low-urgency turns don't block high-urgency turns
- User gets faster feedback

### Resource Efficiency
- While one turn waits for tool results, another turn can execute
- Idle time is minimized
- Resources are used more efficiently

## Parallelism Challenges

### Complexity
- Parallel execution is harder to reason about
- Debugging is more difficult
- Testing requires more scenarios

### Consistency
- Shared working memory must remain consistent
- Concurrent updates must be coordinated
- Race conditions must be prevented

### Observability
- Tracing parallel execution is complex
- Logs from parallel turns are interleaved
- Debugging requires careful analysis

## Technology Implications

Parallelism strategy affects technology choices:

**For Daemon Parallelism**:
- Async/await support
- Concurrent task scheduling
- Synchronization primitives

**For Turn Parallelism**:
- Message queues or event streams
- Distributed state management
- Conflict resolution strategies

**For Shared Working Memory**:
- Concurrent data structure support
- Locking or optimistic concurrency
- Consistency guarantees

These are implementation details chosen when technology stack is selected.

## Key Principle

**Parallelism is an optimization, not a requirement.** The system can function with sequential execution, but parallelism enables better performance and responsiveness. Technology choices should support parallelism but not require it.

