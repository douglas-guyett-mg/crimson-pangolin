# Hybrid Indexing - Test Conditions

## Test Suite: Dense Indexing

### Test 11.1: Dense Index Stores Embeddings
**Objective**: Verify that dense index stores embeddings correctly

**Setup**:
- Create hybrid index
- Write item with text

**Steps**:
1. Write item
2. Verify embedding is stored
3. Retrieve embedding

**Expected Outcome**:
- Embedding is stored
- Embedding has correct dimension
- Embedding can be retrieved

**Verification**:
- Embedding exists in vector database
- Embedding dimension matches configuration
- Embedding is normalized

---

### Test 11.2: Dense Search Finds Semantically Similar Items
**Objective**: Verify that dense search finds semantically similar items

**Setup**:
- Write items about cities
- Search for semantically similar query

**Steps**:
1. Write items: "Paris is a city", "London is a city", "France is a country"
2. Search for "urban areas"
3. Verify results

**Expected Outcome**:
- City items are found
- Country item is not found
- Results are ranked by similarity

**Verification**:
- Search results include city items
- Search results exclude country item
- Results are ranked by similarity score

---

### Test 11.3: Dense Search Handles Synonyms
**Objective**: Verify that dense search handles synonyms

**Setup**:
- Write items with synonymous terms
- Search for synonym

**Steps**:
1. Write items: "automobile", "car", "vehicle"
2. Search for "car"
3. Verify all items are found

**Expected Outcome**:
- All synonymous items are found
- Results are ranked by similarity

**Verification**:
- Search results include all items
- Results are ranked appropriately

---

### Test 11.4: Dense Search Respects Threshold
**Objective**: Verify that dense search respects similarity threshold

**Setup**:
- Write items with varying similarity
- Search with threshold

**Steps**:
1. Write items
2. Search with threshold=0.8
3. Verify only high-similarity items are returned

**Expected Outcome**:
- Only items with similarity >= 0.8 are returned

**Verification**:
- All returned items have similarity >= 0.8

---

## Test Suite: Sparse Indexing

### Test 11.5: Sparse Index Stores Terms
**Objective**: Verify that sparse index stores terms correctly

**Setup**:
- Create hybrid index
- Write item with text

**Steps**:
1. Write item: "Paris is a beautiful city"
2. Verify terms are indexed
3. Retrieve terms

**Expected Outcome**:
- Terms are indexed
- Stopwords are removed
- Terms can be retrieved

**Verification**:
- Index contains ["paris", "beautiful", "city"]
- Index does not contain ["is", "a"]

---

### Test 11.6: Sparse Search Finds Keyword Matches
**Objective**: Verify that sparse search finds keyword matches

**Setup**:
- Write items with specific keywords
- Search for keyword

**Steps**:
1. Write items: "Paris is a city", "London is a city", "France is a country"
2. Search for "Paris"
3. Verify results

**Expected Outcome**:
- Only items containing "Paris" are found

**Verification**:
- Search results include only Paris item

---

### Test 11.7: Sparse Search Handles Phrases
**Objective**: Verify that sparse search handles phrases

**Setup**:
- Write items with phrases
- Search for phrase

**Steps**:
1. Write items: "New York City", "New York State", "York England"
2. Search for "New York"
3. Verify results

**Expected Outcome**:
- Items containing "New York" phrase are found

**Verification**:
- Search results include New York items

---

### Test 11.8: Sparse Search Respects Threshold
**Objective**: Verify that sparse search respects relevance threshold

**Setup**:
- Write items with varying relevance
- Search with threshold

**Steps**:
1. Write items
2. Search with threshold=0.7
3. Verify only high-relevance items are returned

**Expected Outcome**:
- Only items with relevance >= 0.7 are returned

**Verification**:
- All returned items have relevance >= 0.7

---

## Test Suite: Metadata Indexing

### Test 11.9: Metadata Index Filters by agent_id
**Objective**: Verify that metadata index filters by agent_id

**Setup**:
- Write items from different agents
- Filter by agent_id

**Steps**:
1. Write items from agent_001 and agent_002
2. Search with filter agent_id="agent_001"
3. Verify results

**Expected Outcome**:
- Only items from agent_001 are returned

**Verification**:
- All results have agent_id="agent_001"

---

### Test 11.10: Metadata Index Filters by Tags
**Objective**: Verify that metadata index filters by tags

**Setup**:
- Write items with different tags
- Filter by tags

**Steps**:
1. Write items with tags ["city"], ["country"], ["city", "capital"]
2. Search with filter tags=["city"]
3. Verify results

**Expected Outcome**:
- Only items with "city" tag are returned

**Verification**:
- All results have "city" tag

---

### Test 11.11: Metadata Index Supports Range Queries
**Objective**: Verify that metadata index supports range queries

**Setup**:
- Write items with timestamps
- Query by time range

**Steps**:
1. Write items with timestamps
2. Search with filter timestamp_range=[start, end]
3. Verify results

**Expected Outcome**:
- Only items in time range are returned

**Verification**:
- All results have timestamps in range

---

## Test Suite: Hybrid Search

### Test 11.12: Hybrid Search Combines Dense and Sparse
**Objective**: Verify that hybrid search combines dense and sparse results

**Setup**:
- Write items
- Perform hybrid search

**Steps**:
1. Write items
2. Perform hybrid search
3. Verify results combine both approaches

**Expected Outcome**:
- Results include both semantically similar and keyword-matching items
- Results are ranked by combined score

**Verification**:
- Results include items from both dense and sparse searches
- Results are ranked appropriately

---

### Test 11.13: Hybrid Search Respects Limits
**Objective**: Verify that hybrid search respects limit parameter

**Setup**:
- Write 20 items
- Search with limit=5

**Steps**:
1. Perform hybrid search with limit=5
2. Verify result count

**Expected Outcome**:
- Returns at most 5 items

**Verification**:
- `len(results)` <= 5

---

### Test 11.14: Hybrid Search Applies Metadata Filters
**Objective**: Verify that hybrid search applies metadata filters

**Setup**:
- Write items from different agents
- Search with metadata filter

**Steps**:
1. Write items from agent_001 and agent_002
2. Perform hybrid search with filter agent_id="agent_001"
3. Verify results

**Expected Outcome**:
- Only items from agent_001 are returned

**Verification**:
- All results have agent_id="agent_001"

---

## Test Suite: Index Maintenance

### Test 11.15: Index Updates on Write
**Objective**: Verify that indexes are updated when items are written

**Setup**:
- Write item
- Verify indexes are updated

**Steps**:
1. Write item
2. Verify dense index contains embedding
3. Verify sparse index contains terms
4. Verify metadata index contains metadata

**Expected Outcome**:
- All indexes are updated

**Verification**:
- Item is searchable via all index types

---

### Test 11.16: Index Updates on Delete
**Objective**: Verify that indexes are updated when items are deleted

**Setup**:
- Write item
- Delete item
- Verify indexes are updated

**Steps**:
1. Write item
2. Delete item
3. Search for item
4. Verify item is not found

**Expected Outcome**:
- Item is removed from all indexes

**Verification**:
- Item is not found in any search

---

### Test 11.17: Index Rebuild
**Objective**: Verify that indexes can be rebuilt

**Setup**:
- Write items
- Rebuild indexes
- Verify indexes are correct

**Steps**:
1. Write items
2. Call rebuild_indexes()
3. Verify indexes are correct

**Expected Outcome**:
- Indexes are rebuilt correctly

**Verification**:
- Searches work correctly after rebuild

---

## Test Suite: Performance

### Test 11.18: Dense Search Performance
**Objective**: Verify that dense search is performant

**Setup**:
- Write 1000 items
- Perform dense search

**Steps**:
1. Write 1000 items
2. Perform dense search
3. Measure time

**Expected Outcome**:
- Search completes in reasonable time (< 1 second)

**Verification**:
- Search time is acceptable

---

### Test 11.19: Sparse Search Performance
**Objective**: Verify that sparse search is performant

**Setup**:
- Write 1000 items
- Perform sparse search

**Steps**:
1. Write 1000 items
2. Perform sparse search
3. Measure time

**Expected Outcome**:
- Search completes quickly (< 100ms)

**Verification**:
- Search time is acceptable

---

### Test 11.20: Metadata Search Performance
**Objective**: Verify that metadata search is performant

**Setup**:
- Write 1000 items
- Perform metadata search

**Steps**:
1. Write 1000 items
2. Perform metadata search
3. Measure time

**Expected Outcome**:
- Search completes quickly (< 100ms)

**Verification**:
- Search time is acceptable

---

## Test Suite: Edge Cases

### Test 11.21: Empty Index
**Objective**: Verify behavior with empty index

**Setup**:
- Create index with no items

**Steps**:
1. Perform dense search
2. Perform sparse search
3. Perform metadata search

**Expected Outcome**:
- All searches return empty results
- No errors

**Verification**:
- All searches return empty lists

---

### Test 11.22: Single Item Index
**Objective**: Verify behavior with single item

**Setup**:
- Write 1 item

**Steps**:
1. Perform searches
2. Verify results

**Expected Outcome**:
- Single item is found

**Verification**:
- Searches return the single item

---

### Test 11.23: Large Items
**Objective**: Verify handling of large items

**Setup**:
- Write items with large text (100,000 tokens)

**Steps**:
1. Write large items
2. Perform searches
3. Verify results

**Expected Outcome**:
- Large items are indexed correctly
- Searches work correctly

**Verification**:
- Large items are found in searches

