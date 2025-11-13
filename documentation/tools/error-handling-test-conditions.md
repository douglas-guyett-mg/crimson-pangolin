# Tool Error Handling - Test Conditions

**Status:** v1.0 (Stable)
**Last Updated:** 2025-11-07

## Test Categories

### 1. Error Classification Tests

#### 1.1 Network Timeout Classified as Transient
**Setup:**
- Tool: flight_search
- Error: "Connection timeout after 30s"

**Action:**
- Classify error

**Expected:**
- Classification: Transient
- Retry: Yes
- Reason: Network timeouts are temporary

#### 1.2 Invalid Input Classified as Permanent
**Setup:**
- Tool: flight_search
- Error: "Invalid airport code: XYZ"

**Action:**
- Classify error

**Expected:**
- Classification: Permanent
- Retry: No
- Reason: Invalid input won't change on retry

#### 1.3 Rate Limit Classified as Transient
**Setup:**
- Tool: flight_search
- Error: "Rate limit exceeded (429)"

**Action:**
- Classify error

**Expected:**
- Classification: Transient
- Retry: Yes
- Reason: Rate limits reset over time

#### 1.4 Auth Failure Classified as Permanent
**Setup:**
- Tool: flight_search
- Error: "Authentication failed (401)"

**Action:**
- Classify error

**Expected:**
- Classification: Permanent
- Retry: No
- Reason: Auth won't change on retry

#### 1.5 Tool Manifest Can Override Classification
**Setup:**
- Tool: custom_api
- Manifest specifies: "treat 503 as permanent"
- Error: "Service unavailable (503)"

**Action:**
- Classify error using manifest override

**Expected:**
- Classification: Permanent (overridden)
- Retry: No
- Reason: Tool manifest override respected

---

### 2. Retry Policy Tests

#### 2.1 Exponential Backoff Delays Correct
**Setup:**
- Tool: flight_search
- Default retry policy
- Transient error

**Actions:**
- Attempt 1: Fails immediately
- Attempt 2: Wait 100ms ± jitter, retry
- Attempt 3: Wait 200ms ± jitter, retry
- Attempt 4: Wait 400ms ± jitter, retry
- Attempt 5: Wait 800ms ± jitter, retry

**Expected:**
- Delays: 100ms ±10ms, 200ms ±20ms, 400ms ±40ms, 800ms ±80ms (measured from attempt start to next attempt start)
- Total time: 1500ms ±150ms (P95 within this range across 20 runs)
- All 5 attempts executed (verified: attempt counter = 5)

#### 2.2 Jitter Applied Correctly
**Setup:**
- Tool: flight_search
- Retry delay: 100ms
- Jitter: ±10%

**Actions:**
- Execute 10 retries
- Measure actual delays

**Expected:**
- 100% of delays fall within range [90ms, 110ms]
- At least 80% of delays are unique values (allowing for some random collision)
- Mean delay: 100ms ±5ms across 10 retries

#### 2.3 Max Attempts Respected
**Setup:**
- Tool: flight_search
- Max attempts: 5
- All attempts fail with transient error

**Actions:**
- Attempt 1-5: All fail

**Expected:**
- After attempt 5: Stop retrying
- Return error to caller
- Do not attempt 6

#### 2.4 Max Total Time Respected
**Setup:**
- Tool: flight_search
- Max total retry time: 2000ms
- Each attempt takes 600ms

**Actions:**
- Attempt 1: 0ms (immediate)
- Attempt 2: 600ms (total: 600ms)
- Attempt 3: 600ms (total: 1200ms)
- Attempt 4: 600ms (total: 1800ms)
- Attempt 5: Would exceed 2000ms

**Expected:**
- Attempt 5 not executed
- Stop after attempt 4
- Total time: ~1800ms

#### 2.5 Tool-Specific Overrides Work
**Setup:**
- Tool: custom_api
- Manifest specifies:
  - initial_delay_ms: 50
  - max_delay_ms: 2000
  - max_attempts: 3

**Actions:**
- Attempt 1: Fails
- Attempt 2: Wait 50ms, retry
- Attempt 3: Wait 100ms, retry

**Expected:**
- Delays: 50ms, 100ms (not default 100ms, 200ms)
- Max attempts: 3 (not default 5)
- Overrides respected

---

### 3. Circuit Breaker Tests

#### 3.1 Circuit Opens After N Failures
**Setup:**
- Tool: flight_search
- Failure threshold: 5
- Circuit initially CLOSED

**Actions:**
- Failure 1: Circuit remains CLOSED
- Failure 2: Circuit remains CLOSED
- Failure 3: Circuit remains CLOSED
- Failure 4: Circuit remains CLOSED
- Failure 5: Circuit opens

**Expected:**
- After 5 failures: Circuit state = OPEN
- Attempt 6: Return error immediately (no retry)
- Turn Trace records: "Circuit breaker opened for flight_search"

#### 3.2 Circuit Half-Opens After Timeout
**Setup:**
- Tool: flight_search
- Circuit: OPEN
- Timeout: 30s
- Current time: t=0

**Actions:**
- At t=0: Circuit OPEN, return error immediately
- At t=15s: Circuit still OPEN, return error immediately
- At t=30s: Timeout elapsed, circuit transitions to HALF_OPEN
- At t=30s: Allow one retry attempt

**Expected:**
- At t=30s ±100ms: Circuit state = HALF_OPEN (verified by state query)
- Exactly 1 retry allowed (verified: subsequent requests blocked until first retry completes)
- If retry succeeds: Circuit → CLOSED within 50ms
- If retry fails: Circuit → OPEN within 50ms and timeout timer resets to t=0

#### 3.3 Circuit Closes After Success in Half-Open
**Setup:**
- Tool: flight_search
- Circuit: HALF_OPEN
- One retry allowed

**Actions:**
- Retry attempt: Succeeds

**Expected:**
- Circuit state: CLOSED
- Failure counter: Reset to 0
- Normal operation resumes

#### 3.4 Permanent Errors Don't Increment Circuit Counter
**Setup:**
- Tool: flight_search
- Circuit: CLOSED
- Failure threshold: 5

**Actions:**
- Permanent error 1: Invalid input
- Permanent error 2: Auth failure
- Permanent error 3: Not found
- Permanent error 4: Invalid input
- Permanent error 5: Auth failure

**Expected:**
- Circuit remains CLOSED
- Failure counter: 0 (not incremented)
- Reason: Permanent errors don't indicate service issues

#### 3.5 Transient Errors Increment Circuit Counter
**Setup:**
- Tool: flight_search
- Circuit: CLOSED
- Failure threshold: 5

**Actions:**
- Transient error 1: Timeout
- Transient error 2: Rate limit
- Transient error 3: Connection reset
- Transient error 4: Timeout
- Transient error 5: Rate limit

**Expected:**
- After error 5: Circuit opens
- Failure counter: 5
- Circuit state: OPEN

---

### 4. Timeout Tests

#### 4.1 Per-Tool Timeout Terminates Tool
**Setup:**
- Tool: flight_search
- Timeout: 30s
- Tool starts at t=0

**Actions:**
- Tool running at t=25s (normal)
- Tool running at t=30s (timeout reached)

**Expected:**
- Tool terminated immediately at t=30s
- Error: "Tool timeout after 30s"
- Classification: Transient
- Retry: Yes (with backoff)

#### 4.2 Per-Turn Timeout Collects Partial Results
**Setup:**
- Turn timeout: 300s
- Tool 1: flight_search (completes at t=50s)
- Tool 2: hotel_search (running at t=300s)
- Tool 3: activity_search (running at t=300s)

**Actions:**
- Turn starts at t=0
- Tool 1 completes at t=50s (success)
- Tool 2 running at t=100s (normal)
- Tool 3 running at t=100s (normal)
- Turn timeout reached at t=300s

**Expected:**
- Collect results from Tool 1 (success)
- Skip Tools 2 & 3 (not completed)
- Continue with available results
- Response: "Completed flight search, but hotel search timed out"

#### 4.3 Timeout Treated as Transient Error
**Setup:**
- Tool: flight_search
- Timeout: 30s
- Tool exceeds timeout

**Actions:**
- Tool times out
- Classify error

**Expected:**
- Classification: Transient
- Retry: Yes
- Reason: Timeout is temporary

#### 4.4 Timeouts Recorded in Turn Trace
**Setup:**
- Tool: flight_search
- Timeout: 30s
- Tool exceeds timeout

**Actions:**
- Tool times out
- Check Turn Trace

**Expected:**
- Turn Trace entry:
  ```
  {
    event_type: "ToolTimeout",
    tool_id: "flight_search",
    timeout_ms: 30000,
    timestamp: "2025-11-05T10:30:45Z"
  }
  ```

---

### 5. Cascading Failure Tests

#### 5.1 Dependent Tools Skipped When Dependency Fails
**Setup:**
- Tool A: flight_search (fails with permanent error)
- Tool B: compare_prices (depends on Tool A output)
- Tool C: create_itinerary (depends on Tool B output)

**Actions:**
- Tool A fails (permanent error)
- Tool B cannot run (missing input)
- Tool C cannot run (missing input)

**Expected:**
- Tool A: Failed
- Tool B: Skipped (dependency failed)
- Tool C: Skipped (dependency failed)
- Turn continues with other independent tools

#### 5.2 Optional Tools Skipped, Required Tools Escalate
**Setup:**
- Tool A: flight_search (fails)
- Tool B: compare_prices (optional, depends on Tool A)
- Tool C: create_itinerary (required, depends on Tool A)

**Actions:**
- Tool A fails
- Evaluate Tool B (optional)
- Evaluate Tool C (required)

**Expected:**
- Tool B: Skipped (optional)
- Tool C: Escalate (required but missing input)
- Turn Trace records: "Required tool skipped due to dependency failure"

#### 5.3 Default Values Used When Available
**Setup:**
- Tool A: flight_search (fails)
- Tool B: compare_prices (depends on Tool A, has default value)

**Actions:**
- Tool A fails
- Tool B needs Tool A output
- Default value available

**Expected:**
- Tool B: Executes with default value
- Turn continues normally
- Turn Trace records: "Used default value for Tool B"

#### 5.4 Independent Tools Continue Despite Failures
**Setup:**
- Tool A: flight_search (fails)
- Tool B: hotel_search (independent)
- Tool C: activity_search (independent)

**Actions:**
- Tool A fails
- Tool B executes (independent)
- Tool C executes (independent)

**Expected:**
- Tool A: Failed
- Tool B: Succeeds
- Tool C: Succeeds
- Turn continues with results from B and C

---

### 6. Integration Tests

#### 6.1 Error Handler Receives Error Notifications
**Setup:**
- Tool: flight_search
- Tool fails with transient error
- Error Handler daemon active

**Actions:**
- Tool fails
- Executor detects error
- Executor notifies Error Handler

**Expected:**
- Error Handler receives notification
- Error Handler has access to:
  - Error details
  - Tool ID
  - Attempt count
  - Turn context

#### 6.2 Error Handler Decides Recovery Strategy
**Setup:**
- Tool: flight_search
- Error: Transient (timeout)
- Circuit: CLOSED
- Retry count: 0

**Actions:**
- Error Handler analyzes error
- Error Handler decides strategy

**Expected:**
- Decision: Retry
- Reason: Transient error + circuit closed + retries available

#### 6.3 Retry Succeeds After Transient Error
**Setup:**
- Tool: flight_search
- Attempt 1: Fails with timeout
- Attempt 2: Succeeds

**Actions:**
- Attempt 1: Fails
- Error Handler decides: Retry
- Attempt 2: Succeeds

**Expected:**
- Final result: Success
- Turn Trace records: "Tool succeeded on retry 2"

#### 6.4 Alternative Tool Used When Primary Fails
**Setup:**
- Primary tool: flight_search (fails permanently)
- Alternative tool: flight_search_backup (available)

**Actions:**
- Primary tool fails
- Error Handler decides: Use alternative
- Alternative tool executes

**Expected:**
- Primary tool: Failed
- Alternative tool: Executed
- Result: Success (if alternative succeeds)
- Turn Trace records: "Used alternative tool"

#### 6.5 Escalation Triggered for Permanent Errors
**Setup:**
- Tool: flight_search
- Error: Permanent (invalid input)
- Circuit: CLOSED

**Actions:**
- Tool fails
- Error Handler analyzes error
- Error Handler decides strategy

**Expected:**
- Decision: Escalate
- Reason: Permanent error (no retry)
- User notified

#### 6.6 Turn Trace Records All Error Handling Decisions
**Setup:**
- Multiple tools with various errors
- Error Handler active

**Actions:**
- Tools fail
- Error Handler handles errors
- Check Turn Trace

**Expected:**
- Turn Trace contains entries for:
  - Each error
  - Error classification
  - Circuit breaker state
  - Recovery decision
  - Outcome (success/failure)

---

### 7. Edge Cases

#### 7.1 Rapid Sequential Failures
**Setup:**
- Tool: flight_search
- 10 rapid failures (all transient)

**Actions:**
- Failure 1-5: Retry with backoff
- Failure 6-10: Circuit opens after 5

**Expected:**
- Failures 1-5: Retried
- Failures 6-10: Circuit open, no retry
- Circuit state: OPEN

#### 7.2 Mixed Transient and Permanent Errors
**Setup:**
- Tool: flight_search
- Error 1: Transient (timeout)
- Error 2: Permanent (invalid input)
- Error 3: Transient (rate limit)

**Actions:**
- Error 1: Retry
- Error 2: Escalate (don't retry)
- Error 3: Retry

**Expected:**
- Error 1: Retried
- Error 2: Escalated
- Error 3: Retried
- Circuit counter: 1 (only transient errors count)

#### 7.3 Timeout During Retry
**Setup:**
- Tool: flight_search
- Per-tool timeout: 30s
- Per-turn timeout: 60s
- Attempt 1: Fails at t=25s
- Attempt 2: Starts at t=25s, exceeds per-tool timeout at t=55s

**Actions:**
- Attempt 1: Fails (transient)
- Retry scheduled
- Attempt 2: Starts
- Attempt 2: Exceeds per-tool timeout

**Expected:**
- Attempt 2: Terminated at t=55s
- Treated as transient error
- Retry scheduled (if retries available)
- Per-turn timeout still active (60s)

#### 7.4 Per-Turn Timeout During Retry
**Setup:**
- Tool: flight_search
- Per-tool timeout: 30s
- Per-turn timeout: 60s
- Attempt 1: Fails at t=25s
- Attempt 2: Starts at t=25s, per-turn timeout at t=60s

**Actions:**
- Attempt 1: Fails
- Retry scheduled
- Attempt 2: Starts at t=25s
- Per-turn timeout reached at t=60s

**Expected:**
- Attempt 2: Interrupted at t=60s
- Collect partial results
- Do not retry
- Continue with available results

