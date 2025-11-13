# Daemon Interface — Test Conditions

**Status:** Specification v1.0  
**Date:** 2025-11-01

---

## 1. Core Interface Tests

### 1.1 get_purpose() Returns String
**Test:** All daemons implement get_purpose() and return a string
**Acceptance Criteria:**
- `daemon.get_purpose()` returns a non-empty string
- String is human-readable (not binary or encoded)
- String describes what the daemon does
- String is consistent across multiple calls

### 1.2 get_instructions() Returns String
**Test:** All daemons implement get_instructions() and return a string
**Acceptance Criteria:**
- `daemon.get_instructions()` returns a non-empty string
- String is human-readable
- String describes how the daemon operates
- String is consistent across multiple calls

### 1.3 Purpose and Instructions Are Distinct
**Test:** Purpose and instructions are different
**Acceptance Criteria:**
- `get_purpose()` and `get_instructions()` return different strings
- Purpose is 50-200 characters, contains exactly 1 complete sentence describing primary function
- Instructions are more detailed (multiple sentences/paragraphs)

---

## 2. Async Operation Tests

### 2.1 Operations Return Futures
**Test:** Daemon operations return futures/promises, not direct results
**Setup:** Call a daemon operation (e.g., memory daemon's `remember()`)
**Acceptance Criteria:**
- Operation returns immediately (doesn't block)
- Return value is a future/promise object
- Future can be awaited later
- Caller can do other work while waiting

### 2.2 Multiple Operations Can Run in Parallel
**Test:** Multiple daemon operations can execute concurrently
**Setup:**
- Call operation A on daemon (returns future A)
- Call operation B on daemon (returns future B)
- Await both futures
**Acceptance Criteria:**
- Both operations execute (not sequentially)
- Total execution time <= max(time_A, time_B) + 50ms overhead, operations overlap by >= 80% of shorter operation duration
- Results are correct for both operations

### 2.3 Operations Complete Successfully
**Test:** Async operations eventually complete
**Setup:** Call daemon operation and await result
**Acceptance Criteria:**
- Future resolves with a result
- Result is of expected type
- Result is consistent with operation semantics

### 2.4 Timeout Behavior
**Test:** Operations timeout if they take too long
**Setup:** Call operation with timeout set to 100ms
**Acceptance Criteria:**
- If operation completes within timeout: returns result
- If operation exceeds timeout: returns timeout error
- Timeout error includes: {timeout_ms: <value>, suggested_retry_delay_ms: <2x timeout>, alternative_operation: <name or null>}
- Operation can be retried after timeout

### 2.5 Error Handling
**Test:** Errors are returned asynchronously
**Setup:** Call operation that will fail (e.g., invalid parameters)
**Acceptance Criteria:**
- Future resolves with error (not exception)
- Error includes: error code, message, context
- Caller can handle error without crashing
- Other operations continue normally

---

## 3. Introspection Tests

### 3.1 Daemon Can Describe Itself
**Test:** Calling get_purpose() and get_instructions() provides useful information
**Setup:** Call both methods on a daemon
**Acceptance Criteria:**
- Purpose clearly states what daemon does
- Instructions clearly state how daemon operates
- Purpose + Instructions combined include: (1) primary function, (2) input/output format, (3) typical use case example, (4) constraints/limitations - verified by 3 independent developers achieving 100% functional understanding

### 3.2 Daemon Status Available
**Test:** Daemon status can be queried
**Setup:** Call `daemon.get_status()`
**Acceptance Criteria:**
- Returns one of: "ready", "busy", "error", "shutdown"
- Status reflects actual daemon state
- Status updates as daemon processes operations

### 3.3 Interface Implementation Queryable
**Test:** Daemon can report which interfaces it implements
**Setup:** Call `daemon.implements("MemoryInterface")`
**Acceptance Criteria:**
- Returns true if daemon implements that interface
- Returns false if daemon doesn't implement that interface
- Works for all known interfaces

---

## 4. Lifecycle Tests

### 4.1 Daemon Initialization
**Test:** Daemon initializes correctly
**Setup:** Create new daemon instance
**Acceptance Criteria:**
- Daemon is created without errors
- `get_status()` returns "ready"
- `get_purpose()` and `get_instructions()` are available
- Daemon is ready to receive operations

### 4.2 Daemon Shutdown
**Test:** Daemon shuts down cleanly
**Setup:** Create daemon, perform operations, then shutdown
**Acceptance Criteria:**
- In-flight operations given 30-second grace period, operations exceeding grace period terminated, resources cleaned up within 5 seconds
- `get_status()` returns "shutdown"
- New operations are rejected after shutdown
- Resources are cleaned up

### 4.3 In-Flight Operations During Shutdown
**Test:** In-flight operations complete during shutdown
**Setup:**
- Start long-running operation
- Initiate shutdown while operation is running
**Acceptance Criteria:**
- In-flight operations given 30-second grace period, operations exceeding grace period terminated, resources cleaned up within 5 seconds
- Operation result is returned correctly
- Shutdown waits for operation (up to 30 seconds)
- If 30-second grace period expires, operation is cancelled

---

## 5. Consistency Tests

### 5.1 Purpose Consistency
**Test:** get_purpose() returns same value across calls
**Setup:** Call get_purpose() multiple times
**Acceptance Criteria:**
- All calls return identical string
- No variation or randomness

### 5.2 Instructions Consistency
**Test:** get_instructions() returns same value across calls
**Setup:** Call get_instructions() multiple times
**Acceptance Criteria:**
- All calls return identical string
- No variation or randomness

### 5.3 Status Consistency
**Test:** get_status() reflects actual daemon state
**Setup:** Perform operations and check status
**Acceptance Criteria:**
- Status is "ready" when idle
- Status is "busy" when processing operations
- Status transitions are correct

---

## 6. Integration Tests

### 6.1 Multiple Daemons Coexist
**Test:** Multiple daemon instances can run simultaneously
**Setup:** Create 3 different daemon instances
**Acceptance Criteria:**
- All daemons initialize successfully
- Each daemon has unique purpose/instructions
- Operations on one daemon don't affect others
- All daemons can be queried independently

### 6.2 Daemon Replacement
**Test:** Daemons can be swapped without changing caller code
**Setup:** 
- Caller code uses daemon via interface
- Replace daemon instance with different implementation
- Caller code runs unchanged
**Acceptance Criteria:**
- Caller code works with new daemon
- New daemon implements same interface
- Results are semantically equivalent

### 6.3 Daemon Composition
**Test:** Daemons can call other daemons
**Setup:** Daemon A calls daemon B asynchronously
**Acceptance Criteria:**
- Daemon A receives future from daemon B
- Daemon A can await result
- Composition works correctly
- No deadlocks or circular dependencies

---

## 7. Error Recovery Tests

### 7.1 Daemon Recovery from Error
**Test:** Daemon recovers after operation fails
**Setup:** 
- Call operation that fails
- Call another operation after failure
**Acceptance Criteria:**
- First operation returns error
- Daemon status is "error" or recovers to "ready"
- Second operation succeeds
- Daemon is usable after error

### 7.2 Partial Failure Handling
**Test:** Daemon handles partial failures gracefully
**Setup:** Call operation that partially succeeds
**Acceptance Criteria:**
- Operation returns partial result (not full failure)
- Error indicates which parts succeeded/failed
- Caller can use partial result if appropriate
- Daemon remains operational

