# Daemon Coordination Protocol - Test Conditions

**Status:** v1.0  
**Date:** 2025-11-05

## Test Categories

### 1. Message Format Tests

#### 1.1 Required Field (query) Enforced
**Setup:**
- Daemon: consciousness
- Message: missing query field

**Action:**
- Send message without query

**Expected:**
- Error: "Missing required field: query"
- Message rejected
- Error logged to Turn Trace

#### 1.2 Optional Fields Accepted
**Setup:**
- Daemon: consciousness
- Message: query + metadata + data

**Action:**
- Send message with optional fields

**Expected:**
- Message accepted
- All fields passed to daemon
- Daemon processes successfully

#### 1.3 Daemon-Specific Fields Validated
**Setup:**
- Daemon: consciousness
- Declared optional fields: category, version
- Message: query + category="safety" + version="latest"

**Action:**
- Send message with valid daemon-specific fields

**Expected:**
- Message accepted
- Fields passed to daemon
- Daemon processes successfully

#### 1.4 Invalid Daemon-Specific Fields Rejected
**Setup:**
- Daemon: consciousness
- Declared optional fields: category, version
- Message: query + invalid_field="value"

**Action:**
- Send message with invalid field

**Expected:**
- Error: "Unknown field: invalid_field"
- Message rejected
- Error logged to Turn Trace

#### 1.5 get_instructions() Declares Optional Fields
**Setup:**
- Daemon: context_assembler

**Action:**
- Call get_instructions()

**Expected:**
- Returns object with optional_fields:
  ```
  {
    tags: "Filter by tags",
    importance_threshold: "Minimum importance",
    max_items: "Maximum items"
  }
  ```

---

### 2. Async Semantics Tests

#### 2.1 Queries Return Futures
**Setup:**
- Daemon: consciousness
- Query: "What mandates apply?"

**Action:**
- Call consciousness.query(...)

**Expected:**
- Returns future/promise immediately
- Future not yet resolved
- Caller can do other work

#### 2.2 Multiple Queries Execute in Parallel
**Setup:**
- Daemon 1: consciousness
- Daemon 2: context_assembler
- Daemon 3: episodic_memory
- All queries take ~100ms each

**Action:**
- Query all three daemons
- Measure total time

**Expected:**
- Total time: 100ms to 150ms (P95, indicating parallel execution)
- Not 280ms+ (which would indicate sequential execution)
- All 3 futures resolved successfully (verified: future.is_done() == true for all 3)

#### 2.3 Timeout Enforced
**Setup:**
- Daemon: slow_daemon
- Query: "Process this"
- Timeout: 1000ms
- Daemon takes 5000ms to respond

**Action:**
- Query daemon with timeout
- Wait for result

**Expected:**
- At 1000ms ±50ms: TimeoutError raised (measured time from query start to error)
- Query cancelled (verified: daemon stops processing, no result returned)
- Error logged to Turn Trace with event_type="DaemonTimeout" and timeout_ms=1000

#### 2.4 Results Available After Await
**Setup:**
- Daemon: consciousness
- Query: "What mandates apply?"

**Action:**
- Query daemon
- Await future
- Check result

**Expected:**
- Result available after await
- Result contains expected data
- No errors

#### 2.5 Default Timeout Applied
**Setup:**
- Daemon: consciousness
- Query: "What mandates apply?"
- No explicit timeout specified

**Action:**
- Query daemon
- Check timeout value

**Expected:**
- Default timeout: exactly 5000ms (verified by checking timeout field in query metadata)
- Timeout applied automatically (verified: query will timeout at 5000ms ±50ms if not completed)

---

### 3. Ordering Guarantee Tests

#### 3.1 No Ordering: Queries Processed in Any Order
**Setup:**
- Daemon: context_assembler (query_ordering: "none")
- Query 1: "Get user context"
- Query 2: "Get conversation history"
- Query 3: "Get episodic memory"

**Action:**
- Send all three queries
- Measure processing order

**Expected:**
- Queries processed in any order
- Results may arrive out of order
- All queries complete successfully

#### 3.2 FIFO: Queries Processed in Order Received
**Setup:**
- Daemon: consciousness (query_ordering: "fifo")
- Query 1: "What is mission mandate?" (sent at t=0)
- Query 2: "What is safety mandate?" (sent at t=10ms)
- Query 3: "What is operational mandate?" (sent at t=20ms)

**Action:**
- Send all three queries
- Measure processing order

**Expected:**
- Query 1 processed first
- Query 2 processed second
- Query 3 processed third
- Results arrive in order

#### 3.3 Causal: Dependencies Respected
**Setup:**
- Daemon: error_handler (query_ordering: "causal")
- Query A: "Classify error" (no dependencies)
- Query B: "Decide recovery" (depends on Query A result)

**Action:**
- Send Query B before Query A completes
- Measure execution order

**Expected:**
- Query A processed first
- Query B waits for Query A result
- Query B processed after Query A
- Dependency respected

#### 3.4 Ordering Policy Declared in get_instructions()
**Setup:**
- Daemon: consciousness

**Action:**
- Call get_instructions()

**Expected:**
- Returns object with query_ordering: "fifo"
- Policy clearly declared

#### 3.5 Requesting Specific Ordering
**Setup:**
- Daemon: context_assembler (default: no ordering)
- Query: "Get context"
- Request: ordering="fifo"

**Action:**
- Query with ordering request

**Expected:**
- Query processed with FIFO ordering
- Overrides default "no ordering"
- Ordering applied successfully

#### 3.6 Ordering Logged to Turn Trace
**Setup:**
- Daemon: consciousness
- Query: "What mandates apply?"

**Action:**
- Query daemon
- Check Turn Trace

**Expected:**
- Turn Trace entry:
  ```
  {
    event_type: "DaemonQuery",
    daemon_id: "consciousness",
    query_ordering: "fifo",
    ordering_requested: "fifo",
    timestamp: "..."
  }
  ```

---

### 4. Conflict Resolution Tests

#### 4.1 Optimistic Locking Detects Conflicts
**Setup:**
- Daemon A: reads goal v5
- Daemon B: reads goal v5
- Daemon A: writes goal v6 (success)
- Daemon B: writes goal v6 (conflict)

**Action:**
- Execute concurrent writes

**Expected:**
- Daemon A: success
- Daemon B: ConflictError
- Error includes current state (v6)

#### 4.2 Retries Succeed After Conflict
**Setup:**
- Daemon B: received ConflictError
- Current state: v6

**Action:**
- Daemon B retries: reads v6, writes v7

**Expected:**
- Retry succeeds
- Goal updated to v7
- No error

#### 4.3 Conflicting Decisions Arbitrated by Executor
**Setup:**
- Constraint Checker: "Violates mandate"
- Plan Generator: "Proceed anyway"
- Executor: receives both decisions

**Action:**
- Executor arbitrates

**Expected:**
- Executor queries Consciousness for tiebreaker
- Executor makes final decision
- Decision logged to Turn Trace

#### 4.4 Conflicts Logged to Turn Trace
**Setup:**
- Conflict occurs between daemons

**Action:**
- Check Turn Trace

**Expected:**
- Turn Trace entry:
  ```
  {
    event_type: "DaemonConflict",
    daemon_1: "constraint_checker",
    daemon_2: "plan_generator",
    conflict_type: "decision_conflict",
    resolution: "executor_arbitration",
    decision: "constraint_checker_wins",
    timestamp: "..."
  }
  ```

---

### 5. Deadlock Prevention Tests

#### 5.1 Timeout Prevents Infinite Wait
**Setup:**
- Daemon A: queries Daemon B
- Daemon B: queries Daemon A (circular)
- Timeout: 5000ms

**Action:**
- Execute circular queries
- Wait for timeout

**Expected:**
- At 5000ms ±100ms: TimeoutError raised for at least one daemon
- Deadlock prevented (verified: system does not hang indefinitely)
- Error logged to Turn Trace with event_type="DaemonDeadlock" or "DaemonTimeout"

#### 5.2 Circular Dependencies Detected
**Setup:**
- Daemon A → Daemon B → Daemon C → Daemon A

**Action:**
- Analyze dependency graph

**Expected:**
- Circular dependency detected
- Warning logged
- Recommendation: restructure daemons

#### 5.3 Escalation to Executor on Timeout
**Setup:**
- Daemon A: queries Daemon B
- Timeout occurs

**Action:**
- Timeout triggered

**Expected:**
- Escalate to Executor
- Executor decides:
  - Retry with longer timeout
  - Use default value
  - Fail gracefully

#### 5.4 Acyclic Dependency Graph Maintained
**Setup:**
- All daemons in system

**Action:**
- Analyze dependency graph

**Expected:**
- Graph is acyclic
- No circular dependencies
- Daemons form DAG (directed acyclic graph)

---

### 6. Error Handling Tests

#### 6.1 Query Errors Caught and Handled
**Setup:**
- Daemon: consciousness
- Query: "What mandates apply?"
- Daemon encounters error

**Action:**
- Query daemon
- Daemon throws error

**Expected:**
- Error caught
- Error propagated to caller
- Caller can handle error

#### 6.2 Error Types Classified Correctly
**Setup:**
- Error 1: Query timeout
- Error 2: Daemon unavailable
- Error 3: Invalid query format
- Error 4: Processing error
- Error 5: Conflict error

**Action:**
- Trigger each error type

**Expected:**
- Error 1: TimeoutError
- Error 2: DaemonUnavailableError
- Error 3: InvalidQueryError
- Error 4: ProcessingError
- Error 5: ConflictError

#### 6.3 Errors Logged to Turn Trace
**Setup:**
- Daemon: consciousness
- Query fails

**Action:**
- Query daemon
- Check Turn Trace

**Expected:**
- Turn Trace entry:
  ```
  {
    event_type: "DaemonError",
    daemon_id: "consciousness",
    error_type: "TimeoutError",
    error_message: "Query exceeded 5000ms timeout",
    timestamp: "..."
  }
  ```

#### 6.4 Error Handler Daemon Decides Recovery
**Setup:**
- Daemon query fails
- Error Handler daemon active

**Action:**
- Error occurs
- Error Handler receives notification

**Expected:**
- Error Handler analyzes error
- Error Handler decides recovery:
  - Retry
  - Use alternative daemon
  - Use default value
  - Escalate

---

### 7. Integration Tests

#### 7.1 Query Lifecycle Complete
**Setup:**
- Executor needs decision
- Multiple daemons queried

**Action:**
- Execute full query lifecycle

**Expected:**
- Executor queries daemons
- Daemons process queries
- Results arrive
- Executor uses results
- All logged to Turn Trace

#### 7.2 Synchronization Points Work
**Setup:**
- Context Assembler, Constraint Checker, Plan Generator (parallel)
- Synchronization point before Executor

**Action:**
- Execute parallel queries
- Wait at synchronization point

**Expected:**
- All three daemons complete
- Synchronization point reached
- Executor proceeds

#### 7.3 Turn Trace Records All Queries
**Setup:**
- Multiple daemon queries during turn

**Action:**
- Execute turn
- Check Turn Trace

**Expected:**
- Turn Trace contains entry for each query:
  - daemon_id
  - query text
  - ordering policy
  - status
  - duration
  - timestamp

---

### 8. Edge Cases

#### 8.1 Rapid Sequential Queries to Same Daemon
**Setup:**
- Daemon: consciousness
- 10 rapid queries

**Action:**
- Send 10 queries in quick succession

**Expected:**
- All queries queued
- All processed (FIFO)
- All results returned

#### 8.2 Query with Empty Optional Fields
**Setup:**
- Daemon: consciousness
- Query: "What mandates apply?"
- metadata: {}
- data: {}

**Action:**
- Send query with empty optional fields

**Expected:**
- Query accepted
- Daemon processes successfully

#### 8.3 Very Large Query Payload
**Setup:**
- Daemon: context_assembler
- Query: "Get context"
- data: 10MB of context

**Action:**
- Send large query

**Expected:**
- Query accepted (or rejected with clear error)
- If accepted: processed successfully
- If rejected: error logged

#### 8.4 Query During Turn Commit
**Setup:**
- Turn ending
- Commit phase starting
- Daemon query arrives

**Action:**
- Query daemon during commit

**Expected:**
- Query queued or rejected
- Behavior defined and logged
- No data corruption

---

### 9. High Load Concurrent Scenarios

#### 9.1 High Load Concurrent Turn Execution (100 Turns)
**Objective:** Verify system stability and performance under high concurrent load with multiple turns executing in parallel, each invoking multiple daemons.

**Setup:**
- System configured to handle concurrent turns (thread pool or async executor)
- 100 turns queued for execution (turn_001 through turn_100)
- Each turn configured to invoke 3-5 daemons:
  - Consciousness (query for mandates)
  - Context Assembler (retrieve context)
  - Episodic Memory (query relevant memories)
  - Plan Generator (generate plan)
  - Constraint Checker (validate constraints)
- Each daemon query configured with:
  - Default timeout: 5000ms
  - Query ordering: daemon-specific (consciousness: FIFO, others: none)
- Single-turn baseline latency measured: ~150ms (P95)
- Expected concurrent P95 latency threshold: < 300ms (< 2x baseline)

**Actions:**
1. Measure single-turn baseline:
   - Execute 1 turn with 3-5 daemon invocations
   - Measure P95 latency: ~150ms
2. Queue 100 turns for concurrent execution at t=0
3. System executes all turns in parallel (limited by available resources)
4. Each turn:
   - Starts
   - Queries 3-5 daemons in parallel
   - Waits for all daemon responses
   - Processes results
   - Commits updates
   - Completes
5. Measure metrics:
   - Total execution time
   - Per-turn latency (P50, P95, P99)
   - Daemon pool utilization
   - Resource consumption (CPU, memory)
   - Error rates
6. Verify all turns complete successfully

**Expected Outcome:**
- All 100 turns complete successfully:
  - 100 turn traces generated
  - Each trace has outcome: "success"
  - 0 turns failed or timed out
- No daemon deadlocks:
  - No circular waits detected
  - All daemon queries resolve (no infinite waits)
  - Daemon pool remains healthy
- No resource starvation:
  - All daemons receive queries and respond
  - No daemons blocked indefinitely
  - Thread/worker pool operates within capacity
- Performance within acceptable bounds:
  - P95 turn latency: < 300ms (< 2x baseline of 150ms)
  - P99 turn latency: < 500ms
  - P50 turn latency: < 200ms
  - Total execution time: < 10 seconds for all 100 turns

**Verification:**
- Turn execution metrics:
  - Total turns executed: 100
  - Successful turns: 100 (100%)
  - Failed turns: 0 (0%)
  - Timed out turns: 0 (0%)
- Turn Trace analysis (for all 100 turns):
  - Each trace contains:
    - turn_id: "turn_001" through "turn_100"
    - outcome: "success"
    - start_timestamp: <ISO8601>
    - end_timestamp: <ISO8601>
    - duration_ms: <value>
    - daemons_invoked: 3-5 (array of daemon IDs)
    - daemon_query_count: 3-5 total queries
    - errors: [] (empty array)
- Latency distribution:
  - P50 latency: < 200ms (verified: sort all durations, take median)
  - P95 latency: < 300ms (verified: sort all durations, take 95th percentile)
  - P99 latency: < 500ms (verified: sort all durations, take 99th percentile)
  - Max latency: < 1000ms
  - Min latency: > 50ms (sanity check)
- Daemon pool metrics:
  - Total daemon invocations: 300-500 (100 turns × 3-5 daemons)
  - Successful daemon queries: 100% (300-500 queries)
  - Failed daemon queries: 0%
  - Timed out daemon queries: 0%
  - Average daemon response time: < 100ms
  - Daemon pool utilization: 60-90% (healthy range)
  - No deadlock events logged
  - No resource exhaustion errors
- Resource utilization:
  - CPU usage: < 90% (sustained)
  - Memory usage: < 2GB increase from baseline
  - Thread pool: No thread exhaustion
  - Connection pool: No connection exhaustion
- Concurrency safety:
  - No race conditions detected
  - No data corruption (all final states valid)
  - All audit trails complete and ordered
  - All version numbers sequential and consistent
- Error logs:
  - 0 DaemonDeadlock events
  - 0 ResourceStarvation events
  - 0 ThreadPoolExhaustion events
  - 0 ConflictError events (or acceptable rate < 5%)

**Measurable Acceptance Criteria:**
- 100/100 turns complete successfully (100% success rate)
- P95 turn latency < 300ms (threshold: < 2x single-turn baseline of 150ms)
- 0 daemon deadlocks detected (verified in logs and metrics)
- 0 resource starvation events (verified in logs)
- 300-500 daemon queries executed successfully (100% success rate)
- Daemon pool utilization: 60-90% (healthy range, not overloaded or underutilized)
- 0 timeout errors (all queries complete within 5000ms)
- Total execution time < 10 seconds (all 100 turns)
- All 100 turn traces retrievable and valid
- Memory usage increase < 2GB from baseline
- CPU usage < 90% sustained throughout test

---

