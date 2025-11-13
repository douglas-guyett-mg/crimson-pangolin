# Daemon Decision-Making Model

**Status:** Specification v1.0
**Date:** 2025-11-01
**Note:** This document uses "daemon" terminology. See `documentation/turn/README.md` for the distinction between daemons (internal cognitive agents) and tools (external capabilities).

## Core Principle

Every daemon decision follows the same pattern: **combine consciousness + working memory + episodic memory + input to make a decision that can improve over time**.

## Decision Inputs

When a daemon needs to make a decision, it gathers information from four sources by querying other daemons using the Daemon Interface (see `documentation/daemon-interface.md`):

### 1. Consciousness Daemon
Si's core identity and governance framework:
- **Values and Mandates**: What does Si care about? What are its priorities?
- **Identity**: Who is Si? What are its core characteristics?
- **Governance Rules**: What constraints apply to Si's decisions?
- **Capabilities Catalog**: What is Si capable of doing?

**How to access:** `consciousness.query("What mandates apply to financial security?")`

Example: Router querying consciousness to understand what situations are considered "urgent"

### 2. Working Memory Daemon
The current context and conversation state:
- **Conversation History**: What has been discussed so far?
- **Turn Trace**: What happened in previous turns?
- **Current Context**: What is the immediate situation?
- **Shared State**: What information is available from parallel turns?

**How to access:** `working_memory.query("What is the current conversation context?")`

### 3. Episodic Memory Daemon
Si's learned experiences and patterns:
- **Previous Experiences**: What similar situations has Si encountered?
- **Learned Patterns**: What patterns have proven effective?
- **Skills**: What capabilities has Si developed through experience?
- **Outcomes**: What worked and what didn't?

**How to access:** `episodic_memory.query("What patterns have we learned about suspicious financial emails?", metadata={tags: ["financial", "security"]})`

Example: Router querying episodic memory to recall that similar situations in the past required immediate response

### 4. Input
The immediate question or situation:
- **Content**: What is being asked?
- **Intent**: What does the user actually want?
- **Urgency Signals**: Are there time-sensitive elements?
- **Constraints**: Are there limitations on what can be done?

Example: Responder analyzing the user's message to understand what response is needed

## Decision Process

```
Daemon receives task
    ↓
Query other daemons for decision inputs (async):
  - consciousness.query("What mandates apply?")
  - working_memory.query("What is the context?")
  - episodic_memory.query("What patterns have we learned?")
  - Input (question, intent, constraints)
    ↓
Wait for results (can do other work in parallel)
    ↓
Make decision using all available information
    ↓
Log decision to Turn Trace
    ↓
Execute decision (may invoke tools or other daemons)
    ↓
Evaluator daemon assesses outcome
    ↓
Reflector daemon learns from outcome
    ↓
Learning updates consciousness/episodic memory/skills
    ↓
Next turn: daemon has better information for similar decisions
```

**Key point:** All daemon queries are asynchronous (return futures), so a daemon can:
1. Query multiple daemons in parallel
2. Do other work while waiting for results
3. Use results when they arrive
4. Never block waiting for a single response

## Example: Router Daemon's Urgency Decision

### Scenario
User: "I got a suspicious email from my bank about a transfer"

### Decision Process

**Step 1: Router queries other daemons (async)**
```
future_consciousness = consciousness.query(
  "What mandates apply to financial security?"
)
future_working = working_memory.query(
  "What is the current conversation context?"
)
future_episodic = episodic_memory.query(
  "What patterns have we learned about suspicious financial emails?",
  metadata={tags: ["financial", "security"]}
)
```

**Step 2: Router can do other work while waiting**
- Activate other daemons in parallel
- Prepare for different urgency levels
- Set up response templates

**Step 3: Results arrive**
```
consciousness_result = await future_consciousness
# Returns: Mandate "Protect user from financial harm", Value "Act quickly in emergencies"

working_result = await future_working
# Returns: First message from user, no prior context, user is concerned

episodic_result = await future_episodic
# Returns: Pattern "Suspicious financial emails often require immediate action"
```

**Step 4: Router makes decision**
- Combines all inputs
- Decides: **HIGH URGENCY**
- Logs decision to Turn Trace with rationale

**Step 5: Router executes decision**
- Activates Executor daemon to handle immediately
- Activates Reflector daemon to learn from outcome
- Sends immediate response to user

### Outcome Assessment
Evaluator daemon checks: Did the urgency decision work?
- Did user avoid taking bad action? ✓
- Did we respond quickly enough? ✓
- Did background research complete successfully? ✓

### Learning
Reflector daemon updates:
- Consciousness: Reinforces mandate about financial protection
- Episodic Memory: Stores this as successful high-urgency case
- Skills: Refines pattern recognition for financial urgency

### Next Time
Router sees similar situation → queries daemons → gets better information → makes better decision because it learned from this experience

## Daemons That Use This Model

All coordination daemons follow this decision-making pattern:

- **Router Daemon**: Decides urgency, which daemons to activate
- **Clarifier Daemon**: Decides what clarification questions to ask
- **Executor Daemon**: Decides which tools to call and in what order
- **Error Handler Daemon**: Decides how to handle failures
- **Evaluator Daemon**: Decides if turn is complete
- **Reflector Daemon**: Decides what to learn and how to update knowledge
- **Responder Daemon**: Decides what response to generate

Memory daemons also use this pattern when deciding how to handle queries:
- **Consciousness Daemon**: Decides which consciousness items to return
- **Episodic Memory Daemon**: Decides which memories are relevant
- **Semantic Memory Daemon**: Decides which semantic knowledge to return
- **Working Memory Daemon**: Decides what context is most important

## Key Insight: Continuous Improvement

Because every decision uses episodic memory (learned experiences), Si's decisions improve over time. The system doesn't need perfect rules upfront - it learns from experience.

This is why the learning loop (see `learning-loop.md`) is so critical to the architecture.

## Daemon Communication

All daemon-to-daemon communication uses the Daemon Interface (see `documentation/daemon-interface.md`):
- `query(natural_language_query, metadata?, data?)` - Send a query
- `get_purpose()` - Understand what the daemon does
- `get_instructions()` - Understand how the daemon operates

All operations are asynchronous, enabling parallelism and efficient resource usage.

