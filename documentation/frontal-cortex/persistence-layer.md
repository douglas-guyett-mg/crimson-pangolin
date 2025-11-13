# Frontal Cortex Persistence Layer

**Status:** Specification v1.0  
**Date:** 2025-11-05  
**Technology:** Agnostic (implementation details deferred)

## Overview

The Frontal Cortex persistence layer provides durable storage for Goals, Pending Actions, Questions, Stories, and Action Type Registry entries. It must support:
- **Concurrent updates** from parallel turns without data loss
- **Audit trails** for all mutations (versioning and lineage)
- **Access control** for multi-agent and multi-tenant scenarios
- **Efficient querying** for turn-start retrieval and WM integration

---

## 1. Conflict Resolution: Optimistic Locking with Field-Level Versioning

### 1.1 Versioning Strategy

Each FC item (Goal, PendingAction, Question, Story) maintains:

- **Global Version** (`version: integer`): Incremented on any mutation. Used to detect concurrent updates.
- **Field-Level Versions** (`field_versions: {field_name: integer}`): Tracks which fields changed. Enables merge-friendly updates for independent fields.
- **Last Modified Timestamp** (`updated_at: timestamp`): For ordering and audit purposes.

### 1.2 Conflict Detection and Resolution

**Write Operation:**
1. Client reads item with current `version` (e.g., v5)
2. Client modifies one or more fields
3. Client submits write with `expected_version: 5`
4. Server checks: if current version ≠ 5, return **ConflictError** with current state
5. On conflict, client retries: reads fresh state, reapplies changes, writes again

**Merge-Friendly Updates:**
- If Turn A updates `progress` and Turn B updates `priority` (different fields), both succeed
- Server increments global version and updates only the changed field's version
- Example:
  - Goal v5: progress_version=3, priority_version=2
  - Turn A writes progress (expects v5) → Goal v6: progress_version=4, priority_version=2
  - Turn B writes priority (expects v5) → ConflictError (now v6), Turn B retries and succeeds

### 1.3 Retry Semantics

- Clients MUST implement exponential backoff on ConflictError
- Maximum retry attempts: configurable (default: 3)
- Retry delay: exponential with jitter (e.g., 10ms, 20ms, 40ms)
- If max retries exceeded: return error to caller; Turn Trace records conflict and retry exhaustion

---

## 2. Versioning and Audit Trail

### 2.1 Mutation History

Every mutation creates an immutable audit entry:
- `mutation_id`: unique identifier for this change
- `item_id`: which Goal/Action/Question/Story was changed
- `mutation_type`: create | update | delete (soft-delete)
- `previous_version`: version before this mutation
- `new_version`: version after this mutation
- `changed_fields`: list of fields that changed
- `change_delta`: before/after values for each changed field
- `mutated_by`: agent_id or system component that made the change
- `mutated_from_turn_id`: which turn triggered this mutation
- `timestamp`: when the mutation occurred
- `rationale`: optional explanation (e.g., from Turn Trace)

### 2.2 Lineage Tracking

- Each item maintains `lineage_ids: [mutation_id, ...]` in order
- Enables full audit trail: query all mutations for an item
- Supports revert: restore item to any previous version by replaying mutations in reverse

### 2.3 Soft Delete

- Deletions are soft: set `deleted_at: timestamp` and `status: archived`
- Item remains queryable for audit purposes
- Queries exclude soft-deleted items by default (unless explicitly requested)

---

## 3. Access Control and Sharing

### 3.1 Ownership and Scope

Each FC item has:
- `owner_agent_id`: the agent that owns this item
- `owner_org_id`: the organization (tenant) that owns this item
- `scope`: private | shared_with_agents | shared_with_org | public

### 3.2 ACL Model

Access Control List (ACL) for each item:
- `acl: [{principal_type, principal_id, permissions}]`
- `principal_type`: agent | org | role
- `principal_id`: agent_id, org_id, or role name
- `permissions`: [read, write, delete, share]

**Default ACLs:**
- Owner agent: [read, write, delete, share]
- Owner org: [read] (org can view but not modify)
- Others: [] (no access)

### 3.3 Permission Semantics

- **read**: Can retrieve item and its history
- **write**: Can update item fields (subject to conflict resolution)
- **delete**: Can soft-delete item
- **share**: Can modify ACL (grant/revoke access to other agents/orgs)

### 3.4 Cross-Agent Sharing

Agents can share goals/actions with other agents:
1. Agent A calls `share_item(item_id, target_agent_id, permissions: [read, write])`
2. Server adds entry to ACL: `{principal_type: agent, principal_id: target_agent_id, permissions: [read, write]}`
3. Agent B can now read/write the item
4. All mutations by Agent B are recorded with `mutated_by: agent_b_id`

---

## 4. Query Patterns and Retrieval

### 4.1 Turn-Start Retrieval

At turn start, WM calls:
```
get_active_items(agent_id, org_id, filters: {
  item_types: [goal, pending_action, question, story],
  status: [active, in_progress],
  priority: [high, critical],
  limit: 10
}) -> items[]
```

Returns items ordered by:
1. Priority (critical > high > medium > low)
2. Due date (soonest first)
3. Blocking status (blockers first)
4. Relevance to current context (if available)

### 4.2 Audit Trail Queries

```
get_item_history(item_id, version_range?: {from, to}) -> mutations[]
get_mutations_by_turn(turn_id) -> mutations[]
get_mutations_by_agent(agent_id, time_range?: {from, to}) -> mutations[]
```

### 4.3 Filtering and Search

```
query_items(filters: {
  agent_id,
  org_id,
  item_type: goal | pending_action | question | story,
  status: [active, completed, archived],
  tags: [tag1, tag2],
  created_after: timestamp,
  created_before: timestamp,
  goal_id: (for actions/questions linked to a goal),
  owner: agent | user,
  blocking: true | false
}) -> items[]
```

---

## 5. Persistence Operations

### 5.1 Core Operations

- `create_item(item: Goal | PendingAction | Question | Story) -> item_id`
- `read_item(item_id) -> item`
- `update_item(item_id, updates: {field: value}, expected_version: int) -> item | ConflictError`
- `delete_item(item_id) -> Ack` (soft delete)
- `get_item_history(item_id) -> mutations[]`
- `share_item(item_id, principal_id, permissions) -> Ack`
- `revoke_access(item_id, principal_id) -> Ack`

### 5.2 Batch Operations

- `batch_update_items(updates: [{item_id, fields, expected_version}]) -> {succeeded: [], failed: []}`
- `batch_query_items(queries: [query_spec]) -> results[]`

---

## 6. Test Conditions

### 6.1 Conflict Resolution Tests

- ✅ Concurrent updates to different fields succeed (merge-friendly)
- ✅ Concurrent updates to same field fail with ConflictError
- ✅ Retry after conflict succeeds with fresh data
- ✅ Max retry exhaustion returns error
- ✅ Version numbers increment correctly
- ✅ Field-level versions track independent updates

### 6.2 Audit Trail Tests

- ✅ Every mutation creates immutable audit entry
- ✅ Lineage is complete and ordered
- ✅ Revert restores item to previous version
- ✅ Soft-deleted items excluded from queries by default
- ✅ Soft-deleted items queryable with explicit flag

### 6.3 ACL Tests

- ✅ Owner has full permissions by default
- ✅ Non-owner cannot read/write without ACL entry
- ✅ Shared items readable/writable by target agent
- ✅ Share operation updates ACL correctly
- ✅ Revoke operation removes access
- ✅ Mutations by shared agents recorded correctly

### 6.4 Query Tests

- ✅ Turn-start retrieval returns items in priority order
- ✅ Filtering by status, tags, goal_id works correctly
- ✅ Audit trail queries return complete history
- ✅ Batch queries return results in order

---

## 7. Integration with Turn Execution

### 7.1 Turn-Start Phase

1. WM calls `get_active_items(agent_id, org_id, filters)`
2. Persistence layer returns top-K goals, pending actions, questions/stories
3. WM injects into prompt for turn planning

### 7.2 Turn-End Commit Phase

1. Executor calls `commit_at_turn_end(turn_ctx, updates: {progress, new_actions, completed_actions, story_updates})`
2. Persistence layer applies updates with optimistic locking
3. On conflict: record in Turn Trace, retry or escalate
4. On success: record mutations in audit trail with turn_id

### 7.3 Error Handling

- ConflictError: Turn Trace records conflict, Executor retries or escalates
- PermissionError: Turn Trace records access denial, Executor escalates
- StorageError: Turn Trace records error, Error Handler daemon decides retry/escalate

---

## 8. Relationship to Other Systems

- **Working Memory**: Reads FC items at turn start, updates at turn end
- **Turn Trace**: Records all FC mutations with turn_id and rationale
- **Consciousness**: May inform FC queries (e.g., which goals align with current mode)
- **Scratch Page**: Observations that warrant action become FC goals/actions

