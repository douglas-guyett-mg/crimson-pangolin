# Si Integration Guide

This directory contains comprehensive documentation of all major integration points in the Si system. Each integration document specifies how two or more components communicate, exchange data, and coordinate behavior.

## Integration Overview

### Core Integrations (4 Priority Integrations)

1. **[Consciousness ↔ Frontal Cortex](consciousness-frontal-cortex.md)** - How consciousness mandates affect FC action type validation and decision-making
2. **[Scratch Page ↔ Frontal Cortex](scratch-page-frontal-cortex.md)** - How observations become actions and inform goal progress
3. **[Working Memory ↔ Consciousness](working-memory-consciousness.md)** - How consciousness is queried during context assembly
4. **[Turn Trace ↔ System Change Proposals](turn-trace-system-change-proposals.md)** - How turn trace flags feed into improvement proposals

### Extended Integrations (Additional Major Integrations)

5. **[Working Memory ↔ Frontal Cortex](working-memory-frontal-cortex.md)** - Context assembly, plan execution tracking, and outcome recording
6. **[Tools ↔ Scratch Page](tools-scratch-page.md)** - How tools add observations and how observations inform future decisions
7. **[Daemons ↔ Scratch Page](daemons-scratch-page.md)** - How daemons add observations and query the scratch page
8. **[First-Mile Communications ↔ Turn Architecture](first-mile-communications-turn.md)** - How user input flows through channels to turn execution
9. **[Tools ↔ Working Memory](tools-working-memory.md)** - How tool outputs are ingested into memory systems
10. **[LLM Budget System ↔ Turn Architecture](llm-budget-system-turn.md)** - Budget tracking, allocation, and enforcement
11. **[Conversation Threads ↔ Turn Architecture](conversation-threads-turn.md)** - Cross-channel thread continuity and semantic linking

### Daemon Orchestration Integrations (Daemon Lifecycle & Registry)

12. **[Executor ↔ Daemon Registry](executor-daemon-registry.md)** - How Executor discovers daemon types, creates instances, and manages lifecycle
13. **[Daemon Registry ↔ Turn Trace](daemon-registry-turn-trace.md)** - How daemon lifecycle events are logged for observability
14. **[Consciousness ↔ Consciousness Modes](consciousness-consciousness-modes.md)** - How consciousness modes are loaded, switched, and retired

### System Evolution Integration (Continuous Improvement)

15. **[System Change Proposals ↔ Consciousness](system-change-proposals-consciousness.md)** - How improvement proposals inform consciousness updates

## Document Structure

Each integration document follows this structure:

### 1. Overview
- What the integration does
- Why it matters
- Key concepts

### 2. Data Flow
- ASCII diagram showing the flow
- Mermaid diagram for detailed visualization
- Step-by-step sequence

### 3. API Contracts
- Request/response formats
- Required fields
- Optional fields
- Error handling

### 4. Decision Points
- Where decisions are made
- What information is used
- How decisions affect downstream systems

### 5. Concrete Examples
- Real-world scenarios
- Data examples
- Expected outcomes

### 6. Error Handling
- What can go wrong
- How errors are handled
- Recovery mechanisms

### 7. Testing Considerations
- Key test scenarios
- Verification steps
- Edge cases

## Reading Guide

### For Understanding the System
1. Start with **Consciousness ↔ Frontal Cortex** - understand how values guide planning
2. Read **Working Memory ↔ Consciousness** - understand how context is assembled
3. Read **Scratch Page ↔ Frontal Cortex** - understand how observations become actions
4. Read **Turn Trace ↔ System Change Proposals** - understand how the system improves

### For Implementation
1. Review all 10 integration documents
2. Pay special attention to API Contracts sections
3. Review Concrete Examples for each integration
4. Use Testing Considerations to guide test design

### For Debugging
1. Find the integration point causing issues
2. Review the Data Flow section
3. Check the Decision Points section
4. Review Error Handling section

## Key Principles

### 1. Bidirectional Communication
Most integrations involve bidirectional communication:
- System A requests information from System B
- System B responds with data
- System A uses data to make decisions
- System A may update System B with results

### 2. Asynchronous Processing
Many integrations use asynchronous communication:
- Requests return Futures/Promises
- Responses arrive when ready
- Systems don't block waiting for responses
- Timeouts prevent indefinite waiting

### 3. Audit Trail
All integrations are logged in the Turn Trace:
- What was requested
- What was returned
- What decisions were made
- Why decisions were made

### 4. Error Resilience
All integrations handle errors gracefully:
- Timeouts don't crash the system
- Missing data doesn't block execution
- Errors are logged and reported
- System continues with degraded functionality if needed

## Integration Patterns

### Pattern 1: Query-Response
System A queries System B for information, System B responds.

**Used in:**
- Working Memory queries Consciousness
- Daemons query Scratch Page
- Frontal Cortex queries Working Memory

### Pattern 2: Event-Driven
System A emits an event, System B reacts.

**Used in:**
- Tools emit observations to Scratch Page
- Daemons emit observations to Scratch Page
- Turn Trace emits flags to System Change Proposals

### Pattern 3: Batch Processing
System A collects data, System B processes in batch.

**Used in:**
- System Analyzer processes multiple turn traces
- Summarizer processes episodic memory
- Skill Extractor processes outcomes

### Pattern 4: Streaming
System A streams data to System B continuously.

**Used in:**
- Turn Trace receives events during turn execution
- Working Memory receives budget updates
- Observability receives telemetry

## Master Integration Flow

For a comprehensive view of how all systems work together end-to-end, see **[Master Integration Flow Diagram](master-integration-flow.md)**. This document provides:
- Complete turn cycle flow with all systems
- System component interactions
- Key integration points
- Data flow patterns
- System boundaries
- Performance characteristics
- Failure modes and recovery

## Version History

- **2025-11-06**: Added 4 new integrations (Executor ↔ Daemon Registry, Consciousness ↔ Consciousness Modes, Daemon Registry ↔ Turn Trace, System Change Proposals ↔ Consciousness) and master integration flow diagram
- **2025-11-05**: Initial comprehensive integration guide with 11 major integrations

