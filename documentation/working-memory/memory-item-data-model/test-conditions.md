# Memory Item Data Model - Test Conditions

## Test Suite: Core Field Validation

### Test 3.1: Create MemoryItem with All Required Fields
**Objective**: Verify that a MemoryItem can be created with all required fields

**Setup**:
- Prepare valid values for all required fields:
  - id: "mem_abc123"
  - type: "observation"
  - text: "Sample memory text"
  - tokens_estimate: 10
  - timestamp: "2025-10-30T14:23:45Z"
  - agent_id: "agent_001"

**Steps**:
1. Create a MemoryItem with these values
2. Verify all fields are set correctly

**Expected Outcome**:
- MemoryItem is created successfully
- All fields are accessible and match input values

**Verification**:
- `item.id` equals "mem_abc123"
- `item.type` equals "observation"
- `item.text` equals "Sample memory text"
- `item.tokens_estimate` equals 10
- `item.timestamp` equals "2025-10-30T14:23:45Z"
- `item.agent_id` equals "agent_001"

---

### Test 3.2: Create MemoryItem with Optional Fields
**Objective**: Verify that optional fields (tags, metadata) can be set

**Setup**:
- Create a MemoryItem with required fields
- Add tags: ["tag1", "tag2"]
- Add metadata: {"source": "user_input", "confidence": 0.95}

**Steps**:
1. Create the MemoryItem
2. Verify optional fields are set

**Expected Outcome**:
- Optional fields are stored and retrievable
- Defaults are applied if not provided

**Verification**:
- `item.tags` equals ["tag1", "tag2"]
- `item.metadata["source"]` equals "user_input"
- `item.metadata["confidence"]` equals 0.95

---

### Test 3.3: Validate Required Fields
**Objective**: Verify that missing required fields are caught

**Setup**:
- Attempt to create a MemoryItem without each required field

**Steps**:
1. Try creating without id
2. Try creating without type
3. Try creating without text
4. Try creating without tokens_estimate
5. Try creating without timestamp
6. Try creating without agent_id

**Expected Outcome**:
- Each attempt fails with a validation error
- Error message indicates which field is missing

**Verification**:
- ValidationError is raised for each missing field
- Error message includes field name

---

### Test 3.4: Validate Type Field Values
**Objective**: Verify that type field only accepts valid enum values

**Setup**:
- Valid types: "conversation", "action", "observation", "goal", "skill", "fact", "pattern", "error", "custom"
- Invalid type: "invalid_type"

**Steps**:
1. Create MemoryItem with each valid type
2. Attempt to create with invalid type

**Expected Outcome**:
- Valid types are accepted
- Invalid type is rejected with error

**Verification**:
- All valid types create successfully
- Invalid type raises ValidationError

---

### Test 3.5: Validate tokens_estimate is Non-Negative
**Objective**: Verify that tokens_estimate must be >= 0

**Setup**:
- Valid values: 0, 1, 100, 1000
- Invalid values: -1, -100

**Steps**:
1. Create MemoryItem with valid token counts
2. Attempt to create with negative token counts

**Expected Outcome**:
- Valid values are accepted
- Negative values are rejected

**Verification**:
- Valid values create successfully
- Negative values raise ValidationError

---

### Test 3.6: Validate Timestamp Format
**Objective**: Verify that timestamp must be valid ISO 8601 format

**Setup**:
- Valid: "2025-10-30T14:23:45Z", "2025-10-30T14:23:45+00:00"
- Invalid: "2025-10-30", "14:23:45", "invalid"

**Steps**:
1. Create MemoryItem with valid timestamps
2. Attempt to create with invalid timestamps

**Expected Outcome**:
- Valid ISO 8601 timestamps are accepted
- Invalid formats are rejected

**Verification**:
- Valid timestamps create successfully
- Invalid timestamps raise ValidationError

---

## Test Suite: Store-Specific Fields

### Test 3.7: Working Memory Store Fields
**Objective**: Verify that Working Memory specific fields can be set in store_metadata

**Setup**:
- Create a MemoryItem with required fields
- Add Working Memory fields via store_metadata: position=5, step_index=42, importance_score=0.8

**Steps**:
1. Create the MemoryItem
2. Verify store-specific fields are set in store_metadata

**Expected Outcome**:
- Store-specific fields are stored and retrievable in store_metadata
- They don't interfere with core fields

**Verification**:
- `item.store_metadata["working_memory"]["position"]` equals 5
- `item.store_metadata["working_memory"]["step_index"]` equals 42
- `item.store_metadata["working_memory"]["importance_score"]` equals 0.8

---

### Test 3.8: Episodic Memory Store Fields
**Objective**: Verify that Episodic Memory specific fields can be set in store_metadata

**Setup**:
- Create a MemoryItem with required fields
- Add Episodic Memory fields via store_metadata: parent_id="mem_parent", chunk_index=2, episode_id="ep_001"

**Steps**:
1. Create the MemoryItem
2. Verify store-specific fields are set in store_metadata

**Expected Outcome**:
- Episodic fields are stored and retrievable in store_metadata

**Verification**:
- `item.store_metadata["episodic"]["parent_id"]` equals "mem_parent"
- `item.store_metadata["episodic"]["chunk_index"]` equals 2
- `item.store_metadata["episodic"]["episode_id"]` equals "ep_001"

---

## Test Suite: Serialization

### Test 3.9: JSON Serialization
**Objective**: Verify that MemoryItem can be serialized to JSON

**Setup**:
- Create a MemoryItem with all fields populated

**Steps**:
1. Serialize to JSON
2. Verify JSON is valid
3. Verify all fields are present

**Expected Outcome**:
- Serialization succeeds
- JSON is valid and parseable
- All fields are included

**Verification**:
- JSON string is valid
- JSON contains all required fields
- JSON can be parsed back to object

---

### Test 3.10: JSON Deserialization
**Objective**: Verify that MemoryItem can be deserialized from JSON

**Setup**:
- Create a MemoryItem and serialize to JSON
- Deserialize back to MemoryItem

**Steps**:
1. Serialize original item to JSON
2. Deserialize JSON to new MemoryItem
3. Compare original and deserialized

**Expected Outcome**:
- Deserialization succeeds
- All fields match original
- No data loss

**Verification**:
- Deserialized item equals original item
- All fields (including metadata) are preserved
- Data types are correct

---

### Test 3.11: Serialization with Nested Metadata
**Objective**: Verify that complex metadata structures serialize correctly

**Setup**:
- Create MemoryItem with nested metadata:
  ```
  metadata: {
    "source": "tool_output",
    "nested": {
      "key1": "value1",
      "key2": [1, 2, 3]
    }
  }
  ```

**Steps**:
1. Serialize to JSON
2. Deserialize from JSON
3. Verify nested structure is preserved

**Expected Outcome**:
- Nested structures are preserved
- No data loss or corruption

**Verification**:
- `item.metadata["nested"]["key1"]` equals "value1"
- `item.metadata["nested"]["key2"]` equals [1, 2, 3]

---

## Test Suite: Immutability and Identity

### Test 3.12: Item ID is Immutable
**Objective**: Verify that item ID cannot be changed after creation

**Setup**:
- Create a MemoryItem with id="mem_abc123"

**Steps**:
1. Attempt to change the id field
2. Verify it remains unchanged

**Expected Outcome**:
- ID cannot be modified
- Either raises error or silently ignores change

**Verification**:
- `item.id` still equals "mem_abc123" after modification attempt

---

### Test 3.13: Timestamp is Immutable
**Objective**: Verify that timestamp cannot be changed after creation

**Setup**:
- Create a MemoryItem with timestamp="2025-10-30T14:23:45Z"

**Steps**:
1. Attempt to change the timestamp
2. Verify it remains unchanged

**Expected Outcome**:
- Timestamp cannot be modified
- Either raises error or silently ignores change

**Verification**:
- `item.timestamp` still equals "2025-10-30T14:23:45Z"

---

## Test Suite: Edge Cases

### Test 3.14: Empty Text Field
**Objective**: Verify behavior with empty text

**Setup**:
- Create MemoryItem with text=""

**Steps**:
1. Create the item
2. Verify it's stored

**Expected Outcome**:
- Empty text is allowed (though may not be useful)
- Item is created successfully

**Verification**:
- `item.text` equals ""

---

### Test 3.15: Large Text Field
**Objective**: Verify handling of very large text

**Setup**:
- Create MemoryItem with text containing 100,000 characters
- Set tokens_estimate appropriately

**Steps**:
1. Create the item
2. Serialize and deserialize
3. Verify no truncation

**Expected Outcome**:
- Large text is handled correctly
- No truncation or data loss

**Verification**:
- Deserialized text matches original
- Length is preserved

