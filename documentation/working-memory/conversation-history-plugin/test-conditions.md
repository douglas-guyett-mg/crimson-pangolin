# Conversation History Daemon - Test Conditions

## Test Suite: Rolling Buffer

### Test 7.1: Buffer Maintains Last N Utterances
**Objective**: Verify that buffer maintains the last N utterances

**Setup**:
- Create daemon with buffer_size=5
- Call query() to add 7 utterances

**Steps**:
1. Call query() to add utterances 1-7
2. Call query() to retrieve all utterances
3. Verify buffer contains utterances 3-7

**Expected Outcome**:
- Buffer contains exactly 5 utterances
- Oldest utterances (1-2) were evicted

**Verification**:
- query("Get all utterances") returns utterances 3-7
- Buffer size equals 5

---

### Test 7.2: FIFO Eviction
**Objective**: Verify that oldest utterances are evicted first

**Setup**:
- Create daemon with buffer_size=3
- Call query() to add 5 utterances

**Steps**:
1. Call query() to add utterances A, B, C, D, E
2. Call query() to retrieve all utterances
3. Verify buffer contains C, D, E

**Expected Outcome**:
- Oldest utterances (A, B) are evicted
- Newest utterances (C, D, E) are preserved

**Verification**:
- query("Get all utterances") returns [C, D, E]

---

### Test 7.3: Utterance Indexing
**Objective**: Verify that utterances are indexed correctly

**Setup**:
- Create daemon and call query() to add 5 utterances

**Steps**:
1. Write utterances
2. Verify utterance_index values

**Expected Outcome**:
- Utterances have sequential utterance_index values
- Indexes reflect position in buffer

**Verification**:
- Utterance 1 has utterance_index=0
- Utterance 2 has utterance_index=1
- Utterance 5 has utterance_index=4

---

## Test Suite: Speaker Tracking

### Test 7.4: Speaker Field is Recorded
**Objective**: Verify that speaker field is recorded for each utterance

**Setup**:
- Write utterances from different speakers

**Steps**:
1. Write user utterance
2. Write assistant utterance
3. Write system utterance
4. Verify speaker fields

**Expected Outcome**:
- Each utterance has correct speaker field
- Speakers are "user", "assistant", or "system"

**Verification**:
- User utterance has speaker="user"
- Assistant utterance has speaker="assistant"
- System utterance has speaker="system"

---

### Test 7.5: Turn Number is Tracked
**Objective**: Verify that turn_number is tracked

**Setup**:
- Write utterances across multiple turns

**Steps**:
1. Write 2 utterances at turn 1
2. Write 2 utterances at turn 2
3. Write 2 utterances at turn 3
4. Verify turn numbers

**Expected Outcome**:
- Each utterance has correct turn_number
- Turn numbers reflect conversation flow

**Verification**:
- First 2 utterances have turn_number=1
- Next 2 utterances have turn_number=2
- Last 2 utterances have turn_number=3

---

## Test Suite: Summarization

### Test 7.6: Evicted Utterances Are Summarized
**Objective**: Verify that evicted utterances are summarized

**Setup**:
- Create daemon with buffer_size=3
- Call query() to add 5 utterances

**Steps**:
1. Call query() to add utterances 1-3
2. Call query() to add utterances 4-5 (triggers eviction)
3. Call query() to retrieve summary

**Expected Outcome**:
- Utterances 1-2 are evicted
- Summary is generated for evicted utterances
- Summary is available via query()

**Verification**:
- query("Get summary") returns summary with length >= 50 characters
- Summary contains at least 80% of key entities (names, topics, actions) from utterances 1-2 as measured by exact or semantic match

---

### Test 7.7: Summary Respects Token Limit
**Objective**: Verify that summary respects token limit

**Setup**:
- Create daemon with summary_token_limit=100
- Call query() to add many utterances to trigger summarization

**Steps**:
1. Call query() to add utterances to trigger eviction
2. Call query() to retrieve summary with token limit

**Expected Outcome**:
- Summary token count <= 100

**Verification**:
- query("Get summary", data={token_limit: 100}) returns summary with token_count <= 100

---

### Test 7.8: Incremental Summaries
**Objective**: Verify that new summaries are merged with existing summaries

**Setup**:
- Create daemon with buffer_size=2
- Call query() to add 6 utterances (triggers multiple evictions)

**Steps**:
1. Write utterances 1-2
2. Write utterances 3-4 (evicts 1-2, generates summary 1)
3. Write utterances 5-6 (evicts 3-4, generates summary 2)
4. Verify final summary

**Expected Outcome**:
- Final summary includes information from all evicted utterances
- Summaries are merged incrementally

**Verification**:
- `get_summary()` contains at least 75% of key entities and topics from all 4 evicted utterances (1-4) as measured by entity extraction and overlap comparison

---

## Test Suite: Read Operations

### Test 7.9: get_last_n Returns Recent Utterances
**Objective**: Verify that get_last_n returns the most recent utterances

**Setup**:
- Write 10 utterances
- Call get_last_n(5)

**Steps**:
1. Write utterances 1-10
2. Call get_last_n(5)
3. Verify results

**Expected Outcome**:
- Returns utterances 6-10
- In chronological order

**Verification**:
- `get_last_n(5)` returns [6, 7, 8, 9, 10]

---

### Test 7.10: read() Returns Utterances in Chronological Order
**Objective**: Verify that read() returns utterances in chronological order

**Setup**:
- Write 5 utterances
- Call read()

**Steps**:
1. Write utterances
2. Call read()
3. Verify order

**Expected Outcome**:
- Utterances are returned in chronological order
- Oldest first

**Verification**:
- `read()` returns utterances sorted by timestamp ascending (verified by comparing timestamps[i] <= timestamps[i+1] for all i)

---

### Test 7.11: search() Finds Utterances by Content
**Objective**: Verify that search() finds utterances matching query

**Setup**:
- Write utterances with varied content
- Search for specific keyword

**Steps**:
1. Write utterances
2. Search for keyword
3. Verify results

**Expected Outcome**:
- Returns utterances containing keyword
- Ranked by relevance score descending

**Verification**:
- 100% of search results contain the exact keyword or semantic match with cosine similarity >= 0.85
- Results are sorted by relevance_score descending (verified by scores[i] >= scores[i+1] for all i)

---

### Test 7.12: Filter by Speaker
**Objective**: Verify that read/search can filter by speaker

**Setup**:
- Write utterances from different speakers
- Filter for "user" speaker

**Steps**:
1. Write utterances from user, assistant, system
2. Read with filter speaker="user"
3. Verify results

**Expected Outcome**:
- Only user utterances are returned

**Verification**:
- All results have speaker="user"

---

## Test Suite: Swappable Backends

### Test 7.13: In-Memory Backend Works
**Objective**: Verify that in-memory backend works correctly

**Setup**:
- Create plugin with backend="in-memory"
- Write utterances

**Steps**:
1. Write utterances
2. Read utterances
3. Verify data is accessible

**Expected Outcome**:
- In-memory backend stores and retrieves data

**Verification**:
- `read()` returns written utterances

---

### Test 7.14: Durable Backend Works
**Objective**: Verify that durable backend works correctly

**Setup**:
- Create plugin with backend="durable"
- Write utterances

**Steps**:
1. Write utterances
2. Read utterances
3. Verify data is persistent

**Expected Outcome**:
- Durable backend stores and retrieves data
- Data persists across sessions

**Verification**:
- `read()` returns 100% of written utterances with exact content match
- Data persists across process restart (verified by killing and restarting process, then confirming read() returns same data)

---

### Test 7.15: Backend Switching
**Objective**: Verify that backend can be switched

**Setup**:
- Create plugin with in-memory backend
- Write utterances
- Switch to durable backend

**Steps**:
1. Write utterances to in-memory backend
2. Switch backend to durable
3. Verify data is accessible

**Expected Outcome**:
- Backend can be switched
- Data is accessible after switch

**Verification**:
- `read()` returns utterances after switch

---

## Test Suite: Token Budget

### Test 7.16: Respect Token Budget
**Objective**: Verify that plugin respects max_tokens limit

**Setup**:
- Create plugin with max_tokens=1000
- Write utterances totaling > 1000 tokens

**Steps**:
1. Write utterances
2. Verify total tokens

**Expected Outcome**:
- Total tokens <= 1000
- Utterances were evicted to stay within budget

**Verification**:
- `store.total_tokens()` <= 1000

---

## Test Suite: Edge Cases

### Test 7.17: Empty Buffer
**Objective**: Verify behavior with empty buffer

**Setup**:
- Create plugin with no utterances

**Steps**:
1. Call get_last_n(5)
2. Call read()
3. Call get_summary()

**Expected Outcome**:
- Operations return empty results
- No errors

**Verification**:
- `get_last_n(5)` returns empty list
- `read()` returns empty list
- `get_summary()` returns empty summary

---

### Test 7.18: Single Utterance
**Objective**: Verify behavior with single utterance

**Setup**:
- Write 1 utterance

**Steps**:
1. Call get_last_n(1)
2. Call read()
3. Delete utterance

**Expected Outcome**:
- Single utterance operations work correctly

**Verification**:
- `get_last_n(1)` returns 1 utterance
- `read()` returns 1 utterance
- After delete, buffer is empty

---

### Test 7.19: Large Utterances
**Objective**: Verify handling of large utterances

**Setup**:
- Write utterances with large text (10,000+ tokens)

**Steps**:
1. Write large utterances
2. Verify they're stored correctly
3. Retrieve and verify

**Expected Outcome**:
- Large utterances are handled correctly
- No truncation or data loss

**Verification**:
- Retrieved utterances match original with 100% character-for-character equality
- Retrieved token count equals original token count (±0 tolerance)

