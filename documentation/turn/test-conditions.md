# Turn Architecture Test Conditions

## Overview

This document specifies the test conditions for the turn architecture. Every testable component must have clear, measurable test conditions that verify correct behavior.

## Test Categories

### 1. Router Tests

#### Test 1.1: Urgency Assessment - High Urgency
**Condition**: Router receives input with financial threat keywords
**Expected**: Router assesses urgency as HIGH
**Verification**: Urgency level = HIGH

#### Test 1.2: Urgency Assessment - Medium Urgency
**Condition**: Router receives input with time-sensitive but non-critical keywords
**Expected**: Router assesses urgency as MEDIUM
**Verification**: Urgency level = MEDIUM

#### Test 1.3: Urgency Assessment - Low Urgency
**Condition**: Router receives input with no urgency signals
**Expected**: Router assesses urgency as LOW
**Verification**: Urgency level = LOW

#### Test 1.4: Hobgoblin Activation - Simple Query
**Condition**: Router receives simple query ("What's 2+2?")
**Expected**: Router activates only Responder
**Verification**: Activated hobgoblins = [Responder]

#### Test 1.5: Hobgoblin Activation - Complex Query
**Condition**: Router receives complex query requiring multiple steps
**Expected**: Router activates Context Assembler, Plan Generator, Executor, Evaluator, Responder
**Verification**: Activated hobgoblins includes all required hobgoblins

#### Test 1.6: Hobgoblin Activation - Ambiguous Query
**Condition**: Router receives ambiguous query
**Expected**: Router activates Clarifier
**Verification**: Activated hobgoblins includes Clarifier

#### Test 1.7: Learning - Urgency Improvement
**Condition**: Router makes urgency decision, receives feedback, makes similar decision again
**Expected**: Second decision is better informed than first
**Verification**: Confidence in pattern increases

### 2. Clarifier Tests

#### Test 2.1: Ambiguity Detection
**Condition**: Clarifier receives ambiguous input
**Expected**: Clarifier detects ambiguity
**Verification**: Ambiguity detected = true

#### Test 2.2: Question Generation
**Condition**: Clarifier detects ambiguity
**Expected**: Clarifier generates clarifying questions
**Verification**: Questions generated > 0

#### Test 2.3: Question Quality
**Condition**: Clarifier generates questions
**Expected**: Questions are clear, specific, and focused
**Verification**: User can understand and answer questions

#### Test 2.4: Turn Completion
**Condition**: Clarifier generates questions
**Expected**: Turn ends with clarification request
**Verification**: Response sent to user, turn ends

### 3. Context Assembler Tests

#### Test 3.1: Context Retrieval
**Condition**: Context Assembler receives query
**Expected**: Context Assembler retrieves relevant context
**Verification**: Context retrieved > 0

#### Test 3.2: Context Relevance
**Condition**: Context Assembler retrieves context
**Expected**: Retrieved context is relevant to query
**Verification**: Context relevance score > threshold

#### Test 3.3: Parallel Execution
**Condition**: Context Assembler runs in parallel with other hobgoblins
**Expected**: All hobgoblins complete without conflicts
**Verification**: No race conditions, consistent results

### 4. Constraint Checker Tests

#### Test 4.1: Constraint Identification
**Condition**: Constraint Checker receives proposed action
**Expected**: Constraint Checker identifies applicable constraints
**Verification**: Constraints identified > 0

#### Test 4.2: Compliance Check - Compliant
**Condition**: Proposed action complies with constraints
**Expected**: Constraint Checker marks as COMPLIANT
**Verification**: Status = COMPLIANT

#### Test 4.3: Compliance Check - Violation
**Condition**: Proposed action violates constraints
**Expected**: Constraint Checker marks as VIOLATION
**Verification**: Status = VIOLATION

#### Test 4.4: Alternative Suggestion
**Condition**: Constraint Checker detects violation
**Expected**: Constraint Checker suggests compliant alternative
**Verification**: Alternative provided and is compliant

### 5. Plan Generator Tests

#### Test 5.1: Plan Creation
**Condition**: Plan Generator receives requirements
**Expected**: Plan Generator creates execution plan
**Verification**: Plan created with steps > 0

#### Test 5.2: Step Ordering
**Condition**: Plan has dependent steps
**Expected**: Steps are ordered correctly
**Verification**: Dependencies respected in order

#### Test 5.3: Parallelism Identification
**Condition**: Plan has independent steps
**Expected**: Plan Generator identifies parallelizable steps
**Verification**: Independent steps marked as parallel

#### Test 5.4: Plan Optimization
**Condition**: Plan Generator creates plan
**Expected**: Plan is optimized for efficiency
**Verification**: Execution time is minimal

### 6. Executor Tests

#### Test 6.1: Tool Execution
**Condition**: Executor receives plan with tool call
**Expected**: Executor calls tool with correct parameters
**Verification**: Tool called with correct parameters

#### Test 6.2: Result Collection
**Condition**: Tool returns result
**Expected**: Executor collects and stores result
**Verification**: Result stored in working memory

#### Test 6.3: Sequential Execution
**Condition**: Plan has dependent steps
**Expected**: Executor executes steps in order
**Verification**: Steps executed in correct order

#### Test 6.4: Parallel Execution
**Condition**: Plan has independent steps
**Expected**: Executor executes steps in parallel
**Verification**: Steps executed concurrently

#### Test 6.5: Error Handling
**Condition**: Tool fails
**Expected**: Executor alerts Error Handler
**Verification**: Error Handler receives notification

### 7. Error Handler Tests

#### Test 7.1: Error Detection
**Condition**: Tool fails
**Expected**: Error Handler detects error
**Verification**: Error detected = true

#### Test 7.2: Retry Strategy
**Condition**: Transient error occurs
**Expected**: Error Handler retries
**Verification**: Tool called again

#### Test 7.3: Alternative Strategy
**Condition**: Tool fails, alternative available
**Expected**: Error Handler uses alternative tool
**Verification**: Alternative tool called

#### Test 7.4: Graceful Failure
**Condition**: All recovery strategies fail
**Expected**: Error Handler fails gracefully
**Verification**: User informed, alternatives suggested

### 8. Evaluator Tests

#### Test 8.1: Goal Achievement Assessment
**Condition**: Turn completes
**Expected**: Evaluator assesses if goal was achieved
**Verification**: Assessment produced

#### Test 8.2: Quality Assessment
**Condition**: Turn completes
**Expected**: Evaluator assesses result quality
**Verification**: Quality score produced

#### Test 8.3: Efficiency Assessment
**Condition**: Turn completes
**Expected**: Evaluator assesses execution efficiency
**Verification**: Efficiency score produced

#### Test 8.4: Outcome Assessment
**Condition**: Evaluator completes assessment
**Expected**: Outcome assessment provided to Reflector
**Verification**: Assessment includes all required fields

### 9. Reflector Tests

#### Test 9.1: Pattern Learning
**Condition**: Reflector receives outcome assessment
**Expected**: Reflector identifies patterns
**Verification**: Patterns extracted

#### Test 9.2: Knowledge Update
**Condition**: Reflector identifies patterns
**Expected**: Reflector updates episodic memory
**Verification**: Memory updated with new patterns

#### Test 9.3: Confidence Adjustment
**Condition**: Reflector learns from outcome
**Expected**: Confidence in patterns adjusted
**Verification**: Confidence scores updated

#### Test 9.4: Continuous Improvement
**Condition**: Multiple turns with similar patterns
**Expected**: Decisions improve over time
**Verification**: Success rate increases

### 10. Responder Tests

#### Test 10.1: Response Generation
**Condition**: Responder receives information to communicate
**Expected**: Responder generates response
**Verification**: Response generated

#### Test 10.2: Response Clarity
**Condition**: Responder generates response
**Expected**: Response is clear and understandable
**Verification**: User can understand response

#### Test 10.3: Response Appropriateness
**Condition**: Responder generates response
**Expected**: Response is appropriate for situation
**Verification**: Tone and format are appropriate

#### Test 10.4: Response Delivery
**Condition**: Responder generates response
**Expected**: Response is sent to user
**Verification**: Response delivered

### 11. Integration Tests

#### Test 11.1: Simple Turn - Direct Answer
**Condition**: User asks simple question (e.g., "What's 2+2?")
**Expected**: Turn completes with direct answer
**Verification**:
- Router activates Responder only
- Responder generates answer
- User receives answer
- Turn completes successfully

#### Test 11.2: Complex Turn - Full Pipeline
**Condition**: User asks complex question requiring analysis (e.g., "Analyze this data and create a report")
**Expected**: Turn completes with comprehensive answer
**Verification**:
- Router activates Context Assembler, Plan Generator, Executor, Evaluator, Responder
- Context Assembler retrieves relevant context
- Plan Generator creates execution plan
- Executor executes plan successfully
- Evaluator assesses outcome
- Responder generates comprehensive response
- User receives complete answer

#### Test 11.3: Ambiguous Turn - Clarification
**Condition**: User asks ambiguous question (e.g., "Fix the bug")
**Expected**: Turn ends with clarification request
**Verification**:
- Router activates Clarifier
- Clarifier detects ambiguity
- Clarifier generates clarifying questions
- Responder sends clarification request
- Turn ends
- User can respond with clarification

#### Test 11.4: Urgent Turn - Immediate Response
**Condition**: User asks urgent question (e.g., "Suspicious email from my bank")
**Expected**: Immediate response sent, background work continues
**Verification**:
- Router assesses urgency as HIGH
- Router activates Responder immediately
- Responder generates immediate response
- Response sent to user
- Executor continues background work
- Evaluator assesses background work
- Reflector learns from outcome
- Follow-up response sent when background work completes

#### Test 11.5: Constraint-Sensitive Turn
**Condition**: User asks question that may violate constraints
**Expected**: Constraints verified before action
**Verification**:
- Router activates Constraint Checker
- Constraint Checker identifies applicable constraints
- Constraint Checker verifies compliance
- If compliant: Plan Generator creates plan, Executor executes
- If violation: Constraint Checker suggests alternative
- Responder communicates result
- Turn completes appropriately

#### Test 11.6: Error Recovery Turn
**Condition**: Error occurs during turn execution
**Expected**: Error is handled gracefully
**Verification**:
- Executor detects error
- Error Handler receives notification
- Error Handler analyzes error
- Error Handler selects recovery strategy
- Recovery strategy executed
- If recovery succeeds: Turn continues
- If recovery fails: Graceful failure
- Evaluator assesses outcome
- Reflector learns from error handling

#### Test 11.7: Parallel Turns
**Condition**: Multiple users ask questions simultaneously
**Expected**: Turns execute in parallel without conflicts
**Verification**:
- Turn 1 and Turn 2 start simultaneously
- Each turn has isolated context
- No interference between turns
- All turns complete successfully
- Results are correct for each turn
- Parallel execution improves performance

#### Test 11.8: Learning Loop - Improvement Over Time
**Condition**: Multiple turns with similar patterns
**Expected**: System learns and improves
**Verification**:
- Turn 1: Router makes decision, Evaluator assesses, Reflector learns
- Turn 2: Similar situation, Router uses learned patterns
- Turn 2 decision is better informed than Turn 1
- Success rate increases
- Confidence scores increase
- User satisfaction improves

#### Test 11.9: Hobgoblin Communication
**Condition**: Hobgoblins communicate during turn
**Expected**: Communication is correct and complete
**Verification**:
- Router passes context to activated hobgoblins
- Context Assembler provides context to other hobgoblins
- Plan Generator passes plan to Executor
- Executor passes results to Evaluator
- Evaluator passes assessment to Reflector
- All communication is correct
- No information lost
- All hobgoblins receive necessary information

#### Test 11.10: Turn Lifecycle - Complete Sequence
**Condition**: Complete turn from input to response
**Expected**: All lifecycle stages complete
**Verification**:
- Input received
- Router assesses urgency and activates hobgoblins
- Hobgoblins execute in correct order
- Results collected
- Evaluator assesses outcome
- Responder generates response
- Response sent to user
- Reflector learns from outcome
- Turn marked complete
- Lifecycle complete

## Test Execution Strategy

### Unit Tests
- Test each hobgoblin independently
- Verify decision-making logic
- Verify output format
- Test with mocked dependencies
- Verify error handling
- Test edge cases

**Execution**: Run in parallel, each hobgoblin tested independently

### Integration Tests
- Test hobgoblins working together
- Verify communication between hobgoblins
- Verify turn completion
- Test data flow between hobgoblins
- Verify context passing
- Test error propagation

**Execution**: Run sequentially to ensure proper ordering, then run parallel scenarios

### End-to-End Tests
- Test complete turns from input to response
- Verify user satisfaction
- Verify learning loop
- Test with real user scenarios
- Verify response quality
- Test multiple turn sequences

**Execution**: Run with realistic data, measure user satisfaction

### Performance Tests
- Measure turn execution time (target: < 2s for simple turns, < 10s for complex)
- Measure LLM call count (target: minimize unnecessary calls)
- Measure resource usage (target: < 100MB memory per turn)
- Measure parallel turn throughput
- Measure response latency (target: < 500ms for immediate responses)

**Execution**: Run with load testing, measure under various conditions

### Regression Tests
- Run all tests after each change
- Verify no regressions in performance
- Verify no regressions in accuracy
- Verify no regressions in user satisfaction

**Execution**: Automated, run on every commit

## Success Criteria

A turn is successful if:
1. ✓ User receives appropriate response
2. ✓ Response is timely (urgency-appropriate)
3. ✓ Response is accurate and helpful
4. ✓ No constraints are violated
5. ✓ System learns from outcome
6. ✓ Similar future turns are better

### Detailed Success Metrics

**Correctness**:
- Goal achievement: >= 90% of turns achieve stated goal
- Accuracy: >= 95% of responses are factually accurate
- Compliance: 100% of turns comply with constraints

**Performance**:
- Simple turns: < 500ms response time
- Complex turns: < 2s response time
- Immediate responses: < 100ms
- Parallel turns: linear scaling up to 10 concurrent turns

**Quality**:
- User satisfaction: >= 4.0/5.0 average rating
- Response clarity: >= 90% of users understand response
- Actionability: >= 85% of users can act on response

**Learning**:
- Improvement rate: >= 5% improvement per 10 similar turns
- Pattern accuracy: >= 85% of learned patterns are correct
- Skill transfer: >= 80% of skills transfer to new contexts

## Test Data Requirements

### Input Scenarios
- Simple queries (10+ examples)
- Complex queries (10+ examples)
- Ambiguous queries (10+ examples)
- Urgent queries (10+ examples)
- Constraint-sensitive queries (10+ examples)
- Error scenarios (10+ examples)
- Edge cases (10+ examples)

### Expected Outputs
- Correct responses for each scenario
- Expected execution paths
- Expected hobgoblin activations
- Expected performance metrics

### Mock Data
- Mock consciousness objects
- Mock working memory
- Mock episodic memory
- Mock tool responses
- Mock user profiles

## Key Principles

**All test conditions must be measurable and verifiable.** Vague or aspirational test conditions are not acceptable. Each test must have clear pass/fail criteria.

**Tests must be comprehensive.** Cover unit tests, integration tests, end-to-end tests, performance tests, and regression tests.

**Tests must be maintainable.** Use clear naming, good documentation, and reusable test fixtures.

**Tests must be fast.** Unit tests should complete in < 1s, integration tests in < 5s, end-to-end tests in < 30s.

**Tests must be reliable.** No flaky tests, deterministic results, reproducible failures.

