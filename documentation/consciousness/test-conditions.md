# Consciousness Data Model — Test Conditions

**Status:** v1.0 (Stable)
**Last Updated:** 2025-11-07

---

## 1. Data Model Integrity Tests

### 1.1 Required Fields
**Test:** All consciousness items must have required fields
- `id`, `type`, `version`, `name`, `description`, `content`
- `tags`, `approval_status`, `effective_date`, `priority`
- `created_by`, `created_at`, `change_rationale`

**Acceptance Criteria:**
- Attempting to create an item without any required field fails with clear error
- All existing consciousness items pass validation

### 1.2 Field Type Validation
**Test:** Fields must have correct types
- `version` must be valid semantic version (MAJOR.MINOR.PATCH)
- `approval_status` must be one of: "draft", "approved", "deprecated"
- `priority` must be integer 1-10
- `effective_date` and `sunset_date` must be valid ISO8601 timestamps
- `tags` must be non-empty array of strings

**Acceptance Criteria:**
- Invalid values are rejected with clear error messages
- Type coercion does not occur (strict validation)

### 1.3 Logical Consistency
**Test:** Items must satisfy logical constraints
- If `approval_status = "approved"`, then `approved_by` and `approved_at` must be set
- If `approval_status = "draft"`, then `approved_by` and `approved_at` must be null
- If `sunset_date` is set, it must be after `effective_date`
- `effective_date` must be <= current time for approved items (or in future for scheduled)

**Acceptance Criteria:**
- Violations are caught and reported
- All existing items satisfy these constraints

---

## 2. Daemon Interface Tests

### 2.1 Consciousness Implements Daemon Interface
**Test:** Consciousness daemon implements all required methods
- `get_purpose()` returns a string describing consciousness purpose
- `get_instructions()` returns consciousness specifications
- `query(prompt, metadata)` returns a Future

**Acceptance Criteria:**
- All three methods are callable and return expected types
- `get_purpose()` returns "Provide SI's self-concept, identity, mandates, capabilities, and governance rules"
- `query()` returns a Future that resolves to ConsciousnessResult

### 2.2 Query by Natural Language
**Test:** `consciousness.query("What mandates apply to my current mode?", {current_mode: "code_review"})` returns relevant mandates
**Acceptance Criteria:**
- Returns only items where `type = "mandate"` and `applies_to` includes "code_review" or is empty
- Results are sorted by priority (highest first)
- Results include only approved items effective at current time
- Returns empty array if no mandates exist
- Works for all item types: mandate, capability, mode, governance_rule

### 2.3 Query Result Includes Rationale
**Test:** `consciousness.query()` returns ConsciousnessResult with rationale
**Acceptance Criteria:**
- Result includes `rationale: string` explaining why items were selected
- Result includes `effective_at: ISO8601` timestamp
- Result includes `items: ConsciousnessItem[]` array
- Rationale is human-readable and suitable for logging

### 2.4 Asynchronous Query Execution
**Test:** `consciousness.query()` returns a Future that can be awaited
**Acceptance Criteria:**
- Query returns immediately (non-blocking)
- Caller can do other work while waiting
- Future resolves to ConsciousnessResult
- Multiple queries can be parallelized

### 2.5 Query Result Ordering
**Test:** Results are sorted by priority (highest first)
**Acceptance Criteria:**
- Items with `priority=10` appear before `priority=1`
- Items with same priority maintain insertion order
- Sorting is deterministic

---

## 3. Selection Logic (v1) Tests

### 3.1 Core Self Always Included
**Test:** Every consciousness query includes core_si_self mandate
**Acceptance Criteria:**
- `query_consciousness("mandate", ["core_si_self"])` returns exactly one item
- That item has `approval_status = "approved"`
- That item is the latest version (highest version number)

### 3.2 Current Mode Always Included
**Test:** Every consciousness query includes current mode
**Setup:** Set current mode to "research_mode"
**Acceptance Criteria:**
- `query_consciousness("mode", ["research_mode"])` returns the current mode
- Mode has `approval_status = "approved"`
- Mode is the latest version

### 3.3 v1 Selection Returns Exactly Two Items
**Test:** v1 selection logic returns core_si_self + current_mode
**Acceptance Criteria:**
- Result array has exactly 2 items
- First item is core_si_self mandate
- Second item is current mode
- No other items are included

---

## 4. Versioning Tests

### 4.1 Version Numbering
**Test:** Versions follow semantic versioning
**Acceptance Criteria:**
- Valid versions: "1.0.0", "1.1.0", "2.0.0", "1.0.1"
- Invalid versions are rejected: "1.0", "v1.0.0", "1.0.0.0"

### 4.2 Latest Version Selection
**Test:** System always uses latest approved version
**Setup:** Create mandate with versions 1.0.0, 1.1.0, 2.0.0 (all approved)
**Acceptance Criteria:**
- Query returns version 2.0.0
- Older versions are still retrievable via version history
- Deprecated versions are not returned in default queries

### 4.3 Effective Date Filtering
**Test:** Items with future effective_date are not returned
**Setup:** Create item with `effective_date = tomorrow`
**Acceptance Criteria:**
- Query with `effective_at = today` does not return the item
- Query with `effective_at = tomorrow` returns the item
- Scheduled items can be queried explicitly

---

## 5. Change Management Tests

### 5.1 Draft Items Not Returned by Default
**Test:** Draft consciousness items are excluded from default queries
**Setup:** Create mandate with `approval_status = "draft"`
**Acceptance Criteria:**
- Default query does not return draft item
- Explicit query with `approval_status="draft"` returns it
- Draft items cannot be used in turn execution

### 5.2 Approval Workflow
**Test:** Items transition from draft → approved
**Setup:** Create draft mandate
**Acceptance Criteria:**
- Initially: `approval_status = "draft"`, `approved_by = null`, `approved_at = null`
- After approval: `approval_status = "approved"`, `approved_by = <user>`, `approved_at = <timestamp>`
- Approval is idempotent (approving twice doesn't change state)

### 5.3 Deprecation Workflow
**Test:** Items can be deprecated with sunset_date
**Setup:** Create approved mandate, then deprecate it
**Acceptance Criteria:**
- Set `approval_status = "deprecated"` and `sunset_date = tomorrow`
- Query with `effective_at = today` returns item (still active)
- Query with `effective_at = tomorrow + 1 day` does not return item
- Deprecated items are retrievable via history

---

## 6. Audit Trail Tests

### 6.1 Creation Audit
**Test:** All items record creation metadata
**Acceptance Criteria:**
- `created_by` is set to user/system that created item
- `created_at` is set to creation timestamp
- `change_rationale` documents why item was created

### 6.2 Approval Audit
**Test:** All approved items record approval metadata
**Acceptance Criteria:**
- `approved_by` is set to approver
- `approved_at` is set to approval timestamp
- Approval history is immutable (cannot be changed retroactively)

---

## 7. Integration Tests

### 7.1 Consciousness Assembly for Turn
**Test:** Consciousness can be assembled for a turn execution
**Setup:** Set current mode to "research_mode"
**Acceptance Criteria:**
- Retrieve core_si_self mandate
- Retrieve research_mode mode
- Both items are approved and effective
- Can be formatted into prompt text

### 7.2 Multiple Versions Coexist
**Test:** Multiple versions of same item can coexist
**Setup:** Create mandate v1.0.0 (approved), v1.1.0 (approved), v2.0.0 (draft)
**Acceptance Criteria:**
- All three versions are stored
- Query returns v1.1.0 (latest approved)
- Can retrieve full version history
- Draft version is not returned in default queries

---

## 8. Mode Scoping Tests

### 8.1 Mode is Locked at Turn Start
**Objective:** Verify that mode is determined at turn start and remains locked for entire turn

**Setup:**
- Turn starts with Mode A
- User switches to Mode B mid-turn

**Steps:**
1. Turn starts, Router determines mode = "Mode A"
2. Consciousness is queried with metadata: {current_mode: "Mode A"}
3. Consciousness returns mandates for Mode A
4. User switches to Mode B (mid-turn)
5. Consciousness is queried again with metadata: {current_mode: "Mode A"} (still Mode A)
6. Consciousness returns mandates for Mode A (not Mode B)
7. Turn completes

**Expected Outcome:**
- Mode remains "Mode A" throughout turn
- All Consciousness queries return Mode A mandates
- Mode switch is queued for next turn

**Verification:**
- Turn Trace shows all queries with current_mode: "Mode A"
- Consciousness returns consistent mandates for Mode A
- Mode switch does not affect current turn

---

### 8.2 Mode Switch Takes Effect at Next Turn
**Objective:** Verify that mode switches only take effect at turn start

**Setup:**
- Turn 1 completes with Mode A
- User switches to Mode B
- Turn 2 starts

**Steps:**
1. Turn 1 executes with Mode A
2. Turn 1 completes
3. User switches to Mode B
4. Turn 2 starts
5. Router determines mode = "Mode B"
6. Consciousness is queried with metadata: {current_mode: "Mode B"}
7. Consciousness returns mandates for Mode B

**Expected Outcome:**
- Turn 2 starts with Mode B
- Consciousness returns Mode B mandates
- Mode switch is effective

**Verification:**
- Turn 1 Turn Trace shows current_mode: "Mode A"
- Turn 2 Turn Trace shows current_mode: "Mode B"
- Consciousness returns different mandates for each turn

---

### 8.3 Consciousness Returns Correct Mandates for Mode
**Objective:** Verify that Consciousness returns appropriate mandates based on current_mode metadata

**Setup:**
- Mode A has mandates: [M1, M2, M3]
- Mode B has mandates: [M4, M5, M6]
- Core mandates: [C1, C2] (always present)

**Steps:**
1. Query Consciousness with {current_mode: "Mode A"}
2. Verify response includes M1, M2, M3, C1, C2
3. Query Consciousness with {current_mode: "Mode B"}
4. Verify response includes M4, M5, M6, C1, C2

**Expected Outcome:**
- Mode A query returns Mode A mandates + core mandates
- Mode B query returns Mode B mandates + core mandates
- Core mandates always present in both responses

**Verification:**
- Response items match expected mandates
- Core mandates present in all responses
- Mode-specific mandates differ between modes

---

### 8.4 Mid-Turn Mode Switch is Ignored
**Objective:** Verify that mode switches during turn execution do not affect current turn

**Setup:**
- Turn executing with Mode A
- User switches to Mode B mid-turn
- Multiple Consciousness queries during turn

**Steps:**
1. Turn starts with Mode A
2. Query 1: Consciousness.query("What mandates?", {current_mode: "Mode A"})
3. User switches to Mode B
4. Query 2: Consciousness.query("What mandates?", {current_mode: "Mode A"}) (still Mode A)
5. Query 3: Consciousness.query("What mandates?", {current_mode: "Mode A"}) (still Mode A)
6. Turn completes

**Expected Outcome:**
- All queries return Mode A mandates
- Mode switch does not affect any queries
- Turn completes with Mode A

**Verification:**
- All queries in Turn Trace show current_mode: "Mode A"
- Consciousness returns consistent Mode A mandates
- No Mode B mandates appear in turn

---

## 9. Concurrent Scenario Tests

### 9.1 Concurrent Mode Switch Ignored During Turn
**Objective:** Verify that mode changes during turn execution are isolated and do not affect in-flight turns

**Setup:**
- Turn T1 starts at t=0 with Mode = "research"
- Consciousness locked at turn start (mode snapshot taken)
- User switches Mode = "code_review" at t=50ms (mid-turn)
- Turn T2 starts at t=200ms (after T1 completes)

**Steps:**
1. T1 starts, Router determines mode = "research"
2. T1 Query 1 (t=10ms): Consciousness.query("What are my research mandates?", {current_mode: "research"})
3. User switches mode to "code_review" (t=50ms, during T1 execution)
4. T1 Query 2 (t=60ms): Consciousness.query("What constraints apply?", {current_mode: "research"}) - still uses "research"
5. T1 Query 3 (t=80ms): Consciousness.query("What capabilities do I have?", {current_mode: "research"}) - still uses "research"
6. T1 completes (t=100ms)
7. T2 starts (t=200ms), Router determines mode = "code_review"
8. T2 Query 1 (t=210ms): Consciousness.query("What are my mandates?", {current_mode: "code_review"})

**Expected Outcome:**
- All queries in T1 (Query 1, 2, 3) return mandates for "research" mode
- Query 1: Returns research-specific mandates (e.g., "thorough_investigation", "cite_sources")
- Query 2: Returns research-specific constraints (e.g., "verify_facts", "document_findings")
- Query 3: Returns research-specific capabilities (e.g., "web_search", "data_analysis")
- T2 Query 1 returns mandates for "code_review" mode (e.g., "check_style", "identify_bugs")
- Mode switch at t=50ms does not affect T1
- T2 correctly uses new mode

**Verification:**
- Turn Trace for T1 shows:
  - turn_id: "T1"
  - mode_at_start: "research"
  - mode_locked: true
  - mode_lock_timestamp: t=0
  - All 3 queries logged with current_mode: "research"
  - No mode change events during T1 execution
  - turn_end_mode: "research"
- Turn Trace for T2 shows:
  - turn_id: "T2"
  - mode_at_start: "code_review"
  - mode_locked: true
  - mode_lock_timestamp: t=200ms
  - Query logged with current_mode: "code_review"
- Consciousness query results:
  - T1 Query 1, 2, 3: All return items where applies_to includes "research" or is empty
  - T2 Query 1: Returns items where applies_to includes "code_review" or is empty
  - No overlap in mode-specific mandates between T1 and T2
- Mode change log shows:
  - mode_change_requested: t=50ms, from "research" to "code_review"
  - mode_change_effective: t=200ms (at T2 start)
  - affected_turn: "T2" (not "T1")

**Measurable Acceptance Criteria:**
- 100% of T1 queries (3/3) use "research" mode metadata
- 0 instances of "code_review" mandates returned during T1
- T2 uses "code_review" mode with 100% consistency
- Mode lock timestamp for T1 < mode change request timestamp < T1 completion timestamp
- Mode change becomes effective only at T2 start (t=200ms), not during T1

---

