# Hobgoblin Decision-Making Model

## Core Principle

Every hobgoblin decision follows the same pattern: **combine consciousness + working memory + episodic memory + input to make a decision that can improve over time**.

## Decision Inputs

When a hobgoblin needs to make a decision, it considers:

### 1. Consciousness
Si's core identity and governance framework:
- **Values and Mandates**: What does Si care about? What are its priorities?
- **Identity**: Who is Si? What are its core characteristics?
- **Governance Rules**: What constraints apply to Si's decisions?
- **Capabilities Catalog**: What is Si capable of doing?

Example: Router checking consciousness to understand what situations are considered "urgent"

### 2. Working Memory
The current context and conversation state:
- **Conversation History**: What has been discussed so far?
- **Turn Trace**: What happened in previous turns?
- **Current Context**: What is the immediate situation?
- **Shared State**: What information is available from parallel turns?

Example: Context Assembler pulling relevant conversation history and turn trace

### 3. Episodic Memory
Si's learned experiences and patterns:
- **Previous Experiences**: What similar situations has Si encountered?
- **Learned Patterns**: What patterns have proven effective?
- **Skills**: What capabilities has Si developed through experience?
- **Outcomes**: What worked and what didn't?

Example: Router recalling that similar situations in the past required immediate response

### 4. Input
The immediate question or situation:
- **Content**: What is being asked?
- **Intent**: What does the user actually want?
- **Urgency Signals**: Are there time-sensitive elements?
- **Constraints**: Are there limitations on what can be done?

Example: Responder analyzing the user's message to understand what response is needed

## Decision Process

```
Hobgoblin receives task
    ↓
Gather decision inputs:
  - Consciousness (values, mandates, governance)
  - Working Memory (context, history, turn trace)
  - Episodic Memory (experiences, patterns, skills)
  - Input (question, intent, constraints)
    ↓
Make decision using all available information
    ↓
Execute decision
    ↓
Evaluator assesses outcome
    ↓
Reflector learns from outcome
    ↓
Learning updates consciousness/episodic memory/skills
    ↓
Next turn: hobgoblin has better information for similar decisions
```

## Example: Router's Urgency Decision

### Scenario
User: "I got a suspicious email from my bank about a transfer"

### Decision Inputs

**Consciousness**:
- Mandate: "Protect user from financial harm"
- Value: "Act quickly in emergencies"
- Governance: "Can contact external services if necessary"

**Working Memory**:
- Conversation history: First message from user
- Turn trace: No previous turns
- Current context: User is concerned about financial security

**Episodic Memory**:
- Pattern: "Suspicious financial emails often require immediate action"
- Skill: "Can verify with banks quickly"
- Experience: "Previous similar situations benefited from immediate response"

**Input**:
- Content: Suspicious email about transfer
- Intent: User wants help understanding if it's real
- Urgency signals: "suspicious", "bank", "transfer" (financial keywords)
- Constraints: None apparent

### Decision
Router decides: **HIGH URGENCY**
- Immediate response needed to prevent user taking bad action
- Background research should continue in parallel
- Follow-up response when research completes

### Outcome Assessment
Evaluator checks: Did the urgency decision work?
- Did user avoid taking bad action? ✓
- Did we respond quickly enough? ✓
- Did background research complete successfully? ✓

### Learning
Reflector updates:
- Consciousness: Reinforces mandate about financial protection
- Episodic Memory: Stores this as successful high-urgency case
- Skills: Refines pattern recognition for financial urgency

### Next Time
Router sees similar situation → makes better decision because it learned from this experience

## Hobgoblins That Use This Model

All hobgoblins follow this decision-making pattern:

- **Router**: Decides urgency, which hobgoblins to activate
- **Clarifier**: Decides what clarification questions to ask
- **Context Assembler**: Decides what context is relevant
- **Constraint Checker**: Decides if constraints are violated
- **Plan Generator**: Decides what steps to take
- **Executor**: Decides which tools to call and in what order
- **Error Handler**: Decides how to handle failures
- **Evaluator**: Decides if turn is complete
- **Reflector**: Decides what to learn and how to update knowledge
- **Responder**: Decides what response to generate

## Key Insight: Continuous Improvement

Because every decision uses episodic memory (learned experiences), Si's decisions improve over time. The system doesn't need perfect rules upfront - it learns from experience.

This is why the learning loop (see `learning-loop.md`) is so critical to the architecture.

