# Working Memory Store - Test Conditions

## Test Suite: Capacity Management

### Test 4.1: Respect Item Count Limit
**Objective**: Verify that the store respects the max_items limit (64)

**Setup**:
- Create a Working Memory Store with max_items=64
- Create 70 items with importance_score=0.5

**Steps**:
1. Write all 70 items to the store
2. Verify store size

**Expected Outcome**:
- Store contains exactly 64 items
- 6 items were evicted
- Oldest items were evicted first

**Verification**:
- `store.size()` equals 64
- Eviction report shows 6 items evicted
- Evicted items are the first 6 written

---

### Test 4.2: Respect Token Budget
**Objective**: Verify that the store respects the max_tokens limit (4000)

**Setup**:
- Create a Working Memory Store with max_tokens=4000
- Create items with tokens_estimate such that total would exceed 4000

**Steps**:
1. Write items until total tokens would exceed 4000
2. Verify total tokens

**Expected Outcome**:
- Total tokens <= 4000
- Items were evicted to stay within budget
- Oldest items were evicted first

**Verification**:
- `store.total_tokens()` <= 4000
- Eviction occurred
- Evicted items are oldest

---

### Test 4.3: Item Count Limit Takes Precedence
**Objective**: Verify behavior when both limits are approached

**Setup**:
- Create store with max_items=64, max_tokens=4000
- Create 65 items with 100 tokens each (6500 total)

**Steps**:
1. Write all items
2. Verify which limit was enforced

**Expected Outcome**:
- Store respects both limits
- Item count limit is enforced (64 items max)
- Token budget is also respected

**Verification**:
- `store.size()` <= 64
- `store.total_tokens()` <= 4000

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

