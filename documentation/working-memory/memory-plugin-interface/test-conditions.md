# MemoryPlugin Interface - Test Conditions

## Test Suite: Core Operations

### Test 2.1: memorize() Stores Item Successfully
**Objective**: Verify that memorize() stores a memory item and returns confirmation

**Setup**:
- Create a memory plugin instance
- Create a MemoryItem with: id, type, text, tokens_estimate, timestamp, agent_id, tags, metadata

**Steps**:
1. Call `plugin.memorize(item, metadata)`
2. Verify the return value
3. Verify item is stored and retrievable

**Expected Outcome**:
- Memorize succeeds without error
- Returns confirmation with item reference (format depends on memory type)
- Item is stored and retrievable via remember()

**Verification**:
- Return value includes `success: true`
- Return value includes item reference
- Subsequent remember() call returns the stored item

---

### Test 2.2: remember() Retrieves Relevant Items
**Objective**: Verify that remember() retrieves items relevant to a prompt

**Setup**:
- Create a memory plugin instance
- Memorize 5 items with distinct content

**Steps**:
1. Call `plugin.remember(prompt, metadata)` with a prompt matching some items
2. Verify the returned items

**Expected Outcome**:
- Returns items relevant to the prompt
- Results are ranked by relevance (ranking strategy depends on memory type)
- Results include metadata and confidence scores

**Verification**:
- Returned list contains relevant items
- Items are ranked by relevance score
- Each item includes original fields (text, type, tags, metadata)

---

### Test 2.3: forget() Removes Items Based on Instructions
**Objective**: Verify that forget() removes items based on natural language instructions

**Setup**:
- Create a memory plugin instance
- Memorize 5 items

**Steps**:
1. Call `plugin.forget(forget_instructions, mode)` with instructions like "oldest"
2. Verify the item was forgotten
3. Verify other items remain

**Expected Outcome**:
- Forget succeeds without error
- Item matching instructions is removed or faded
- Other items remain unaffected

**Verification**:
- Return value includes `success: true`
- Subsequent remember() does not return the forgotten item
- Other items are still retrievable

---

### Test 2.4: summarize() Generates Compressed Summary
**Objective**: Verify that summarize() generates a summary within token limits

**Setup**:
- Create a memory plugin instance
- Memorize 10 items with varied content

**Steps**:
1. Call `plugin.summarize(scope, token_limit)` with scope="all" and token_limit=100
2. Verify the returned summary

**Expected Outcome**:
- Returns a summary text
- Summary token count <= token_limit
- Summary includes source item references

**Verification**:
- `summary.text` is non-empty
- `summary.token_count` <= 100
- `summary.source_items` is a list of item references

---

### Test 2.5: get_capacity_info() Returns Capacity Metadata
**Objective**: Verify that get_capacity_info() returns accurate capacity information

**Setup**:
- Create a memory plugin instance
- Memorize several items

**Steps**:
1. Call `plugin.get_capacity_info()`
2. Verify the returned information

**Expected Outcome**:
- Returns current item count
- Returns current token usage
- Returns capacity limits (if applicable)
- Returns available space (if applicable)

**Verification**:
- `capacity_info.item_count` >= 0
- `capacity_info.token_usage` >= 0
- `capacity_info.max_items` is defined (if applicable)
- `capacity_info.max_tokens` is defined (if applicable)

---

## Test Suite: Natural Language Instruction Handling

### Test 2.6: forget() Interprets Common Instructions
**Objective**: Verify that forget() correctly interprets common natural language instructions

**Setup**:
- Create a memory plugin instance
- Memorize 5 items with varying importance/recency

**Steps**:
1. Call `plugin.forget("oldest", "soft")`
2. Verify the oldest item is forgotten
3. Call `plugin.forget("least important", "soft")`
4. Verify the least important item is forgotten

**Expected Outcome**:
- "oldest" instruction removes the oldest/least recent item
- "least important" instruction removes the lowest importance item
- Other items remain unaffected

**Verification**:
- After first forget, oldest item is not returned by remember()
- After second forget, least important item is not returned by remember()
- Other items are still retrievable

---

### Test 2.7: forget() Handles Type-Specific Instructions
**Objective**: Verify that forget() handles memory-type-specific instructions

**Setup**:
- Create a memory plugin instance (specific type)
- Memorize several items

**Steps**:
1. Call `plugin.forget(type_specific_instruction, "soft")` with an instruction specific to this memory type
2. Verify the instruction is interpreted correctly

**Expected Outcome**:
- Type-specific instruction is understood and executed
- Correct items are forgotten based on the instruction

**Verification**:
- Instruction is executed without error
- Expected items are forgotten
- Unexpected items remain

---

### Test 2.8: remember() Respects Metadata Filters
**Objective**: Verify that remember() respects metadata filters

**Setup**:
- Create a memory plugin instance
- Memorize 5 items with different agent_ids and tags

**Steps**:
1. Call `plugin.remember(prompt, metadata)` with filters for specific agent_id
2. Verify only matching items are returned

**Expected Outcome**:
- Only items matching the filter are returned
- Items with different agent_id are excluded

**Verification**:
- All returned items have the specified agent_id
- No items with other agent_ids are returned

---

## Test Suite: Error Handling

### Test 2.9: memorize() Handles Invalid Item
**Objective**: Verify that memorize() handles invalid items gracefully

**Setup**:
- Create a memory plugin instance
- Create an invalid MemoryItem (missing required fields)

**Steps**:
1. Call `plugin.memorize(invalid_item, metadata)`
2. Verify error handling

**Expected Outcome**:
- Returns error or raises exception
- Error message is clear
- Plugin state is not corrupted

**Verification**:
- Return value includes `success: false` or exception is raised
- Error message indicates what field is missing

---

### Test 2.10: remember() Handles Empty Results
**Objective**: Verify that remember() handles prompts with no matches

**Setup**:
- Create a memory plugin instance
- Memorize items that don't match a specific prompt

**Steps**:
1. Call `plugin.remember(prompt, metadata)` with a prompt that matches nothing
2. Verify the return value

**Expected Outcome**:
- Returns empty list (not null or error)
- No exception is raised

**Verification**:
- `results` is an empty list
- No exception is raised

---

### Test 2.11: forget() Handles Unsupported Instructions
**Objective**: Verify that forget() handles unsupported instructions gracefully

**Setup**:
- Create a memory plugin instance
- Memorize several items

**Steps**:
1. Call `plugin.forget(unsupported_instruction, "soft")`
2. Verify error handling

**Expected Outcome**:
- Returns error or raises exception
- Error message indicates the instruction is not supported
- Plugin state is not corrupted

**Verification**:
- Return value includes `success: false` or exception is raised
- Error message indicates unsupported instruction
- Plugin state is unchanged

---

## Test Suite: Budgeting and Constraints

### Test 2.12: memorize() Respects Token Budget
**Objective**: Verify that memorize() respects token budgets

**Setup**:
- Create a memory plugin instance with token_budget=1000
- Memorize items totaling 800 tokens

**Steps**:
1. Attempt to memorize an item with 300 tokens (would exceed budget)
2. Verify behavior

**Expected Outcome**:
- Either: (a) memorize fails with budget error, or (b) older items are evicted to make room
- Budget is never exceeded

**Verification**:
- Total tokens in store <= 1000
- If eviction occurred, oldest/least important items were removed

---

### Test 2.13: forget() with Soft Mode Preserves Data
**Objective**: Verify that forget() with soft mode marks items but preserves them

**Setup**:
- Create a memory plugin instance
- Memorize 3 items

**Steps**:
1. Call `plugin.forget(item_reference, "soft")`
2. Attempt to remember the soft-forgotten item
3. Verify it's marked as deleted but still retrievable

**Expected Outcome**:
- Soft-forgotten item is marked as deleted
- Item is still in the store (for audit purposes)
- Item may be excluded from normal remember() calls

**Verification**:
- Item has `deleted: true` flag
- Item is still retrievable with appropriate query

---

### Test 2.14: forget() with Hard Mode Permanently Removes Data
**Objective**: Verify that forget() with hard mode permanently removes items

**Setup**:
- Create a memory plugin instance
- Memorize 3 items

**Steps**:
1. Call `plugin.forget(item_reference, "hard")`
2. Attempt to remember the hard-forgotten item
3. Verify it's completely removed

**Expected Outcome**:
- Hard-forgotten item is permanently removed
- Item is no longer retrievable
- Other items remain unaffected

**Verification**:
- Item is not returned by remember()
- Other items are still retrievable

---

## Test Suite: Semantic Flexibility

### Test 2.15: Different Memory Types Implement Operations Differently
**Objective**: Verify that different memory types can implement the same operations with different semantics

**Setup**:
- Create instances of two different memory type implementations (e.g., Working Memory and Episodic Memory)
- Memorize the same items in both

**Steps**:
1. Call `remember(prompt)` on both implementations
2. Verify results are different (different ranking, different selection)

**Expected Outcome**:
- Both return relevant items
- But the ranking and selection strategy is different
- Each follows its own semantics

**Verification**:
- Working Memory returns items ranked by importance
- Episodic Memory returns items ranked by time recency
- Both return relevant items but in different order

---

