# Semantic Memory Store - Test Conditions

## Key Strategy Tests

### Test 6.0: Key Format Is Semantic
**Objective**: Verify that keys follow the semantic format and represent entities/concepts

**Setup**:
- Prepare various facts to store
- Generate keys for each fact

**Steps**:
1. For each fact, verify key follows format `{entity_type}:{entity_identifier}`
2. Verify keys are meaningful and human-readable
3. Verify same entity gets same key

**Expected Outcome**:
- All keys follow the format
- Keys are semantic and meaningful
- Same entity always gets same key

**Verification**:
- Key format matches `{entity_type}:{entity_identifier}` pattern
- Key is human-readable (e.g., "location:new_york", not a hash)
- Multiple facts about same entity use same key

---

### Test 6.0b: Same Key Indicates Same Entity
**Objective**: Verify that same key is used for updates to the same entity

**Setup**:
- Create two facts about the same entity
- Generate keys for both

**Steps**:
1. Fact 1: "New York is in the United States" → key="location:new_york"
2. Fact 2: "New York is the largest city in the US" → key="location:new_york"
3. Verify keys are identical

**Expected Outcome**:
- Both facts get the same key
- This indicates they are updates to the same entity

**Verification**:
- Key for fact 1 equals key for fact 2
- Both keys are "location:new_york"

---

### Test 6.0c: Different Keys Indicate Different Entities
**Objective**: Verify that different entities get different keys

**Setup**:
- Create facts about different entities
- Generate keys for each

**Steps**:
1. Fact 1: "New York is in the United States" → key="location:new_york"
2. Fact 2: "Paris is in France" → key="location:paris"
3. Verify keys are different

**Expected Outcome**:
- Different entities get different keys
- This indicates they are separate facts

**Verification**:
- Key for fact 1 != key for fact 2
- Keys are "location:new_york" and "location:paris"

---

## Test Suite: Upsert Operations

### Test 6.1: Insert New Item
**Objective**: Verify that new items can be inserted

**Setup**:
- Create Semantic Memory Store
- Prepare a new item with key="fact:new_york"

**Steps**:
1. Write the item
2. Verify it's stored
3. Retrieve it

**Expected Outcome**:
- Item is inserted successfully
- Version is 1
- Item is retrievable

**Verification**:
- Write returns success
- `read()` returns the item
- Item version equals 1

---

### Test 6.2: Update Existing Item
**Objective**: Verify that existing items can be updated

**Setup**:
- Write an item with key="fact:new_york"
- Update the same item with new text

**Steps**:
1. Write initial item (version 1)
2. Write updated item with same key (version 2)
3. Verify version is incremented

**Expected Outcome**:
- Item is updated
- Version is incremented to 2
- Lineage links to version 1

**Verification**:
- Write returns version 2
- `read()` returns updated text
- Lineage includes version 1

---

### Test 6.3: Upsert Creates New if Not Exists
**Objective**: Verify that upsert inserts if key doesn't exist

**Setup**:
- Create store with no items
- Call upsert with new key

**Steps**:
1. Call upsert(key="fact:paris", item)
2. Verify item is inserted

**Expected Outcome**:
- Item is inserted
- Version is 1

**Verification**:
- Upsert returns success
- Version equals 1

---

### Test 6.4: Upsert Updates if Exists
**Objective**: Verify that upsert updates if key exists

**Setup**:
- Write an item with key="fact:paris"
- Call upsert with same key and new text

**Steps**:
1. Write initial item
2. Call upsert with same key
3. Verify version is incremented

**Expected Outcome**:
- Item is updated
- Version is incremented

**Verification**:
- Upsert returns version 2
- Text is updated

---

## Test Suite: Versioning and Lineage

### Test 6.5: Version Numbers Increment
**Objective**: Verify that version numbers increment with each update

**Setup**:
- Write an item
- Update it 5 times

**Steps**:
1. Write item (version 1)
2. Update 5 times
3. Verify final version is 6

**Expected Outcome**:
- Each update increments version
- Final version is 6

**Verification**:
- Final item version equals 6

---

### Test 6.6: Lineage Tracks Previous Versions
**Objective**: Verify that lineage links to previous versions

**Setup**:
- Write an item
- Update it 3 times

**Steps**:
1. Write item (version 1)
2. Update (version 2)
3. Update (version 3)
4. Update (version 4)
5. Verify lineage

**Expected Outcome**:
- Lineage includes [1, 2, 3]
- Current version is 4

**Verification**:
- `item.lineage` equals [1, 2, 3]
- `item.version` equals 4

---

### Test 6.7: Get Specific Version
**Objective**: Verify that specific versions can be retrieved

**Setup**:
- Write an item and update it 3 times

**Steps**:
1. Write item (version 1)
2. Update (version 2)
3. Update (version 3)
4. Retrieve version 1
5. Retrieve version 2
6. Retrieve version 3

**Expected Outcome**:
- Each version is retrievable
- Content matches what was written

**Verification**:
- `get_version(item_id, 1)` returns version 1 content
- `get_version(item_id, 2)` returns version 2 content
- `get_version(item_id, 3)` returns version 3 content

---

## Test Suite: Soft Delete

### Test 6.8: Soft Delete Marks Item
**Objective**: Verify that soft delete marks items without removing them

**Setup**:
- Write an item
- Soft delete it

**Steps**:
1. Write item
2. Soft delete
3. Verify item is marked as deleted

**Expected Outcome**:
- Item is marked with deleted=true
- Item is still in store
- Normal queries exclude it

**Verification**:
- Item has `deleted: true` flag
- `read()` excludes soft-deleted items

---

### Test 6.9: Soft Deleted Items Can Be Restored
**Objective**: Verify that soft-deleted items can be restored

**Setup**:
- Write an item
- Soft delete it
- Restore it

**Steps**:
1. Write item
2. Soft delete
3. Restore (upsert with deleted=false)
4. Verify item is active

**Expected Outcome**:
- Item is restored
- deleted flag is false
- Item is retrievable

**Verification**:
- Item has `deleted: false` flag
- `read()` includes restored item

---

## Test Suite: Hybrid Search

### Test 6.10: Dense Search Finds Semantically Similar Items
**Objective**: Verify that dense search finds semantically similar items

**Setup**:
- Write items about cities
- Search for semantically similar query

**Steps**:
1. Write items: "Paris is a city in France", "London is a city in England"
2. Search for "urban areas in Europe"
3. Verify results

**Expected Outcome**:
- Both items are found
- Ranked by semantic similarity

**Verification**:
- Search results include both items
- Results are ranked by similarity score

---

### Test 6.11: Sparse Search Finds Keyword Matches
**Objective**: Verify that sparse search finds keyword matches

**Setup**:
- Write items with specific keywords
- Search for keyword

**Steps**:
1. Write items containing "Paris"
2. Search for "Paris"
3. Verify results

**Expected Outcome**:
- Items containing "Paris" are found
- Ranked by keyword relevance

**Verification**:
- Search results include items with "Paris"

---

### Test 6.12: Hybrid Search Combines Dense and Sparse
**Objective**: Verify that hybrid search combines both approaches

**Setup**:
- Write items with both semantic and keyword relevance
- Search with query

**Steps**:
1. Write items
2. Search with query
3. Verify results combine both approaches

**Expected Outcome**:
- Results include both semantically similar and keyword-matching items
- Ranked by combined score

**Verification**:
- Search results include items from both approaches
- Results are ranked by combined score

---

### Test 6.13: search() Respects Limit
**Objective**: Verify that search respects limit parameter

**Setup**:
- Write 20 items
- Search with limit=5

**Steps**:
1. Search for broad query
2. Verify result count

**Expected Outcome**:
- Returns at most 5 items

**Verification**:
- `len(results)` <= 5

---

## Test Suite: LRU + Importance Retention

### Test 6.14: LRU Eviction
**Objective**: Verify that least recently used items are evicted first

**Setup**:
- Create store with max_items=5
- Write 7 items
- Access items in specific order

**Steps**:
1. Write items A, B, C, D, E
2. Access A, B, C (updates access time)
3. Write items F, G (triggers eviction)
4. Verify which items were evicted

**Expected Outcome**:
- Least recently used items (D, E) are evicted
- Recently accessed items (A, B, C) are preserved

**Verification**:
- Store contains A, B, C, F, G
- D and E were evicted

---

### Test 6.15: Importance Protects Items from Eviction
**Objective**: Verify that high-importance items are protected

**Setup**:
- Create store with max_items=5
- Write 3 items with importance=0.8 (high)
- Write 4 items with importance=0.2 (low)

**Steps**:
1. Write all items (7 total, exceeds limit of 5)
2. Verify which items were evicted

**Expected Outcome**:
- High-importance items are preserved
- Low-importance items are evicted

**Verification**:
- All 3 high-importance items remain
- 2 low-importance items remain
- 2 low-importance items were evicted

---

## Test Suite: Capacity Management

### Test 6.16: Respect Item Count Limit
**Objective**: Verify that store respects max_items limit

**Setup**:
- Create store with max_items=10
- Write 15 items

**Steps**:
1. Write all items
2. Verify store size

**Expected Outcome**:
- Store contains exactly 10 items
- 5 items were evicted

**Verification**:
- `store.size()` equals 10

---

### Test 6.17: Respect Token Budget
**Objective**: Verify that store respects max_tokens limit

**Setup**:
- Create store with max_tokens=5000
- Write items totaling 8000 tokens

**Steps**:
1. Write items
2. Verify total tokens

**Expected Outcome**:
- Total tokens <= 5000
- Items were evicted to stay within budget

**Verification**:
- `store.total_tokens()` <= 5000

---

## Test Suite: Metadata Filtering

### Test 6.18: Filter by Tags
**Objective**: Verify that search can filter by tags

**Setup**:
- Write items with different tags
- Search with tag filter

**Steps**:
1. Write items with tags ["city"], ["country"], ["city", "capital"]
2. Search with filter tags=["city"]
3. Verify results

**Expected Outcome**:
- Only items with "city" tag are returned

**Verification**:
- All results have "city" tag

---

### Test 6.19: Filter by Source
**Objective**: Verify that search can filter by source

**Setup**:
- Write items with different sources
- Search with source filter

**Steps**:
1. Write items with source="user_input", "inference", "tool_output"
2. Search with filter source="user_input"
3. Verify results

**Expected Outcome**:
- Only items from "user_input" source are returned

**Verification**:
- All results have source="user_input"

---

## Test Suite: Edge Cases

### Test 6.20: Empty Store Queries
**Objective**: Verify behavior with empty store

**Setup**:
- Create store with no items

**Steps**:
1. Search
2. Read
3. Upsert

**Expected Outcome**:
- Search returns empty results
- Read returns empty results
- Upsert inserts successfully

**Verification**:
- `search()` returns empty list
- `read()` returns empty list
- Upsert succeeds

---

### Test 6.21: Single Item Store
**Objective**: Verify behavior with single item

**Setup**:
- Write 1 item

**Steps**:
1. Search
2. Read
3. Update
4. Delete

**Expected Outcome**:
- Single item operations work correctly

**Verification**:
- `search()` returns 1 item
- `read()` returns 1 item
- Update succeeds
- Delete succeeds

