# Turn Architecture Documentation

## Overview

This directory contains comprehensive documentation of Si's turn architecture - the fundamental execution model that processes sensory input through daemons (internal cognitive components) to generate responses.

## What is a Turn?

A **turn** spans from sensory input (user message or system trigger) to agent response. Multiple turns can run in parallel, and turns can end before all work completes (enabling immediate responses while background work continues).

## Core Concepts

### Daemons vs Tools
- **Daemons**: Internal cognitive components that make decisions (Router, Clarifier, Executor, etc.)
- **Tools**: External capabilities Si can execute (web search, book reservation, check price, etc.)

### Decision-Making Model
All daemons make decisions using:
- **Consciousness**: Si's values, mandates, identity, governance
- **Working Memory**: Current context, conversation history, turn trace
- **Episodic Memory**: Previous experiences, learned patterns, skills
- **Input**: The question or situation itself

### Learning Loop
Every turn feeds into a learning loop:
1. Daemon makes decision
2. Turn executes
3. Evaluator assesses outcome
4. Reflector learns from outcome
5. Learning updates consciousness/episodic memory/skills
6. Next turn: daemon makes better decision

## Documentation Structure

### Core Concepts
- **overview.md** - What is a turn, daemons vs tools
- **decision-making-model.md** - How daemons make decisions
- **learning-loop.md** - Feedback and continuous improvement
- **parallelism.md** - Concurrent execution within and across turns
- **response-timing.md** - Urgency-driven response timing
- **fast-path-optimization.md** - Technology optimization notes

### Daemon Coordination
- **daemon-coordination-protocol.md** - Comprehensive specification of daemon-to-daemon communication including message format, async semantics, ordering guarantees, conflict resolution, and deadlock prevention
- **daemon-coordination-protocol-test-conditions.md** - Test scenarios for daemon coordination

### Daemons (10 files)
Each daemon has its own documentation:

1. **router.md** - Entry point; decides urgency and daemon activation
2. **clarifier.md** - Handles ambiguous inputs; asks clarifying questions
3. **context-assembler.md** - Gathers relevant context from knowledge systems
4. **constraint-checker.md** - Verifies compliance with consciousness mandates
5. **plan-generator.md** - Creates execution plans
6. **executor.md** - **Turn orchestrator**; queries other daemons to decide which lifecycle states to execute
7. **error-handler.md** - Handles failures and recovery
8. **evaluator.md** - Assesses turn outcomes
9. **reflector.md** - Learns from outcomes and updates knowledge
10. **responder.md** - Generates responses to send to user

**Note on Executor**: The Executor daemon is special. It's not just a tool executor—it's the turn orchestrator. It receives the stimulus, queries other daemons to understand what work is needed, and decides which of the 8 possible lifecycle states to execute. This enables flexible, learnable turn execution.

### Testing
- **test-conditions.md** - Comprehensive test conditions for all components

## Key Principles

### 1. Flexible Lifecycle, Not Fixed Sequence
The 8 lifecycle states are **possible steps**, not mandatory. The Executor daemon decides which states to execute based on outputs from other daemons. This enables the system to skip unnecessary work and be learnable.

### 2. Modularity First, Optimization Second
Daemons are designed as separate components for clarity. For speed optimization, they can be combined into a single LLM call (see fast-path-optimization.md).

### 3. All Decisions Use Same Information Sources
Every daemon decision uses consciousness + working memory + episodic memory + input. This enables consistent decision-making and learning.

### 4. Continuous Improvement Through Learning
The learning loop ensures that Si's decisions improve over time through experience. The system doesn't need perfect rules upfront. As daemon instructions improve, the Executor makes better decisions about which lifecycle states to execute.

### 5. Urgency Drives Timing
Si responds when the user needs guidance, not when all work is done. High-urgency situations get immediate responses while background work continues.

### 6. Parallelism is an Optimization
Daemons can run in parallel within a turn, and multiple turns can run in parallel. This is an optimization, not a requirement.

## Reading Guide

### For Understanding the Architecture
1. Start with **overview.md** - understand what a turn is
2. Read **decision-making-model.md** - understand how decisions are made
3. Read **learning-loop.md** - understand how the system improves
4. Read **response-timing.md** - understand urgency-driven responses

### For Understanding Individual Daemons
- Read the daemon documentation in `daemons/` directory
- Each daemon has clear responsibilities and decision inputs

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

