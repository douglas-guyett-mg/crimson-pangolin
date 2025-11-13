# Turn Trace Data Model

**Status:** Specification v1.0  
**Date:** 2025-11-01  
**Purpose:** Define the structure and semantics of Turn Trace - the complete audit log of turn execution

---

## Overview

Turn Trace is a **write-only audit log during turn execution** and a **read-only analysis log after turn completes**. It records everything that happens during a turn for debugging, auditing, learning, and monitoring.

**Key principle:** Turn Trace is NOT meant to be queried by daemons during turn execution for decision-making. That's Working Memory's job. Turn Trace is for post-turn analysis.

---

## Purpose

Turn Trace serves to:

1. **Audit/Compliance** - Complete record of what happened during turn execution
2. **Debugging** - If something goes wrong, replay the trace to understand what happened
3. **Learning** - Reflector daemon uses trace to extract learnings and update consciousness/episodic memory
4. **Monitoring & Observability** - Primary mechanism for system health, performance, and policy monitoring (see Observability section below)
5. **Transparency** - Make system decisions and reasoning transparent

---

## Core Data Structures

### Trace (Top-Level Container)

```
Trace {
  trace_id: string              # Unique identifier for this trace
  turn_id: string               # Which turn this trace belongs to
  agent_id: string              # Which agent executed this turn
  conversation_id: string       # Which conversation (optional)
  started_at: timestamp         # When turn started
  ended_at: timestamp           # When turn ended (null if still running)
  budgets: {                    # Budgets for this turn
    tokens: number
    tools: number
    time_ms: number
  }
  components: string[]          # Which components participated (Router, Executor, etc.)
  spans: Span[]                 # Nested execution contexts
  events: Event[]               # All events that occurred
}
```

### Span (Execution Context)

```
Span {
  span_id: string               # Unique identifier
  trace_id: string              # Parent trace
  parent_span_id: string        # Parent span (null if top-level)
  component: string             # Which component (Router, Executor, Reflector, etc.)
  name: string                  # Human-readable name (e.g., "route_decision", "tool_execution")
  start_ts: timestamp           # When span started (millisecond precision)
  end_ts: timestamp             # When span ended (null if still running)
  depends_on: string[]          # Span IDs that must complete before this span starts
  attributes: {                 # Custom attributes
    [key: string]: any
  }
}
```

**Key Fields:**
- `start_ts` / `end_ts`: Millisecond precision timestamps for performance analysis
- `depends_on`: Explicit dependency links for causality tracking (see Causality Tracking section below)

### Event (Individual Log Entry)

```
Event {
  event_id: string              # Unique identifier
  ts: timestamp                 # When event occurred
  trace_id: string              # Parent trace
  span_id: string               # Parent span (optional)
  component: string             # Which component emitted this
  kind: string                  # Event type (Decision, ToolInvocation, Observation, etc.)
  agent_msg: string             # Natural language explanation from component (optional)
  attributes: {                 # Type-specific fields (see Event Types below)
    [key: string]: any
  }
}
```

---

## Event Types (Extensible Registry)

### Decision Event
```
{
  kind: "Decision"
  attributes: {
    chosen: string              # Which action was chosen
    considered: string[]        # Other options considered
    rationale: string           # Why this decision was made
    score: number               # Decision confidence (0-1)
    policy_notes: string        # Any governance rules applied
  }
}
```

### ToolInvocation Event
```
{
  kind: "ToolInvocation"
  attributes: {
    tool_name: string           # Which tool was called
    attempt: number             # Attempt number (for retries)
    status: string              # queued|started|succeeded|failed|cancelled
    latency_ms: number          # How long it took
    cost: number                # Resource cost (if applicable)
    error_class: string         # Error type (if failed)
    error_msg: string           # Error message (if failed)
  }
}
```

### Observation Event
```
{
  kind: "Observation"
  attributes: {
    summary: string             # What was learned
    confidence: number          # Confidence in observation (0-1)
    source: string              # Where observation came from
  }
}
```

### Error Event
```
{
  kind: "Error"
  attributes: {
    class: string               # Error type
    message: string             # Error message
    retryable: boolean          # Can this be retried?
    backoff_ms: number          # Suggested backoff time
  }
}
```

### Policy Event
```
{
  kind: "Policy"
  attributes: {
    rule_id: string             # Which governance rule
    action: string              # allow|deny|redact|rate_limit
    details: object             # Rule-specific details
  }
}
```

### Summary Event
```
{
  kind: "Summary"
  attributes: {
    level: string               # micro|meso|macro
    text: string                # Summary text
    tokens: number              # Tokens used
  }
}
```

---

## Ordering Guarantees and Causality Tracking

### Timestamp Precision

- **Precision:** Millisecond (ms) - sufficient for performance analysis and ordering
- **Format:** ISO 8601 with millisecond precision (e.g., "2025-11-07T14:30:45.123Z")
- **Monotonicity:** Within a single component, timestamps are monotonically increasing
- **Clock skew:** System must handle clock skew across distributed components (if applicable)

### Causality Tracking

Causality is tracked through **explicit dependency links** in the `depends_on` field:

```
Span A (Router)
  └─ depends_on: []

Span B (Clarifier)
  └─ depends_on: [Span A]

Span C (Executor)
  └─ depends_on: [Span B]
```

**Interpretation:**
- Spans B, C, D can run in parallel (all depend on A, but not on each other)
- Span E must wait for B, C, D to complete
- This is the **critical path** for the turn

### Ordering Rules

1. **Within a component:** Events are strictly ordered (monotonic sequence)
2. **Dependency ordering:** If Span A depends on Span B, then Span B must complete before Span A starts
3. **Parallel spans:** Spans with the same parent but no dependency links can run in parallel
4. **Global order:** Use (timestamp, span_hierarchy, depends_on) for deterministic replay
5. **Spans form a DAG:** Parent→child relationships; dependency links allowed
6. **Deterministic replay:** Given the same trace, replay produces same results

### Causality Examples

#### Example 1: Sequential Execution

```
Turn: "Search for flights and book the cheapest one"

Span 1: Search Flights (T=0-100ms)
  └─ depends_on: []

Span 2: Book Flight (T=100-200ms)
  └─ depends_on: [Span 1]

Interpretation:
- Book Flight depends on Search Flights
- Must run sequentially
- Book Flight cannot start until Search Flights completes
```

#### Example 2: Parallel Execution

```
Turn: "Book flight, hotel, and rental car"

Span 1: Search Flights (T=0-100ms)
  └─ depends_on: []

Span 2: Search Hotels (T=0-120ms)
  └─ depends_on: []

Span 3: Search Rental Cars (T=0-80ms)
  └─ depends_on: []

Span 4: Consolidate Results (T=120-150ms)
  └─ depends_on: [Span 1, Span 2, Span 3]

Interpretation:
- Spans 1, 2, 3 can run in parallel (no dependencies)
- Span 4 must wait for all three to complete
- Critical path: max(100, 120, 80) + 30 = 150ms
```

#### Example 3: Partial Parallelism

```
Turn: "Complex decision with parallel analysis"

Span 1: Router (T=0-50ms)
  └─ depends_on: []

Span 2: Clarifier (T=50-100ms)
  └─ depends_on: [Span 1]

Span 3: Executor (T=100-300ms)
  └─ depends_on: [Span 2]

Interpretation:
- Router runs first (T=0-50ms)
- Spans 2, 3, 4 run in parallel (all depend on Router, but not on each other)
- Executor waits for all three to complete
- Critical path: 50 + max(70, 50, 100) + 150 = 300ms
```

### Concurrent Tool Invocations

When multiple tools are invoked in parallel:

```
Span: Executor Tool Invocation Phase (T=150-300ms)
  ├─ Tool Invocation 1: Search Flights (T=150-200ms)
  │   └─ depends_on: []
  ├─ Tool Invocation 2: Search Hotels (T=150-220ms)
  │   └─ depends_on: []
  └─ Tool Invocation 3: Search Rental Cars (T=150-180ms)
      └─ depends_on: []

Interpretation:
- All three tools can run in parallel
- No dependencies between them
- All must complete before Executor continues
```

### Debugging with Causality

When something fails, causality tracking enables effective debugging:

```
Scenario: Executor failed because it didn't have complete context

Turn Trace shows:
- Span: Clarifier (T=50-100ms)
  └─ depends_on: [Router]
  └─ Result: "User wants budget-conscious option"

- Span: Executor (T=100-300ms)
  └─ depends_on: [Clarifier]
  └─ Result: "Executed plan costing $8000"

Analysis:
- Executor should have received Clarifier's budget constraint
- Executor proceeded without complete context
- This is why the plan violated the budget

Fix:
- Ensure Executor receives all context from Clarifier
- Verify budget constraints before execution
```

### Replay with Causality

To replay a turn deterministically:

1. **Extract dependency graph** from trace
2. **Topologically sort** spans by dependencies
3. **Execute in order**, respecting dependencies
4. **Compare results** with original trace

```
Original trace dependency graph:
  Router → Clarifier → Executor

Replay execution:
1. Run Router (no dependencies)
2. Run Clarifier (depends on Router)
3. Run Executor (depends on Clarifier)

Result: Should match original trace exactly
```

---

## Lifecycle

### During Turn Execution (Write Phase)
- Components emit events via `log_event(kind, component, payload, agent_msg?, span_id?)`
- Events are appended to trace in order
- Spans track nested execution contexts
- No reading/querying during this phase

### After Turn Completes (Read Phase)
- Trace is finalized and closed
- Reflector daemon reads trace to extract learnings
- Episodic Memory daemon reads trace to store turn events
- Debugging tools can query trace for analysis
- Monitoring systems can analyze trace for health/performance

---

## Retention and Compaction

### Retention Policy
- **Active traces:** Kept in memory during turn execution
- **Recent traces:** Kept in fast storage for 24 hours
- **Archived traces:** Moved to cold storage after 24 hours
- **Deleted traces:** Removed after 90 days (configurable)

### Compaction
- Traces can be compacted to reduce storage
- Compaction removes low-importance events
- Keeps high-importance events and all errors
- Preserves deterministic ordering

---

## Observability & Monitoring

Turn Trace is the **primary observability mechanism** for Si. All monitoring, metrics, dashboards, and alerting are built on top of Turn Trace data.

### Logging Capability Requirements

Every component that participates in a turn MUST emit events to Turn Trace:

1. **Decision Events** - Every significant decision (Router, Executor, FC, etc.)
2. **Tool Invocation Events** - Every tool call with latency, status, errors
3. **Error Events** - Every error, with classification (transient vs permanent)
4. **Policy Events** - Every policy check (consciousness mandates, constraints, etc.)
5. **Observation Events** - Every observation or insight discovered
6. **Summary Events** - Periodic summaries at different levels (micro/meso/macro)

### Monitoring Use Cases

**Real-Time Monitoring:**
- Track turn execution in progress
- Monitor budget consumption (tokens, tools, time)
- Detect errors and anomalies
- Alert on policy violations

**Post-Turn Analysis:**
- Calculate performance metrics (latency, success rate, cost)
- Identify patterns and trends
- Detect regressions or degradation
- Generate dashboards and reports

**System Health:**
- Tool success/failure rates
- Error classification and frequency
- Policy violation patterns
- Resource utilization trends

### Metrics Derived from Turn Trace

**Performance Metrics:**
- Turn latency (end_ts - started_at)
- Tool invocation latency (per tool)
- Budget utilization (tokens, tools, time)
- Component execution time (per component)

**Quality Metrics:**
- Tool success rate
- Error rate and classification
- Policy violation rate
- Decision confidence scores

**Resource Metrics:**
- Tokens consumed per turn
- Tool calls per turn
- Memory operations per turn
- Cost per turn

### Integration Points

- **Reflector Daemon:** Reads trace after turn completes to extract learnings
- **Episodic Memory:** Reads trace to store turn events
- **Frontal Cortex:** Reads trace to update goals and pending actions
- **Debugging Tools:** Query trace to understand what happened
- **Monitoring Systems:** Analyze trace for health and performance metrics
- **Dashboards:** Display real-time and historical metrics from traces
- **Alerting:** Trigger alerts based on trace events and metrics

