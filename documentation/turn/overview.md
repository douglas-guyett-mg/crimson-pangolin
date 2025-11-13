# Turn Architecture Overview

**Related Documentation**: See `../turn.md` for technical specification, lifecycle states, budgets, invariants, and testing requirements.

## What is a Turn?

A **turn** is the fundamental unit of Si's execution model. It spans from the moment Si receives sensory input (user message or system trigger) to the moment Si sends a response.

```
Sensory Input → Si's Cognitive Process → Response Output = One Turn
```

### Key Characteristics

1. **Bounded by I/O**: A turn begins with input and ends when a response is sent
2. **Work can continue after response**: A turn can end (response sent) while background work continues
3. **Multiple turns can run in parallel**: Si can process multiple inputs simultaneously
4. **Shared working memory**: Parallel turns access and update the same working memory

### Example: Suspicious Email Scenario

**Turn 1:**
- Input: "I got a suspicious email from my bank about a transfer"
- Si's process: Router detects urgency → Responder generates immediate response → Executor starts research
- Output: "Stop, don't do anything. Let me contact your bank to verify."
- End of Turn 1 (response sent, but research continues in background)

**Turn 2 (parallel):**
- Input: (none - this is continuation of Turn 1's background work)
- Si's process: Executor finishes research → Responder generates follow-up
- Output: "I confirmed with your bank - that email is fraudulent. Ignore it."
- End of Turn 2

## Daemons vs Tools

Si's mind is composed of two distinct types of components:

### Daemons (Internal Cognitive Components)

**What they are**: Internal decision-making components that form Si's cognitive architecture. All daemons implement the Daemon Interface (see `documentation/daemon-interface.md`).

**What they do**: Make decisions, coordinate work, learn from experience.

**Examples**:
- **Router**: Decides urgency level and which daemons should be involved
- **Executor**: Orchestrates the turn by deciding which lifecycle states to execute and coordinating other daemons
- **Plan Generator**: Decides what steps to take
- **Evaluator**: Decides if work is complete
- **Reflector**: Learns from outcomes and updates Si's knowledge

**Key insight**: Daemons are always part of Si's internal thinking, regardless of whether tools are used. The **Executor daemon** is special: it orchestrates the entire turn by querying other daemons and deciding which lifecycle states to execute.

### Tools (External Capabilities)

**What they are**: External capabilities Si can invoke to take action in the world.

**What they do**: Execute specific tasks (search, reserve, check, etc.).

**Examples**:
- Web search
- Expert archive search
- Book reservation
- Check wine price
- Check phone number owner
- Contact bank
- Send message

**Key insight**: Tools are only used when a daemon (usually Executor) decides they're needed.

## The Turn Flow

Si's turn follows a flexible, data-driven flow:

```
User Input
  ↓
Router: "Can we answer this quickly using Scratch Page?"
  ├─ YES → Responder generates answer → Send to user
  └─ NO → Continue
  ↓
Executor ALWAYS spawned (regardless of Router's decision)
  ↓
Executor queries Working Memory for context
  ↓
Executor queries FC: "What's the plan?"
  ↓
FC returns natural language plan with steps:
  [Natural language narrative]

  ## Step 1: [Name]
  Success Criteria: [...]
  Actions: [...]
  Parallelizable with: [...]

  ## Step 2: [Name]
  ...
  ↓
Executor executes plan:
  - For each step (or parallel group):
    - Use LLM to interpret step description
    - Decide which daemons/tools to call
    - Execute decisions
    - Check success criteria
  - Handle errors and constraints
  - Log all context to Turn Trace for learning
  ↓
Send response to user (if not already sent by Router)
  ↓
Update Working Memory at commit
```

**Key insight**: The daemon flow is **not fixed** - it's determined by FC's plan. Executor is flexible and can call any daemons in any order/parallelization based on what FC decides.

## The Distinction Matters

This separation is crucial because:

1. **Daemons are always thinking** - even if no tools are used, daemons are making decisions
2. **Tools are optional** - some turns use many tools, others use none
3. **Daemons improve over time** - through the learning loop, daemons make better decisions
4. **Tools are static** - they do what they're designed to do, but don't learn
5. **Executor orchestrates, not dictates** - Executor decides which states to execute based on other daemons' outputs, not by following a fixed sequence

## Core Daemons

Si's cognitive architecture includes these core daemons:

### Decision Daemons
1. **Router** - Entry point; decides urgency and approach (immediate response, clarification, or execution)
2. **Clarifier** - Handles ambiguous inputs; identifies what needs clarification
3. **Gut** - Async monitor; watches execution and raises concerns about deviation from user intent

### Execution Daemons
4. **Executor** - Turn orchestrator; gathers context, queries FC for plans, executes, monitors Gut
5. **FC (Frontal Cortex)** - Planning daemon; creates/updates plans based on context

### Communication Daemons
6. **Responder** - Generates user-facing messages (questions or responses)

### Learning Daemons
7. **Evaluator** - Assesses outcomes and produces assessments
8. **Reflector** - Learns from outcomes and updates knowledge systems
9. **Responder** - Generates responses to send to the user

## Decision-Making Foundation

All daemons make decisions using the same information sources:

- **Consciousness**: Si's values, mandates, identity, and governance rules
- **Working Memory**: Current context, conversation history, turn trace
- **Episodic Memory**: Previous experiences, learned patterns, skills
- **Input**: The question or situation itself

This means daemons can improve their decisions over time through learning (see Learning Loop).

## Next Steps

See related documentation:
- `daemons/` - Detailed documentation for each daemon
- `decision-making-model.md` - How daemons make decisions
- `learning-loop.md` - How Si improves through experience
- `parallelism.md` - How turns and daemons run in parallel
- `response-timing.md` - How urgency drives response timing

