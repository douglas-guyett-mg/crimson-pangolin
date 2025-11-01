# Turn Architecture Overview

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

## Hobgoblins vs Tools

Si's mind is composed of two distinct types of components:

### Hobgoblins (Internal Cognitive Daemons)

**What they are**: Internal decision-making components that form Si's cognitive architecture.

**What they do**: Make decisions, coordinate work, learn from experience.

**Examples**:
- Router: Decides which hobgoblins to activate and what urgency level to assign
- Plan Generator: Decides what steps to take
- Executor: Decides which tools to call and in what order
- Evaluator: Decides if work is complete
- Reflector: Learns from outcomes and updates Si's knowledge

**Key insight**: Hobgoblins are always part of Si's internal thinking, regardless of whether tools are used.

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

**Key insight**: Tools are only used when a hobgoblin (usually Executor) decides they're needed.

## The Distinction Matters

This separation is crucial because:

1. **Hobgoblins are always thinking** - even if no tools are used, hobgoblins are making decisions
2. **Tools are optional** - some turns use many tools, others use none
3. **Hobgoblins improve over time** - through the learning loop, hobgoblins make better decisions
4. **Tools are static** - they do what they're designed to do, but don't learn

## Core Hobgoblins

Si's cognitive architecture includes these core hobgoblins (more may be added):

1. **Router** - Entry point; decides urgency, which hobgoblins to activate
2. **Clarifier** - Handles ambiguous inputs; asks follow-up questions
3. **Context Assembler** - Gathers relevant context from consciousness, working memory, turn trace
4. **Constraint Checker** - Verifies decisions against consciousness mandates and governance rules
5. **Plan Generator** - Creates execution plans when needed
6. **Executor** - Orchestrates tool execution
7. **Error Handler** - Handles failures; decides whether to retry, escalate, or fail gracefully
8. **Evaluator** - Assesses whether the turn is complete and successful
9. **Reflector** - Learns from the turn; updates consciousness, episodic memory, and skills
10. **Responder** - Generates responses to send to the user

## Decision-Making Foundation

All hobgoblins make decisions using the same information sources:

- **Consciousness**: Si's values, mandates, identity, and governance rules
- **Working Memory**: Current context, conversation history, turn trace
- **Episodic Memory**: Previous experiences, learned patterns, skills
- **Input**: The question or situation itself

This means hobgoblins can improve their decisions over time through learning (see Learning Loop).

## Next Steps

See related documentation:
- `hobgoblins/` - Detailed documentation for each hobgoblin
- `decision-making-model.md` - How hobgoblins make decisions
- `learning-loop.md` - How Si improves through experience
- `parallelism.md` - How turns and hobgoblins run in parallel
- `response-timing.md` - How urgency drives response timing

