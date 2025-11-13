# Frontal Cortex Persistence Layer - Test Conditions

**Status:** v1.0  
**Date:** 2025-11-05

## Test Categories

### 1. Conflict Resolution and Versioning

#### 1.1 Concurrent Updates to Different Fields (Merge-Friendly)
**Scenario:** Two turns update different fields of the same goal simultaneously.

**Setup:**
- Goal v5: {progress: 50%, priority: high, status: active}
- Turn A reads Goal v5
- Turn B reads Goal v5

**Actions:**
- Turn A updates: progress = 75%, expected_version = 5
- Turn B updates: priority = critical, expected_version = 5

**Expected Outcome:**
- Both updates succeed
- Final Goal state: {progress: 75%, priority: critical, status: active}
- Global version: v7 (incremented twice)
- progress_version: 4, priority_version: 3
- No ConflictError raised

#### 1.1b Concurrent Updates to Different Fields (Merge Success - Detailed)
**Scenario:** Two concurrent turns update non-overlapping fields of the same goal, demonstrating successful merge behavior.

**Setup:**
- Goal v5 exists with state:
  - id: "goal_123"
  - progress: 50%
  - priority: "high"
  - status: "active"
  - global_version: 5
  - progress_version: 3
  - priority_version: 2
  - status_version: 1
- Turn A (turn_id: "turn_A_001") starts at t=0, reads Goal v5
- Turn B (turn_id: "turn_B_001") starts at t=5ms, reads Goal v5

**Actions:**
1. Turn A (t=10ms): Calls `update_goal(goal_id="goal_123", updates={progress: 75%}, expected_version=5)`
2. Turn B (t=12ms): Calls `update_goal(goal_id="goal_123", updates={priority: "critical"}, expected_version=5)`
3. Persistence layer processes updates with field-level versioning

**Expected Outcome:**
- Turn A update succeeds:
  - Goal becomes v6
  - progress: 75% (updated)
  - priority: "high" (unchanged)
  - status: "active" (unchanged)
  - global_version: 6
  - progress_version: 4 (incremented)
  - priority_version: 2 (unchanged)
  - status_version: 1 (unchanged)
- Turn B update succeeds (merge-friendly):
  - Goal becomes v7
  - progress: 75% (preserved from Turn A)
  - priority: "critical" (updated)
  - status: "active" (unchanged)
  - global_version: 7
  - progress_version: 4 (preserved)
  - priority_version: 3 (incremented)
  - status_version: 1 (unchanged)
- Final Goal state: {progress: 75%, priority: "critical", status: "active"}
- No ConflictError raised for either turn

**Verification:**
- Audit trail contains 2 mutation entries:
  - Mutation 1:
    - mutation_id: <UUID_1>
    - item_id: "goal_123"
    - mutation_type: "update"
    - previous_version: 5
    - new_version: 6
    - changed_fields: ["progress"]
    - mutated_by: "turn_A_001"
    - mutation_timestamp: t=10ms
    - field_changes: {progress: {old: 50%, new: 75%, version: 3→4}}
  - Mutation 2:
    - mutation_id: <UUID_2>
    - item_id: "goal_123"
    - mutation_type: "update"
    - previous_version: 6
    - new_version: 7
    - changed_fields: ["priority"]
    - mutated_by: "turn_B_001"
    - mutation_timestamp: t=12ms
    - field_changes: {priority: {old: "high", new: "critical", version: 2→3}}
- Lineage chain verified:
  - v5 → v6 (Turn A: progress update)
  - v6 → v7 (Turn B: priority update)
  - Complete and ordered chain with no gaps
- Turn Trace entries:
  - Turn A: "FC update succeeded: goal_123, fields=[progress], version: 5→6"
  - Turn B: "FC update succeeded: goal_123, fields=[priority], version: 6→7, merge_applied=true"
- Final goal state query returns:
  - global_version: 7
  - All field versions correct (progress_version=4, priority_version=3, status_version=1)
  - Both updates reflected in final state

**Measurable Acceptance Criteria:**
- 0 ConflictErrors raised (both updates succeed)
- 2 mutations created in audit trail
- Global version increments from 5 → 7 (2 increments)
- progress_version increments from 3 → 4 (Turn A only)
- priority_version increments from 2 → 3 (Turn B only)
- 100% of Turn A's change (progress=75%) preserved in final state
- 100% of Turn B's change (priority="critical") preserved in final state
- Lineage chain is complete with 2 entries connecting v5→v6→v7
- Merge operation detected and logged for Turn B (merge_applied=true)

#### 1.2 Concurrent Updates to Same Field (Conflict)
**Scenario:** Two turns update the same field simultaneously.

**Setup:**
- Goal v5: {progress: 50%}
- Turn A reads Goal v5
- Turn B reads Goal v5

**Actions:**
- Turn A updates: progress = 75%, expected_version = 5 → succeeds, Goal becomes v6
- Turn B updates: progress = 80%, expected_version = 5 → fails with ConflictError

**Expected Outcome:**
- Turn A succeeds, Goal v6: {progress: 75%}
- Turn B receives ConflictError with current state (v6, progress: 75%)
- Turn B can retry with fresh data

#### 1.3 Retry After Conflict
**Scenario:** Turn retries after receiving ConflictError.

**Setup:**
- Goal v6: {progress: 75%}
- Turn B has ConflictError from previous attempt

**Actions:**
- Turn B reads fresh Goal v6
- Turn B updates: progress = 80%, expected_version = 6

**Expected Outcome:**
- Update succeeds
- Goal becomes v7: {progress: 80%}
- Audit trail shows two mutations: v5→v6 (Turn A), v6→v7 (Turn B)

#### 1.4 Max Retry Exhaustion
**Scenario:** Turn exhausts max retries due to repeated conflicts.

**Setup:**
- Max retries: 3
- Goal continuously updated by other turns

**Actions:**
- Turn A attempts update, gets ConflictError
- Turn A retries (attempt 1): ConflictError
- Turn A retries (attempt 2): ConflictError
- Turn A retries (attempt 3): ConflictError

**Expected Outcome:**
- After 3 retries, return error to caller
- Turn Trace records: "FC update failed after 3 retries, item_id=X, field=Y"
- Error Handler daemon can decide to escalate or fail gracefully

#### 1.5 Version Numbers Increment Correctly
**Scenario:** Verify version numbering is sequential and consistent.

**Setup:**
- Goal v1 (initial creation)

**Actions:**
- Mutation 1: update progress → v2
- Mutation 2: update priority → v3
- Mutation 3: update status → v4

**Expected Outcome:**
- Versions are sequential: 1, 2, 3, 4
- Each mutation increments global version by 1
- Field-level versions track independent changes

#### 1.6 Field-Level Versions Track Independent Updates
**Scenario:** Verify field-level versioning enables merge-friendly updates.

**Setup:**
- Goal: {progress_version: 2, priority_version: 1, status_version: 1}

**Actions:**
- Update progress → progress_version becomes 3, others unchanged
- Update priority → priority_version becomes 2, others unchanged

**Expected Outcome:**
- progress_version: 3, priority_version: 2, status_version: 1
- Independent fields can be updated without conflict

---

### 2. Audit Trail and Versioning

#### 2.1 Every Mutation Creates Immutable Audit Entry
**Scenario:** Verify all mutations are recorded.

**Setup:**
- Goal created

**Actions:**
- Create Goal
- Update progress
- Update priority
- Soft-delete Goal

**Expected Outcome:**
- 4 audit entries created (create, update, update, delete)
- Each entry has: mutation_id, item_id, mutation_type, previous_version, new_version, changed_fields, timestamp
- Entries are immutable (cannot be modified)

#### 2.2 Lineage is Complete and Ordered
**Scenario:** Verify lineage tracks all mutations in order.

**Setup:**
- Goal with 5 mutations

**Actions:**
- Call `get_item_history(goal_id)`

**Expected Outcome:**
- Returns 5 mutations in chronological order
- Each mutation references previous version
- Lineage chain verified: mutation[i].new_version equals mutation[i+1].previous_version for all i, no gaps in sequence, versions are 1→2→3→4→5, mutation[0].previous_version=null, all 5 mutations have valid mutation_id and timestamp

#### 2.3 Revert Restores Item to Previous Version
**Scenario:** Verify revert functionality.

**Setup:**
- Goal v5: {progress: 80%, priority: critical}
- Lineage: v1 → v2 → v3 → v4 → v5

**Actions:**
- Call `revert_to_version(goal_id, version=3)`

**Expected Outcome:**
- Goal restored to v3 state
- New version created: v6 (revert creates new version)
- Lineage: v1 → v2 → v3 → v4 → v5 → v6 (revert)
- Audit entry records: "reverted from v5 to v3"

#### 2.4 Soft-Deleted Items Excluded from Queries by Default
**Scenario:** Verify soft-delete behavior.

**Setup:**
- Goal A (active)
- Goal B (active)
- Soft-delete Goal B

**Actions:**
- Query active goals

**Expected Outcome:**
- Returns only Goal A
- Goal B not included (soft-deleted)

#### 2.5 Soft-Deleted Items Queryable with Explicit Flag
**Scenario:** Verify soft-deleted items can be retrieved for audit.

**Setup:**
- Goal B (soft-deleted)

**Actions:**
- Query with `include_deleted: true`

**Expected Outcome:**
- Returns Goal B with `deleted_at: timestamp` and `status: archived`
- Audit trail still accessible

---

### 3. Access Control and Sharing

#### 3.1 Owner Has Full Permissions by Default
**Scenario:** Verify owner permissions.

**Setup:**
- Agent A creates Goal

**Actions:**
- Agent A reads Goal → succeeds
- Agent A updates Goal → succeeds
- Agent A deletes Goal → succeeds
- Agent A shares Goal → succeeds

**Expected Outcome:**
- All operations succeed
- ACL shows: {principal_type: agent, principal_id: A, permissions: [read, write, delete, share]}

#### 3.2 Non-Owner Cannot Access Without ACL Entry
**Scenario:** Verify access control.

**Setup:**
- Agent A creates Goal
- Agent B (no ACL entry)

**Actions:**
- Agent B reads Goal → PermissionError
- Agent B updates Goal → PermissionError

**Expected Outcome:**
- Both operations fail with PermissionError
- Turn Trace records: {event: "access_denied", principal_id: "agent_b_id", resource_type: "goal", resource_id: <goal_id>, attempted_operation: "read|write", timestamp: <ISO8601>, acl_checked: true}

#### 3.3 Shared Items Readable/Writable by Target Agent
**Scenario:** Verify sharing works.

**Setup:**
- Agent A creates Goal
- Agent A shares Goal with Agent B (permissions: [read, write])

**Actions:**
- Agent B reads Goal → succeeds
- Agent B updates Goal → succeeds

**Expected Outcome:**
- Both operations succeed
- Mutations recorded with `mutated_by: agent_b_id`

#### 3.4 Share Operation Updates ACL Correctly
**Scenario:** Verify ACL updates.

**Setup:**
- Goal with ACL: [{principal_type: agent, principal_id: A, permissions: [read, write, delete, share]}]

**Actions:**
- Agent A calls `share_item(goal_id, agent_b_id, permissions: [read, write])`

**Expected Outcome:**
- ACL updated: adds entry for Agent B
- Agent B can now read/write
- Audit entry records: "shared with agent_b_id, permissions: [read, write]"

#### 3.5 Revoke Operation Removes Access
**Scenario:** Verify access revocation.

**Setup:**
- Goal shared with Agent B

**Actions:**
- Agent A calls `revoke_access(goal_id, agent_b_id)`

**Expected Outcome:**
- ACL entry for Agent B removed
- Agent B can no longer read/write
- Audit entry records: "revoked access for agent_b_id"

#### 3.6 Mutations by Shared Agents Recorded Correctly
**Scenario:** Verify mutation tracking for shared access.

**Setup:**
- Goal shared with Agent B

**Actions:**
- Agent B updates Goal

**Expected Outcome:**
- Mutation recorded with `mutated_by: agent_b_id`
- Audit entry contains: {mutated_by: 'agent_b_id', mutation_timestamp: <ISO8601>, previous_value: <val>, new_value: <val>, mutation_type: 'update', agent_permissions_verified: true}
- Owner (Agent A) can see all mutations

---

### 4. Query Patterns and Retrieval

#### 4.1 Turn-Start Retrieval Returns Items in Priority Order
**Scenario:** Verify retrieval ordering.

**Setup:**
- Goal 1: priority=low, due_at=tomorrow
- Goal 2: priority=critical, due_at=next_week
- Goal 3: priority=high, due_at=today
- Goal 4: priority=medium, due_at=today

**Actions:**
- Call `get_active_items(agent_id, filters: {limit: 10})`

**Expected Outcome:**
- Returns in order: Goal 2 (critical), Goal 3 (high, due today), Goal 4 (medium, due today), Goal 1 (low)
- Sort order: 1) priority (critical>high>medium>low), 2) due_date ASC (nulls last), 3) blocking=true before blocking=false, 4) created_at ASC for final tie-break

#### 4.2 Filtering by Status Works Correctly
**Scenario:** Verify status filtering.

**Setup:**
- Goal A: status=active
- Goal B: status=completed
- Goal C: status=in_progress

**Actions:**
- Query with `status: [active, in_progress]`

**Expected Outcome:**
- Returns Goal A and Goal C
- Goal B excluded

#### 4.3 Filtering by Tags Works Correctly
**Scenario:** Verify tag filtering.

**Setup:**
- Goal A: tags=[urgent, security]
- Goal B: tags=[security]
- Goal C: tags=[feature]

**Actions:**
- Query with `tags: [security]`

**Expected Outcome:**
- Returns Goal A and Goal B
- Goal C excluded

#### 4.4 Audit Trail Queries Return Complete History
**Scenario:** Verify audit trail retrieval.

**Setup:**
- Goal with 10 mutations

**Actions:**
- Call `get_item_history(goal_id)`

**Expected Outcome:**
- Returns all 10 mutations in chronological order
- Each mutation has: mutation_id (UUID), item_id, mutation_type (enum), previous_version (int), new_version (int), changed_fields (array), mutated_by (agent_id), mutation_timestamp (ISO8601), transaction_id (UUID)

#### 4.5 Batch Queries Return Results in Order
**Scenario:** Verify batch query semantics.

**Setup:**
- Query 1: get goals with priority=critical
- Query 2: get pending actions with status=active
- Query 3: get questions for goal_id=X

**Actions:**
- Call `batch_query_items([query1, query2, query3])`

**Expected Outcome:**
- Returns results array with 3 elements
- Results[0] = goals, Results[1] = actions, Results[2] = questions
- Order matches query order

---

### 5. Integration with Turn Execution

#### 5.1 Turn-Start Retrieval Injects into WM
**Scenario:** Verify WM integration at turn start.

**Setup:**
- 5 active goals, 3 pending actions

**Actions:**
- Turn starts, WM calls `get_active_items(agent_id, org_id, filters)`

**Expected Outcome:**
- WM receives items
- Items injected into prompt
- Turn Trace records: "FC items retrieved: 5 goals, 3 actions"

#### 5.2 Turn-End Commit Applies Updates
**Scenario:** Verify commit phase.

**Setup:**
- Goal v5: progress=50%
- Turn completes with update: progress=75%

**Actions:**
- Executor calls `commit_at_turn_end(turn_ctx, updates: {progress: 75%})`

**Expected Outcome:**
- Goal updated to v6: progress=75%
- Audit entry created with turn_id
- Turn Trace records: "FC commit succeeded"

#### 5.3 Conflict During Commit Recorded in Turn Trace
**Scenario:** Verify error handling.

**Setup:**
- Goal v5
- Turn attempts update with expected_version=5
- Another turn already updated to v6

**Actions:**
- Executor calls `commit_at_turn_end(turn_ctx, updates: {...}, expected_version: 5)`

**Expected Outcome:**
- ConflictError returned
- Turn Trace records: "FC commit failed: ConflictError, item_id=X, current_version=6"
- Error Handler daemon notified

---

### 6. Edge Cases and Stress Tests

#### 6.1 Rapid Sequential Updates
**Scenario:** Multiple updates in quick succession.

**Setup:**
- Goal v1

**Actions:**
- 10 rapid updates from same turn

**Expected Outcome:**
- All succeed
- Versions: v1 → v2 → v3 → ... → v11
- Lineage chain verified: each mutation.previous_version equals prior mutation.new_version, no gaps in version sequence, all 10 mutations retrievable via get_item_history(), first mutation has previous_version=null

#### 6.2 Concurrent Updates from Many Turns
**Scenario:** 10 turns updating same goal simultaneously.

**Setup:**
- Goal v1
- 10 turns each read v1 and attempt update

**Expected Outcome:**
- First turn succeeds (v2)
- Remaining 9 get ConflictError
- All 9 can retry and eventually succeed
- Final version: v11
- Lineage contains exactly 10 mutation entries, sequential versions 1→2→3...→11, each mutation has unique mutated_by (one of the 10 turn_ids), complete chain verified via get_item_history() returning all 10 mutations

#### 6.3 Large Batch Operations
**Scenario:** Batch update 100 items.

**Setup:**
- 100 goals to update

**Actions:**
- Call `batch_update_items([100 updates])`

**Expected Outcome:**
- Transaction isolation level: SERIALIZABLE, all 100 updates committed or all rolled back, no partial state visible, confirmed by database transaction log
- Results show succeeded/failed counts
- Audit trail contains exactly 100 mutation entries (on success) or 0 (on rollback), each with: mutation_id, item_id, mutation_type, previous_version, new_version, changed_fields, mutated_by, mutation_timestamp, transaction_id (same for all 100)

#### 6.4 Query Performance with Large Dataset
**Scenario:** Query 10,000 items with filters.

**Setup:**
- 10,000 goals in system

**Actions:**
- Query with multiple filters

**Expected Outcome:**
- Query returns results in < 500ms for 10,000 items, < 2 seconds for 100,000 items, with appropriate indexes
- Results accurate and complete: returned count matches expected count (exact), all returned items satisfy filter criteria (100% match), no false positives, no missing items, checksum of result set matches expected checksum

