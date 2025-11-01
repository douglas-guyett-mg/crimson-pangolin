# Router Hobgoblin - Test Conditions

## Overview

This document specifies comprehensive test conditions for the Router hobgoblin. Router is the entry point for every turn and makes critical decisions about urgency assessment and hobgoblin activation.

---

## Unit Tests: Urgency Assessment

### Test R.1.1: HIGH Urgency - Financial Threat
**Condition**: Router receives input containing financial threat keywords (e.g., "suspicious email from my bank", "unauthorized transaction", "credit card fraud")
**Expected**: Router assesses urgency as HIGH
**Verification**: 
- Urgency level = HIGH
- Confidence score > 0.8
- Reasoning includes financial threat pattern

### Test R.1.2: HIGH Urgency - Health Emergency
**Condition**: Router receives input indicating health emergency (e.g., "chest pain", "can't breathe", "severe allergic reaction")
**Expected**: Router assesses urgency as HIGH
**Verification**:
- Urgency level = HIGH
- Confidence score > 0.8
- Reasoning includes health emergency pattern

### Test R.1.3: HIGH Urgency - Security Threat
**Condition**: Router receives input indicating security threat (e.g., "hacked account", "malware detected", "unauthorized access")
**Expected**: Router assesses urgency as HIGH
**Verification**:
- Urgency level = HIGH
- Confidence score > 0.8
- Reasoning includes security threat pattern

### Test R.1.4: MEDIUM Urgency - Time-Sensitive Decision
**Condition**: Router receives input with deadline but not critical (e.g., "job interview tomorrow", "flight leaves in 3 hours", "event this weekend")
**Expected**: Router assesses urgency as MEDIUM
**Verification**:
- Urgency level = MEDIUM
- Confidence score > 0.7
- Reasoning includes time-sensitive pattern

### Test R.1.5: MEDIUM Urgency - Moderate Time Pressure
**Condition**: Router receives input with moderate time pressure (e.g., "need answer by end of week", "deadline next month")
**Expected**: Router assesses urgency as MEDIUM
**Verification**:
- Urgency level = MEDIUM
- Confidence score > 0.7

### Test R.1.6: LOW Urgency - General Information
**Condition**: Router receives input with no time pressure (e.g., "What's the capital of France?", "Tell me about history", "How does photosynthesis work?")
**Expected**: Router assesses urgency as LOW
**Verification**:
- Urgency level = LOW
- Confidence score > 0.8
- No urgency signals detected

### Test R.1.7: LOW Urgency - Exploratory Question
**Condition**: Router receives exploratory input (e.g., "I'm curious about...", "Tell me more about...", "What are some examples of...")
**Expected**: Router assesses urgency as LOW
**Verification**:
- Urgency level = LOW
- Confidence score > 0.8

### Test R.1.8: Urgency Reassessment - Context Changes
**Condition**: Router receives input that initially seems LOW urgency, but context reveals HIGH urgency (e.g., "I got an email" → "from my bank" → "about a transfer I didn't make")
**Expected**: Router updates urgency assessment to HIGH
**Verification**:
- Initial urgency = LOW
- After context = HIGH
- Reassessment triggered by new information

---

## Unit Tests: Hobgoblin Activation

### Test R.2.1: Simple Query - Responder Only
**Condition**: Router receives simple query requiring direct answer (e.g., "What's 2+2?", "What time is it?")
**Expected**: Router activates only Responder
**Verification**:
- Activated hobgoblins = [Responder]
- No other hobgoblins activated
- Execution path is direct

### Test R.2.2: Ambiguous Query - Clarifier Activated
**Condition**: Router receives ambiguous input (e.g., "Fix the bug", "Help me with the project", "What should I do?")
**Expected**: Router activates Clarifier
**Verification**:
- Activated hobgoblins includes Clarifier
- Clarifier is activated before other hobgoblins
- Turn ends with clarification request

### Test R.2.3: Complex Query - Full Pipeline
**Condition**: Router receives complex query requiring analysis (e.g., "Analyze this data and create a report", "Find the best travel options for my trip")
**Expected**: Router activates Context Assembler, Plan Generator, Executor, Evaluator, Responder
**Verification**:
- Activated hobgoblins includes all required hobgoblins
- Hobgoblins activated in correct order
- All hobgoblins receive necessary context

### Test R.2.4: Urgent Query - Responder First
**Condition**: Router receives HIGH urgency query (e.g., "Suspicious email from my bank")
**Expected**: Router activates Responder immediately, then Executor for background work
**Verification**:
- Responder activated first
- Response sent immediately
- Executor continues in background
- Evaluator and Reflector activated after background work

### Test R.2.5: Constraint-Sensitive Query - Constraint Checker Activated
**Condition**: Router receives query that may violate constraints (e.g., "Can you help me with this financial decision?", "Should I invest in this?")
**Expected**: Router activates Constraint Checker
**Verification**:
- Activated hobgoblins includes Constraint Checker
- Constraint Checker runs before Executor
- Constraints are verified before action

### Test R.2.6: Learning Query - Reflector Activated
**Condition**: Router receives query similar to previous turns
**Expected**: Router activates Reflector to learn from previous outcomes
**Verification**:
- Reflector is activated
- Previous patterns are considered
- Learning improves decision quality

---

## Integration Tests: Hobgoblin Coordination

### Test R.3.1: Parallel Hobgoblin Activation
**Condition**: Router activates multiple hobgoblins that can run in parallel
**Expected**: Hobgoblins run concurrently without conflicts
**Verification**:
- All hobgoblins complete successfully
- No race conditions detected
- Results are consistent
- Execution time is optimal

### Test R.3.2: Sequential Hobgoblin Activation
**Condition**: Router activates hobgoblins with dependencies
**Expected**: Hobgoblins execute in correct order
**Verification**:
- Dependencies respected
- Each hobgoblin receives output from previous
- No deadlocks occur

### Test R.3.3: Context Passing
**Condition**: Router passes context to activated hobgoblins
**Expected**: All hobgoblins receive necessary context
**Verification**:
- Context includes urgency level
- Context includes turn goals
- Context includes relevant history
- All hobgoblins can access context

---

## Edge Cases and Failure Modes

### Test R.4.1: Conflicting Urgency Signals
**Condition**: Input contains conflicting urgency signals (e.g., "I'm not in a hurry, but my account was hacked")
**Expected**: Router resolves conflict and selects appropriate urgency
**Verification**:
- Urgency assessment is reasonable
- Reasoning explains conflict resolution
- User safety prioritized

### Test R.4.2: Ambiguous Hobgoblin Activation
**Condition**: Query could activate multiple different hobgoblin patterns
**Expected**: Router selects most appropriate pattern
**Verification**:
- Selected pattern is reasonable
- Reasoning explains selection
- Alternative patterns considered

### Test R.4.3: Missing Context
**Condition**: Router receives query with insufficient context
**Expected**: Router activates Clarifier or requests more information
**Verification**:
- Clarifier activated if ambiguous
- User asked for clarification
- Turn doesn't proceed without sufficient context

### Test R.4.4: Malformed Input
**Condition**: Router receives malformed or nonsensical input
**Expected**: Router handles gracefully
**Verification**:
- No crash or error
- Appropriate response generated
- User informed of issue

### Test R.4.5: Parallel Turn Coordination
**Condition**: Multiple turns arrive simultaneously
**Expected**: Router handles each turn independently
**Verification**:
- Each turn gets unique context
- No interference between turns
- All turns complete successfully

---

## Learning and Improvement

### Test R.5.1: Urgency Learning
**Condition**: Router makes urgency decision, receives feedback, makes similar decision again
**Expected**: Second decision is better informed
**Verification**:
- Confidence score increases
- Reasoning improves
- Fewer reassessments needed

### Test R.5.2: Activation Pattern Learning
**Condition**: Router uses activation pattern, receives feedback on effectiveness
**Expected**: Router improves pattern selection
**Verification**:
- Successful patterns used more frequently
- Unsuccessful patterns avoided
- New patterns learned

### Test R.5.3: Continuous Improvement
**Condition**: Multiple turns with similar patterns
**Expected**: Router decisions improve over time
**Verification**:
- Success rate increases
- Confidence scores increase
- User satisfaction improves

---

## Performance Tests

### Test R.6.1: Decision Latency
**Condition**: Router makes urgency and activation decisions
**Expected**: Decision completes within acceptable time
**Verification**:
- Decision time < 100ms
- No blocking operations
- Responsive to user input

### Test R.6.2: Context Retrieval Performance
**Condition**: Router retrieves context for decision
**Expected**: Context retrieval is efficient
**Verification**:
- Context retrieval < 50ms
- No unnecessary data fetched
- Caching works effectively

---

## Success Criteria

Router is successful if:
1. ✓ Urgency assessment is accurate
2. ✓ Hobgoblin activation is appropriate
3. ✓ Context is correctly passed to hobgoblins
4. ✓ Decisions improve over time through learning
5. ✓ Performance is acceptable
6. ✓ Edge cases are handled gracefully

