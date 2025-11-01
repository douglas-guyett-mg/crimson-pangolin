# Context Assembler Hobgoblin - Test Conditions

## Overview

This document specifies comprehensive test conditions for the Context Assembler hobgoblin. Context Assembler gathers relevant context from consciousness, working memory, and episodic memory to inform decision-making.

---

## Unit Tests: Context Retrieval

### Test CA.1.1: Consciousness Retrieval
**Condition**: Context Assembler retrieves consciousness objects
**Expected**: Consciousness objects retrieved successfully
**Verification**:
- Mandates retrieved
- Values retrieved
- Governance rules retrieved
- Capabilities retrieved
- All objects have valid structure

### Test CA.1.2: Working Memory Retrieval
**Condition**: Context Assembler retrieves working memory
**Expected**: Working memory retrieved successfully
**Verification**:
- Conversation history retrieved
- Turn trace retrieved
- Current context retrieved
- Parallel turns identified
- All data is current

### Test CA.1.3: Episodic Memory Retrieval
**Condition**: Context Assembler retrieves episodic memory
**Expected**: Episodic memory retrieved successfully
**Verification**:
- Similar situations found
- Learned patterns retrieved
- Previous outcomes retrieved
- Skills retrieved
- Memories are relevant

### Test CA.1.4: Conversation History Retrieval
**Condition**: Context Assembler retrieves conversation history
**Expected**: History retrieved in correct order
**Verification**:
- Messages in chronological order
- All messages included
- Message metadata preserved
- Context is complete

### Test CA.1.5: Turn Trace Retrieval
**Condition**: Context Assembler retrieves turn trace
**Expected**: Turn trace retrieved with all details
**Verification**:
- Previous turn decisions included
- Previous turn outcomes included
- Hobgoblin activations included
- Tool calls included
- Errors included

---

## Unit Tests: Context Relevance

### Test CA.2.1: Relevance Scoring
**Condition**: Context Assembler scores context relevance
**Expected**: Relevance scores are accurate
**Verification**:
- Highly relevant context scored > 0.8
- Moderately relevant context scored 0.5-0.8
- Irrelevant context scored < 0.5
- Scores are consistent

### Test CA.2.2: Relevant Context Selection
**Condition**: Context Assembler selects relevant context
**Expected**: Selected context is relevant to query
**Verification**:
- Selected context > 0 items
- All selected items relevant
- Irrelevant items excluded
- Selection is comprehensive

### Test CA.2.3: Context Filtering
**Condition**: Context Assembler filters irrelevant context
**Expected**: Irrelevant context excluded
**Verification**:
- Irrelevant items filtered out
- No false positives
- No false negatives
- Filtering is accurate

### Test CA.2.4: Context Prioritization
**Condition**: Context Assembler prioritizes context
**Expected**: Most relevant context prioritized
**Verification**:
- Highly relevant context first
- Moderately relevant context second
- Less relevant context last
- Prioritization is logical

### Test CA.2.5: Context Deduplication
**Condition**: Context Assembler encounters duplicate context
**Expected**: Duplicates removed
**Verification**:
- Duplicate items identified
- Duplicates removed
- No data loss
- Context is clean

---

## Integration Tests: Context Completeness

### Test CA.3.1: Complete Context Assembly
**Condition**: Context Assembler assembles complete context
**Expected**: All necessary context included
**Verification**:
- Consciousness included
- Working memory included
- Episodic memory included
- Input included
- Context is comprehensive

### Test CA.3.2: Context Consistency
**Condition**: Context Assembler assembles context from multiple sources
**Expected**: Context is consistent
**Verification**:
- No contradictions
- Consistent terminology
- Consistent values
- Consistent state

### Test CA.3.3: Context Freshness
**Condition**: Context Assembler retrieves context
**Expected**: Context is current
**Verification**:
- Working memory is current
- Turn trace is current
- Parallel turns reflected
- No stale data

### Test CA.3.4: Context Availability
**Condition**: Context Assembler makes context available to hobgoblins
**Expected**: All hobgoblins can access context
**Verification**:
- Context accessible to all hobgoblins
- No access conflicts
- Context not modified by hobgoblins
- Context remains consistent

---

## Parallel Execution Tests

### Test CA.4.1: Parallel Context Retrieval
**Condition**: Context Assembler runs in parallel with other hobgoblins
**Expected**: All hobgoblins complete without conflicts
**Verification**:
- No race conditions
- Consistent results
- All hobgoblins complete successfully
- Execution time is optimal

### Test CA.4.2: Concurrent Memory Access
**Condition**: Multiple hobgoblins access context simultaneously
**Expected**: No conflicts or data corruption
**Verification**:
- All reads succeed
- No data corruption
- Consistent results
- No deadlocks

### Test CA.4.3: Parallel Turn Context
**Condition**: Multiple turns run in parallel
**Expected**: Each turn gets correct context
**Verification**:
- Turn 1 context isolated from Turn 2
- No interference between turns
- Each turn has complete context
- Parallel turns complete successfully

---

## Edge Cases and Failure Modes

### Test CA.5.1: Missing Context
**Condition**: Context Assembler encounters missing context
**Expected**: Handles gracefully
**Verification**:
- No crash or error
- Partial context returned
- Missing items noted
- Hobgoblins informed of gaps

### Test CA.5.2: Corrupted Context
**Condition**: Context Assembler encounters corrupted data
**Expected**: Handles gracefully
**Verification**:
- Corrupted data identified
- Corrupted data excluded
- Valid data preserved
- Error logged

### Test CA.5.3: Stale Context
**Condition**: Context Assembler retrieves stale data
**Expected**: Detects staleness
**Verification**:
- Staleness detected
- Fresh data retrieved
- Timestamp checked
- Context is current

### Test CA.5.4: Context Explosion
**Condition**: Context Assembler retrieves too much context
**Expected**: Limits context size
**Verification**:
- Context size limited
- Most relevant items included
- Less relevant items excluded
- Performance maintained

### Test CA.5.5: Empty Context
**Condition**: Context Assembler retrieves no context
**Expected**: Handles gracefully
**Verification**:
- No crash or error
- Empty context returned
- Hobgoblins informed
- Turn continues appropriately

---

## Learning and Improvement

### Test CA.6.1: Relevance Learning
**Condition**: Context Assembler scores relevance, receives feedback
**Expected**: Relevance scoring improves
**Verification**:
- Scoring accuracy increases
- False positives decrease
- False negatives decrease
- Scoring becomes more precise

### Test CA.6.2: Context Selection Learning
**Condition**: Context Assembler selects context, receives feedback
**Expected**: Selection improves
**Verification**:
- More relevant context selected
- Less relevant context excluded
- Selection becomes more accurate
- Hobgoblins more satisfied

### Test CA.6.3: Continuous Improvement
**Condition**: Multiple turns with context assembly
**Expected**: Context assembly improves
**Verification**:
- Relevance scoring improves
- Context selection improves
- Hobgoblin satisfaction increases

---

## Performance Tests

### Test CA.7.1: Context Retrieval Latency
**Condition**: Context Assembler retrieves context
**Expected**: Retrieval completes quickly
**Verification**:
- Retrieval time < 200ms
- No blocking operations
- Responsive to hobgoblins

### Test CA.7.2: Context Assembly Latency
**Condition**: Context Assembler assembles context
**Expected**: Assembly completes quickly
**Verification**:
- Assembly time < 300ms
- Efficient processing
- No unnecessary operations

### Test CA.7.3: Memory Usage
**Condition**: Context Assembler manages context
**Expected**: Memory usage is reasonable
**Verification**:
- Memory usage < 100MB
- No memory leaks
- Efficient storage

---

## Success Criteria

Context Assembler is successful if:
1. ✓ All necessary context is retrieved
2. ✓ Context is relevant to query
3. ✓ Context is consistent
4. ✓ Context is current
5. ✓ Context is available to all hobgoblins
6. ✓ Parallel execution works correctly
7. ✓ Performance is acceptable
8. ✓ Edge cases are handled gracefully

