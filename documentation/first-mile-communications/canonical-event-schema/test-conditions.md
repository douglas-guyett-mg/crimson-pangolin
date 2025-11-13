# Si First-Mile Communications — Canonical Event Schema Test Conditions

## Purpose
Enumerate measurable verification criteria to ensure canonical events conform to the schema and maintain data integrity across all channels and event types.

## Test Suite: Core Field Validation

### Test 1.1: Required Core Fields Are Present
**Objective**: Verify that all required core fields are present in every canonical event

**Setup**:
- Create canonical events from each channel (messaging, browser, mobile)
- Create events of each type (message, audio_frame, control, compliance)

**Steps**:
1. For each event, verify presence of: session_id, event_id, timestamp, event_type, participant_id, channel
2. Verify no required fields are missing or null

**Expected Outcome**:
- All required core fields are present
- No required fields are null or empty

**Verification**:
- session_id is present and non-empty
- event_id is present and non-empty
- timestamp is present and non-empty
- event_type is present and non-empty
- participant_id is present and non-empty
- channel is present and non-empty

---

### Test 1.2: session_id Is Valid UUID
**Objective**: Verify that session_id conforms to UUID format

**Setup**:
- Create canonical events with various session_ids

**Steps**:
1. For each event, validate session_id format
2. Verify session_id is valid UUID v4 or v5

**Expected Outcome**:
- All session_ids are valid UUIDs
- session_ids follow consistent format

**Verification**:
- session_id matches UUID regex pattern
- session_id can be parsed as UUID

---

### Test 1.3: event_id Is Unique
**Objective**: Verify that each event has a unique event_id

**Setup**:
- Create 1000 canonical events across all channels and types

**Steps**:
1. Collect all event_ids
2. Check for duplicates

**Expected Outcome**:
- All event_ids are unique
- No duplicates exist

**Verification**:
- Number of unique event_ids equals total events
- No event_id appears more than once

---

### Test 1.4: timestamp Is Valid ISO 8601
**Objective**: Verify that timestamp conforms to ISO 8601 format with timezone

**Setup**:
- Create canonical events with various timestamps

**Steps**:
1. For each event, validate timestamp format
2. Verify timezone is present

**Expected Outcome**:
- All timestamps are valid ISO 8601
- All timestamps include timezone information

**Verification**:
- timestamp matches ISO 8601 pattern with timezone
- timestamp can be parsed as datetime

---

### Test 1.5: event_type Is Valid Enumeration
**Objective**: Verify that event_type is one of the allowed values

**Setup**:
- Create events with various event_types

**Steps**:
1. For each event, verify event_type is in allowed set
2. Attempt to create event with invalid event_type

**Expected Outcome**:
- Valid event_types are accepted
- Invalid event_types are rejected

**Verification**:
- event_type is one of: "message", "audio_frame", "control", "compliance"
- Invalid event_types cause validation failure

---

### Test 1.6: channel Is Valid Enumeration
**Objective**: Verify that channel is one of the allowed values

**Setup**:
- Create events from each channel

**Steps**:
1. For each event, verify channel is in allowed set
2. Attempt to create event with invalid channel

**Expected Outcome**:
- Valid channels are accepted
- Invalid channels are rejected

**Verification**:
- channel is one of: "messaging", "browser", "mobile"
- Invalid channels cause validation failure

---

## Test Suite: Content Field Validation

### Test 2.1: Message Events Have text_content
**Objective**: Verify that message events contain text_content

**Setup**:
- Create message events from each channel

**Steps**:
1. For each message event, verify text_content is present
2. Verify text_content is non-empty

**Expected Outcome**:
- All message events have text_content
- text_content is non-empty

**Verification**:
- text_content is present and non-empty
- text_content is a string

---

### Test 2.2: Audio Frame Events Have audio_metadata
**Objective**: Verify that audio_frame events contain audio_metadata

**Setup**:
- Create audio_frame events from browser and mobile

**Steps**:
1. For each audio_frame event, verify audio_metadata is present
2. Verify audio_metadata contains required subfields

**Expected Outcome**:
- All audio_frame events have audio_metadata
- audio_metadata contains: format, sample_rate, duration_ms, encoding

**Verification**:
- audio_metadata is present
- audio_metadata.format is present and valid
- audio_metadata.sample_rate is present and numeric
- audio_metadata.duration_ms is present and numeric
- audio_metadata.encoding is present

---

### Test 2.3: modality Is Valid Enumeration
**Objective**: Verify that modality is one of the allowed values

**Setup**:
- Create events with various modalities

**Steps**:
1. For each event, verify modality is in allowed set
2. Verify modality matches event_type

**Expected Outcome**:
- modality is one of: "text", "audio", "blended"
- modality is consistent with event_type

**Verification**:
- modality is one of: "text", "audio", "blended"
- Message events have modality "text" or "blended"
- Audio frame events have modality "audio" or "blended"

---

## Test Suite: Metadata Preservation

### Test 3.1: Channel Metadata Is Preserved
**Objective**: Verify that channel-specific metadata is preserved

**Setup**:
- Create events from each channel with channel-specific metadata

**Steps**:
1. Create event with channel_metadata
2. Verify channel_metadata is preserved in output

**Expected Outcome**:
- Channel metadata is preserved unchanged
- Channel metadata is accessible to downstream systems

**Verification**:
- metadata.channel_metadata is present
- metadata.channel_metadata contains expected fields

---

### Test 3.2: Compliance Metadata Is Present
**Objective**: Verify that compliance metadata is present and valid

**Setup**:
- Create events with compliance metadata

**Steps**:
1. For each event, verify compliance metadata is present
2. Verify compliance fields are valid

**Expected Outcome**:
- All events have compliance metadata
- Compliance fields are valid

**Verification**:
- metadata.compliance is present
- metadata.compliance.consent_level is present
- metadata.compliance.recording_disclosed is boolean
- metadata.compliance.retention_policy is present

---

## Test Suite: Correlation and Tracing

### Test 4.1: correlation_id Links Related Events
**Objective**: Verify that correlation_id enables event linking

**Setup**:
- Create related events with same correlation_id

**Steps**:
1. Create multiple events with same correlation_id
2. Query events by correlation_id

**Expected Outcome**:
- Events with same correlation_id are linked
- Queries return all related events

**Verification**:
- Events with same correlation_id are retrievable together
- correlation_id is optional but when present is consistent

---

### Test 4.2: parent_event_id Links to Origin
**Objective**: Verify that parent_event_id links to originating event

**Setup**:
- Create event with parent_event_id

**Steps**:
1. Create parent event
2. Create child event with parent_event_id
3. Verify linkage

**Expected Outcome**:
- parent_event_id references valid parent event
- Linkage enables conversation reconstruction

**Verification**:
- parent_event_id references existing event_id
- Parent event can be retrieved using parent_event_id

---

## Test Suite: Cross-Channel Consistency

### Test 5.1: Same Content Produces Identical Schema
**Objective**: Verify that same content from different channels produces identical canonical structure

**Setup**:
- Send identical text message via messaging, browser, and mobile

**Steps**:
1. Capture canonical events from each channel
2. Compare core fields and content

**Expected Outcome**:
- Core fields and content are identical
- Only channel-specific metadata differs

**Verification**:
- session_id, participant_id, text_content have 100% character-for-character equality across all 3 channels
- channel field differs appropriately (messaging ≠ browser ≠ mobile)
- event_type, event_id format, timestamp format are identical across channels
- metadata.channel_metadata differs but all 3 contain the same top-level keys (verified by set equality of keys)

---

## Test Suite: Edge Cases

### Test 6.1: Empty text_content Is Rejected
**Objective**: Verify that empty text_content is rejected for message events

**Setup**:
- Attempt to create message event with empty text_content

**Steps**:
1. Create message event with empty text_content
2. Verify validation fails

**Expected Outcome**:
- Empty text_content causes validation failure
- Error message indicates required field

**Verification**:
- Validation fails with appropriate error
- Event is not created

---

### Test 6.2: Very Long text_content Is Accepted
**Objective**: Verify that very long text_content is accepted

**Setup**:
- Create message event with 100,000 character text_content

**Steps**:
1. Create event with long text_content
2. Verify event is created and preserved

**Expected Outcome**:
- Long text_content is accepted
- Content is preserved without truncation

**Verification**:
- Event is created successfully (no error thrown)
- text_content length equals exactly 100,000 characters (verified: len(text_content) == 100000)
- Content hash matches original (verified: SHA256(retrieved) == SHA256(original))

