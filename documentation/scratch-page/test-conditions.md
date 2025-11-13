# Scratch Page System - Test Conditions

**Status**: Specification v0.1  
**Last Updated**: 2025-10-31

## Test Requirements Overview

All test conditions must be deterministic and reproducible. Tests should use fixed seeds for any randomization and include both positive and negative cases.

---

## Core Operations Tests

### Test Suite 1: add_observation()

**TC-1.1: Add valid observation**
- Input: Valid ObservationItem with all required fields
- Expected: Observation created with id and timestamps
- Verify: id is unique ULID, created_at is current time, status is "active"

**TC-1.2: Add observation with minimal fields**
- Input: ObservationItem with only required fields (type, content, source)
- Expected: Observation created with defaults for optional fields
- Verify: owner defaults to "agent", status defaults to "active", confidence defaults to 0.5

**TC-1.3: Reject invalid observation type**
- Input: ObservationItem with unregistered type
- Expected: Error returned, observation not created
- Verify: Error message indicates invalid type

**TC-1.4: Reject empty content**
- Input: ObservationItem with empty content string
- Expected: Error returned, observation not created
- Verify: Error message indicates content required

**TC-1.5: Add observation with expiration**
- Input: ObservationItem with expires_at set to future time
- Expected: Observation created and stored
- Verify: expires_at is preserved

**TC-1.6: Add observation with metadata**
- Input: ObservationItem with arbitrary metadata key-value pairs
- Expected: Observation created with metadata preserved
- Verify: Metadata is retrievable unchanged

### Test Suite 2: list_observations()

**TC-2.1: List all observations**
- Setup: Add 5 observations with different types
- Input: list_observations({})
- Expected: All 5 observations returned
- Verify: Order is by recency (newest first)

**TC-2.2: Filter by type**
- Setup: Add 3 pending_confirmation, 2 contextual_insight observations
- Input: list_observations({ type: "pending_confirmation" })
- Expected: Only 3 pending_confirmation observations returned
- Verify: No other types included

**TC-2.3: Filter by status**
- Setup: Add observations with mixed statuses (active, resolved, archived)
- Input: list_observations({ status: "active" })
- Expected: Only active observations returned
- Verify: Resolved and archived excluded

**TC-2.4: Filter by owner**
- Setup: Add observations with owner="agent" and owner="user"
- Input: list_observations({ owner: "user" })
- Expected: Only user-owned observations returned
- Verify: Agent-owned excluded

**TC-2.5: Filter by tags**
- Setup: Add observations with various tags
- Input: list_observations({ tags: ["security"] })
- Expected: Only observations with "security" tag returned
- Verify: Tag matching is exact

**TC-2.6: Filter by source**
- Setup: Add observations from different tools
- Input: list_observations({ source: "wine_recommender" })
- Expected: Only observations from wine_recommender returned
- Verify: Other sources excluded

**TC-2.7: Multiple filters combined**
- Setup: Add mixed observations
- Input: list_observations({ type: "risk_alert", status: "active", tags: ["fraud"] })
- Expected: Only observations matching all filters returned
- Verify: Filters are AND-ed together

**TC-2.8: Empty result set**
- Setup: Add observations but none match filter
- Input: list_observations({ type: "nonexistent_type" })
- Expected: Empty array returned
- Verify: No error, just empty result

### Test Suite 3: get_observation()

**TC-3.1: Get existing observation**
- Setup: Add observation with known id
- Input: get_observation(id)
- Expected: Observation returned
- Verify: All fields match original

**TC-3.2: Get non-existent observation**
- Input: get_observation("nonexistent_id")
- Expected: null returned
- Verify: No error thrown

### Test Suite 4: update_observation()

**TC-4.1: Update status**
- Setup: Add observation with status="active"
- Input: update_observation(id, { status: "resolved" })
- Expected: Observation updated, updated_at changed
- Verify: Status is now "resolved", created_at unchanged

**TC-4.2: Update content**
- Setup: Add observation
- Input: update_observation(id, { content: "new content" })
- Expected: Content updated
- Verify: New content is retrievable

**TC-4.3: Update metadata**
- Setup: Add observation with metadata
- Input: update_observation(id, { metadata: { key: "new_value" } })
- Expected: Metadata updated
- Verify: New metadata is retrievable

**TC-4.4: Reject invalid status transition**
- Setup: Add observation with status="archived"
- Input: update_observation(id, { status: "active" })
- Expected: Error returned, status unchanged
- Verify: Error indicates invalid transition

**TC-4.5: Reject update to immutable fields**
- Setup: Add observation
- Input: update_observation(id, { id: "new_id", created_at: "2025-01-01" })
- Expected: Error returned, fields unchanged
- Verify: Error indicates immutable fields

**TC-4.6: Update confidence**
- Setup: Add observation with confidence=0.5
- Input: update_observation(id, { confidence: 0.95 })
- Expected: Confidence updated
- Verify: New confidence is retrievable

### Test Suite 5: archive_observation()

**TC-5.1: Archive active observation**
- Setup: Add observation with status="active"
- Input: archive_observation(id)
- Expected: Status changed to "archived"
- Verify: Observation no longer in active list

**TC-5.2: Archive non-existent observation**
- Input: archive_observation("nonexistent_id")
- Expected: Error returned
- Verify: Error indicates not found

### Test Suite 6: clear_resolved()

**TC-6.1: Clear single resolved observation**
- Setup: Add observation with status="resolved"
- Input: clear_resolved([id])
- Expected: Observation removed
- Verify: get_observation(id) returns null

**TC-6.2: Clear multiple resolved observations**
- Setup: Add 3 resolved observations
- Input: clear_resolved([id1, id2, id3])
- Expected: All 3 removed
- Verify: All return null on get

**TC-6.3: Reject clearing active observation**
- Setup: Add observation with status="active"
- Input: clear_resolved([id])
- Expected: Error returned, observation not removed
- Verify: Observation still retrievable

**TC-6.4: Clear with mixed statuses**
- Setup: Add 2 resolved, 1 active
- Input: clear_resolved([resolved1, active, resolved2])
- Expected: Error returned, no observations removed
- Verify: All observations still present

---

## Lifecycle Tests

### Test Suite 7: Observation Lifecycle

**TC-7.1: Complete lifecycle**
- Add observation (status="active")
- Update observation (status="pending_review")
- Update observation (status="resolved")
- Archive observation (status="archived")
- Verify: All transitions succeed

**TC-7.2: Expiration handling**
- Setup: Add observation with expires_at = now + 1 second
- Wait 2 seconds
- Input: list_observations({})
- Expected: Expired observation not returned
- Verify: Observation is automatically removed or marked expired

**TC-7.3: Auto-archive resolved**
- Setup: Configuration with auto_archive_resolved=true
- Add observation, update to resolved
- Wait for auto-archive trigger
- Input: list_observations({ status: "active" })
- Expected: Resolved observation not in active list
- Verify: Observation is archived

---

## Integration Tests

### Test Suite 8: Tool Integration

**TC-8.1: Tool adds observation**
- Setup: Tool execution context
- Tool calls: add_observation({ type: "contextual_insight", ... })
- Expected: Observation created with source=tool_name
- Verify: Observation retrievable by list_observations()

**TC-8.2: Tool reads observations**
- Setup: Add observations with relevant tags
- Tool calls: list_observations({ tags: ["wine"] })
- Expected: Relevant observations returned
- Verify: Tool can use observations in decision-making

### Test Suite 9: Daemon Integration

**TC-9.1: Daemon adds pending thought**
- Setup: Daemon execution context
- Daemon calls: add_observation({ type: "observation", content: "pending thought" })
- Expected: Observation created
- Verify: Observation retrievable

**TC-9.2: Router uses observations in urgency assessment**
- Setup: Add risk_alert observation
- Router decision-making
- Expected: Router considers alert in urgency assessment
- Verify: HIGH urgency assigned when alert present

### Test Suite 10: Memory Integration

**TC-10.1: Memory surfaces relevant observations**
- Setup: Add contextual_insight observations
- Memory system calls: list_observations({ tags: ["wine"] })
- Expected: Relevant observations returned
- Verify: Observations included in context assembly

---

## Concurrency Tests

### Test Suite 11: Concurrent Access

**TC-11.1: Concurrent adds**
- Setup: Multiple threads/processes
- Each adds observation simultaneously
- Expected: All observations created with unique ids
- Verify: No duplicates, no lost data

**TC-11.2: Concurrent read/write**
- Setup: One thread adds, another reads
- Expected: Reads see consistent state
- Verify: No partial updates visible

**TC-11.3: Concurrent updates**
- Setup: Multiple threads update same observation
- Expected: Last-write-wins or error
- Verify: Consistent final state

---

## Edge Cases

### Test Suite 12: Edge Cases

**TC-12.1: Very long content**
- Input: Observation with 10,000 character content
- Expected: Observation created and stored
- Verify: Content retrievable unchanged

**TC-12.2: Many observations**
- Setup: Add 1,000 observations
- Input: list_observations({})
- Expected: All returned (or paginated)
- Verify: Performance acceptable

**TC-12.3: Special characters in content**
- Input: Observation with unicode, emojis, special chars
- Expected: Observation created
- Verify: Content retrievable unchanged

**TC-12.4: Null/undefined fields**
- Input: ObservationItem with null optional fields
- Expected: Observation created with defaults
- Verify: Null fields replaced with defaults

---

## Acceptance Criteria

- All test suites pass with 100% success rate
- No data loss under concurrent access
- Lifecycle transitions are enforced
- Filters work correctly in isolation and combination
- Integration with tools and daemons works as specified
- Performance is acceptable for 1,000+ observations

