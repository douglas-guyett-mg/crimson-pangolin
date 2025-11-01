# Episodic Memory Store (v1: Append-Only with Soft-Delete) - Test Conditions

## Test Suite: Append-Only Behavior

### Test 5.1: Items Are Appended in Order
**Objective**: Verify that items are appended in chronological order

**Setup**:
- Create Episodic Memory Store
- Write 5 items with timestamps 1 second apart

**Steps**:
1. Write items A, B, C, D, E
2. Query all items
3. Verify order

**Expected Outcome**:
- Items are returned in chronological order
- Order matches write order

**Verification**:
- `query_all()` returns [A, B, C, D, E]
- Timestamps are in ascending order

---

### Test 5.2: Items Cannot Be Modified
**Objective**: Verify that items cannot be modified after writing

**Setup**:
- Write an item to the store
- Attempt to modify it

**Steps**:
1. Write item A
2. Attempt to change item A's text
3. Verify item is unchanged

**Expected Outcome**:
- Item cannot be modified
- Either raises error or silently ignores change

**Verification**:
- Item A's text remains unchanged

---

## Test Suite: Chunking

### Test 5.3: Large Items Are Chunked
**Objective**: Verify that large items are automatically chunked

**Setup**:
- Create store with chunk_size=2000
- Write an item with 5000 tokens

**Steps**:
1. Write the large item
2. Verify it was chunked
3. Query the chunks

**Expected Outcome**:
- Item is split into 3 chunks (2000 + 2000 + 1000)
- Chunks are linked via parent_id
- Chunks have chunk_index values

**Verification**:
- Write returns 3 chunk IDs
- All chunks have same parent_id
- chunk_index values are 0, 1, 2

---

### Test 5.4: Chunks Are Reassembled on Retrieval
**Objective**: Verify that chunks are reassembled when retrieved

**Setup**:
- Write a large item that was chunked into 3 chunks
- Retrieve the item

**Steps**:
1. Write large item (5000 tokens)
2. Retrieve the item
3. Verify it's reassembled

**Expected Outcome**:
- Retrieved item contains full text
- No indication of chunking to caller
- Text matches original

**Verification**:
- Retrieved item text equals original text
- Retrieved item has no chunk_index field (or it's hidden)

---

### Test 5.5: Chunk Semantics Are Preserved
**Objective**: Verify that chunking preserves semantic meaning

**Setup**:
- Write a large item with semantic structure
- Chunk it
- Retrieve and reassemble

**Steps**:
1. Write item with clear semantic boundaries
2. Verify chunks respect boundaries
3. Retrieve and verify semantics

**Expected Outcome**:
- Chunks don't split mid-sentence or mid-concept
- Reassembled text is semantically coherent

**Verification**:
- Chunks end at sentence boundaries
- Reassembled text reads naturally

---

## Test Suite: Episode Grouping

### Test 5.6: Items Can Be Grouped into Episodes
**Objective**: Verify that items can be grouped by episode_id

**Setup**:
- Write 5 items with episode_id="ep_001"
- Write 3 items with episode_id="ep_002"

**Steps**:
1. Write all items
2. Query episode "ep_001"
3. Query episode "ep_002"

**Expected Outcome**:
- query_episode("ep_001") returns 5 items
- query_episode("ep_002") returns 3 items
- Items are in chronological order within each episode

**Verification**:
- `query_episode("ep_001")` returns exactly 5 items
- `query_episode("ep_002")` returns exactly 3 items

---

### Test 5.7: Episode Queries Return Chronological Order
**Objective**: Verify that episode queries return items in chronological order

**Setup**:
- Write 5 items to episode "ep_001" with timestamps 1 second apart

**Steps**:
1. Write items in order
2. Query episode
3. Verify order

**Expected Outcome**:
- Items are returned in chronological order
- Order matches write order

**Verification**:
- `query_episode("ep_001")` returns items in timestamp order

---

## Test Suite: Time Range Queries

### Test 5.8: query_range Returns Items in Time Window
**Objective**: Verify that query_range returns items within time bounds

**Setup**:
- Write 10 items with timestamps spanning 1 hour
- Query for items in a 20-minute window

**Steps**:
1. Write items at 10:00, 10:06, 10:12, 10:18, 10:24, 10:30, 10:36, 10:42, 10:48, 10:54
2. Query range 10:10 to 10:30
3. Verify results

**Expected Outcome**:
- Returns items at 10:12, 10:18, 10:24, 10:30
- Excludes items outside range

**Verification**:
- `query_range(10:10, 10:30)` returns 4 items
- All returned items have timestamps in range

---

### Test 5.9: query_range Respects Boundaries
**Objective**: Verify that query_range respects start and end boundaries

**Setup**:
- Write items at exact boundary times

**Steps**:
1. Write items at 10:00, 10:10, 10:20, 10:30
2. Query range 10:10 to 10:20 (inclusive)
3. Verify boundary items are included

**Expected Outcome**:
- Items at 10:10 and 10:20 are included
- Items at 10:00 and 10:30 are excluded

**Verification**:
- `query_range(10:10, 10:20)` returns items at 10:10 and 10:20

---

## Test Suite: Deletion Semantics

### Test 5.10: Soft Delete Marks Item as Deleted (Default Behavior)
**Objective**: Verify that soft delete marks items without removing them from the log

**Setup**:
- Write 5 items
- Soft delete one item using mode="soft"

**Steps**:
1. Write items A, B, C, D, E
2. Call delete(item_id=C, mode="soft")
3. Query all items
4. Verify item C is marked as deleted but retained

**Expected Outcome**:
- Item C is marked with deleted=true
- Item C is still in the log (for audit trail)
- Normal queries exclude item C
- Soft-deleted items can be undeleted if needed

**Verification**:
- Item C has `deleted: true` flag
- `query_all()` excludes item C (returns 4 items)
- Direct access to item C shows deleted flag
- Deletion timestamp is recorded

---

### Test 5.11: Hard Delete Removes Item Permanently (Compliance Exception)
**Objective**: Verify that hard delete permanently removes items from the log for compliance

**Setup**:
- Write 5 items
- Hard delete one item using mode="hard"

**Steps**:
1. Write items A, B, C, D, E
2. Call delete(item_id=C, mode="hard")
3. Query all items
4. Verify item C is completely removed

**Expected Outcome**:
- Item C is permanently removed from log
- Cannot be retrieved by any query
- Log size decreases
- Deletion is irreversible

**Verification**:
- `query_all()` returns 4 items
- Item C cannot be retrieved by ID
- Item C does not appear in any query results
- Hard delete is logged separately for audit

---

### Test 5.12: Soft Delete Is Reversible
**Objective**: Verify that soft-deleted items can be undeleted

**Setup**:
- Write item
- Soft delete it
- Undelete it

**Steps**:
1. Write item A
2. Soft delete item A
3. Call undelete(item_id=A)
4. Query all items

**Expected Outcome**:
- Item A is restored to active state
- Item A appears in queries again
- Deletion history is preserved

**Verification**:
- After undelete, `query_all()` includes item A
- Item A has `deleted: false` flag
- Deletion and undelete timestamps are recorded

---

## Test Suite: Search Operations

### Test 5.13: search() Finds Items by Content
**Objective**: Verify that search finds items matching query

**Setup**:
- Write 10 items with varied content
- Search for specific keyword

**Steps**:
1. Write items
2. Search for keyword
3. Verify results

**Expected Outcome**:
- Returns items containing keyword
- Results are ranked by relevance

**Verification**:
- Search results contain keyword
- Results are ranked

---

### Test 5.14: search() Respects Limit
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

## Test Suite: Summarization

### Test 5.15: summarize() Generates Episode Summary
**Objective**: Verify that summarize generates summary of episode

**Setup**:
- Write 10 items to an episode
- Summarize the episode

**Steps**:
1. Write items
2. Call summarize(scope="episode:ep_001", token_limit=200)
3. Verify summary

**Expected Outcome**:
- Returns summary text
- Token count <= 200
- Includes source item references

**Verification**:
- `summary.text` is non-empty
- `summary.token_count` <= 200
- `summary.source_items` is a list

---

## Test Suite: Edge Cases

### Test 5.16: Empty Store Queries
**Objective**: Verify behavior with empty store

**Setup**:
- Create store with no items

**Steps**:
1. Query all items
2. Query episode
3. Query time range

**Expected Outcome**:
- All queries return empty results
- No errors

**Verification**:
- `query_all()` returns empty list
- `query_episode()` returns empty list
- `query_range()` returns empty list

---

### Test 5.17: Single Item Store
**Objective**: Verify behavior with single item

**Setup**:
- Write 1 item

**Steps**:
1. Query all items
2. Query by episode
3. Delete item

**Expected Outcome**:
- Single item operations work correctly

**Verification**:
- `query_all()` returns 1 item
- After delete, store is empty

---

### Test 5.18: Large Log Performance
**Objective**: Verify performance with large number of items

**Setup**:
- Write 10,000 items to the log

**Steps**:
1. Write items
2. Query by time range
3. Search for items
4. Verify performance

**Expected Outcome**:
- Queries complete in reasonable time
- No degradation with large log

**Verification**:
- Query time is acceptable (< 1 second for typical queries)
- Results are accurate

