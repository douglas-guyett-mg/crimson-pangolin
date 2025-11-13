# System Change Proposals - Test Conditions

**Purpose:** Define comprehensive test scenarios for the System Change Proposals system, including Change Flagging Daemon, System Analyzer Daemon, and integration tests.

**Status:** v1.0 (2025-11-05)

---

## Change Flagging Daemon Tests

### Scenario 1.1: FC suggests new action type (happy path)
**Given:** Frontal Cortex daemon detects missing action type during turn execution  
**When:** FC calls `flag_change_needed(flag_type: "new_action_type", details: {...}, severity: "info", rationale: "...")`  
**Then:** Flag is created with all required fields

**Test Data:**
```json
{
  "flag_type": "new_action_type",
  "details": {
    "action_type_name": "parallel_task",
    "description": "Execute multiple tasks in parallel",
    "reason": "User asked to run 5 research tasks simultaneously",
    "suggested_policies": {
      "requires_confirmation": false,
      "auto_complete": true
    }
  },
  "severity": "info",
  "rationale": "Current action types don't support parallel execution"
}
```

**Acceptance Criteria:**
- [ ] Flag ID is unique and follows format "flag_YYYYMMDD_NNN"
- [ ] flag_type equals "new_action_type"
- [ ] details.action_type_name equals "parallel_task"
- [ ] severity equals "info"
- [ ] source_daemon equals "frontal_cortex"
- [ ] timestamp is ISO8601 format and within 1 second of current time
- [ ] context.turn_id is populated with current turn ID
- [ ] context.rationale matches input rationale
- [ ] Flag is added to turn_trace.system_change_flags array
- [ ] Flag object is valid JSON and matches schema

**Verification:**
- Query turn_trace.system_change_flags and verify flag exists
- Validate flag structure against change-needed-flag schema
- Confirm flag_id is unique across all flags in turn
- Verify timestamp is recent (within 1 second)
- Check that all required fields are present (no null/undefined)

---

### Scenario 1.2: Executor suggests new tool (happy path)
**Given:** Executor daemon needs a tool that doesn't exist  
**When:** Executor calls `flag_change_needed(flag_type: "new_tool", details: {...}, severity: "warning", rationale: "...")`  
**Then:** Flag is created with tool-specific details

**Test Data:**
```json
{
  "flag_type": "new_tool",
  "details": {
    "tool_name": "pdf_analyzer",
    "tool_purpose": "Extract text, tables, and metadata from PDF files",
    "use_case": "User uploaded a PDF and asked for analysis",
    "suggested_inputs": ["pdf_file_path", "extraction_type"],
    "suggested_outputs": ["text_content", "tables", "metadata"]
  },
  "severity": "warning",
  "rationale": "User needs PDF analysis capability"
}
```

**Acceptance Criteria:**
- [ ] flag_type equals "new_tool"
- [ ] details.tool_name equals "pdf_analyzer"
- [ ] details.suggested_inputs is array with 2 elements
- [ ] details.suggested_outputs is array with 3 elements
- [ ] severity equals "warning"
- [ ] source_daemon equals "executor"
- [ ] context.error_message is populated (tool not found)
- [ ] Flag is stored in turn trace

**Verification:**
- Query flag and verify all tool-specific fields present
- Validate suggested_inputs and suggested_outputs are arrays
- Confirm error_message indicates tool not found
- Check flag persists after turn execution

---

### Scenario 1.3: Multiple flags in single turn
**Given:** Multiple daemons suggest changes in same turn  
**When:** FC flags new action type AND Executor flags new tool in same turn  
**Then:** Both flags are created and stored

**Test Data:**
```
Turn execution:
  - FC calls flag_change_needed(flag_type: "new_action_type", ...)
  - Executor calls flag_change_needed(flag_type: "new_tool", ...)
```

**Acceptance Criteria:**
- [ ] turn_trace.system_change_flags array has 2 elements
- [ ] First flag has flag_type "new_action_type"
- [ ] Second flag has flag_type "new_tool"
- [ ] Both flags have unique flag_ids
- [ ] Both flags have same turn_id
- [ ] Both flags have different source_daemons
- [ ] Flags are in order they were created (FIFO)

**Verification:**
- Query turn_trace.system_change_flags and count elements
- Verify flag_ids are unique
- Confirm both flags have correct turn_id
- Check ordering matches creation order

---

### Scenario 1.4: Flag with critical severity
**Given:** Daemon detects blocking issue that prevents task completion  
**When:** Daemon calls `flag_change_needed(..., severity: "critical", ...)`  
**Then:** Flag is created with critical severity

**Test Data:**
```json
{
  "flag_type": "capability_gap",
  "details": {
    "capability_name": "image_generation",
    "description": "Generate images from text descriptions",
    "use_case": "User asked to create an image but capability doesn't exist"
  },
  "severity": "critical",
  "rationale": "User task cannot complete without this capability"
}
```

**Acceptance Criteria:**
- [ ] severity equals "critical"
- [ ] Flag is marked as blocking
- [ ] Flag is prioritized for System Analyzer processing
- [ ] Flag includes context about why it's blocking

**Verification:**
- Query flag and verify severity is "critical"
- Confirm flag is included in high-priority queue
- Check that System Analyzer processes critical flags first

---

### Scenario 1.5: Malformed flag - missing required fields
**Given:** Daemon calls flag_change_needed() with missing required fields  
**When:** flag_change_needed(flag_type: "new_action_type", details: {}, severity: null, rationale: "")  
**Then:** Flag creation fails with validation error

**Acceptance Criteria:**
- [ ] Flag is NOT created
- [ ] Validation error is returned to calling daemon
- [ ] Error message indicates which fields are missing
- [ ] Turn execution continues (error is non-blocking)
- [ ] Error is logged in turn trace

**Verification:**
- Attempt to create flag with missing fields
- Verify flag is not added to turn_trace.system_change_flags
- Check error is returned to daemon
- Confirm turn execution completes despite error

---

### Scenario 1.6: Duplicate flag in same turn
**Given:** Same daemon flags same change twice in one turn  
**When:** FC calls flag_change_needed(flag_type: "new_action_type", action_type_name: "parallel_task") twice  
**Then:** Both flags are created (deduplication happens in System Analyzer, not here)

**Acceptance Criteria:**
- [ ] Both flags are created with unique flag_ids
- [ ] Both flags have same flag_type and details
- [ ] Both flags have same turn_id
- [ ] Both flags have same source_daemon
- [ ] Timestamps differ slightly (milliseconds)

**Verification:**
- Query turn_trace.system_change_flags and verify 2 flags exist
- Confirm flag_ids are different
- Check timestamps are different
- Verify both flags have identical details

---

### Scenario 1.7: Flag persistence across turn boundaries
**Given:** Flag is created during turn execution  
**When:** Turn completes and new turn begins  
**Then:** Flag persists in turn trace and is available to System Analyzer

**Acceptance Criteria:**
- [ ] Flag is stored in persistent turn trace storage
- [ ] Flag is retrievable after turn execution completes
- [ ] Flag is retrievable in subsequent turns
- [ ] Flag data is not modified or corrupted
- [ ] Flag can be queried by turn_id and flag_id

**Verification:**
- Create flag in turn N
- Complete turn N
- Query turn trace in turn N+1 and verify flag exists
- Confirm flag data matches original
- Verify flag can be retrieved by both turn_id and flag_id

---

### Scenario 1.8: Flag with all optional fields populated
**Given:** Daemon provides all optional fields in flag  
**When:** flag_change_needed() called with complete details including suggested_documentation  
**Then:** All fields are stored correctly

**Test Data:**
```json
{
  "flag_type": "new_action_type",
  "details": { ... },
  "severity": "warning",
  "rationale": "...",
  "suggested_documentation": {
    "file_path": "documentation/frontal-cortex/action-types/parallel_task.md",
    "sections": ["Overview", "Use Cases", "Policies", "Examples"]
  }
}
```

**Acceptance Criteria:**
- [ ] All optional fields are stored
- [ ] suggested_documentation is preserved exactly
- [ ] No fields are truncated or modified
- [ ] Flag structure remains valid

**Verification:**
- Query flag and verify all optional fields present
- Confirm suggested_documentation matches input
- Validate complete flag structure

---

### Scenario 1.9: Rapid flag creation (stress test)
**Given:** Multiple daemons create flags simultaneously  
**When:** 10 daemons each call flag_change_needed() within 100ms  
**Then:** All flags are created without loss or corruption

**Acceptance Criteria:**
- [ ] All 10 flags are created
- [ ] All flag_ids are unique
- [ ] No flags are lost or corrupted
- [ ] Timestamps are accurate (within milliseconds)
- [ ] turn_trace.system_change_flags has exactly 10 elements

**Verification:**
- Create 10 flags in rapid succession
- Query turn_trace and count flags
- Verify all flag_ids are unique
- Confirm no data corruption
- Check timestamps are sequential

---

### Scenario 1.10: Flag with user context
**Given:** Flag is created with user_id and conversation_id  
**When:** flag_change_needed() called with context including user_id and conversation_id  
**Then:** Context is stored and available for proposal generation

**Test Data:**
```json
{
  "context": {
    "turn_id": "turn_20251104_042",
    "user_id": "user_123",
    "conversation_id": "conv_456",
    "action_attempted": "Create parallel research tasks",
    "rationale": "User needs parallel execution"
  }
}
```

**Acceptance Criteria:**
- [ ] context.user_id is stored
- [ ] context.conversation_id is stored
- [ ] context.action_attempted is stored
- [ ] All context fields are retrievable
- [ ] Context is used by System Analyzer for evidence aggregation

**Verification:**
- Query flag and verify all context fields present
- Confirm user_id and conversation_id are correct
- Check that System Analyzer can retrieve and use context

---

## System Analyzer Daemon Tests

### Scenario 2.1: Read flagged changes from single turn
**Given:** Turn trace contains 1 flag  
**When:** System Analyzer calls `query_flagged_changes()`  
**Then:** Flag is retrieved and ready for processing

**Acceptance Criteria:**
- [ ] Flag is retrieved from turn trace
- [ ] Flag structure is valid
- [ ] All required fields are present
- [ ] Flag data is not modified during retrieval
- [ ] Query completes within 1 second

**Verification:**
- Call query_flagged_changes()
- Verify returned flag matches stored flag
- Confirm all fields are present
- Check query performance

---

### Scenario 2.2: Read flagged changes from multiple turns
**Given:** Turn traces contain 5 flags across 3 different turns  
**When:** System Analyzer calls `query_flagged_changes()`  
**Then:** All 5 flags are retrieved

**Acceptance Criteria:**
- [ ] All 5 flags are returned
- [ ] Flags are grouped by turn_id
- [ ] Flags maintain their original structure
- [ ] Query includes metadata about source turns
- [ ] Query completes within 2 seconds

**Verification:**
- Create flags in 3 different turns
- Call query_flagged_changes()
- Verify all 5 flags are returned
- Confirm grouping by turn_id
- Check query performance

---

### Scenario 2.3: Filter flagged changes by flag_type
**Given:** Turn traces contain 3 "new_action_type" flags and 2 "new_tool" flags  
**When:** System Analyzer calls `query_flagged_changes(flag_type: "new_action_type")`  
**Then:** Only 3 "new_action_type" flags are returned

**Acceptance Criteria:**
- [ ] Exactly 3 flags are returned
- [ ] All returned flags have flag_type "new_action_type"
- [ ] "new_tool" flags are excluded
- [ ] Filter is case-sensitive
- [ ] Query completes within 1 second

**Verification:**
- Create mixed flags
- Query with flag_type filter
- Verify only matching flags returned
- Confirm count is correct

---

### Scenario 2.4: Filter flagged changes by severity
**Given:** Turn traces contain 2 "info", 3 "warning", 1 "critical" flags  
**When:** System Analyzer calls `query_flagged_changes(severity: "critical")`  
**Then:** Only 1 "critical" flag is returned

**Acceptance Criteria:**
- [ ] Exactly 1 flag is returned
- [ ] Returned flag has severity "critical"
- [ ] Other severity flags are excluded
- [ ] Query completes within 1 second

**Verification:**
- Create flags with different severities
- Query with severity filter
- Verify only critical flag returned

---

### Scenario 2.5: Filter flagged changes by date range
**Given:** Flags created on 2025-11-01, 2025-11-02, 2025-11-03, 2025-11-04  
**When:** System Analyzer calls `query_flagged_changes(since: "2025-11-02", until: "2025-11-03")`  
**Then:** Only flags from 2025-11-02 and 2025-11-03 are returned

**Acceptance Criteria:**
- [ ] Exactly 2 flags are returned (from 2 days)
- [ ] Flags from 2025-11-01 are excluded
- [ ] Flags from 2025-11-04 are excluded
- [ ] Date filtering is inclusive on both ends
- [ ] Query completes within 1 second

**Verification:**
- Create flags with different timestamps
- Query with date range
- Verify correct flags returned
- Confirm date boundaries are inclusive

---

### Scenario 2.6: Exact match deduplication
**Given:** Proposal already exists for "parallel_task" action type  
**When:** System Analyzer processes new flag for "parallel_task"  
**Then:** Existing proposal is updated with new evidence, no new proposal created

**Test Data:**
```
Existing proposal:
  - proposal_id: "prop_20251101_001"
  - proposal_type: "new_action_type"
  - details.action_type_name: "parallel_task"
  - evidence.total_flags: 1

New flag:
  - flag_type: "new_action_type"
  - details.action_type_name: "parallel_task"
```

**Acceptance Criteria:**
- [ ] Existing proposal is found
- [ ] New proposal is NOT created
- [ ] Existing proposal.evidence.total_flags is incremented to 2
- [ ] New flag is added to affected_turns
- [ ] Proposal.last_flagged is updated to new timestamp
- [ ] Proposal status remains "pending"

**Verification:**
- Query proposals before and after
- Verify proposal count doesn't increase
- Confirm evidence is updated
- Check affected_turns includes new flag

---

### Scenario 2.7: Similar match deduplication
**Given:** Proposal exists for "parallel_task" action type  
**When:** System Analyzer processes flag for "concurrent_task" (similar purpose)  
**Then:** New proposal is created but linked as related_proposals

**Acceptance Criteria:**
- [ ] New proposal is created with proposal_id "prop_20251104_002"
- [ ] New proposal.related_proposals includes "prop_20251101_001"
- [ ] Existing proposal.related_proposals includes "prop_20251104_002"
- [ ] Both proposals have status "pending"
- [ ] Note indicates similarity

**Verification:**
- Create new proposal
- Query both proposals
- Verify related_proposals links exist
- Confirm both proposals are independent

---

### Scenario 2.8: No match deduplication
**Given:** No proposal exists for "image_generation" capability  
**When:** System Analyzer processes flag for "image_generation"  
**Then:** New proposal is created

**Acceptance Criteria:**
- [ ] New proposal is created
- [ ] proposal_id is unique
- [ ] proposal_type is "capability_gap"
- [ ] status is "pending"
- [ ] related_proposals is empty array
- [ ] created_at is current timestamp

**Verification:**
- Query proposals before and after
- Verify new proposal exists
- Confirm proposal_id is unique
- Check status is "pending"

---

### Scenario 2.9: Aggregate evidence from multiple flags
**Given:** 3 flags for "parallel_task" from different users and turns  
**When:** System Analyzer generates proposal  
**Then:** Evidence aggregates all flags

**Test Data:**
```
Flag 1: severity "info", user_123, turn_20251101_042
Flag 2: severity "warning", user_456, turn_20251103_015
Flag 3: severity "warning", user_789, turn_20251104_043
```

**Acceptance Criteria:**
- [ ] evidence.total_flags equals 3
- [ ] evidence.severity_distribution.info equals 1
- [ ] evidence.severity_distribution.warning equals 2
- [ ] evidence.affected_users includes all 3 users
- [ ] evidence.affected_turns includes all 3 turns
- [ ] evidence.first_flagged is earliest timestamp
- [ ] evidence.last_flagged is latest timestamp
- [ ] evidence.source_daemons includes all source daemons

**Verification:**
- Generate proposal from 3 flags
- Query proposal.evidence
- Verify all aggregations are correct
- Confirm counts match input

---

### Scenario 2.10: Generate proposal with all required fields
**Given:** Flags for new action type  
**When:** System Analyzer generates proposal  
**Then:** Proposal includes all required fields

**Acceptance Criteria:**
- [ ] proposal_id is present and unique
- [ ] proposal_type is present
- [ ] title is present and descriptive
- [ ] description is present
- [ ] rationale is present
- [ ] evidence is present with all sub-fields
- [ ] details is present with type-specific fields
- [ ] suggested_documentation is present
- [ ] status equals "pending"
- [ ] created_at is ISO8601 timestamp
- [ ] created_by equals "system_analyzer"

**Verification:**
- Generate proposal
- Validate against proposal schema
- Confirm all required fields present
- Check no null/undefined values

---

### Scenario 2.11: Store proposal persistently
**Given:** Proposal is generated  
**When:** System Analyzer calls `store_proposal(proposal)`  
**Then:** Proposal is persisted and retrievable

**Acceptance Criteria:**
- [ ] Proposal is stored in persistent storage
- [ ] Proposal is assigned proposal_id
- [ ] Proposal is retrievable by proposal_id
- [ ] Proposal data is not modified during storage
- [ ] Proposal is available for human review
- [ ] Storage completes within 1 second

**Verification:**
- Store proposal
- Query by proposal_id
- Verify data matches original
- Confirm proposal is accessible

---

### Scenario 2.12: Handle empty flag list
**Given:** No flags exist in turn traces  
**When:** System Analyzer calls `query_flagged_changes()`  
**Then:** Empty list is returned, no errors

**Acceptance Criteria:**
- [ ] Empty array is returned
- [ ] No error is thrown
- [ ] Query completes successfully
- [ ] System Analyzer handles gracefully

**Verification:**
- Call query_flagged_changes() with no flags
- Verify empty array returned
- Confirm no errors logged

---

### Scenario 2.13: Handle conflicting proposals
**Given:** Two proposals exist with conflicting recommendations  
**When:** System Analyzer detects conflict  
**Then:** Conflict is flagged and both proposals are marked

**Test Data:**
```
Proposal 1: "parallel_task should auto-complete"
Proposal 2: "parallel_task should require confirmation"
```

**Acceptance Criteria:**
- [ ] Conflict is detected
- [ ] Both proposals are marked with conflict flag
- [ ] Conflict is documented in proposal.review_notes
- [ ] Human is alerted to review conflict
- [ ] Neither proposal is auto-approved

**Verification:**
- Create conflicting proposals
- Run System Analyzer
- Verify conflict is detected
- Check both proposals are marked

---

### Scenario 2.14: Rapid proposal generation (stress test)
**Given:** 100 flags across 50 turns  
**When:** System Analyzer processes all flags  
**Then:** All proposals are generated without loss or corruption

**Acceptance Criteria:**
- [ ] All proposals are generated
- [ ] All proposal_ids are unique
- [ ] No data is lost or corrupted
- [ ] Processing completes within 10 seconds
- [ ] Evidence aggregation is accurate

**Verification:**
- Create 100 flags
- Run System Analyzer
- Verify all proposals created
- Confirm proposal_ids are unique
- Check processing time

---

### Scenario 2.15: Proposal with suggested documentation
**Given:** Flag includes suggested_documentation  
**When:** System Analyzer generates proposal  
**Then:** Suggested documentation is included in proposal

**Test Data:**
```
Flag suggested_documentation:
  - file_path: "documentation/frontal-cortex/action-types/parallel_task.md"
  - sections: ["Overview", "Use Cases", "Policies", "Examples"]
```

**Acceptance Criteria:**
- [ ] proposal.suggested_documentation is present
- [ ] file_path is included
- [ ] sections array is included
- [ ] Documentation structure is helpful for human writer
- [ ] All sections are actionable

**Verification:**
- Generate proposal from flag with suggested_documentation
- Query proposal
- Verify suggested_documentation is present and complete

---

## Integration Tests

### Scenario 3.1: End-to-end flow - single flag to proposal
**Given:** Turn execution with FC suggesting new action type  
**When:** Turn completes and System Analyzer runs  
**Then:** Proposal is generated and available for human review

**Flow:**
```
1. Turn starts
2. FC detects missing "parallel_task" action type
3. Change Flagging Daemon creates flag
4. Flag stored in turn trace
5. Turn completes
6. System Analyzer runs
7. System Analyzer reads flag
8. System Analyzer generates proposal
9. Proposal stored for human review
```

**Acceptance Criteria:**
- [ ] Flag is created during turn
- [ ] Flag persists after turn
- [ ] System Analyzer retrieves flag
- [ ] Proposal is generated
- [ ] Proposal is stored
- [ ] Proposal is accessible for review
- [ ] End-to-end flow completes within 5 seconds

**Verification:**
- Execute full flow
- Verify flag exists after turn
- Confirm proposal exists after System Analyzer runs
- Check proposal is accessible

---

### Scenario 3.2: Multiple flags aggregated into single proposal
**Given:** 3 turns with same flag type  
**When:** System Analyzer processes all flags  
**Then:** Single proposal is generated with aggregated evidence

**Flow:**
```
Turn 1: FC flags "parallel_task"
Turn 2: FC flags "parallel_task"
Turn 3: FC flags "parallel_task"
System Analyzer runs
→ Single proposal with evidence.total_flags = 3
```

**Acceptance Criteria:**
- [ ] 3 flags are created across 3 turns
- [ ] System Analyzer finds all 3 flags
- [ ] Single proposal is generated (not 3)
- [ ] Proposal.evidence.total_flags equals 3
- [ ] Proposal.affected_turns includes all 3 turns
- [ ] Proposal.affected_users includes all users

**Verification:**
- Create 3 flags in different turns
- Run System Analyzer
- Verify single proposal created
- Confirm evidence aggregation

---

### Scenario 3.3: Proposal lifecycle - pending to approved
**Given:** Proposal is generated and pending  
**When:** Human reviews and approves proposal  
**Then:** Proposal status changes to "approved"

**Flow:**
```
1. Proposal generated with status "pending"
2. Human reviews proposal
3. Human approves proposal
4. Proposal status updated to "approved"
5. Proposal.reviewed_by is set
6. Proposal.reviewed_at is set
7. Proposal.review_notes is populated
```

**Acceptance Criteria:**
- [ ] Proposal starts with status "pending"
- [ ] Human can review proposal
- [ ] Proposal status can be updated to "approved"
- [ ] reviewed_by is set to human identifier
- [ ] reviewed_at is set to current timestamp
- [ ] review_notes can be added
- [ ] Approved proposal is available for implementation

**Verification:**
- Create proposal
- Update status to "approved"
- Verify all fields updated
- Confirm proposal is marked for implementation

---

### Scenario 3.4: Proposal lifecycle - pending to rejected
**Given:** Proposal is generated and pending  
**When:** Human reviews and rejects proposal  
**Then:** Proposal status changes to "rejected"

**Acceptance Criteria:**
- [ ] Proposal status changes to "rejected"
- [ ] reviewed_by is set
- [ ] reviewed_at is set
- [ ] review_notes explains rejection reason
- [ ] Rejected proposal is archived

**Verification:**
- Create proposal
- Update status to "rejected"
- Verify rejection is recorded
- Confirm proposal is archived

---

### Scenario 3.5: Proposal lifecycle - approved to implemented
**Given:** Proposal is approved  
**When:** Implementation is completed  
**Then:** Proposal status changes to "implemented"

**Acceptance Criteria:**
- [ ] Proposal status changes to "implemented"
- [ ] implementation_pr is set to PR link
- [ ] implementation_status is set to "completed"
- [ ] Implemented proposal is marked as complete
- [ ] Related proposals are notified

**Verification:**
- Approve proposal
- Update status to "implemented"
- Set implementation_pr
- Verify proposal is marked complete

---

### Scenario 3.6: Turn trace integration - flags persist across system boundaries
**Given:** Flag is created in turn execution  
**When:** Turn completes and System Analyzer queries turn traces  
**Then:** Flag is retrievable and unchanged

**Acceptance Criteria:**
- [ ] Flag is stored in turn trace
- [ ] Flag persists after turn execution
- [ ] Flag is retrievable by System Analyzer
- [ ] Flag data is not modified
- [ ] Flag can be queried by turn_id, flag_id, flag_type

**Verification:**
- Create flag in turn
- Complete turn
- Query flag from System Analyzer
- Verify data integrity

---

### Scenario 3.7: Multiple proposal types in single analysis run
**Given:** Flags for new_action_type, new_tool, and tool_enhancement  
**When:** System Analyzer processes all flags  
**Then:** Proposals are generated for each type

**Acceptance Criteria:**
- [ ] Proposal for new_action_type is generated
- [ ] Proposal for new_tool is generated
- [ ] Proposal for tool_enhancement is generated
- [ ] Each proposal has correct proposal_type
- [ ] Each proposal has type-specific details
- [ ] All proposals are stored

**Verification:**
- Create flags of different types
- Run System Analyzer
- Verify all proposals created
- Confirm each has correct type

---

### Scenario 3.8: Deduplication across multiple analysis runs
**Given:** Proposal exists from previous analysis run  
**When:** System Analyzer runs again with same flag type  
**Then:** Existing proposal is updated, not duplicated

**Acceptance Criteria:**
- [ ] Existing proposal is found
- [ ] New proposal is NOT created
- [ ] Existing proposal is updated with new evidence
- [ ] Proposal count remains same
- [ ] Evidence is aggregated correctly

**Verification:**
- Run System Analyzer (creates proposal)
- Run System Analyzer again (same flags)
- Verify proposal count is 1
- Confirm evidence is updated

---

### Scenario 3.9: Error handling - malformed flag in turn trace
**Given:** Turn trace contains malformed flag  
**When:** System Analyzer processes flags  
**Then:** Error is handled gracefully

**Acceptance Criteria:**
- [ ] Malformed flag is skipped
- [ ] Error is logged
- [ ] Other flags are still processed
- [ ] System Analyzer completes successfully
- [ ] Error report is generated

**Verification:**
- Create malformed flag
- Run System Analyzer
- Verify error is logged
- Confirm other flags processed

---

### Scenario 3.10: Performance - System Analyzer processes 1000 flags
**Given:** 1000 flags across 500 turns  
**When:** System Analyzer processes all flags  
**Then:** Processing completes within acceptable time

**Acceptance Criteria:**
- [ ] All 1000 flags are processed
- [ ] Processing completes within 30 seconds
- [ ] Memory usage is reasonable
- [ ] No data loss or corruption
- [ ] Proposals are generated correctly

**Verification:**
- Create 1000 flags
- Run System Analyzer
- Measure processing time
- Verify all flags processed
- Confirm memory usage

---

## Summary

**Total Test Scenarios:** 40+
- Change Flagging Daemon: 10 scenarios
- System Analyzer Daemon: 15 scenarios
- Integration Tests: 10 scenarios

**Coverage Areas:**
- ✅ Happy path scenarios
- ✅ Error handling and edge cases
- ✅ Data validation
- ✅ Persistence and retrieval
- ✅ Deduplication logic
- ✅ Evidence aggregation
- ✅ Proposal generation
- ✅ Integration flows
- ✅ Performance and stress tests
- ✅ Lifecycle management

**Implementation Notes:**
- All scenarios include detailed test data
- All scenarios include specific acceptance criteria
- All scenarios include verification steps
- Tests should be automated where possible
- Performance tests should be run regularly

