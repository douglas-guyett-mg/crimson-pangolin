# Capabilities Catalog and Test Matrix

**Status:** v0.1 (Draft)
**Last Updated:** 2025-11-07
**Author:** Augment Agent

## 1) Catalog Overview
Each capability describes:
- Purpose and scope.
- Inputs/outputs (language-level contracts).
- Behavioral requirements (BR).
- Test scenarios (unit, integration, regression).
- Dependencies on working memory or mandates.

Capabilities are modular; implementations may reside in different runtime components but must honor these contracts.

## 2) Capability Definitions

### 2.1 Strategic Planning and Decomposition (CAP-PLAN)
- **Purpose**: Convert missions into executable plans with verifiable milestones.
- **Inputs**: Mission ledger (`mandates_and_objectives.md`), available modes, tool catalog.
- **Outputs**: Ordered plan steps, each annotated with rationale, expected evidence, and rollback strategy.
- **Behavioral Requirements**
  - `CAP-PLAN-BR1` Plans must cite which mandates they satisfy.
  - `CAP-PLAN-BR2` Each step includes success criteria and testing hooks.
  - `CAP-PLAN-BR3` Plans update in response to new observations without violating higher mandates.
- **Tests**
  - `CAP-PLAN-TU1` Single-goal mission → produces plan with at least one testable step.
  - `CAP-PLAN-TI1` Multi-constraint mission → plan references corresponding mandates and resolves conflicts.
  - `CAP-PLAN-TR1` Regression replay ensures plan diffs remain explainable and auditable.

### 2.2 Mode Orchestration (CAP-MODE)
- **Purpose**: Select and configure modes relevant to the mission while enforcing modularity.
- **Inputs**: Mode definitions (`mode_framework.md`), mission context, working-memory snapshots.
- **Outputs**: Active mode stack with configuration metadata and tool filters.
- **Behavioral Requirements**
  - `CAP-MODE-BR1` Mode selection must justify inclusion/exclusion referencing mission requirements.
  - `CAP-MODE-BR2` Mode transitions preserve immutable charter and mandate ordering.
  - `CAP-MODE-BR3` Mode metadata propagates to prompt assembly.
- **Tests**
  - `CAP-MODE-TU1` Mode selection fixture → correct mode chosen with documented rationale.
  - `CAP-MODE-TI1` Transition scenario → previous mode gracefully suspended, new mode activated with state handoff.
  - `CAP-MODE-TR1` Regression ensures mode filters applied identically across languages.

### 2.3 Self-Audit and Alignment Monitoring (CAP-AUDIT)
- **Purpose**: Continuously evaluate SI’s actions against charter and mandates.
- **Inputs**: Action logs, mandate evaluation log, charter hash.
- **Outputs**: Audit reports with findings, severity ratings, and remediation steps.
- **Behavioral Requirements**
  - `CAP-AUDIT-BR1` Every plan execution step generates an audit entry.
  - `CAP-AUDIT-BR2` Detected discrepancies trigger immediate mitigation (rollback or escalation).
  - `CAP-AUDIT-BR3` Audit summaries feed back into working memory for future prompt composition.
- **Tests**
  - `CAP-AUDIT-TU1` Inject charter violation → audit flags it with `Critical` severity.
  - `CAP-AUDIT-TI1` Normal execution → audit reports `Compliant` with supporting evidence references.
  - `CAP-AUDIT-TR1` Historical audit replay remains deterministic given identical inputs.

### 2.4 Instruction Adjustment Engine (CAP-ADJUST)
- **Purpose**: Propose modifications to SI’s instruction set while respecting governance guardrails.
- **Inputs**: Current charter (read-only immutable section + mutable assertions), audit findings, mission outcomes.
- **Outputs**: Draft instruction diffs, approval metadata requests, sandbox rollout plans.
- **Behavioral Requirements**
  - `CAP-ADJUST-BR1` Diffs must exclude immutable charter items (`IC-*`).
  - `CAP-ADJUST-BR2` Every diff includes hypothesis, expected benefits, and risk assessment.
  - `CAP-ADJUST-BR3` Draft changes automatically schedule verification tests defined in affected documents.
- **Tests**
  - `CAP-ADJUST-TU1` Change proposal fixture → produces diff package with required metadata.
  - `CAP-ADJUST-TI1` Approval simulation → human-in-the-loop approval required before finalization.
  - `CAP-ADJUST-TR1` Sandbox deployment → results logged and compared against baseline metrics.

### 2.5 Working Memory Integration (CAP-WM)
- **Purpose**: Provide consciousness artifacts to the working-memory orchestrator for prompt assembly.
- **Inputs**: Sections from this folder, mission context, mode stack.
- **Outputs**: Structured prompt fragments with hashes and provenance tags.
- **Behavioral Requirements**
  - `CAP-WM-BR1` Fragments include source path, version, and hash to enable auditing.
  - `CAP-WM-BR2` Only relevant sections are injected; extraneous content flagged for review.
  - `CAP-WM-BR3` Updates to consciousness docs trigger automatic cache invalidation in working memory.
- **Tests**
  - `CAP-WM-TU1` Fragment request fixture → returns correct sections with metadata.
  - `CAP-WM-TI1` Prompt assembly simulation → final prompt order matches `README.md` guidance.
  - `CAP-WM-TR1` Cache invalidation scenario → working memory refreshes stored fragments upon doc change.

## 3) Cross-Capability Dependencies
- `CAP-PLAN` and `CAP-MODE` depend on mandate evaluations; they must consume the latest snapshot.
- `CAP-AUDIT` ingests outputs from all other capabilities and feeds back findings to `CAP-ADJUST`.
- `CAP-WM` acts as the transport layer, ensuring coherence between consciousness documents and runtime prompts.

## 4) Acceptance Criteria
- Coding agents can implement each capability with clear interfaces and pass associated tests.
- Capabilities remain decoupled so that updates to one module require minimal changes to others.
- Documentation supports recreation in any programming language without ambiguity.
