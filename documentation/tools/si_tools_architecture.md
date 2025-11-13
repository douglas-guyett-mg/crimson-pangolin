# Si Tooling Architecture — Technical Plan (Platform-Independent)

**Status:** v0.1 (Draft)
**Last Updated:** 2025-11-07
**Author:** Augment Agent

## 1) Purpose
Define a modular, technology-agnostic tooling framework that allows Si to discover, invoke, and evaluate self-contained tools. The plan must be sufficiently detailed to regenerate an implementation in any language, enabling tools to be added or removed by managing their directories alone.

## 2) Scope and Non-Goals
- In scope: directory conventions, manifests, invocation contracts, output handling, lifecycle hooks, observability, and testing requirements.
- Non-goals: choosing a concrete runtime, task scheduler, or packaging format; prescribing specific programming languages or vendor products.

## 3) Alignment with Si Core Engineering Tenants
- **Documentation as code:** this plan, accompanying schemas, and test directives are the authoritative specification for the tooling system.
- **Tests first:** every interface and policy below includes explicit test requirements and sample scenarios.
- **Modular first:** tools are plug-and-play units with zero shared mutable state; discovery and invocation happen through declarative manifests.
- **Technology agnostic:** manifests, contracts, and policies avoid language- or vendor-specific assumptions.

## 4) Tooling Principles
1. **Self-contained:** each tool’s code, manifest, tests, and docs live under `tools/{tool_name}/`.
2. **Declarative discovery:** Si reads manifests to understand capabilities, inputs, outputs, and policies without parsing executable code.
3. **Stable contracts:** requests and responses conform to shared schemas, enabling consistent working-memory ingestion.
4. **Stateless core, pluggable adapters:** tools declare required resources; runtimes provide adapters (HTTP, CLI, SDK) via configuration.
5. **Deterministic metadata:** versioned manifests, semantic versioning, and checksum tracking prevent drift between documentation and implementation.
6. **Observability by default:** every invocation emits structured telemetry and optional memory write suggestions.

## 5) Repository Layout (Docs as Code View)
```
documentation/
  tools/
    si_tools_architecture.md        # This document (authoritative spec)
    tool_manifest_schema.yaml       # Declarative schema (docs-as-code)
    tool_io_contract.yaml           # Request/response schema
    test_matrix.md                  # Required scenarios for validation (future work)
tools/
  {tool_name}/
    tool.yaml                       # Manifest (must validate against schema)
    README.md                       # Tool-specific instructions/tests
    src/                            # Implementation (language-specific)
    tests/                          # Unit + contract tests
    fixtures/                       # Optional sample inputs/outputs
```

## 6) Tool Manifest Specification
- Implemented in `documentation/tools/tool_manifest_schema.yaml`.
- Key fields:
  - `id`: stable, globally unique slug (`kebab-case`).
  - `version`: semantic version (`MAJOR.MINOR.PATCH`).
  - `display_name` & `description`: human-readable summary.
  - `entrypoint`: declarative invocation (e.g., `{type: cli, command: ...}`).
  - `capabilities`: list of domain verbs or categories.
  - `inputs` / `outputs`: JSON Schema references or inline OpenAPI fragments.
  - `side_effects`: declare external mutations (filesystem, network).
  - `resources`: required credentials, rate limits, or sandbox rules.
  - `policies`: guardrails (confirmation needed, max runtime, retry policy).
  - `observability`: telemetry expectations (metrics, logs, traces).

All manifests must pass validation and include a changelog entry in `README.md` when `version` increments.

## 7) Tool Invocation Contract
- Canonical schema defined in `documentation/tools/tool_io_contract.yaml`.
- `ToolRequest` (Si → Tool):
  - `request_id`, `tool_id`, `tool_version`, `timestamp`.
  - `agent_context`: task id, plan id, caller metadata.
  - `inputs`: structured payload matching manifest’s `inputs.schema`.
  - `invocation_policies`: resolved runtime limits (timeout, retries).
  - `memory_snippet`: optional distilled context shared with the tool.
- `ToolResponse` (Tool → Si):
  - `request_id` (echo), `status` (`ok|error|partial`).
  - `outputs`: structured data complying with manifest.
  - `observations`: freeform text + tagged metadata (for WM ingestion).
  - `memory_writes`: array of normalized `MemoryItem` drafts (type `tool_result`).
  - `telemetry`: runtime metrics (latency, tokens, cost).
  - `error`: structured fault info (code, message, retryable) when `status != ok`.

Responses must be serializable to JSON; runtime adapters may convert native objects.

## 8) Working Memory Integration
- `memory_writes` must align with `MemoryItem` schema (see `working-memory.md`).
- Minimal fields: `type=tool_result`, `text` summarizing outcome, `metadata` with `tool_id`, `tool_version`, `raw_outputs_location`.
- Tool adapters enforce token budgets, perform redaction (with tooling-provided hints), and optionally synthesize summaries if outputs exceed limits.
- Validation tests: golden scenarios verifying that `memory_writes` are usable for downstream planning (QA tasks, follow-ups).

## 9) Tool Lifecycle
1. **Discovery:** Si scans `tools/**/tool.yaml`, validates against schema, and builds runtime catalog.
2. **Capability negotiation:** catalog entries expose verbs/tags used by the planner to match tasks → tools.
3. **Invocation:** planner issues `ToolRequest`; adapter executes entrypoint with resolved inputs/policies.
4. **Post-processing:** adapter validates `ToolResponse`, normalizes memory entries, emits telemetry.
5. **Removal:** deleting a tool directory removes it from the catalog after the next scan; stale references log warnings.

## 10) Observability and Governance
- Every invocation logs:
  - `request_id`, `tool_id`, `duration_ms`, `status`.
  - `policy_decisions`: escalations, confirmations, overrides.
  - `resource_usage`: tokens, cost, API calls (if known).
- Metrics feed the broader observability layer defined in `working_memory_system.md`.
- Access control: manifests declare required scopes; runtime ensures credentials isolation per tool.

## 11) Tool Invocation Ordering and Data Flow

See `documentation/tools/tool-invocation-ordering.md` for comprehensive specification covering:
- **Static Planning**: FC specifies execution order and data flow in natural language
- **Dynamic Execution**: Executor runs tools and adapts based on actual outputs
- **Data Flow**: How tool outputs become inputs to downstream tools
- **Adaptive Decision-Making**: When constraints are violated, Executor loops back to FC
- **Sub-Turn Spawning**: Breaking tasks into parallel sub-turns for efficiency
- **Turn Trace Recording**: All decisions and adaptations logged for learning

See `documentation/tools/tool-invocation-ordering-test-conditions.md` for comprehensive test scenarios.

## 12) Error Handling

See `documentation/tools/error-handling.md` for comprehensive error handling specification covering:
- **Error Classification**: Transient vs. permanent errors
- **Retry Policies**: Exponential backoff with tool-specific overrides
- **Circuit Breaker Pattern**: Prevent cascading failures
- **Timeout Semantics**: Per-tool (hard stop) and per-turn (graceful degradation)
- **Cascading Failures**: Dependency-based failure propagation
- **Integration**: Error Handler daemon decision-making

See `documentation/tools/error-handling-test-conditions.md` for comprehensive test scenarios.

## 13) Testing Requirements
- **Manifest validation:** schema compliance, required fields, semantic version progression.
- **Contract tests:** request/response round-trips using fixtures; enforce deterministic outputs for golden cases.
- **Failure handling:** simulate timeouts, partial successes, redaction failures; verify structured `error` payloads.
- **Observability hooks:** ensure telemetry is emitted even on failure paths.
- **Compatibility tests:** removing a tool does not break catalog initialization.

Tests must be runnable via language-agnostic commands (e.g., `make contract-test`), with instructions documented per tool.

## 14) Best Practices & Recommendations
- Prefer pure functions; isolate side effects behind adapters declared in `resources`.
- Keep manifests narrowly scoped; compose complex behaviors via orchestrated plans rather than monolithic tools.
- Provide sample prompts showing how Si should use the tool, aiding prompt-engineering for the planner.
- Maintain `CHANGELOG.md` per tool detailing manifest and behavior updates.
- Leverage linting/formatters appropriate to the implementation language but keep docs and manifests purely declarative.

## 15) Open Questions
1. Do we require signed manifests or checksums for supply-chain guarantees?
2. Should tool removal trigger automated deprecation workflows in working memory?
3. What is the default cadence for catalog rescans in long-running deployments?
4. How are secrets injected (env vars, vault adapters) while keeping manifests technology-agnostic?
5. Are there regulatory constraints dictating where tool outputs may persist?

Stakeholder decisions on these items are needed before moving from draft to implementation.
