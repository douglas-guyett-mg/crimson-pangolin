# Gut Daemon - Test Conditions

## Overview

This document specifies comprehensive test conditions for the Gut Daemon. Gut is an async monitor that detects when Si is deviating from user intent and raises concerns to the Executor.

---

## Unit Tests: Semantic Similarity Detection

### Test GUT.1.1: Perfect Alignment
**Condition**: Current action perfectly matches user intent
**Expected**: No concern raised
**Verification**:
- Similarity score > 0.9
- Concern raised = false
- No alert to Executor

### Test GUT.1.2: Minor Deviation
**Condition**: Current action slightly off from user intent
**Expected**: MEDIUM severity concern
**Verification**:
- Similarity score 0.6 - 0.8
- Concern raised = true
- Severity = MEDIUM
- Concern message provided

### Test GUT.1.3: Major Deviation
**Condition**: Current action significantly different from user intent
**Expected**: HIGH severity concern
**Verification**:
- Similarity score 0.3 - 0.6
- Concern raised = true
- Severity = HIGH
- Concern message provided

### Test GUT.1.4: Complete Misalignment
**Condition**: Current action completely unrelated to user intent
**Expected**: CRITICAL severity concern
**Verification**:
- Similarity score < 0.3
- Concern raised = true
- Severity = CRITICAL
- Concern message provided

---

## Integration Tests: Async Monitoring

### Test GUT.2.1: Parallel Monitoring
**Condition**: Gut runs in parallel with Executor
**Expected**: Concerns raised without blocking execution
**Verification**:
- Executor continues execution
- Gut raises concerns asynchronously
- No deadlocks
- Concerns delivered to Executor

### Test GUT.2.2: Multiple Concerns
**Condition**: Multiple deviations detected during turn
**Expected**: All concerns raised with correct severity
**Verification**:
- All deviations detected
- Each concern has severity level
- Concerns ordered by severity
- Executor receives all concerns

### Test GUT.2.3: Concern Delivery
**Condition**: Gut raises concern
**Expected**: Executor receives concern via daemon interface
**Verification**:
- Concern message delivered
- Severity level included
- Timestamp included
- Executor can query concern

---

## Integration Tests: Executor Attention Process

### Test GUT.3.1: Low Severity Ignored
**Condition**: Gut raises MEDIUM severity concern
**Expected**: Executor logs but continues
**Verification**:
- Concern logged
- Execution continues
- No interruption
- No FC/Planner query

### Test GUT.3.2: High Severity Interrupts
**Condition**: Gut raises HIGH severity concern
**Expected**: Executor interrupts and checks plan
**Verification**:
- Execution interrupted
- FC queried for plan check
- Planner consulted
- Plan updated or confirmed

### Test GUT.3.3: Critical Severity Escalates
**Condition**: Gut raises CRITICAL severity concern
**Expected**: Executor stops and escalates
**Verification**:
- Execution stopped
- User escalation triggered
- Turn halted
- Concern logged

---

## Edge Cases and Failure Modes

### Test GUT.4.1: Ambiguous User Intent
**Condition**: User intent is unclear or ambiguous
**Expected**: Gut handles gracefully
**Verification**:
- No crash or error
- Conservative concern raising
- Concerns marked as uncertain
- Executor informed of ambiguity

### Test GUT.4.2: Evolving Intent
**Condition**: User intent changes during turn
**Expected**: Gut adapts to new intent
**Verification**:
- New intent detected
- Previous concerns reassessed
- New concerns raised if needed
- Executor informed of change

### Test GUT.4.3: Tool Chain Complexity
**Condition**: Complex multi-step tool chain
**Expected**: Gut monitors entire chain
**Verification**:
- Each step evaluated
- Cumulative deviation tracked
- Concerns raised at appropriate points
- No false positives from complexity

---

## Learning and Improvement

### Test GUT.5.1: Concern Validation
**Condition**: Gut raises concern, Executor validates
**Expected**: Learning improves future detection
**Verification**:
- Concern validity recorded
- Accuracy metrics updated
- Similar patterns learned
- Future similar cases handled better

### Test GUT.5.2: False Positive Reduction
**Condition**: Multiple turns with feedback
**Expected**: False positive rate decreases
**Verification**:
- Initial false positive rate measured
- After N turns, rate decreases
- Threshold adjusts based on feedback
- Precision improves

### Test GUT.5.3: False Negative Reduction
**Condition**: Multiple turns with feedback
**Expected**: False negative rate decreases
**Verification**:
- Initial false negative rate measured
- After N turns, rate decreases
- Sensitivity improves
- Recall improves

---

## Performance Tests

### Test GUT.6.1: Monitoring Latency
**Condition**: Gut monitors execution
**Expected**: Concerns raised quickly
**Verification**:
- Concern latency < 100ms
- No blocking of Executor
- Async operation confirmed

### Test GUT.6.2: Memory Usage
**Condition**: Gut monitors long turn
**Expected**: Memory usage reasonable
**Verification**:
- Memory usage < 50MB
- No memory leaks
- Efficient buffering

---

## Success Criteria

Gut is successful if:
1. ✓ Deviations from user intent are detected
2. ✓ Severity levels are accurate
3. ✓ Concerns are raised asynchronously
4. ✓ Executor can act on concerns
5. ✓ False positives are minimized
6. ✓ False negatives are minimized
7. ✓ Performance is acceptable
8. ✓ Learning improves accuracy over time

