# Gut Daemon

## Purpose

The Gut Daemon is an async monitor that watches Si's execution and raises internal concerns when something feels wrong. It detects when Si is deviating from the user's intent or input stimulus, acting as an internal voice that alerts the Executor to potential issues.

**Note**: Gut Daemon runs asynchronously in parallel with Executor, continuously monitoring the turn.

## Responsibilities

1. **Monitor Execution**: Watch what Si is doing during the turn
2. **Detect Deviation**: Identify when Si is deviating from user intent
3. **Assess Severity**: Determine how serious the deviation is
4. **Raise Concern**: Alert Executor with concern message and severity level
5. **Enable Redirection**: Allow Executor to decide if concern warrants attention

## What Gut Watches For

### Deviation from User Intent
- "Why are we using a short code lookup tool to ask about an email?"
- "This plan doesn't match what the user asked for"
- "We're going down a rabbit hole that's not relevant"
- "This tool call doesn't make sense for this input"

### NOT Concerned With
- Token limits or budget overruns
- Performance metrics
- Optimization opportunities
- Non-critical inefficiencies

## Decision Inputs

Gut makes decisions using:

### Original User Input
- **User Query**: What did the user actually ask for?
- **Intent**: What is the user trying to accomplish?

### Current Execution
- **Current Action**: What is Si doing right now?
- **Current Plan**: What steps are being taken?
- **Tool Calls**: What tools are being invoked?

### Consciousness
- **Mandates**: What should Si prioritize?
- **Values**: What matters to Si?

## Concern Detection Process

```
Gut monitors execution (async)
    ↓
Compare current action against original user intent
    ↓
Semantic similarity check:
  - How similar is current action to user intent?
  - Similarity score: 0.0 - 1.0
    ↓
If similarity < threshold (e.g., 0.6):
  → Raise concern
    ↓
Assess severity:
  - Minor deviation: MEDIUM severity
  - Major deviation: HIGH severity
  - Critical misalignment: CRITICAL severity
    ↓
Output concern message + severity level
    ↓
Executor reads concern via daemon interface
    ↓
Executor decides if concern warrants attention
```

## Severity Levels

### MEDIUM Severity
**Example**: "Tool choice seems slightly off-topic but might be relevant"
**Executor Action**: Log concern, continue monitoring

### HIGH Severity
**Example**: "This plan doesn't match the user's request"
**Executor Action**: Interrupt flow, ask FC/Planner to check and update plan

### CRITICAL Severity
**Example**: "Complete misalignment with user intent"
**Executor Action**: Stop execution, escalate to user

## Executor's Attention Process

When Gut raises a concern:

```
Gut raises concern (severity level)
    ↓
Executor reads concern
    ↓
If severity > threshold:
  → Interrupt current flow
  → Query FC: "Should we update the plan?"
  → Query Planner: "Is this plan still aligned?"
  → Decide: Continue, redirect, or escalate
Else:
  → Log concern, continue execution
```

## Learning Through Gut

Gut improves through the learning loop:

**Feedback Sources**:
- Was the concern valid?
- Did the concern help prevent problems?
- Were there false positives (flagged when not actually a problem)?
- Were there false negatives (missed actual deviations)?

**Learning Updates**:
- Episodic Memory: Store patterns of valid concerns
- Skills: Improve deviation detection accuracy
- Consciousness: Refine what "user intent" means

**Example**:
- Turn 1: Gut flags tool choice as off-topic
- Executor checks with FC
- FC confirms: "Yes, this tool is wrong"
- Evaluator: Gut's concern was valid → GOOD
- Reflector: Learn "This type of deviation is important to catch"

## Algorithm Details

### Semantic Similarity Check (Current Implementation)

Gut uses semantic similarity to detect deviation:

1. **Extract user intent** from original query
2. **Extract current action** from execution trace
3. **Compute similarity score** (0.0 - 1.0)
4. **Compare to threshold** (default: 0.6)
5. **Raise concern if below threshold**

**Future Enhancement**: Combine semantic similarity with LLM-based reasoning for more sophisticated deviation detection.

## Key Principle

**Gut is Si's internal voice.** By continuously monitoring and raising concerns about deviation from user intent, Gut helps Executor catch problems early and redirect when needed, ensuring Si stays focused on what the user actually wants.

