# Tool Error Handling Specification

**Status:** Specification v1.0  
**Date:** 2025-11-05  
**Technology:** Agnostic

## Overview

Tool error handling is a critical subsystem that ensures Si can recover from tool failures gracefully. This specification defines:
- Error classification (transient vs. permanent)
- Retry policies with exponential backoff
- Circuit breaker patterns
- Timeout semantics at tool and turn levels
- Cascading failure handling
- Integration with Error Handler daemon

---

## 1. Error Classification

All tool errors are classified into two categories:

### 1.1 Transient Errors (Retryable)

Errors that are temporary and likely to succeed on retry:
- **Network timeouts**: Tool didn't respond within timeout
- **Rate limit exceeded**: API rate limit hit, but will reset
- **Temporary service unavailable**: Service temporarily down (503, 429)
- **Connection reset**: Network connection dropped mid-request
- **Partial timeout**: Tool returned partial results before timeout

**Retry Strategy:** Retry with exponential backoff

**Example:**
```
Tool: flight_search
Error: "Connection timeout after 30s"
Classification: Transient (network timeout)
Action: Retry with backoff
```

### 1.2 Permanent Errors (Non-Retryable)

Errors that indicate a fundamental problem unlikely to resolve on retry:
- **Invalid input**: Tool received malformed parameters
- **Authentication failure**: Invalid credentials or permissions
- **Not found**: Resource doesn't exist (404)
- **Unsupported operation**: Tool doesn't support requested operation
- **Schema validation failure**: Response doesn't match expected schema
- **Tool not found**: Tool doesn't exist or is disabled

**Retry Strategy:** Do not retry; escalate or use alternative

**Example:**
```
Tool: flight_search
Error: "Invalid airport code: XYZ"
Classification: Permanent (invalid input)
Action: Escalate to user or use alternative tool
```

### 1.3 Error Classification Algorithm

```
Tool returns error
  ↓
1. Check error code/type
   - Network errors (timeout, connection reset) → Transient
   - Rate limit (429, 503) → Transient
   - Invalid input (400, validation) → Permanent
   - Auth failure (401, 403) → Permanent
   - Not found (404) → Permanent
   - Unknown → Classify as Transient (safer default)
  ↓
2. Check tool manifest
   - Tool may override classification
   - Example: "treat 503 as permanent for this tool"
  ↓
3. Return classification
```

---

## 2. Retry Policies

### 2.1 Default Retry Policy (Exponential Backoff)

All tools use exponential backoff by default:

```
Attempt 1: Immediate (0ms delay)
Attempt 2: 100ms + jitter (±10%)
Attempt 3: 200ms + jitter (±10%)
Attempt 4: 400ms + jitter (±10%)
Attempt 5: 800ms + jitter (±10%)
Max attempts: 5 (configurable)
Max total retry time: 2 seconds (configurable)
```

**Jitter:** Random ±10% variation prevents thundering herd when multiple tools retry simultaneously.

**Example:**
```
Attempt 1: 0ms
Attempt 2: 100ms + 8ms jitter = 108ms
Attempt 3: 200ms + 15ms jitter = 215ms
Attempt 4: 400ms + 22ms jitter = 422ms
Attempt 5: 800ms + 18ms jitter = 818ms
Total: 1563ms
```

### 2.2 Tool-Specific Retry Overrides

Tools can override default retry policy in their manifest:

```yaml
tool:
  id: flight_search
  retry_policy:
    strategy: exponential_backoff
    initial_delay_ms: 50        # Override default 100ms
    max_delay_ms: 2000          # Override default 800ms
    multiplier: 2.0             # Override default 2.0
    jitter_percent: 15          # Override default 10%
    max_attempts: 3             # Override default 5
    max_total_time_ms: 5000     # Override default 2000ms
```

### 2.3 Retry Decision Logic

```
Tool fails with transient error
  ↓
1. Check retry count
   - If retry_count < max_attempts: proceed to step 2
   - If retry_count >= max_attempts: go to Circuit Breaker
  ↓
2. Check total retry time
   - If total_time < max_total_time: proceed to step 3
   - If total_time >= max_total_time: go to Circuit Breaker
  ↓
3. Calculate backoff delay
   - delay = initial_delay * (multiplier ^ (retry_count - 1))
   - delay = min(delay, max_delay)
   - delay += random jitter
  ↓
4. Wait and retry
   - Sleep for delay milliseconds
   - Retry tool invocation
```

---

## 3. Circuit Breaker Pattern

### 3.1 Circuit Breaker States

```
CLOSED (normal operation)
  ↓ (after N failures)
OPEN (stop retrying)
  ↓ (after timeout)
HALF_OPEN (test if recovered)
  ↓ (success or failure)
CLOSED or OPEN
```

### 3.2 Circuit Breaker Configuration

```
failure_threshold: 5           # Open after 5 consecutive failures
success_threshold: 2           # Close after 2 consecutive successes
timeout_ms: 30000              # Wait 30s before trying again
```

### 3.3 Circuit Breaker Logic

```
Tool fails
  ↓
1. Increment failure counter
   - failure_count++
  ↓
2. Check if threshold exceeded
   - If failure_count >= failure_threshold:
       Set circuit to OPEN
       Record: "Circuit breaker opened for tool_id"
       Return error immediately (no retry)
   - Else: Proceed with retry
  ↓
3. If circuit is OPEN
   - Check if timeout elapsed
   - If timeout elapsed: Set circuit to HALF_OPEN, try one retry
   - If timeout not elapsed: Return error immediately
  ↓
4. If circuit is HALF_OPEN
   - Allow one retry attempt
   - If succeeds: Set circuit to CLOSED, reset failure_count
   - If fails: Set circuit to OPEN, restart timeout
```

### 3.4 Error Classification + Circuit Breaker

```
Tool fails
  ↓
1. Classify error (transient vs permanent)
  ↓
2. If permanent error:
   - Do NOT retry
   - Do NOT increment circuit breaker counter
   - Escalate immediately
  ↓
3. If transient error:
   - Check circuit breaker state
   - If CLOSED: Retry with backoff
   - If OPEN: Return error (circuit open)
   - If HALF_OPEN: Allow one retry
```

---

## 4. Timeout Semantics

### 4.1 Per-Tool Timeout

Each tool has a maximum execution time:

```yaml
tool:
  id: flight_search
  timeout_ms: 30000             # 30 second timeout
```

**Behavior:** Hard stop
- If tool exceeds timeout, immediately terminate execution
- Treat as transient error (timeout)
- Retry with backoff (up to circuit breaker limits)
- If all retries exhausted, escalate

**Example:**
```
Tool starts at t=0
Tool running at t=25s (normal)
Tool running at t=30s (timeout reached)
Tool terminated immediately
Error: "Tool timeout after 30s"
Classification: Transient
Action: Retry with backoff
```

### 4.2 Per-Turn Timeout

Each turn has a maximum execution time:

```
Turn starts at t=0
Turn timeout: 5 minutes (300s)
```

**Behavior:** Graceful degradation
- If turn exceeds timeout, do NOT terminate running tools
- Instead, collect partial results from completed tools
- Skip remaining tools
- Continue with available results
- Inform user of timeout

**Example:**
```
Turn starts at t=0
Tool 1 completes at t=50s (success)
Tool 2 running at t=100s (normal)
Tool 3 running at t=100s (normal)
Turn timeout reached at t=300s
Action: Collect results from Tool 1, skip Tools 2 & 3
Response: "Completed search for flights, but hotel search timed out"
```

### 4.3 Timeout Interaction

```
Per-tool timeout: 30s (hard stop)
Per-turn timeout: 300s (graceful degradation)

Scenario 1: Tool exceeds per-tool timeout
  → Tool terminated, treated as transient error, retry
  → If retries succeed before per-turn timeout: continue
  → If retries fail before per-turn timeout: escalate
  → If per-turn timeout reached: collect partial results

Scenario 2: Turn exceeds per-turn timeout
  → Collect partial results from completed tools
  → Skip remaining tools
  → Continue with available results
```

---

## 5. Cascading Failure Handling

### 5.1 Dependency-Based Cascading

When a tool fails and other tools depend on its output:

```
Tool A fails
  ↓
Tool B depends on Tool A output
  ↓
1. Classify Tool A failure
   - If transient: retry Tool A
   - If permanent: proceed to step 2
  ↓
2. Check if Tool B is optional
   - If optional: skip Tool B, continue
   - If required: proceed to step 3
  ↓
3. Check for default value
   - If default available: use default, continue
   - If no default: escalate
```

### 5.2 Cascading Error Propagation

```
Tool A fails (permanent error)
  ↓
Tool B depends on Tool A
  ↓
Tool C depends on Tool B
  ↓
Decision:
  - Skip Tool B (cannot run without Tool A)
  - Skip Tool C (cannot run without Tool B)
  - Continue with remaining independent tools
  - Escalate if critical path blocked
```

---

## 6. Integration with Error Handler Daemon

### 6.1 Error Flow

```
Tool execution fails
  ↓
Executor detects error
  ↓
Executor notifies Error Handler daemon
  ↓
Error Handler:
  1. Receives error details
  2. Classifies error (transient vs permanent)
  3. Checks circuit breaker state
  4. Decides recovery strategy:
     - Retry (if transient + circuit closed)
     - Alternative tool (if available)
     - Partial success (if acceptable)
     - Escalate (if required)
     - Fail gracefully (if no recovery)
  ↓
Error Handler executes recovery
  ↓
Executor continues or aborts based on outcome
```

### 6.2 Error Handler Decision Inputs

Error Handler uses:
- **Error classification** (transient vs permanent)
- **Circuit breaker state** (closed, open, half-open)
- **Retry count** (how many times already retried)
- **Turn timeout** (how much time remaining)
- **Alternative tools** (available alternatives)
- **Consciousness** (error handling policies)
- **Episodic memory** (similar errors and outcomes)

---

## 7. Test Conditions

### 7.1 Error Classification Tests

- ✅ Network timeout classified as transient
- ✅ Invalid input classified as permanent
- ✅ Rate limit classified as transient
- ✅ Auth failure classified as permanent
- ✅ Tool manifest can override classification

### 7.2 Retry Policy Tests

- ✅ Exponential backoff delays correct (100ms, 200ms, 400ms, 800ms)
- ✅ Jitter applied correctly (±10%)
- ✅ Max attempts respected (default 5)
- ✅ Max total time respected (default 2s)
- ✅ Tool-specific overrides work

### 7.3 Circuit Breaker Tests

- ✅ Circuit opens after N failures
- ✅ Circuit half-opens after timeout
- ✅ Circuit closes after success in half-open state
- ✅ Permanent errors don't increment circuit counter
- ✅ Circuit state persists across retries

### 7.4 Timeout Tests

- ✅ Per-tool timeout terminates tool
- ✅ Per-turn timeout collects partial results
- ✅ Timeout treated as transient error
- ✅ Timeouts recorded in Turn Trace

### 7.5 Cascading Failure Tests

- ✅ Dependent tools skipped when dependency fails
- ✅ Optional tools skipped, required tools escalate
- ✅ Default values used when available
- ✅ Independent tools continue despite failures

### 7.6 Integration Tests

- ✅ Error Handler receives error notifications
- ✅ Error Handler decides recovery strategy
- ✅ Retry succeeds after transient error
- ✅ Alternative tool used when primary fails
- ✅ Escalation triggered for permanent errors
- ✅ Turn Trace records all error handling decisions

---

## 8. Observability and Monitoring

### 8.1 Error Metrics

Track per tool:
- `error_count`: Total errors
- `transient_error_count`: Transient errors
- `permanent_error_count`: Permanent errors
- `retry_count`: Total retries
- `retry_success_rate`: % of retries that succeeded
- `circuit_breaker_opens`: Times circuit opened
- `timeout_count`: Times tool timed out

### 8.2 Turn Trace Recording

Every error handling decision recorded:
```
{
  event_type: "ToolError",
  tool_id: "flight_search",
  error: "Connection timeout",
  classification: "transient",
  circuit_breaker_state: "closed",
  retry_count: 2,
  decision: "retry",
  timestamp: "2025-11-05T10:30:45Z"
}
```

---

## 9. Relationship to Other Systems

- **Executor**: Detects tool errors, notifies Error Handler
- **Error Handler Daemon**: Decides recovery strategy
- **Turn Trace**: Records all error handling decisions
- **Consciousness**: Provides error handling policies
- **Episodic Memory**: Stores learned error recovery patterns

