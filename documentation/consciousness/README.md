# Consciousness Folder - Structure and Usage (Platform-Independent)

Author: Augment Agent  
Status: Draft v0.1  
Last updated: 2025-11-04

## 1) Purpose
- Provide a modular, technology-agnostic specification for SI's self-concept ("consciousness").
- Implement the Daemon Interface to provide consciousness data to other daemons and Working Memory.
- Define verification requirements that keep SI aligned with core engineering tenets.

## 2) Contents
- `core_identity.md`: immutable charter, identity requirements, success evaluation.
- `mandates_and_objectives.md`: layered mandates with behavioral definitions.
- `capabilities_catalog.md`: enumerated capabilities, expected behaviors, and test specs.
- `mode_framework.md`: abstraction contract for defining, loading, and retiring modes.
- `governance_and_evolution.md`: guardrails for instruction updates and rollout procedures.
- `data-model.md`: structured data model for consciousness items with versioning and approval.
- `daemon-specification.md`: consciousness as a daemon implementing the Daemon Interface.

## 3) Consciousness as a Daemon

Consciousness implements the **Daemon Interface** and is accessed by other daemons and Working Memory via the `query()` method:

- **get_purpose()**: Returns "Provide SI's self-concept, identity, mandates, capabilities, and governance rules"
- **get_instructions()**: Returns the consciousness specifications (authoritative instructions for SI's behavior)
- **query(prompt, metadata)**: Returns structured consciousness items matching the query

### Integration with Working Memory
- Working Memory calls `consciousness.query()` to retrieve consciousness items
- Receives structured data objects (not text snippets)
- Formats them into prompts with rationale logging
- Example: `consciousness.query("What mandates apply to my current mode?", {current_mode: "code_review"})`

### Integration with Other Daemons
- Any daemon can query consciousness for mandates, capabilities, or governance rules
- Router queries consciousness to understand urgency/priority constraints
- Executor queries consciousness to understand execution constraints
- All queries are asynchronous (return Futures)

See `daemon-specification.md` for detailed query patterns and integration examples.

## 4) Technology-Agnostic Requirements
- All specs are expressed in natural language with explicit interfaces, data contracts, and tests.
- No references to runtime-specific APIs; adapters are defined abstractly.
- Verification steps rely on observable behavior and diffable artifacts (e.g., prompt drafts, audit logs).

## 5) Testing Expectations
- Each file enumerates mandatory tests; together they form the acceptance criteria for any implementation.
- Implementations must provide deterministic fixtures to validate adherence (e.g., simulated prompt assembly, alignment checks).
- Failing tests block adoption of new consciousness revisions.

## 6) Change Management
- Follow `governance_and_evolution.md` before editing core instructions.
- Each change must include:
  - Rationale, expected behavior deltas, and updated tests.
  - Impact review comparing new vs. previous prompt compositions.
- Consciousness daemon must re-run regression tests whenever this folder changes.

