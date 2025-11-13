# Turn Architecture Test Conditions

## Overview

This document specifies the test conditions for the turn architecture. Every testable component must have clear, measurable test conditions that verify correct behavior.

## Test Categories

### 1. Router Tests

#### Test 1.1: Quick Answer Detection - Scratch Page
**Condition**: Router receives query that can be answered from Scratch Page
**Expected**: Router decides "quick answer" approach
**Verification**: Quick answer = true, Responder activated

#### Test 1.2: Quick Answer Detection - Simple Query
**Condition**: Router receives simple query ("What's 2+2?")
**Expected**: Router decides "quick answer" approach
**Verification**: Quick answer = true, Responder activated

#### Test 1.3: Complex Query Detection
**Condition**: Router receives complex query requiring multiple steps
**Expected**: Router decides "needs execution" approach
**Verification**: Quick answer = false, Executor will run

#### Test 1.4: Ambiguous Query Detection
**Condition**: Router receives ambiguous query
**Expected**: Router decides "needs clarification" approach
**Verification**: Ambiguous = true, Clarifier will be called by Executor

#### Test 1.5: Executor Always Spawned
**Condition**: Router makes any decision (quick answer or not)
**Expected**: Executor is always spawned
**Verification**: Executor spawned = true

#### Test 1.6: Learning - Quick Answer Improvement
**Condition**: Router makes quick answer decision, receives feedback, makes similar decision again
**Expected**: Second decision is better informed than first
**Verification**: Confidence in pattern increases

### 2. Clarifier Tests

#### Test 2.1: Ambiguity Detection
**Condition**: Clarifier receives ambiguous input
**Expected**: Clarifier detects ambiguity
**Verification**: Ambiguity detected = true

#### Test 2.2: Clarification Needs Identification
**Condition**: Clarifier detects ambiguity
**Expected**: Clarifier identifies what needs clarification
**Verification**: Clarification needs identified > 0

#### Test 2.3: Responder Integration
**Condition**: Clarifier identifies clarification needs
**Expected**: Clarifier passes to Responder for question generation
**Verification**: Responder receives clarification needs, generates questions

#### Test 2.4: Turn Completion
**Condition**: Responder generates clarifying questions
**Expected**: Turn ends with clarification request
**Verification**: Response sent to user, turn ends

### 3. Working Memory Context Assembly Tests

**Note**: Context assembly is now handled by Working Memory (Memory Orchestrator), not a separate daemon.

#### Test 3.1: Context Retrieval
**Condition**: Executor queries Working Memory for context
**Expected**: Working Memory retrieves relevant context
**Verification**: Context retrieved > 0

#### Test 3.2: Context Relevance
**Condition**: Working Memory retrieves context
**Expected**: Retrieved context is relevant to query
**Verification**: Context relevance score > threshold

#### Test 3.3: Threshold Logic
**Condition**: Working Memory assembles context
**Expected**: If total_tokens < threshold, return all; else use LLM to select
**Verification**: Threshold logic applied correctly

### 4. Executor Constraint Checking Tests

**Note**: Constraint checking is now a built-in capability of Executor, not a separate daemon.

#### Test 4.1: Constraint Identification
**Condition**: Executor checks proposed action
**Expected**: Executor identifies applicable constraints
**Verification**: Constraints identified > 0

#### Test 4.2: Compliance Check - Compliant
**Condition**: Proposed action complies with constraints
**Expected**: Executor marks as COMPLIANT
**Verification**: Status = COMPLIANT

#### Test 4.3: Compliance Check - Violation
**Condition**: Proposed action violates constraints
**Expected**: Executor marks as VIOLATION
**Verification**: Status = VIOLATION

#### Test 4.4: Violation Handling
**Condition**: Executor detects violation
**Expected**: Executor stops execution and escalates
**Verification**: Execution stopped, user notified

### 5. FC (Frontal Cortex) Planning Tests

**Note**: Planning is a capability of the Frontal Cortex daemon.

#### Test 5.1: Plan Creation
**Condition**: Executor queries FC for plan
**Expected**: FC creates execution plan
**Verification**: Plan created with steps > 0

#### Test 5.2: Step Ordering
**Condition**: Plan has dependent steps
**Expected**: Steps are ordered correctly
**Verification**: Dependencies respected in order

#### Test 5.3: Parallelism Identification
**Condition**: Plan has independent steps
**Expected**: FC identifies parallelizable steps
**Verification**: Independent steps marked as parallel

#### Test 5.4: Plan Update
**Condition**: Gut raises concern about plan
**Expected**: FC updates plan based on concern
**Verification**: Plan updated, new plan returned

### 6. Executor Tests

#### Test 6.1: Always Spawned
**Condition**: Turn starts (regardless of Router's decision)
**Expected**: Executor is always spawned
**Verification**: Executor spawned = true

#### Test 6.2: Context Gathering
**Condition**: Executor starts
**Expected**: Executor queries Working Memory for context
**Verification**: Context retrieved and available

#### Test 6.3: Plan Querying
**Condition**: Executor has context
**Expected**: Executor queries FC for plan
**Verification**: Plan received from FC with daemon list

#### Test 6.4: Flexible Daemon Calling
**Condition**: FC plan specifies daemons to call
**Expected**: Executor calls daemons in order specified by FC
**Verification**: Daemons called in correct order

#### Test 6.5: Parallel Daemon Execution
**Condition**: FC plan marks daemons as parallelizable
**Expected**: Executor calls daemons in parallel
**Verification**: Daemons called concurrently, results collected

#### Test 6.6: Sub-Turn Spawning
**Condition**: FC plan specifies spawn_subturn = true
**Expected**: Executor spawns new turn for parallel work
**Verification**: Sub-turn spawned and running

#### Test 6.7: Result Collection
**Condition**: Daemons return results
**Expected**: Executor collects and stores results
**Verification**: Results stored in working memory

#### Test 6.7: Responder Integration
**Condition**: Execution complete
**Expected**: Executor queries Responder for user message
**Verification**: Responder generates message

### 7. Error Handler Tests

**Note**: Error Handler is now a system-wide capability (not just for Executor).

#### Test 7.1: Error Detection
**Condition**: Tool fails
**Expected**: Error Handler detects error
**Verification**: Error detected = true

#### Test 7.2: Severity Assessment - Low
**Condition**: Non-critical error occurs
**Expected**: Error Handler assesses as LOW severity
**Verification**: Severity = LOW, action = log only

#### Test 7.3: Severity Assessment - Medium
**Condition**: Tool fails but alternative exists
**Expected**: Error Handler assesses as MEDIUM severity
**Verification**: Severity = MEDIUM, action = log + notify

#### Test 7.4: Severity Assessment - High
**Condition**: Critical tool fails
**Expected**: Error Handler assesses as HIGH severity
**Verification**: Severity = HIGH, action = log + notify + interrupt

#### Test 7.5: Severity Assessment - Critical
**Condition**: Multiple failures or safety violation
**Expected**: Error Handler assesses as CRITICAL severity
**Verification**: Severity = CRITICAL, action = log + stop + escalate

#### Test 7.6: Recovery Strategy
**Condition**: Error with recovery option
**Expected**: Error Handler executes recovery
**Verification**: Recovery executed successfully

### 8. Gut Daemon Tests

#### Test 8.1: Deviation Detection
**Condition**: Current action deviates from user intent
**Expected**: Gut detects deviation
**Verification**: Deviation detected = true

#### Test 8.2: Severity Assessment - Medium
**Condition**: Minor deviation from user intent
**Expected**: Gut assesses as MEDIUM severity
**Verification**: Severity = MEDIUM

#### Test 8.3: Severity Assessment - High
**Condition**: Significant deviation from user intent
**Expected**: Gut assesses as HIGH severity
**Verification**: Severity = HIGH

#### Test 8.4: Severity Assessment - Critical
**Condition**: Complete misalignment with user intent
**Expected**: Gut assesses as CRITICAL severity
**Verification**: Severity = CRITICAL

#### Test 8.5: Async Monitoring
**Condition**: Gut monitors execution in parallel
**Expected**: Concerns raised without blocking Executor
**Verification**: No deadlocks, concerns delivered

#### Test 8.6: Executor Attention Process
**Condition**: Gut raises HIGH severity concern
**Expected**: Executor reads concern and interrupts
**Verification**: Executor queries FC for plan update

### 9. Evaluator Tests

#### Test 9.1: Goal Achievement Assessment
**Condition**: Turn completes
**Expected**: Evaluator assesses if goal was achieved
**Verification**: Assessment produced

#### Test 9.2: Quality Assessment
**Condition**: Turn completes
**Expected**: Evaluator assesses result quality
**Verification**: Quality score produced

#### Test 9.3: Efficiency Assessment
**Condition**: Turn completes
**Expected**: Evaluator assesses execution efficiency
**Verification**: Efficiency score produced

#### Test 9.4: Outcome Assessment
**Condition**: Evaluator completes assessment
**Expected**: Outcome assessment provided to Reflector
**Verification**: Assessment includes all required fields

### 10. Reflector Tests

#### Test 10.1: Pattern Learning
**Condition**: Reflector receives outcome assessment
**Expected**: Reflector identifies patterns
**Verification**: Patterns extracted

#### Test 10.2: Knowledge Update
**Condition**: Reflector identifies patterns
**Expected**: Reflector updates episodic memory
**Verification**: Memory updated with new patterns

#### Test 10.3: Confidence Adjustment
**Condition**: Reflector learns from outcome
**Expected**: Confidence in patterns adjusted
**Verification**: Confidence scores updated

#### Test 10.4: Continuous Improvement
**Condition**: Multiple turns with similar patterns
**Expected**: Decisions improve over time
**Verification**: Success rate increases

### 11. Responder Tests

#### Test 11.1: Response Generation
**Condition**: Responder receives information to communicate
**Expected**: Responder generates response
**Verification**: Response generated

#### Test 11.2: Question Generation
**Condition**: Responder receives clarification needs
**Expected**: Responder generates clarifying questions
**Verification**: Questions generated

#### Test 11.3: Response Clarity
**Condition**: Responder generates response
**Expected**: Response is clear and understandable
**Verification**: User can understand response

#### Test 11.4: Response Appropriateness
**Condition**: Responder generates response
**Expected**: Response is appropriate for situation
**Verification**: Tone and format are appropriate

#### Test 11.5: Response Delivery
**Condition**: Responder generates response
**Expected**: Response is sent to user
**Verification**: Response delivered

### 12. Integration Tests

#### Test 12.1: Simple Turn - Quick Answer
**Condition**: User asks simple question (e.g., "What's 2+2?")
**Expected**: Turn completes with quick answer
**Verification**:
- Router detects quick answer possible
- Responder generates answer and sends to user
- Executor still spawned (for learning/updates)
- Turn completes successfully

#### Test 12.2: Complex Turn - Flexible Daemon Orchestration
**Condition**: User asks complex question requiring analysis (e.g., "Analyze this data and create a report")
**Expected**: Turn completes with comprehensive answer
**Verification**:
- Router detects needs execution
- Executor spawned
- Executor queries Working Memory for context
- Executor queries FC for plan
- FC returns plan with daemon list and parallelization info
- Executor calls daemons in order specified by FC
- Executor collects results
- Responder generates comprehensive response
- User receives complete answer

#### Test 12.3: Ambiguous Turn - Clarification
**Condition**: User asks ambiguous question (e.g., "Fix the bug")
**Expected**: Turn ends with clarification request
**Verification**:
- Router detects ambiguity
- Executor spawned
- FC plan includes Clarifier daemon
- Executor calls Clarifier
- Clarifier detects ambiguity
- Responder generates clarifying questions
- Responder sends clarification request
- Turn ends
- User can respond with clarification

#### Test 12.4: Urgent Turn - Quick Answer + Background Work
**Condition**: User asks urgent question (e.g., "Suspicious email from my bank")
**Expected**: Quick answer sent, background work continues
**Verification**:
- Router detects quick answer possible
- Responder generates immediate response and sends to user
- Executor spawned
- FC plan includes background work (research, learning)
- Executor calls daemons for background work
- Reflector learns from outcome
- Follow-up response sent when background work completes

#### Test 12.5: Constraint-Sensitive Turn
**Condition**: User asks question that may violate constraints
**Expected**: Constraints verified before action
**Verification**:
- Router decides "execution" approach
- Executor queries Working Memory for context
- Executor queries FC for plan
- Executor checks constraints before executing
- If compliant: Executor executes plan
- If violation: Executor stops and escalates
- Responder communicates result
- Turn completes appropriately

#### Test 12.6: Gut Concern Turn
**Condition**: Gut detects deviation from user intent during execution
**Expected**: Executor responds to concern
**Verification**:
- Executor starts execution
- Gut monitors asynchronously
- Gut detects deviation
- Gut raises concern with severity
- If severity > threshold: Executor interrupts
- Executor queries FC for plan update
- Plan updated or confirmed
- Execution continues or redirects

#### Test 12.7: Error Recovery Turn
**Condition**: Error occurs during turn execution
**Expected**: Error is handled gracefully
**Verification**:
- Executor detects error
- Error Handler assesses severity
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

#### Test 11.9: Daemon Communication
**Condition**: Daemons communicate during turn
**Expected**: Communication is correct and complete
**Verification**:
- Router passes context to activated daemons
- Clarifier provides clarifications to other daemons
- Executor passes results to Evaluator
- Evaluator passes assessment to Reflector
- All communication is correct
- No information lost
- All daemons receive necessary information

#### Test 11.10: Turn Lifecycle - Flexible Execution
**Condition**: Complete turn from input to response
**Expected**: Executor decides which lifecycle states to execute based on daemon outputs
**Verification**:
- Input received
- Executor queries other daemons (Router, Clarifier, etc.)
- Executor decides which lifecycle states are needed
- Executor executes only the necessary states
- Response sent to user
- Reflector learns from outcome
- Turn marked complete
- Turn Trace shows which states were executed and why

#### Test 11.11: Turn Lifecycle - State Skipping
**Condition**: Simple query that doesn't need planning or tool execution
**Expected**: Executor skips unnecessary states
**Verification**:
- Input: "What's 2+2?"
- Executor queries daemons
- Executor decides: only Responder needed
- Executor skips: Pre-Retrieval, WM Assembly, Thought/Planning, Action(s), Reflection
- Executor executes: Response, Commit/Flush
- Response sent immediately
- Turn Trace shows states that were skipped

## Test Execution Strategy

### Unit Tests
- Test each daemon independently
- Verify decision-making logic
- Verify output format
- Test with mocked dependencies
- Verify error handling
- Test edge cases

**Execution**: Run in parallel, each daemon tested independently

### Integration Tests
- Test daemons working together
- Verify communication between daemons
- Verify turn completion
- Test data flow between daemons
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
- Expected daemon activations
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

