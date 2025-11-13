# Prompt: Create Si Implementation Architecture (Stage 1)

Objective
- Analyze `documentation/` and produce a modular, sequenced implementation architecture that an agent can execute in 20-30 minute modules.

Inputs
- Entire `documentation/` directory
- `.augment/rules/implementation-workflow.md`, `.augment/rules/documentation.md`, `.augment/rules/file-creation-discipline.md`

Guardrails
- Do not write code. Produce a plan only.
- Respect file-creation discipline. Only create the single plan file below.
- If information is missing, ask one clarifying question at a time; pause for answers.

Required Output (single file)
- Path: `implementation/implementation-architecture.md`
- Content must include the sections in the template below.

Steps
1) Review all relevant docs under `documentation/`; list components, interfaces, and constraints
2) Identify candidate modules; ensure each can be completed in ~20-30 minutes by a single agent
3) Define dependencies among modules and derive a safe execution order
4) Draft the Proposed Implementation Tree under `implementation/`
5) Write the plan using the template; include risks and open questions
6) Validate: every module has purpose, dependencies, success criteria, and testability
7) Save the file to `implementation/implementation-architecture.md`
8) Stop and request human approval to proceed to Stage 2


Note: Detailed module specifications will be generated in Stage 2.

Acceptance Criteria
- Plan contains Implementation Tree, Module Catalog, Sequencing, and Acceptance Gates
- Every module entry contains name (slug), description, dependencies, and a 20-30 minute estimate
- Open questions and assumptions are explicit

Architecture Plan Template (copy and fill)

---
# Si Implementation Architecture

## 1. Scope and Goals
- In scope:
- Out of scope:
- Constraints:

## 2. Proposed Implementation Tree (under `implementation/`)
```
implementation/
  {module-1}/
    overview.md
    implementation-checklist.md
    test-conditions.md
  {module-2}/
    ...
```

## 3. Module Catalog
- For each module, capture:
  - name (slug):
  - description:
  - dependencies:
  - estimated effort (20-30 minutes):

## 4. Dependency Graph and Sequencing
- Textual DAG summary:
- Sequenced list with rationale:

## 5. Open Questions and Assumptions
- Q1:
- Assumptions:

## 6. Acceptance Gates
- Human approval required before Stage 2
- Readiness checklist:
  - [ ] Every module has clear success criteria and tests-first notes
  - [ ] Sequence validated against dependencies
---

