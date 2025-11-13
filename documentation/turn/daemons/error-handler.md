# Error Handler

## Purpose

The Error Handler is a system-wide capability that manages failures across Si. When something goes wrong, Error Handler decides whether to retry, use an alternative approach, escalate, or fail gracefully.

**Note**: Error Handler is not a separate daemon. It's a capability that can be invoked by any component (Executor, Router, Gut, Reflector, or any processing piece) when errors occur.

## Responsibilities

1. **Receive Error**: Accept error notification from any component
2. **Analyze Error**: Understand what went wrong
3. **Assess Severity**: Determine severity level (low, medium, high, critical)
4. **Decide Recovery**: Choose recovery strategy based on severity
5. **Execute Recovery**: Implement the chosen strategy
6. **Report Status**: Inform caller of outcome

## When Error Handler is Invoked

Any component can invoke Error Handler when:
- Tool execution fails
- Tool returns error
- Tool times out
- Tool returns unexpected result
- Constraint violation detected
- Resource limit exceeded
- Any other error condition

**Examples**:
- Flight search tool times out
- Hotel search returns no results
- API rate limit exceeded
- Network connection fails
- Tool returns malformed data
- FC planning fails
- Working Memory query fails

## Decision Inputs

Error Handler makes decisions using:

### Consciousness
- **Governance**: What error handling rules apply?
- **Values**: What matters in error handling?
- **Capabilities**: What recovery options are available?

### Working Memory
- **Error Details**: What went wrong?
- **Context**: What was being attempted?
- **Plan**: What was the original plan?
- **Attempt History**: How many times have we tried?

### Episodic Memory
- **Similar Errors**: How were similar errors handled?
- **Learned Patterns**: What recovery strategies work?
- **Outcomes**: What error handling succeeded?
- **Skills**: What error handling skills has Si developed?

### Input
- **Error**: What is the error?
- **Context**: What was happening?
- **Constraints**: What limitations exist?

## Recovery Strategies

### Strategy 1: Retry
**When**: Transient error (temporary network issue, timeout)
**How**: Retry the same tool call
**Conditions**:
- Error is likely transient
- Haven't exceeded retry limit
- Retrying won't violate constraints

**Example**:
- Error: Flight search times out
- Decision: Retry (network might be temporarily slow)
- Action: Call flight_search again

### Strategy 2: Alternative Tool
**When**: Tool fails but alternative exists
**How**: Use different tool to accomplish same goal
**Conditions**:
- Alternative tool available
- Alternative tool can accomplish goal
- Alternative tool won't violate constraints

**Example**:
- Error: Primary flight search fails
- Decision: Use alternative flight search tool
- Action: Call alternative_flight_search

### Strategy 3: Partial Success
**When**: Tool partially succeeds
**How**: Use partial results and continue
**Conditions**:
- Partial results are useful
- Plan can proceed with partial results
- User can be informed of limitations

**Example**:
- Error: Hotel search returns only 2 results instead of expected 10
- Decision: Use 2 results and continue
- Action: Proceed with comparison using available results

### Strategy 4: Escalate
**When**: Error requires human intervention
**How**: Alert user and ask for guidance
**Conditions**:
- Error cannot be automatically recovered
- User input is needed
- Escalation won't violate constraints

**Example**:
- Error: Need user's credit card to complete booking
- Decision: Escalate to user
- Action: Ask user for authorization

### Strategy 5: Fail Gracefully
**When**: Error cannot be recovered
**How**: Acknowledge failure and provide alternatives
**Conditions**:
- No recovery strategy available
- Continuing would violate constraints
- User should be informed

**Example**:
- Error: All flight search tools fail
- Decision: Fail gracefully
- Action: Inform user that search failed, suggest alternatives

## Severity Levels

Error Handler assesses severity and takes action accordingly:

### Low Severity
**Examples**: Non-critical tool returns unexpected format, minor timeout
**Action**: Log error, continue execution
**Notification**: None (logged only)

### Medium Severity
**Examples**: Tool fails but alternative exists, partial results available
**Action**: Log error, attempt recovery (retry or alternative), notify via email/log
**Notification**: Email notification to user/admin

### High Severity
**Examples**: Critical tool fails, constraint violation, resource limit exceeded
**Action**: Log error, attempt recovery, interrupt flow if needed, escalate to user
**Notification**: Email + interrupt flow + ask for guidance

### Critical Severity
**Examples**: Multiple failures, unrecoverable error, safety violation
**Action**: Log error, stop execution, escalate immediately
**Notification**: Email + immediate escalation + stop turn

## Error Handling Process

```
Any component detects error
    ↓
Error Handler receives error notification
    ↓
Analyze error:
  - What is the error?
  - What was being attempted?
  - How many times have we tried?
  - What recovery options exist?
    ↓
Assess severity:
  - Low / Medium / High / Critical
    ↓
Decide recovery strategy based on severity:
  - Low: Log only
  - Medium: Log + retry/alternative + notify
  - High: Log + retry/alternative + interrupt + escalate
  - Critical: Log + stop + escalate immediately
    ↓
Execute recovery:
  - Implement chosen strategy
  - Monitor for success
  - Collect results
    ↓
Report status:
  - Inform caller of outcome
  - Update working memory
  - Alert other components if needed
    ↓
Continue, retry, or abort based on outcome
```

## Retry Limits

Error Handler respects retry limits:

```
Attempt 1: Fails
  → Retry (Attempt 2)
    ↓
Attempt 2: Fails
  → Retry (Attempt 3)
    ↓
Attempt 3: Fails
  → Try alternative strategy
    ↓
If all strategies fail:
  → Fail gracefully
```

Retry limits prevent infinite loops and resource waste.

## Learning Through Error Handler

Error Handler improves through the learning loop:

**Feedback Sources**:
- Did the recovery strategy work?
- Was the recovery strategy appropriate?
- Could a different strategy have worked better?
- Did error handling prevent worse outcomes?

**Learning Updates**:
- Episodic Memory: Store successful error recovery patterns
- Skills: Improve error handling for similar errors
- Consciousness: Refine error handling policies if needed

**Example**:
- Turn 1: Flight search times out
- Error Handler retries
- Retry succeeds
- Evaluator: Error handling was effective → GOOD
- Reflector: Learn "Timeout errors usually succeed on retry"

## Key Principle

**Error Handler enables resilience.** By handling errors gracefully, Error Handler ensures that temporary failures don't derail the entire turn, and that users are informed when problems occur.

