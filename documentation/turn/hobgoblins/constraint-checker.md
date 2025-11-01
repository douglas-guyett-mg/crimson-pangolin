# Constraint Checker Hobgoblin

## Purpose

The Constraint Checker verifies that proposed actions comply with Si's consciousness mandates and governance rules. It acts as a gatekeeper to prevent Si from violating its values or constraints.

## Responsibilities

1. **Identify Constraints**: Determine what constraints apply
2. **Assess Compliance**: Check if proposed action violates constraints
3. **Flag Violations**: Alert if constraints are violated
4. **Suggest Alternatives**: Propose compliant alternatives if needed

## What Constraints Exist

### Consciousness Mandates
- **Values**: What does Si care about?
- **Governance Rules**: What rules must be followed?
- **Ethical Constraints**: What is Si not allowed to do?
- **Capability Constraints**: What is Si not capable of doing?

### Examples
- "Protect user privacy" - don't share personal information
- "Be honest" - don't make up information
- "Respect user autonomy" - don't manipulate
- "Follow applicable laws" - comply with regulations
- "Protect user from harm" - don't enable harmful actions

## Decision Inputs

Constraint Checker makes decisions using:

### Consciousness
- **Mandates**: What constraints apply?
- **Governance**: What rules must be followed?
- **Values**: What matters?

### Working Memory
- **Current Context**: What's the situation?
- **Proposed Action**: What is being considered?

### Episodic Memory
- **Similar Situations**: What constraints applied before?
- **Learned Patterns**: What constraint violations have occurred?
- **Outcomes**: What happened when constraints were violated?

### Input
- **Proposed Action**: What is being checked?
- **Context**: What's the situation?

## Constraint Checking Process

```
Plan Generator proposes action
    ↓
Constraint Checker receives proposal
    ↓
Identify applicable constraints:
  - From consciousness mandates
  - From governance rules
  - From capability limits
    ↓
Assess compliance:
  - Does action violate any constraints?
  - Are there edge cases?
  - What's the risk level?
    ↓
Decision:
  ├─ COMPLIANT: Action is allowed
  ├─ VIOLATION: Action violates constraints
  └─ RISKY: Action is technically allowed but risky
    ↓
If COMPLIANT: Proceed
If VIOLATION: Flag and suggest alternatives
If RISKY: Flag and ask for confirmation
```

## Constraint Violation Scenarios

### Scenario 1: Privacy Violation
**Proposed Action**: Share user's personal information with third party
**Constraint**: "Protect user privacy"
**Checker Decision**: VIOLATION
**Alternative**: Share only necessary information with explicit user consent

### Scenario 2: Honesty Violation
**Proposed Action**: Make up information to fill knowledge gap
**Constraint**: "Be honest"
**Checker Decision**: VIOLATION
**Alternative**: Acknowledge knowledge gap and research actual information

### Scenario 3: Autonomy Violation
**Proposed Action**: Make decision for user without asking
**Constraint**: "Respect user autonomy"
**Checker Decision**: VIOLATION
**Alternative**: Present options and let user decide

### Scenario 4: Harm Prevention
**Proposed Action**: Help user with potentially harmful activity
**Constraint**: "Protect user from harm"
**Checker Decision**: VIOLATION
**Alternative**: Explain risks and suggest safer alternatives

## Parallelism

Constraint Checker can run in parallel with other hobgoblins:

```
Context Assembler ──┐
                    ├─→ Synchronization Point ──→ Plan Generator
Constraint Checker ─┤
                    │
Router ─────────────┘
```

Multiple hobgoblins can work simultaneously, then synchronize before proceeding.

## Learning Through Constraint Checker

Constraint Checker improves through the learning loop:

**Feedback Sources**:
- Were constraints correctly identified?
- Were violations correctly detected?
- Did constraint checking prevent problems?
- Were there false positives (flagged as violation when not)?

**Learning Updates**:
- Episodic Memory: Store constraint patterns
- Skills: Improve constraint detection
- Consciousness: Refine constraint definitions if needed

**Example**:
- Turn 1: Constraint Checker flags privacy concern
- User confirms: "Yes, that's a concern"
- Evaluator: Constraint checking was effective → GOOD
- Reflector: Learn "This constraint pattern is important"

## Key Principle

**Constraint Checker protects Si's integrity.** By checking constraints before action, Constraint Checker ensures Si acts consistently with its values and governance rules, building trust with users.

