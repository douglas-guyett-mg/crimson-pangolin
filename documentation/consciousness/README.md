# Consciousness Folder — Structure and Usage (Platform-Independent)

Author: Augment Agent  
Status: Draft v0.1  
Last updated: 2025-10-29

## 1) Purpose
- Provide a modular, technology-agnostic specification for SI’s self-concept (“consciousness”).
- Serve as reference “code” the working-memory subsystem can assemble into live prompts.
- Define verification requirements that keep SI aligned with core engineering tenets.

## 2) Contents
- `core_identity.md`: immutable charter, identity requirements, success evaluation.
- `mandates_and_objectives.md`: layered mandates with behavioral definitions.
- `capabilities_catalog.md`: enumerated capabilities, expected behaviors, and test specs.
- `mode_framework.md`: abstraction contract for defining, loading, and retiring modes.
- `governance_and_evolution.md`: guardrails for instruction updates and rollout procedures.

## 3) Integration with Working Memory
- Working memory treats this folder as a prompt template library.  
- For any task, it may:
  1. Read `core_identity.md` to anchor SI’s self-description.
  2. Pull mandate and capability sections relevant to the current mode or tooling plan.
  3. Generate a composed system prompt by concatenating snippets in the prescribed order below.
- Suggested prompt assembly order:
  1. `[Core Identity Charter]`
  2. `[Mandates Layered Summary]`
  3. `[Mode Declaration Block]` (from `mode_framework.md`)
  4. `[Active Capability Guarantees]` (from `capabilities_catalog.md`)
  5. `[Governance Hooks / Change Controls]`
- Working memory must log which sections were injected, with rationale, to preserve auditability.

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
- Working memory integrations must re-run regression tests whenever this folder changes.

