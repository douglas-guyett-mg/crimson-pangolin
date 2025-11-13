# Daemon Coordination Protocol

**Status:** Specification v1.0  
**Date:** 2025-11-05  
**Technology:** Agnostic

## Overview

The Daemon Coordination Protocol defines how daemons communicate with each other during turn execution. It specifies message format, async semantics, ordering guarantees, conflict resolution, and deadlock prevention.

---

## 1. Message Format

### 1.1 Core Message Structure

All daemon-to-daemon communication uses a structured message format:

```
{
  query: string (REQUIRED),
  metadata?: object (optional),
  data?: object (optional),
  [daemon-specific fields]
}
```

**Required Field:**
- `query` (string): Natural language query describing what the daemon needs

**Optional Fields:**
- `metadata` (object): Context about the query (tags, filters, priorities, etc.)
- `data` (object): Structured data to pass to the daemon
- Daemon-specific fields: Each daemon can define additional fields

### 1.2 Daemon-Specific Fields

Each daemon declares what optional fields it accepts in its `get_instructions()` method:

```
get_instructions() returns:
{
  purpose: "What this daemon does",
  query_ordering: "none" | "fifo" | "causal",
  optional_fields: {
    field_name: "description",
    field_name: "description"
  },
  examples: [...]
}
```

**Example: Consciousness Daemon**
```
get_instructions() returns:
{
  purpose: "Provide Si's mandates, capabilities, modes, and governance rules",
  query_ordering: "fifo",
  optional_fields: {
    category: "Filter by mandate category (mission, safety, collaboration, operational)",
    version: "Request specific version (default: effective version)"
  }
}
```

**Example: Clarifier Daemon**
```
get_instructions() returns:
{
  purpose: "Handle ambiguous inputs by asking clarifying questions",
  query_ordering: "fifo",
  optional_fields: {
    ambiguity_type: "Type of ambiguity detected",
    suggested_questions: "Clarifying questions to ask"
  }
}
```

### 1.3 Message Example

```
consciousness.query(
  query: "What mandates apply to financial security?",
  metadata: {
    tags: ["financial", "security"],
    priority: "high"
  },
  category: "safety",
  version: "latest"
)
```

---

## 2. Async Semantics

### 2.1 All Operations Are Asynchronous

All daemon queries return futures/promises:

```
future = daemon.query(message)
# Caller can do other work while waiting
result = await future
```

### 2.2 Parallel Queries

Multiple daemons can be queried in parallel:

```
future_consciousness = consciousness.query(...)
future_context = context_assembler.query(...)
future_episodic = episodic_memory.query(...)

# Do other work while waiting
result_consciousness = await future_consciousness
result_context = await future_context
result_episodic = await future_episodic
```

### 2.3 Timeout Semantics

Each query has a timeout:

```
future = daemon.query(message, timeout_ms=5000)
try:
  result = await future
except TimeoutError:
  # Handle timeout
```

**Default timeout:** 5000ms (configurable per daemon)

---

## 3. Ordering Guarantees

### 3.1 Default: No Ordering Guarantees

By default, queries are processed as soon as possible:
- Queries processed in any order
- Results arrive whenever ready
- Callers must handle out-of-order results

**Use case:** Context Assembler, Plan Generator (independent queries)

### 3.2 FIFO Ordering

Some daemons require FIFO (first-in-first-out) ordering:
- Queries processed in order received
- Ensures consistency
- Slower than no ordering

**Use case:** Consciousness (mandates must be consistent), Frontal Cortex (goals must be consistent)

**Declaration:**
```
get_instructions() returns:
{
  query_ordering: "fifo",
  ...
}
```

### 3.3 Causal Ordering

Some daemons require causal ordering:
- If Query A depends on Query B, ensure B completes first
- Prevents anomalies and race conditions
- Requires dependency tracking

**Use case:** Error Handler (depends on error classification), Evaluator (depends on execution results)

**Declaration:**
```
get_instructions() returns:
{
  query_ordering: "causal",
  ...
}
```

### 3.4 Requesting Specific Ordering

Callers can request specific ordering if needed:

```
consciousness.query(
  query: "What mandates apply?",
  metadata: {ordering: "fifo"}  # Request FIFO
)
```

### 3.5 Ordering in Turn Trace

All ordering decisions are logged:

```
{
  event_type: "DaemonQuery",
  daemon_id: "consciousness",
  query: "What mandates apply?",
  ordering_policy: "fifo",
  ordering_requested: "fifo",
  timestamp: "2025-11-05T10:30:45Z"
}
```

---

## 4. Conflict Resolution

### 4.1 Query Conflicts

When multiple daemons query the same resource, conflicts can occur:

**Example:** Two daemons update Frontal Cortex goal simultaneously

**Resolution Strategy:** Optimistic locking (see `documentation/frontal-cortex/persistence-layer.md`)

1. Daemon A reads goal v5
2. Daemon B reads goal v5
3. Daemon A writes goal v6 (success)
4. Daemon B writes goal v6 (conflict - now v6)
5. Daemon B retries: reads goal v6, writes goal v7 (success)

### 4.2 Conflicting Decisions

When daemons make conflicting decisions:

**Example:** Executor detects constraint violation but user requests override

**Resolution Strategy:** Executor escalates to Error Handler for decision

1. Executor receives conflicting decisions
2. Executor logs conflict to Turn Trace
3. Executor queries Consciousness for tiebreaker
4. Executor makes final decision
5. Executor logs decision to Turn Trace

### 4.3 Conflict Logging

All conflicts logged to Turn Trace:

```
{
  event_type: "DaemonConflict",
  daemon_1: "constraint_checker",
  daemon_2: "plan_generator",
  conflict_type: "decision_conflict",
  resolution: "executor_arbitration",
  decision: "constraint_checker_wins",
  timestamp: "2025-11-05T10:30:45Z"
}
```

---

## 5. Deadlock Prevention

### 5.1 Circular Dependencies

Deadlock occurs when daemons wait for each other in a cycle:

```
Daemon A waits for Daemon B
Daemon B waits for Daemon C
Daemon C waits for Daemon A
→ Deadlock
```

### 5.2 Prevention Strategy: Timeout + Escalation

1. **Timeout**: Each query has timeout (default 5s)
2. **Escalation**: On timeout, escalate to Executor
3. **Executor decides**: Retry, use default, or fail gracefully

```
Daemon A queries Daemon B (timeout: 5s)
  ↓ (5s passes)
Timeout occurs
  ↓
Escalate to Executor
  ↓
Executor decides:
  - Retry with longer timeout
  - Use default value
  - Fail gracefully
```

### 5.3 Acyclic Dependency Graph

Daemons should form an acyclic dependency graph:

```
Router
  ↓
Clarifier, Gut (parallel)
  ↓
Executor
  ↓
Error Handler, Evaluator (parallel)
  ↓
Reflector
```

**Principle:** Daemons should not query daemons that query them.

### 5.4 Deadlock Detection

Turn Trace monitors for deadlocks:

```
If query timeout occurs:
  1. Log timeout to Turn Trace
  2. Check for circular dependencies
  3. If circular: log deadlock warning
  4. Escalate to Executor
```

---

## 6. Error Handling

### 6.1 Query Errors

When a daemon query fails:

```
future = daemon.query(message)
try:
  result = await future
except DaemonError as e:
  # Handle error
  # Error Handler daemon decides recovery
```

### 6.2 Error Types

- **TimeoutError**: Query exceeded timeout
- **DaemonUnavailableError**: Daemon not responding
- **InvalidQueryError**: Query format invalid
- **ProcessingError**: Daemon encountered error processing query
- **ConflictError**: Conflict detected (for Frontal Cortex queries)

### 6.3 Error Logging

All errors logged to Turn Trace:

```
{
  event_type: "DaemonError",
  daemon_id: "consciousness",
  error_type: "TimeoutError",
  error_message: "Query exceeded 5000ms timeout",
  timestamp: "2025-11-05T10:30:45Z"
}
```

---

## 7. Integration with Turn Execution

### 7.1 Query Lifecycle

```
Executor needs decision
  ↓
Executor queries daemon(s)
  ↓
Daemon(s) process query
  ↓
Results arrive (async)
  ↓
Executor uses results
  ↓
Executor logs to Turn Trace
```

### 7.2 Synchronization Points

Daemons can work in parallel until synchronization points:

```
Context Assembler ──┐
                    ├─→ Synchronization Point ──→ Executor
Constraint Checker ─┤
                    │
Plan Generator ─────┘
```

At synchronization points, all parallel queries must complete.

### 7.3 Turn Trace Recording

All daemon queries recorded:

```
{
  event_type: "DaemonQuery",
  daemon_id: "consciousness",
  query: "What mandates apply?",
  query_ordering: "fifo",
  status: "completed",
  result_summary: "3 mandates returned",
  duration_ms: 245,
  timestamp: "2025-11-05T10:30:45Z"
}
```

---

## 8. Test Conditions

### 8.1 Message Format Tests
- ✅ Required field (query) enforced
- ✅ Optional fields accepted
- ✅ Daemon-specific fields validated
- ✅ Invalid fields rejected

### 8.2 Async Semantics Tests
- ✅ Queries return futures
- ✅ Multiple queries execute in parallel
- ✅ Timeout enforced
- ✅ Results available after await

### 8.3 Ordering Tests
- ✅ No ordering: queries processed in any order
- ✅ FIFO: queries processed in order received
- ✅ Causal: dependencies respected
- ✅ Ordering policy declared in get_instructions()

### 8.4 Conflict Resolution Tests
- ✅ Optimistic locking detects conflicts
- ✅ Retries succeed after conflict
- ✅ Conflicting decisions arbitrated by Executor
- ✅ Conflicts logged to Turn Trace

### 8.5 Deadlock Prevention Tests
- ✅ Timeout prevents infinite wait
- ✅ Circular dependencies detected
- ✅ Escalation to Executor on timeout
- ✅ Acyclic dependency graph maintained

### 8.6 Error Handling Tests
- ✅ Query errors caught and handled
- ✅ Error types classified correctly
- ✅ Errors logged to Turn Trace
- ✅ Error Handler daemon decides recovery

---

## 9. Relationship to Other Systems

- **Daemon Interface**: Defines query(), get_purpose(), get_instructions()
- **Turn Trace**: Records all daemon queries and decisions
- **Executor**: Orchestrates daemon queries and synchronization
- **Error Handler**: Handles daemon query failures
- **Consciousness**: Provides mandates and governance rules
- **Frontal Cortex**: Stores goals and pending actions (with optimistic locking)

