# Router Daemon

## Purpose

The Router is the first decision-making daemon. It assesses incoming input and decides:
- **Can we answer this quickly?** (Using Scratch Page or without additional tools?)
- **If YES**: Responder generates answer and sends to user immediately
- **If NO**: Continue (Executor will run regardless)

**Key insight**: Executor ALWAYS runs, regardless of Router's decision. Router may send a quick answer, but Executor still runs for learning and updates.

## Responsibilities

1. **Assess Input**: Understand what the user is asking
2. **Check Scratch Page**: Can this be answered using Scratch Page observations?
3. **Decide Quick Answer**: Can we answer this quickly without additional tools?
4. **If YES**: Activate Responder to send quick answer to user
5. **If NO**: Continue (Executor will handle it)
6. **Provide Context**: Pass relevant context to Responder (if activated)

## Decision Inputs

Router makes decisions using:

### Consciousness
- **Mandates**: What situations are considered urgent?
- **Values**: What does Si prioritize?
- **Governance**: Are there rules about routing?
- **Capabilities**: What is Si capable of doing?

### Working Memory
- **Conversation History**: What's been discussed?
- **Turn Trace**: What happened in previous turns?
- **Current Context**: What's the immediate situation?
- **Parallel Turns**: What else is happening?

### Episodic Memory
- **Similar Situations**: How urgent were similar inputs?
- **Learned Patterns**: What patterns predict urgency?
- **Outcomes**: Did similar routing decisions work well?
- **Skills**: What routing skills has Si developed?

### Input
- **Content**: What is being asked?
- **Intent**: What does the user actually want?
- **Urgency Signals**: Keywords, domain, time pressure, emotional tone
- **Constraints**: Are there limitations?

## Urgency Levels

### HIGH URGENCY
**Characteristics**:
- Time-critical situations
- Potential harm if not addressed immediately
- User needs immediate guidance to prevent bad action

**Examples**:
- Suspicious financial email
- Health emergency
- Security threat
- Urgent decision needed

**Router Decision**:
- Immediate response required
- Background work continues in parallel
- Follow-up response when research completes

### MEDIUM URGENCY
**Characteristics**:
- Time-sensitive but not critical
- User needs guidance but can wait for deeper analysis
- Partial response helpful, complete response better

**Examples**:
- Travel planning with deadline
- Job application with deadline
- Event planning
- Time-bound research

**Router Decision**:
- Partial response after initial analysis
- Deeper work continues in parallel
- Follow-up response with complete results

### LOW URGENCY
**Characteristics**:
- No time pressure
- User wants comprehensive answer
- Thoroughness more important than speed

**Examples**:
- General information requests
- Learning/educational queries
- Exploratory questions
- Historical information

**Router Decision**:
- Complete work before responding
- Single comprehensive response
- No follow-up needed

## Daemon Activation Patterns

Router decides which daemons to activate based on the situation:

### Pattern 1: Simple Direct Answer
```
Input: "What's 2+2?"
Router activates: Responder only
Flow: Input → Responder → Response
```

### Pattern 2: Clarification Needed
```
Input: "Fix the bug"
Router activates: Clarifier
Flow: Input → Clarifier → Ask clarification → End Turn 1
```

### Pattern 3: Complex Analysis
```
Input: "Analyze this data and create a report"
Router activates: Context Assembler, Plan Generator, Executor, Evaluator, Responder
Flow: Input → Context Assembler → Plan Generator → Executor → Evaluator → Responder → Response
```

### Pattern 4: Urgent with Background Work
```
Input: "Suspicious email from my bank"
Router activates: Responder (immediate), Executor (background), Evaluator, Reflector
Flow: Input → Responder (send) → Executor (background) → Evaluator → Reflector
```

## Decision Process

```
Router receives input
    ↓
Gather decision inputs:
  - Consciousness (mandates, values, governance)
  - Working Memory (history, context, parallel turns)
  - Episodic Memory (similar situations, learned patterns)
  - Input (content, intent, urgency signals)
    ↓
Assess urgency: HIGH / MEDIUM / LOW
    ↓
Decide daemon activation pattern
    ↓
Set turn context:
  - Urgency level
  - Activated daemons
  - Expected flow
  - Success criteria
    ↓
Activate daemons and coordinate execution
```

## Learning Through Router

Router improves its decisions through the learning loop:

**Feedback Sources**:
- Did the urgency assessment match the actual situation?
- Did the daemon activation pattern work well?
- Did the user get what they needed?
- Were there unintended consequences?

**Learning Updates**:
- Consciousness: Refine mandates about urgency
- Episodic Memory: Store successful/unsuccessful routing patterns
- Skills: Improve urgency detection and daemon activation

**Example**:
- Turn 1: Router sees "suspicious email" → HIGH urgency → immediate response
- Evaluator: User stopped, research succeeded → GOOD
- Reflector: Learn pattern "financial keywords + user concern = high urgency"
- Turn 2: Router sees similar situation → better informed decision

## Key Principle

**Router is an advisor, not the orchestrator.** It assesses urgency and recommends which daemons should be involved, but the **Executor daemon** makes the final decision about which lifecycle states to execute and orchestrates the turn. This separation enables Router to focus on urgency assessment while Executor handles orchestration.

