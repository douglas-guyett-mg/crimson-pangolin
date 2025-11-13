---
type: "always_apply"
---

# Agent-Driven Modular Implementation Workflow for Si

## Purpose
Codify a consistent, test-first, technology-agnostic workflow that turns Si documentation into working implementation via small, agent-executable modules.

## Core Principles
- Documentation-as-code: plans and specs live in-repo and are the source of truth
- Tests-first: each module defines explicit test conditions before code
- Modularity: scope modules so a single agent can complete them in ~20-30 minutes
- Technology-agnostic: rules focus on outcomes, not vendors
- Safety: follow package manager, file-creation, and execution/verification rules in `.augment/rules/`

## When to Use
Apply this workflow whenever planning or executing implementation for Si or a Si subsystem. Do not skip stages. Use the provided prompts (see Prompts Directory) to drive each stage.

## Directory Conventions
- Architecture plan (Stage 1 output): `implementation/implementation-architecture.md`
- Module workspaces (Stage 2+): `implementation/{module-name}/`
  - `overview.md`
  - `implementation-checklist.md`
  - `test-conditions.md`
- Implementation code (per module): `si/{module-name}/`
- Change proposals (Stage 4): `implementation/proposals/{YYYY-MM-DD}-{module-slug}-architecture-update.md`
- Prompts used by agents: `prompts/`


## Code Isolation and Directory Structure
- All implementation code for the Si project must be created under `si/` at the repository root.
- This keeps implementation code separate from documentation (`documentation/`), plans (`agent-plans/`), and specifications (`implementation/`).
- The directory structure under `si/` should follow the Implementation Tree defined in Stage 1.
- Example: if the architecture defines a module called `working-memory-core`, the code goes in `si/working-memory-core/`, while specifications remain in `implementation/working-memory-core/`.
- The `implementation/{module-name}/` directories contain only the three specification files (`overview.md`, `implementation-checklist.md`, `test-conditions.md`), never actual code.

Respect File Creation Discipline: only create files when the task or these rules explicitly require it.

## Stage 1 — Architecture Planning
Goal: Translate existing documentation into a pragmatic, modular implementation architecture.

Inputs
- Entire `documentation/` directory
- Known constraints, tenants, and rules under `.augment/rules/`

Required Outputs (single file)
- `implementation/implementation-architecture.md` containing:
  1. Scope and goals (what the implementation will deliver, out of scope)
  2. Proposed Implementation Tree (directories and key files under `implementation/`)
  3. Module Catalog: for each module
     - name (slug)
     - description
     - dependencies
     - estimated effort (20-30 minutes)
  4. Dependency Graph summary and Sequencing (ordered list with reasons)
  5. Open Questions and Assumptions
  6. Acceptance Gates (what approval is needed to proceed to Stage 2)

Process
- Analyze `documentation/` for components, interfaces, and invariants
- Identify minimal viable module boundaries (agent-completable units)
- Map dependencies and derive a safe execution sequence
- Validate module scope (< 20-30 minutes). If larger, split and document rationale
- Write the plan (Required Outputs) and request human approval before Stage 2
- Detailed module specifications will be generated in Stage 2

Do/Don’t
- Do not implement code in Stage 1
- Do use the prompt `prompts/create-architecture.md`

## Stage 2 — Per‑Module Implementation Setup
Goal: Prepare a module with complete, execution-ready specifications so an agent can execute it unambiguously.

Inputs
- Approved `implementation/implementation-architecture.md`
- Selected `{module-name}` from the Module Catalog

Required Outputs (directory + 3 files)
- `implementation/{module-name}/overview.md` — High-level description including:
  - What and why; user stories; explicit success criteria
  - Dependencies on other modules; integration points (APIs/files/events)
  - Assumptions, constraints, and non-goals
- `implementation/{module-name}/implementation-checklist.md` — Granular, box-by-box steps:
  - Every file to create/modify with exact paths
  - Every function/class/component signature to implement
  - Every configuration change (where, exact key paths)
  - Verification steps embedded after relevant items
  - Checkboxes `[ ]` for all items; no ambiguous verbs
- `implementation/{module-name}/test-conditions.md` — Tests-first spec:
  - Unit tests per component/function + input/output examples
  - Integration scenarios and environment considerations
  - Acceptance criteria mapping to user stories
  - Edge cases, error handling, and failure modes

Process
- Analyze existing codebase patterns and review relevant documentation before generating specifications
- Confirm scope fits 20-30 minutes. If not, split the module and update the Architecture Plan (Stage 1) before proceeding
- Where uncertainty exists, add a short “Questions” section and pause for clarification
- Use the prompt `prompts/setup-module.md`

## Stage 3 — Implementation Execution
Goal: Implement according to the checklist and verify via tests.

Inputs
- The module’s `overview.md`, `implementation-checklist.md`, `test-conditions.md`

Required Outcomes
- All checklist items completed and checked off
- All tests in `test-conditions.md` implemented and passing
- No scope creep; changes beyond scope are proposed, not implemented

Process & Rules
- Follow the checklist strictly; keep it up to date as you work
- Write code and tests incrementally; run tests frequently
- Create new test files as needed to satisfy `test-conditions.md` (tests-first)
- Place all implementation code under `si/{module-name}/` per Code Isolation and Directory Structure
- Safe-by-default runs are allowed (tests/linters/builds); deployments or package installs require explicit permission
- Always use the appropriate package manager when dependency changes are necessary
- Do not commit/push/merge without explicit permission
- Use the prompt `prompts/implement-module.md`

## Stage 4 — Review and Edit Plan
Goal: Ensure quality, capture learnings, and propose architecture updates.

Checklist
1. Review implementation against the module checklist — all boxes must be checked
2. Run all tests — all must pass cleanly; summarize results
3. Document architectural learnings (discoveries, constraints, pitfalls)
4. Propose architecture updates as a short, focused proposal file:
   - Path: `implementation/proposals/{YYYY-MM-DD}-{module-slug}-architecture-update.md`
   - Include: problem/evidence, proposed change, impact on modules/sequence, alternatives, test implications
5. Human approval gate — wait for explicit approval/rejection before starting the next module
6. If approved, update `implementation/implementation-architecture.md` accordingly (version the changes in-place)

## Module Scoping Guidelines
- Target duration: 20-30 minutes of focused agent work
- Max outputs: a handful of files and < ~300 LOC of production code (rule of thumb)
- Clear external interfaces; avoid cross-cutting changes unless the module is “infrastructure” by design
- Each module must be independently testable per its `test-conditions.md`
- If scope grows: stop, propose a split, and update Stage 1 plan

## Review and Governance
- This workflow complements and does not override:
  - `.augment/rules/file-creation-discipline.md`
  - `.augment/rules/documentation.md`
  - Execution and validation guidance in the agent runtime
- Human approval is required at the end of Stage 1 and Stage 4

## Prompts Directory
The following prompts drive each stage and MUST be used:
- `prompts/create-architecture.md` — Stage 1
- `prompts/setup-module.md` — Stage 2
- `prompts/implement-module.md` — Stage 3
- `prompts/review-and-update.md` — Stage 4 (Review and Edit Plan)

## Success Criteria
- Every module directory contains the three required files with sufficient detail
- Tests pass before a module is considered “done”
- Architecture plan stays current through approved proposals
- Changes remain incremental, reversible, and well-scoped

