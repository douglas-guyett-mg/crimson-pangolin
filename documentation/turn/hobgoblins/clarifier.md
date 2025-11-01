# Clarifier Hobgoblin

## Purpose

The Clarifier handles ambiguous or incomplete user inputs. When Router detects that clarification is needed, Clarifier asks targeted follow-up questions to understand what the user actually wants.

## Responsibilities

1. **Detect Ambiguity**: Identify what's unclear in the input
2. **Ask Questions**: Generate clarifying questions
3. **Gather Information**: Collect missing context
4. **End Turn**: Send clarification request and end the turn

## When Clarifier is Activated

Router activates Clarifier when:
- Input is ambiguous or unclear
- Multiple interpretations are possible
- Critical information is missing
- User intent is unclear

**Examples**:
- User: "Fix the bug" (which bug? what symptoms?)
- User: "I need a recommendation" (for what? what criteria?)
- User: "Help me with my project" (what project? what kind of help?)

## Decision Inputs

Clarifier makes decisions using:

### Consciousness
- **Communication Style**: How should Si ask questions?
- **Governance**: Are there constraints on what can be asked?
- **Values**: What information is most important to clarify?

### Working Memory
- **Conversation History**: What's already been discussed?
- **Turn Trace**: What context is available?
- **Current Context**: What does Si already know?

### Episodic Memory
- **Similar Ambiguities**: How were similar cases clarified?
- **Learned Patterns**: What questions work best?
- **Outcomes**: Which clarification approaches succeeded?

### Input
- **Ambiguous Elements**: What's unclear?
- **Missing Information**: What's needed?
- **User Intent Signals**: What does the user seem to want?

## Clarification Process

```
Router activates Clarifier
    ↓
Clarifier analyzes input:
  - What's ambiguous?
  - What's missing?
  - What's the most important to clarify?
    ↓
Clarifier generates questions:
  - Prioritize by importance
  - Keep questions focused
  - Make questions clear and specific
    ↓
Responder generates response:
  - Acknowledge the input
  - Ask clarifying questions
  - Invite user to provide more information
    ↓
Send response (END TURN 1)
    ↓
User provides clarification
    ↓
Turn 2 starts with clarified input
```

## Question Types

### Focused Questions
Ask about one specific thing:
- "Which bug are you referring to?"
- "What type of recommendation do you need?"
- "What's the deadline for this project?"

### Multiple Choice Questions
Offer options to narrow down:
- "Are you asking about the login bug, the search bug, or something else?"
- "Do you need a restaurant recommendation, a book recommendation, or a travel recommendation?"

### Context Questions
Gather background information:
- "What have you already tried?"
- "When did this start happening?"
- "Who else is involved?"

### Priority Questions
Understand what matters most:
- "What's most important to you: speed, cost, or quality?"
- "Are you looking for a quick fix or a long-term solution?"

## Learning Through Clarifier

Clarifier improves through the learning loop:

**Feedback Sources**:
- Did the user understand the questions?
- Did the clarification questions lead to useful information?
- Did the user provide the information needed?
- Did the clarification enable a good response in Turn 2?

**Learning Updates**:
- Episodic Memory: Store successful clarification patterns
- Skills: Improve question generation and prioritization
- Consciousness: Refine communication style if needed

**Example**:
- Turn 1: Clarifier asks "Which bug?" and "What symptoms?"
- Turn 2: User provides clear information
- Evaluator: Clarification was effective → GOOD
- Reflector: Learn "These two questions work well for bug reports"

## Key Principle

**Clarifier prevents wasted work.** By asking clarifying questions upfront, Clarifier ensures that subsequent hobgoblins (Plan Generator, Executor, etc.) have clear direction and don't waste time on wrong assumptions.

