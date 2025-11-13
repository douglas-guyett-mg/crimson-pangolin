# Response Timing: Urgency-Driven Responses

## Overview

Si doesn't always wait for all work to complete before responding. Instead, it uses urgency assessment to decide when to respond, enabling immediate action in critical situations while continuing deeper work in parallel.

## The Timing Decision

When Router assesses urgency, it determines not just WHAT to do, but WHEN to respond:

- **High Urgency**: Respond immediately, continue work in background
- **Medium Urgency**: Respond after initial analysis, continue deeper work
- **Low Urgency**: Complete work first, then respond

## High Urgency: Immediate Response + Background Work

### Pattern
```
User Input
    ↓
Router: HIGH URGENCY detected
    ↓
Responder: Generate immediate response
    ↓
Send Response (END TURN 1)
    ↓
Executor: Continue research in background
    ↓
Research completes
    ↓
Responder: Generate follow-up response
    ↓
Send Follow-up Response (END TURN 2)
```

### Example: Suspicious Email

**Turn 1:**
- Input: "I got a suspicious email from my bank about a transfer"
- Router detects: HIGH URGENCY (financial threat, time-sensitive)
- Responder generates: "Stop, don't do anything. Let me contact your bank to verify."
- Response sent (END TURN 1)
- Background: Executor starts researching, contacting bank

**Turn 2 (parallel):**
- Input: (none - continuation of Turn 1's work)
- Executor finishes: Bank confirms email is fraudulent
- Responder generates: "I confirmed with your bank - that email is fraudulent. Ignore it."
- Response sent (END TURN 2)

### Why This Matters

In high-urgency situations, the immediate response prevents the user from taking harmful action. The user gets guidance NOW, not after research completes.

## Medium Urgency: Partial Response + Continued Work

### Pattern
```
User Input
    ↓
Router: MEDIUM URGENCY detected
    ↓
Clarifier: Handle any ambiguities
    ↓
Responder: Generate partial response
    ↓
Send Response (END TURN 1)
    ↓
Executor: Continue deeper work
    ↓
Work completes
    ↓
Responder: Generate follow-up with results
    ↓
Send Follow-up Response (END TURN 2)
```

### Example: Travel Planning

**Turn 1:**
- Input: "I need to book a flight to Paris next week"
- Router detects: MEDIUM URGENCY (time-sensitive but not critical)
- Clarifier: Confirms dates and preferences
- Responder generates: "I'll help you plan. Let me search for flights, hotels, and activities for your Paris trip."
- Response sent (END TURN 1)
- Background: Executor searches flights, hotels, activities, prices

**Turn 2 (parallel):**
- Executor finishes: Hotels and activities found
- Responder generates: "Here are the best hotel options and top activities. Total estimated cost: $X"
- Response sent (END TURN 2)

## Low Urgency: Complete Work, Then Respond

### Pattern
```
User Input
    ↓
Router: LOW URGENCY detected
    ↓
Executor: Execute all steps
    ↓
Evaluator: Verify completeness
    ↓
Responder: Generate comprehensive response
    ↓
Send Response (END TURN 1)
```

### Example: General Information

**Turn 1:**
- Input: "Tell me about the history of coffee"
- Router detects: LOW URGENCY (informational, no time pressure)
- Executor executes: Gathers comprehensive information
- Evaluator verifies: All aspects covered
- Responder generates: Comprehensive response with history, trade, culture
- Response sent (END TURN 1)

No follow-up needed - all work completed before responding.

## Factors in Urgency Assessment

Router considers multiple factors when assessing urgency:

### Input Signals
- **Keywords**: "urgent", "emergency", "immediately", "now"
- **Domain**: Financial, health, safety → higher urgency
- **Time pressure**: "by tomorrow", "in 5 minutes" → higher urgency
- **Emotional tone**: Panic, fear → higher urgency

### Consciousness Factors
- **Mandates**: "Protect user from harm" → financial threats are high urgency
- **Values**: What does Si prioritize?
- **Governance**: Are there rules about response timing?

### Working Memory Factors
- **Conversation history**: Is this a continuation of urgent discussion?
- **Turn trace**: Did previous turns indicate urgency?
- **Current context**: What else is happening?

### Episodic Memory Factors
- **Similar situations**: How urgent were similar situations?
- **Learned patterns**: What patterns predict urgency?
- **Outcomes**: Did high urgency decisions work well?

## Multiple Parallel Turns

When multiple turns run in parallel, each has its own urgency assessment:

```
Turn 1: Suspicious email (HIGH URGENCY)
  - Immediate response sent
  - Research continues

Turn 2: Travel planning (MEDIUM URGENCY)
  - Partial response sent
  - Deeper work continues

Turn 3: General question (LOW URGENCY)
  - Complete work first
  - Then respond
```

All three turns can be active simultaneously, each with different response timing.

## Shared Working Memory Across Turns

When turns run in parallel, they share working memory:
- Turn 1 can see Turn 2's progress
- Turn 2 can see Turn 3's findings
- Updates from one turn are visible to others

This enables coordination and prevents duplicate work.

## Technology Implications

Response timing strategy affects technology choices:
- **Async execution**: Daemons must support async operations
- **Message queues**: Turns need to communicate asynchronously
- **Shared state**: Working memory must be accessible across turns
- **Scheduling**: System needs to schedule when responses are sent

These are implementation details chosen when technology stack is selected.

## Key Principle

**Urgency drives timing, not completion.** Si responds when the user needs guidance, not when all work is done. This enables appropriate action in critical situations while maintaining thoroughness in non-urgent situations.

