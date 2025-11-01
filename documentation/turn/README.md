# Turn Architecture Documentation

## Overview

This directory contains comprehensive documentation of Si's turn architecture - the fundamental execution model that processes sensory input through hobgoblins (internal cognitive daemons) to generate responses.

## What is a Turn?

A **turn** spans from sensory input (user message or system trigger) to agent response. Multiple turns can run in parallel, and turns can end before all work completes (enabling immediate responses while background work continues).

## Core Concepts

### Hobgoblins vs Tools
- **Hobgoblins**: Internal cognitive components that make decisions (Router, Clarifier, Context Assembler, etc.)
- **Tools**: External capabilities Si can execute (web search, book reservation, check price, etc.)

### Decision-Making Model
All hobgoblins make decisions using:
- **Consciousness**: Si's values, mandates, identity, governance
- **Working Memory**: Current context, conversation history, turn trace
- **Episodic Memory**: Previous experiences, learned patterns, skills
- **Input**: The question or situation itself

### Learning Loop
Every turn feeds into a learning loop:
1. Hobgoblin makes decision
2. Turn executes
3. Evaluator assesses outcome
4. Reflector learns from outcome
5. Learning updates consciousness/episodic memory/skills
6. Next turn: hobgoblin makes better decision

## Documentation Structure

### Core Concepts
- **overview.md** - What is a turn, hobgoblins vs tools
- **decision-making-model.md** - How hobgoblins make decisions
- **learning-loop.md** - Feedback and continuous improvement
- **parallelism.md** - Concurrent execution within and across turns
- **response-timing.md** - Urgency-driven response timing
- **fast-path-optimization.md** - Technology optimization notes

### Hobgoblins (10 files)
Each hobgoblin has its own documentation:

1. **router.md** - Entry point; decides urgency and hobgoblin activation
2. **clarifier.md** - Handles ambiguous inputs; asks clarifying questions
3. **context-assembler.md** - Gathers relevant context from knowledge systems
4. **constraint-checker.md** - Verifies compliance with consciousness mandates
5. **plan-generator.md** - Creates execution plans
6. **executor.md** - Orchestrates tool execution
7. **error-handler.md** - Handles failures and recovery
8. **evaluator.md** - Assesses turn outcomes
9. **reflector.md** - Learns from outcomes and updates knowledge
10. **responder.md** - Generates responses to send to user

### Testing
- **test-conditions.md** - Comprehensive test conditions for all components

## Key Principles

### 1. Modularity First, Optimization Second
Hobgoblins are designed as separate components for clarity. For speed optimization, they can be combined into a single LLM call (see fast-path-optimization.md).

### 2. All Decisions Use Same Information Sources
Every hobgoblin decision uses consciousness + working memory + episodic memory + input. This enables consistent decision-making and learning.

### 3. Continuous Improvement Through Learning
The learning loop ensures that Si's decisions improve over time through experience. The system doesn't need perfect rules upfront.

### 4. Urgency Drives Timing
Si responds when the user needs guidance, not when all work is done. High-urgency situations get immediate responses while background work continues.

### 5. Parallelism is an Optimization
Hobgoblins can run in parallel within a turn, and multiple turns can run in parallel. This is an optimization, not a requirement.

## Reading Guide

### For Understanding the Architecture
1. Start with **overview.md** - understand what a turn is
2. Read **decision-making-model.md** - understand how decisions are made
3. Read **learning-loop.md** - understand how the system improves
4. Read **response-timing.md** - understand urgency-driven responses

### For Understanding Individual Hobgoblins
- Read the hobgoblin documentation in `hobgoblins/` directory
- Each hobgoblin has clear responsibilities and decision inputs

### For Understanding Optimization
- Read **fast-path-optimization.md** - understand how to optimize for speed

### For Testing
- Read **test-conditions.md** - understand what needs to be tested

## Technology Agnosticism

This documentation is technology-agnostic. It specifies WHAT needs to happen, not HOW to implement it. Technology choices (LLM, framework, language, etc.) are made during implementation.

## Next Steps

1. **Review**: Review this documentation with stakeholders
2. **Clarify**: Ask clarifying questions about any concepts
3. **Implement**: Use this documentation as specification for implementation
4. **Test**: Use test-conditions.md to verify implementation
5. **Optimize**: Use fast-path-optimization.md to optimize for speed

## Questions?

If you have questions about the turn architecture:
1. Check the relevant documentation file
2. Review the examples provided
3. Check test-conditions.md for concrete scenarios
4. Ask for clarification

## Version History

- **2025-10-31**: Initial comprehensive documentation of turn architecture

