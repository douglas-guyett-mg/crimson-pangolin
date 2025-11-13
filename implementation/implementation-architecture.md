---
# Si Implementation Architecture

## 1. Scope and Goals
- In scope:
  - Translate existing documentation under documentation/ into a pragmatic, modular implementation plan that can be executed in 20–30 minute modules by a single agent
  - Define clear module boundaries, dependencies, and a safe sequencing order
  - Align with core engineering tenants: documentation-as-code, tests-first, modularity, technology-agnosticism
  - Establish Implementation Tree under implementation/ (Stage 2 will generate per-module specs; Stage 3 will implement code under si/)
- Out of scope:
  - Writing production code (no code in Stage 1)
  - Infrastructure provisioning, deployments, or vendor-specific wiring (e.g., AWS/Vercel/Supabase specifics)
  - UI/Frontend work
  - Long-term data modeling decisions beyond what’s needed to define module boundaries
- Constraints:
  - File-creation discipline: Stage 1 creates this file only
  - Tests-first: Each module in Stage 2 must include test-conditions.md before implementation
  - Code isolation: All implementation code will live under si/{module-name}/
  - Terminology and interface norms:
    - Use “daemon” for autonomous components; each must be introspectable (get_purpose(), get_instructions())
    - Use “Interface” (not “Plugin”); memory daemons implement a unified interface (query, store)
  - Working Memory (WM):
    - Single unified Summarization Engine used for budget-fitting reductions (not separate sampling/redaction engines)
    - Pluggable memory types per arxiv.org/pdf/2309.02427 with add/subtract semantics
  - Consciousness integration:
    - Consciousness objects provide .to_text() and return an id for Turn Trace logging
  - Turn architecture:
    - Router first; Executor only spins up if needed
    - Executor is LLM-driven; error handling is an Executor capability (not a separate daemon) with graded actions
    - Evaluator is a separate daemon used system-wide
    - Context Assembler may be WM’s assemble_context() (wrapper or alias)
  - Integration norms:
    - Registry/data-driven action/tool types (avoid hard-coded types)
    - Ignore deployment/ and archive/ directories

## 2. Proposed Implementation Tree (under `implementation/`)
```
implementation/
  daemon-registry-core/
    overview.md
    implementation-checklist.md
    test-conditions.md
  turn-trace-core/
    overview.md
    implementation-checklist.md
    test-conditions.md
  wm-memory-type-registry/
    overview.md
    implementation-checklist.md
    test-conditions.md
  wm-summarization-engine/
    overview.md
    implementation-checklist.md
    test-conditions.md
  working-memory-orchestrator/
    overview.md
    implementation-checklist.md
    test-conditions.md
  llm-budget-core/
    overview.md
    implementation-checklist.md
    test-conditions.md
  llm-per-user-cost-tracking/
    overview.md
    implementation-checklist.md
    test-conditions.md
  llm-daemon-prompt-simplification/
    overview.md
    implementation-checklist.md
    test-conditions.md
  tools-core/
    overview.md
    implementation-checklist.md
    test-conditions.md
  tool-invocation-ordering/
    overview.md
    implementation-checklist.md
    test-conditions.md
  tool-memory-access/
    overview.md
    implementation-checklist.md
    test-conditions.md
  scratch-page-core/
    overview.md
    implementation-checklist.md
    test-conditions.md
  conversation-threads-core/
    overview.md
    implementation-checklist.md
    test-conditions.md
  consciousness-integration/
    overview.md
    implementation-checklist.md
    test-conditions.md
  frontal-cortex-integration/
    overview.md
    implementation-checklist.md
    test-conditions.md
  turn-router/
    overview.md
    implementation-checklist.md
    test-conditions.md
  plan-generator/
    overview.md
    implementation-checklist.md
    test-conditions.md
  executor-daemon/
    overview.md
    implementation-checklist.md
    test-conditions.md
  executor-error-handling/
    overview.md
    implementation-checklist.md
    test-conditions.md
  evaluator-daemon/
    overview.md
    implementation-checklist.md
    test-conditions.md
  monitor-daemon/
    overview.md
    implementation-checklist.md
    test-conditions.md
  responder-daemon/
    overview.md
    implementation-checklist.md
    test-conditions.md
  system-change-proposals-core/
    overview.md
    implementation-checklist.md
    test-conditions.md
  change-flagging-daemon/
    overview.md
    implementation-checklist.md
    test-conditions.md
  system-analyzer-daemon/
    overview.md
    implementation-checklist.md
    test-conditions.md
```

## 3. Module Catalog
- name (slug): daemon-registry-core
  - description: Central registry for daemons and capabilities; ensures introspection (get_purpose, get_instructions) and data/registry-driven action types
  - dependencies: none
  - estimated effort (20-30 minutes): 25

- name (slug): turn-trace-core
  - description: Uniform turn trace schema and write API; ensures all daemons (and Consciousness) log with id references
  - dependencies: none
  - estimated effort: 25

- name (slug): wm-memory-type-registry
  - description: Registry and interface for pluggable memory types (add/subtract, query/store) aligned with 2309.02427
  - dependencies: turn-trace-core (for audit) [soft]
  - estimated effort: 25

- name (slug): wm-summarization-engine
  - description: Single summarization engine used by WM to reduce items to fit budget constraints
  - dependencies: wm-memory-type-registry
  - estimated effort: 25

- name (slug): working-memory-orchestrator
  - description: Orchestrates retrieval/assembly via assemble_context(); uses summarization to fit budget; parallelizes per design
  - dependencies: wm-summarization-engine, wm-memory-type-registry, turn-trace-core
  - estimated effort: 30

- name (slug): llm-budget-core
  - description: Core policy and thresholds for budget (token/cost constraints) and fast/slow-path selection
  - dependencies: turn-trace-core [soft]
  - estimated effort: 20

- name (slug): llm-per-user-cost-tracking
  - description: Per-user cost accounting and limits with simple reporting hooks
  - dependencies: llm-budget-core
  - estimated effort: 25

- name (slug): llm-daemon-prompt-simplification
  - description: Prompt simplification passes for daemons to reduce cost when over budget
  - dependencies: llm-budget-core
  - estimated effort: 20

- name (slug): tools-core
  - description: Tool manifest and IO contract; base interfaces and validation
  - dependencies: daemon-registry-core
  - estimated effort: 25

- name (slug): tool-invocation-ordering
  - description: Policies and helpers for tool invocation ordering and composition
  - dependencies: tools-core
  - estimated effort: 20

- name (slug): tool-memory-access
  - description: Controlled access layer for tools to read/write via WM as per policy
  - dependencies: tools-core, working-memory-orchestrator
  - estimated effort: 25

- name (slug): scratch-page-core
  - description: Scratch page model and API for shared pending items/notes across daemons
  - dependencies: turn-trace-core [soft]
  - estimated effort: 20

- name (slug): conversation-threads-core
  - description: Thread grouping across channels; heuristics for continuity and cross-thread context pulls
  - dependencies: scratch-page-core [soft]
  - estimated effort: 25

- name (slug): consciousness-integration
  - description: Access layer for Consciousness objects (to_text, id); fetch guidance per daemon; turn-trace id linking
  - dependencies: working-memory-orchestrator, turn-trace-core
  - estimated effort: 25

- name (slug): frontal-cortex-integration
  - description: FC interaction surface; action type registry (data-driven), persistence hooks
  - dependencies: daemon-registry-core, turn-trace-core
  - estimated effort: 25

- name (slug): turn-router
  - description: Initial decision point; assess urgency, decide fast-path vs plan/execute
  - dependencies: working-memory-orchestrator, llm-budget-core, conversation-threads-core
  - estimated effort: 25

- name (slug): plan-generator
  - description: Generates minimal plan when Router chooses plan/execute path
  - dependencies: working-memory-orchestrator, frontal-cortex-integration
  - estimated effort: 25

- name (slug): executor-daemon
  - description: LLM-driven agent that decides actions (tools/FC/response) using full context (WM, FC, Consciousness, tools, constraints)
  - dependencies: tools-core, working-memory-orchestrator, consciousness-integration, llm-budget-core, frontal-cortex-integration, turn-trace-core
  - estimated effort: 30

- name (slug): executor-error-handling
  - description: Executor capability implementing graded error handling (log, notify, escalate, stop)
  - dependencies: executor-daemon
  - estimated effort: 20

- name (slug): evaluator-daemon
  - description: Independent evaluator used by Router/Executor/Reflector to assess completion/next steps
  - dependencies: working-memory-orchestrator, turn-trace-core
  - estimated effort: 25

- name (slug): monitor-daemon
  - description: Async monitor that watches execution and raises concerns to Executor for redirection
  - dependencies: executor-daemon, turn-trace-core
  - estimated effort: 20

- name (slug): responder-daemon
  - description: Generates user-facing messages from Router or Executor outputs
  - dependencies: turn-router, executor-daemon
  - estimated effort: 20

- name (slug): system-change-proposals-core
  - description: Core data model and process for proposals; storage mechanism abstracted for now
  - dependencies: turn-trace-core
  - estimated effort: 25

- name (slug): change-flagging-daemon
  - description: Flags potential system changes during turns with evidence
  - dependencies: system-change-proposals-core, turn-trace-core
  - estimated effort: 20

- name (slug): system-analyzer-daemon
  - description: Processes flags to generate proposals for human review
  - dependencies: system-change-proposals-core
  - estimated effort: 25

## 4. Dependency Graph and Sequencing
- Textual DAG summary:
  - Base: daemon-registry-core, turn-trace-core
  - WM foundation: wm-memory-type-registry -> wm-summarization-engine -> working-memory-orchestrator
  - LLM budget: llm-budget-core -> {llm-per-user-cost-tracking, llm-daemon-prompt-simplification}
  - Tools: tools-core -> tool-invocation-ordering -> tool-memory-access
  - Integrations: consciousness-integration (uses WM, logs ids), frontal-cortex-integration (action types, persistence hooks)
  - Turn pipeline: turn-router -> plan-generator -> executor-daemon (+ executor-error-handling) -> evaluator-daemon; monitor-daemon watches executor; responder-daemon generates messages
  - Cross-cutting: scratch-page-core and conversation-threads-core feed Router/WM; system-change-proposals stack depends on turn-trace-core

- Sequenced list with rationale:
  1) daemon-registry-core — foundation for daemon discovery and introspection
  2) turn-trace-core — enable consistent logging and IDs across modules early
  3) wm-memory-type-registry — define WM type interfaces
  4) wm-summarization-engine — core reduction mechanism for budgets
  5) working-memory-orchestrator — assemble_context and parallelization
  6) llm-budget-core — centralize policy for fast/slow-path
  7) llm-per-user-cost-tracking — reporting and enforcement hooks
  8) llm-daemon-prompt-simplification — budget-aware prompt simplification
  9) tools-core — standardize tool manifests and contracts
  10) tool-invocation-ordering — deterministic and policy-aware tool sequencing
  11) tool-memory-access — safe WM access for tools
  12) scratch-page-core — shared pending items for cross-daemon context
  13) conversation-threads-core — cross-channel grouping and context pulls
  14) consciousness-integration — fetch to_text() and id with WM
  15) frontal-cortex-integration — action type registry, hooks
  16) turn-router — entry decision (fast-path vs plan/execute)
  17) plan-generator — minimal executable plan when needed
  18) executor-daemon — LLM-driven execution loop
  19) executor-error-handling — graded actions inside Executor
  20) evaluator-daemon — independent evaluation for completion/next steps
  21) monitor-daemon — async oversight for redirections
  22) responder-daemon — user-facing message generation
  23) system-change-proposals-core — data model/process
  24) change-flagging-daemon — flag collection during turns
  25) system-analyzer-daemon — proposal generation

## 5. Open Questions and Assumptions
- Q1: Data store choices for scratch page, turn trace, cost tracking, and proposals — file-based vs DB (e.g., Supabase/Neo4j/Pinecone) for the first pass?
- Q2: Exact summarization strategies and parameters (length budgets, prioritization heuristics) for WM Summarization Engine?
- Q3: Minimal initial set of memory types to support in WM (episodic, semantic, conversation, skills) for the first implementation wave?
- Q4: Conversation thread continuity heuristics across channels and inactivity boundaries — thresholds and semantic similarity approach?
- Q5: LLM budget thresholds and fast/slow-path cutoff rules per daemon (defaults vs per-user overrides)?
- Q6: Tool manifest schema constraints vs flexibility for early tool integrations; how to version manifests?
- Q7: Frontal Cortex API surface at this stage — required operations and action type registry fields for v0?
- Q8: Turn Trace schema fields beyond consciousness id linking — what minimal fields are mandatory system-wide?
- Assumptions:
  - All tests will be defined per module in Stage 2 before code is written (tests-first)
  - WM Context Assembler is the same as WM’s assemble_context() (or a thin wrapper)
  - Error handling lives inside Executor as a capability (not a standalone daemon)
  - New action types/tools are registered via data/registry (no hard-coded lists)
  - For the first wave, persistence can be abstracted behind simple adapters to avoid committing to vendors

## 6. Acceptance Gates
- Human approval required before Stage 2
- Readiness checklist:
  - [ ] Every module above has clear purpose, dependencies, and 20–30 minute scope
  - [ ] Sequence validated against dependencies and risk
  - [ ] Terminology and interface norms adopted (daemon, Interface, registry-driven)
  - [ ] WM constraints (single summarization engine; pluggable types) reflected in scope
---

