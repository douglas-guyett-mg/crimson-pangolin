# Working Memory Store - Test Conditions

## Test Suite: Eviction Policy

### Test 4.1: Evict Low-Importance Items First
**Objective**: Verify that the store evicts low-importance items before high-importance items

**Setup**:
- Create a Working Memory Store
- Add 10 items with importance_score=0.2 (low)
- Add 10 items with importance_score=0.8 (high)

**Steps**:
1. Trigger eviction (by adding more items or calling evict())
2. Verify which items were evicted

**Expected Outcome**:
- Low-importance items (0.2) are evicted first
- High-importance items (0.8) are preserved

**Verification**:
- Evicted items have importance_score < 0.3
- Remaining items include all high-importance items

---

### Test 4.2: FIFO Eviction Within Priority
**Objective**: Verify that items of similar importance are evicted in FIFO order

**Setup**:
- Create a Working Memory Store
- Add 5 items with importance_score=0.5 at different times

**Steps**:
1. Trigger eviction
2. Verify eviction order

**Expected Outcome**:
- Oldest items are evicted first
- Newer items are preserved

**Verification**:
- Evicted items are the oldest ones added
- Remaining items are the newest ones

---

### Test 4.3: Eviction Respects Importance Threshold
**Objective**: Verify that items above importance threshold are protected from eviction

**Setup**:
- Create store with importance_threshold_high=0.7
- Add items with varying importance scores
- Trigger eviction

**Steps**:
1. Add 10 items: 5 with importance=0.8, 5 with importance=0.3
2. Trigger eviction
3. Verify which items were evicted

**Expected Outcome**:
- Low-importance items (0.3) are evicted first
- High-importance items (0.8) are protected

**Verification**:
- Evicted items have importance < 0.7
- Remaining items include those with importance >= 0.7

---

## Test Suite: Eviction Policy

### Test 4.4: Evict Low-Importance Items First
**Objective**: Verify that low-importance items are evicted before high-importance items

**Setup**:
- Create store with max_items=10
- Write 5 items with importance_score=0.2 (low)
- Write 5 items with importance_score=0.8 (high)
- Write 5 more items with importance_score=0.2 (low)

**Steps**:
1. Write all items (15 total, exceeds limit of 10)
2. Verify which items were evicted

**Expected Outcome**:
- Low-importance items are evicted first
- High-importance items are preserved
- Final store contains high-importance items

**Verification**:
- Store contains the 5 high-importance items
- Store contains 5 of the low-importance items
- Evicted items are the oldest low-importance items

---

### Test 4.5: Protect High-Importance Items
**Objective**: Verify that high-importance items (>= 0.7) are protected from eviction

**Setup**:
- Create store with max_items=5
- Write 3 items with importance_score=0.8 (high)
- Write 5 items with importance_score=0.3 (low)

**Steps**:
1. Write all items (8 total, exceeds limit of 5)
2. Verify high-importance items are preserved

**Expected Outcome**:
- All 3 high-importance items are preserved
- Only low-importance items are evicted
- Final store contains 3 high-importance + 2 low-importance

**Verification**:
- All high-importance items remain in store
- Only low-importance items were evicted

---

### Test 4.6: FIFO Within Priority Level
**Objective**: Verify that within the same importance level, older items are evicted first

**Setup**:
- Create store with max_items=5
- Write 7 items all with importance_score=0.5

**Steps**:
1. Write all items
2. Verify eviction order

**Expected Outcome**:
- Oldest items are evicted first
- Newest items are preserved

**Verification**:
- Evicted items are items 1-2 (oldest)
- Remaining items are items 3-7 (newest)

---

## Test Suite: Time-to-Live (TTL)

### Test 4.7: Step-Count TTL Eviction
**Objective**: Verify that items older than step_ttl (20 steps) are evicted

**Setup**:
- Create store with step_ttl=20
- Write 5 items at step 0
- Write 5 items at step 10
- Write 5 items at step 25

**Steps**:
1. Write all items
2. Trigger eviction check at step 25
3. Verify which items are evicted

**Expected Outcome**:
- Items from step 0 (25 steps old) are evicted
- Items from step 10 (15 steps old) are preserved
- Items from step 25 (0 steps old) are preserved

**Verification**:
- Items from step 0 are evicted
- Items from step 10 and 25 remain

---

### Test 4.8: Wall-Time TTL Eviction
**Objective**: Verify that items older than wall_ttl (1 hour) are evicted

**Setup**:
- Create store with wall_ttl=1 hour
- Write items with timestamps 2 hours ago
- Write items with timestamps 30 minutes ago

**Steps**:
1. Write all items
2. Trigger eviction check at current time
3. Verify which items are evicted

**Expected Outcome**:
- Items older than 1 hour are evicted
- Items newer than 1 hour are preserved

**Verification**:
- Items from 2 hours ago are evicted
- Items from 30 minutes ago remain

---

### Test 4.9: Soft Eviction (TTL-Based)
**Objective**: Verify that TTL-expired items are only evicted when capacity is needed

**Setup**:
- Create store with step_ttl=20, max_items=100
- Write 10 items at step 0 (now 25 steps old, TTL-expired)
- Write 5 items at step 25 (current)

**Steps**:
1. Write all items
2. Verify TTL-expired items are still in store
3. Write more items to trigger capacity eviction
4. Verify TTL-expired items are evicted

**Expected Outcome**:
- TTL-expired items remain until capacity is needed
- When capacity is exceeded, TTL-expired items are evicted first

**Verification**:
- TTL-expired items are in store initially
- TTL-expired items are evicted when capacity is exceeded

---

## Test Suite: Position Tracking

### Test 4.10: Position Indices Updated on Write
**Objective**: Verify that position indices are updated when items are written

**Setup**:
- Create store and write 3 items

**Steps**:
1. Write item A (position should be 0)
2. Write item B (position should be 1)
3. Write item C (position should be 2)
4. Verify positions

**Expected Outcome**:
- Positions are assigned correctly
- Positions reflect order in queue

**Verification**:
- Item A has position 0
- Item B has position 1
- Item C has position 2

---

### Test 4.11: Position Indices Updated on Eviction
**Objective**: Verify that position indices are updated when items are evicted

**Setup**:
- Create store with max_items=3
- Write 3 items (positions 0, 1, 2)
- Write 2 more items (triggers eviction)

**Steps**:
1. Write items A, B, C
2. Write items D, E (evicts A, B)
3. Verify positions of remaining items

**Expected Outcome**:
- Remaining items have updated positions
- Positions are 0, 1, 2 for items C, D, E

**Verification**:
- Item C has position 0
- Item D has position 1
- Item E has position 2

---

## Test Suite: Read Operations

### Test 4.12: Read Returns Items in Reverse Chronological Order
**Objective**: Verify that read() returns items newest-first

**Setup**:
- Create store and write 5 items

**Steps**:
1. Write items A, B, C, D, E (in order)
2. Call read() with no filters
3. Verify order

**Expected Outcome**:
- Items are returned in reverse chronological order
- Newest item (E) is first

**Verification**:
- `read()` returns [E, D, C, B, A]

---

### Test 4.13: Search Respects Limit Parameter
**Objective**: Verify that search() respects the limit parameter

**Setup**:
- Create store with 20 items
- Call search() with limit=5

**Steps**:
1. Search for items
2. Verify result count

**Expected Outcome**:
- Returns at most 5 items
- Items are ranked by relevance

**Verification**:
- `len(results)` <= 5

---

## Test Suite: Eviction Reporting

### Test 4.14: Eviction Report Includes Evicted Items
**Objective**: Verify that eviction reports include details about evicted items

**Setup**:
- Create store with max_items=5
- Write 10 items

**Steps**:
1. Write items (triggers eviction)
2. Verify eviction report

**Expected Outcome**:
- Report includes list of evicted items
- Report includes reason for eviction
- Report includes eviction timestamp

**Verification**:
- Eviction report contains evicted item IDs
- Report indicates "capacity exceeded"
- Report includes timestamp

---

## Test Suite: Edge Cases

### Test 4.15: Empty Store Operations
**Objective**: Verify behavior with empty store

**Setup**:
- Create store with no items

**Steps**:
1. Call remember()
2. Call summarize()
3. Call get_capacity_info()

**Expected Outcome**:
- Operations return empty results
- No errors are raised

**Verification**:
- `remember()` returns empty list
- `summarize()` returns empty summary
- `get_capacity_info()` returns zero usage

---

### Test 4.16: Single Item Store
**Objective**: Verify behavior with single item

**Setup**:
- Create store and write 1 item

**Steps**:
1. Read the item
2. Delete the item
3. Verify store is empty

**Expected Outcome**:
- Single item operations work correctly

**Verification**:
- `read()` returns the item
- After delete, store is empty

---

## Test Suite: Concurrent Eviction Scenarios

### Test 4.17: Concurrent Eviction with Cascading Dependencies
**Objective**: Verify that eviction policy correctly handles items with dependency chains and protects referenced items

**Setup**:
- Create Working Memory Store with max_items=10
- Write Item A: {id: "A", importance_score: 0.9, type: "insight", content: "Core finding"}
- Write Item B: {id: "B", importance_score: 0.6, type: "conclusion", content: "Based on insight A", references: ["A"]}
- Write Item C: {id: "C", importance_score: 0.6, type: "action", content: "Follow up from A", references: ["A"]}
- Write Items D through M (10 items): {id: "D" to "M", importance_score: 0.3, type: "observation"}
- Total items: 13 (exceeds max_items by 3)
- Budget triggers eviction of 5 items to free up space

**Steps**:
1. Write all 13 items in order: A, B, C, D, E, F, G, H, I, J, K, L, M
2. Trigger eviction (automatic when item count exceeds max_items)
3. Eviction engine evaluates:
   - Items D-M (importance=0.3) are candidates
   - Item A (importance=0.9) is protected by high importance
   - Items B, C (importance=0.6) are protected because they reference A
4. Eviction selects 5 lowest-importance items: D, E, F, G, H
5. Verify remaining items: A, B, C, I, J, K, L, M (8 items, within capacity)

**Expected Outcome**:
- Eviction removes exactly 5 items (D, E, F, G, H)
- Items evicted are the lowest-importance items that have no dependents
- Item A is preserved (importance=0.9, has 2 dependents)
- Item B is preserved (references A, protected by dependency)
- Item C is preserved (references A, protected by dependency)
- Items I, J, K, L, M are preserved (next-lowest importance after evicted items)
- Final store size: 8 items (within max_items=10)

**Verification**:
- Store contents after eviction:
  - Contains exactly 8 items: A, B, C, I, J, K, L, M
  - Item A: {id: "A", importance: 0.9, dependents: ["B", "C"]}
  - Item B: {id: "B", importance: 0.6, references: ["A"]}
  - Item C: {id: "C", importance: 0.6, references: ["A"]}
  - Items I-M: {id: "I" to "M", importance: 0.3}
- Eviction log entry:
  - evicted_items: ["D", "E", "F", "G", "H"]
  - eviction_reason: "capacity_exceeded"
  - eviction_strategy: "importance_with_dependencies"
  - protected_items: {
      "A": "high_importance_with_dependents",
      "B": "dependency_on_A",
      "C": "dependency_on_A"
    }
  - eviction_timestamp: <ISO8601>
  - items_before: 13
  - items_after: 8
  - target_capacity: 10
- Dependency graph integrity:
  - All references in B and C point to existing item A
  - No dangling references
  - Dependency count for A: 2 (B and C)
- Store metadata:
  - current_size: 8
  - max_items: 10
  - utilization: 0.8 (80%)
  - total_evictions: 5

**Measurable Acceptance Criteria**:
- Exactly 5 items evicted (count=5)
- 0 high-importance items evicted (importance >= 0.7)
- 0 items with dependents evicted
- 100% of dependency references remain valid after eviction
- Eviction log contains all 5 evicted item IDs with reasons
- Final store size is 8 items (target: ≤10)
- All protected items (A, B, C) remain in store
- Evicted items (D-H) are the 5 lowest-importance items without dependencies

---

