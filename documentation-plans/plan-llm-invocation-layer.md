# Plan: LLM Invocation Layer Documentation

## Purpose
Document a modular, technology-agnostic LLM invocation subsystem that lives within Si, exposes a shared contract across providers, supports streaming responses, and enforces user/system token governance per the core engineering tenants.

## Required References
- `documentation/si_core_engineering_tennants.md`
- `documentation/tools/si_tools_architecture.md`
- `working_memory_system.md`
- Any provider policy notes or token budget guidance surfaced during research (capture within the new documentation set).

## Deliverables
Create a new documentation package under `documentation/llm-invocation-layer/` with, at minimum:
- `overview.md`
- `test-conditions.md`
- Additional subdirectories/files if needed (e.g., `provider-adapters/overview.md`) while keeping to the mandated structure.

## Checklist
- [ ] Define subsystem scope, responsibilities, and interaction points with existing Si components (working memory, first-mile communications, observability).
- [ ] Specify canonical request/response contracts, message normalization rules, and streaming semantics.
- [ ] Describe provider adapter responsibilities for OpenAI, Claude, Gemini (extensible to others) including prompt translation, safety policies, and error handling.
- [ ] Document the routing policy engine, including current random selection model, hooks for future intelligent routing, and configuration surfaces.
- [ ] Detail token accounting flows for user-scoped budgets (per plan configurable limits with soft warnings and hard enforcement) and system-token tracking.
- [ ] Capture observability, auditing, and governance requirements (usage telemetry, cost tracking, residency constraints, alerting).
- [ ] Enumerate failure handling, retries, fallback strategy, and degraded-mode behaviors.
- [ ] Define explicit, testable acceptance and verification criteria covering adapter compliance, routing determinism, token budget enforcement, and streaming delivery.
- [ ] Review existing documentation for consistency and flag any conflicts that require remediation before publishing the final docs.

## Notes & Open Items
- Incorporate streaming support requirements (token-by-token delivery, partial tool-call events) in both architecture and testing sections.
- Ensure token governance distinguishes between user-attributed spend and system-level actions; include audit trail expectations even though system tokens are not budget-limited.
