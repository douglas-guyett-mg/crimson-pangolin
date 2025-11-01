# Re-ranking - Test Conditions

## Test Suite: Score Combination

### Test 12.1: Combine Dense and Sparse Scores
**Objective**: Verify that dense and sparse scores are combined correctly

**Setup**:
- Create results with dense and sparse scores
- Set weights: dense=0.4, sparse=0.3

**Steps**:
1. Create result with dense=0.9, sparse=0.7
2. Call rerank()
3. Verify combined score

**Expected Outcome**:
- Combined score = (0.9*0.4) + (0.7*0.3) = 0.51

**Verification**:
- Result score equals 0.51

---

### Test 12.2: Include Recency in Ranking
**Objective**: Verify that recency is included in ranking

**Setup**:
- Create two results with same dense/sparse scores but different timestamps
- Recent result: now
- Old result: 1 hour ago

**Steps**:
1. Create results
2. Call rerank()
3. Verify recent result ranks higher

**Expected Outcome**:
- Recent result has higher score

**Verification**:
- Recent result score > old result score

---

### Test 12.3: Include Importance in Ranking
**Objective**: Verify that importance is included in ranking

**Setup**:
- Create two results with same dense/sparse scores but different importance
- High importance: 0.9
- Low importance: 0.3

**Steps**:
1. Create results
2. Call rerank()
3. Verify high-importance result ranks higher

**Expected Outcome**:
- High-importance result has higher score

**Verification**:
- High-importance result score > low-importance result score

---

### Test 12.4: Include Source Weight in Ranking
**Objective**: Verify that source weight is included in ranking

**Setup**:
- Create two results with same other scores but different sources
- Result A: source="user_input" (weight=1.0)
- Result B: source="inference" (weight=0.6)

**Steps**:
1. Create results
2. Call rerank()
3. Verify user_input result ranks higher

**Expected Outcome**:
- User input result has higher score

**Verification**:
- User input result score > inference result score

---

## Test Suite: Recency Scoring

### Test 12.5: Calculate Recency Score
**Objective**: Verify that recency score is calculated correctly

**Setup**:
- Create items with different timestamps

**Steps**:
1. Create item from now
2. Create item from 1 hour ago
3. Create item from 1 day ago
4. Calculate recency scores

**Expected Outcome**:
- Recent item has high score (close to 1.0)
- Old item has low score (close to 0.0)
- Scores decay exponentially

**Verification**:
- Recent item score > 0.8
- Old item score < 0.2
- Scores follow exponential decay

---

### Test 12.6: Recency Decay Configuration
**Objective**: Verify that recency decay can be configured

**Setup**:
- Configure recency_decay parameter
- Create items with different timestamps

**Steps**:
1. Set recency_decay to different values
2. Calculate recency scores
3. Verify decay rate changes

**Expected Outcome**:
- Different decay rates produce different scores

**Verification**:
- Scores change based on decay configuration

---

## Test Suite: Ranking Order

### Test 12.7: Results Are Ranked by Score
**Objective**: Verify that results are ranked by combined score

**Setup**:
- Create 5 results with different scores

**Steps**:
1. Create results with scores: 0.5, 0.8, 0.3, 0.9, 0.6
2. Call rerank()
3. Verify ranking order

**Expected Outcome**:
- Results are ranked: 0.9, 0.8, 0.6, 0.5, 0.3

**Verification**:
- Results are in descending order by score

---

### Test 12.8: Ranking Is Deterministic
**Objective**: Verify that ranking is deterministic

**Setup**:
- Create results
- Re-rank twice with same input

**Steps**:
1. Re-rank results first time
2. Re-rank results second time
3. Compare rankings

**Expected Outcome**:
- Rankings are identical

**Verification**:
- First ranking equals second ranking

---

## Test Suite: Weight Configuration

### Test 12.9: Set Weights
**Objective**: Verify that weights can be set

**Setup**:
- Create re-ranker
- Set custom weights

**Steps**:
1. Call set_weights({dense: 0.5, sparse: 0.5})
2. Verify weights are set

**Expected Outcome**:
- Weights are updated

**Verification**:
- get_weights() returns new weights

---

### Test 12.10: Get Weights
**Objective**: Verify that weights can be retrieved

**Setup**:
- Create re-ranker with default weights

**Steps**:
1. Call get_weights()
2. Verify weights

**Expected Outcome**:
- Returns current weights

**Verification**:
- Returned weights match configuration

---

### Test 12.11: Weight Changes Affect Ranking
**Objective**: Verify that changing weights changes ranking

**Setup**:
- Create results
- Re-rank with different weights

**Steps**:
1. Re-rank with weights: dense=0.4, sparse=0.3
2. Re-rank with weights: dense=0.1, sparse=0.8
3. Compare rankings

**Expected Outcome**:
- Rankings are different

**Verification**:
- Ranking 1 differs from ranking 2

---

## Test Suite: Diversity

### Test 12.12: Diversity Score Penalizes Similar Results
**Objective**: Verify that diversity score penalizes similar results

**Setup**:
- Create two very similar results
- Create one different result

**Steps**:
1. Create results
2. Calculate diversity scores
3. Verify similar results have lower diversity scores

**Expected Outcome**:
- Similar results have lower diversity scores

**Verification**:
- Diversity score for similar result < diversity score for different result

---

### Test 12.13: Diversity Adjustment Improves Ranking
**Objective**: Verify that diversity adjustment improves ranking

**Setup**:
- Create results where top 2 are very similar
- Enable diversity adjustment

**Steps**:
1. Re-rank with diversity adjustment enabled
2. Verify ranking is adjusted

**Expected Outcome**:
- Ranking is adjusted to improve diversity

**Verification**:
- Different result is ranked higher than it would be without diversity

---

### Test 12.14: Diversity Can Be Disabled
**Objective**: Verify that diversity adjustment can be disabled

**Setup**:
- Create results
- Disable diversity adjustment

**Steps**:
1. Re-rank with diversity disabled
2. Verify diversity is not applied

**Expected Outcome**:
- Diversity adjustment is not applied

**Verification**:
- Ranking is based only on relevance scores

---

## Test Suite: Edge Cases

### Test 12.15: Single Result
**Objective**: Verify behavior with single result

**Setup**:
- Create single result

**Steps**:
1. Call rerank()
2. Verify result

**Expected Outcome**:
- Single result is returned

**Verification**:
- Result is returned unchanged

---

### Test 12.16: Empty Results
**Objective**: Verify behavior with empty results

**Setup**:
- Create empty result list

**Steps**:
1. Call rerank()
2. Verify behavior

**Expected Outcome**:
- Empty list is returned
- No errors

**Verification**:
- Result is empty list

---

### Test 12.17: All Results Have Same Score
**Objective**: Verify behavior when all results have same score

**Setup**:
- Create results with identical scores

**Steps**:
1. Call rerank()
2. Verify ranking

**Expected Outcome**:
- Results are ranked consistently
- Order is deterministic

**Verification**:
- Ranking is consistent across calls

---

### Test 12.18: Very Old Items
**Objective**: Verify handling of very old items

**Setup**:
- Create item from 1 year ago

**Steps**:
1. Calculate recency score
2. Verify score

**Expected Outcome**:
- Score is very low (close to 0)

**Verification**:
- Recency score < 0.1

---

### Test 12.19: Very New Items
**Objective**: Verify handling of very new items

**Setup**:
- Create item from now

**Steps**:
1. Calculate recency score
2. Verify score

**Expected Outcome**:
- Score is very high (close to 1)

**Verification**:
- Recency score > 0.9

---

## Test Suite: Performance

### Test 12.20: Re-ranking Performance
**Objective**: Verify that re-ranking is performant

**Setup**:
- Create 1000 results
- Re-rank

**Steps**:
1. Create 1000 results
2. Call rerank()
3. Measure time

**Expected Outcome**:
- Re-ranking completes quickly (< 100ms)

**Verification**:
- Re-ranking time is acceptable

---

### Test 12.21: Diversity Calculation Performance
**Objective**: Verify that diversity calculation is performant

**Setup**:
- Create 1000 results
- Enable diversity adjustment

**Steps**:
1. Create 1000 results
2. Call rerank() with diversity enabled
3. Measure time

**Expected Outcome**:
- Re-ranking completes in reasonable time (< 500ms)

**Verification**:
- Re-ranking time is acceptable

