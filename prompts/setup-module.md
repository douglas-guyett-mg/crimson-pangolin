# Prompt: Setup Module Workspace (Stage 2)

Objective
- Create `implementation/{module-name}/` with overview.md, implementation-checklist.md, and test-conditions.md detailed enough for unambiguous execution.

Inputs
- `{module-name}` selected from `implementation/implementation-architecture.md`
- The corresponding entry in the Module Catalog

Guardrails
- Do not implement code yet. Prepare specs and checklists only.
- Keep scope 20-30 minutes; if larger, split and update the architecture plan before proceeding.
- Ask one clarifying question at a time if information is missing.
- Generate fully detailed, execution-ready specifications based on codebase analysis that require minimal clarification during Stage 3

Required Outputs
- Directory: `implementation/{module-name}/`
  - `overview.md`
  - `implementation-checklist.md`
  - `test-conditions.md`

Steps
1) Validate the module exists in the Architecture Plan and has clear purpose/dependencies
2) Create directory `implementation/{module-name}/`
3) Use codebase-retrieval to analyze existing patterns, conventions, and related code
4) Review relevant documentation in `documentation/` to understand requirements and constraints
5) Use findings to inform detailed specifications in all three module files
6) Create `overview.md` using the template below
7) Create `implementation-checklist.md` using the template below (checkboxes required)
8) Create `test-conditions.md` using the template below (comprehensive tests-first spec)
9) Self-check for ambiguity; refine until another agent could execute without questions
10) Stop and request review/approval to start Stage 3 for this module

Acceptance Criteria
- All three files exist and are complete per templates
Use codebase analysis to fully specify user stories, inputs/outputs, integration points, concrete file paths under `si/{module-name}/`, function signatures, and class names.

- Every checklist item uses clear, verifiable language with checkboxes
- Test conditions cover unit, integration, acceptance, and edge cases

Templates (copy and fill)

---
File: implementation/{module-name}/overview.md

# {module-name}: Overview

## What and Why
- Description:
- Why it exists / value:

## User Stories (Done when …)
- As a … I can … so that …
- …

## Dependencies
- Upstream modules:
- Downstream impact:

## Integration Points
- Files/APIs/events touched:
- Interfaces/contracts:

## Success Criteria
- Functional:
- Non-functional (performance, safety):
- Verification: linked to `test-conditions.md`

## Assumptions / Non-Goals
- Assumptions:
- Non-goals:
---

---
File: implementation/{module-name}/implementation-checklist.md

# {module-name}: Implementation Checklist

- All code paths must be under `si/{module-name}/` per Code Isolation and Directory Structure.

Guidance
- Check off each item when done. Keep this file in sync as you work.
- Include verification sub-steps after each meaningful action.

## Preparation
- [ ] Confirm prerequisites from dependencies are met
- [ ] Identify files to create/modify (list below)

## Files to Create/Modify
- [ ] Create: si/{module-name}/path/to/file1.ext — purpose
- [ ] Modify: si/{module-name}/path/to/existing_file.ext — purpose

## Implement Components
- [ ] Implement function `name(args) -> return` in si/{module-name}/path/to/file
  - [ ] Add unit tests for `name` per test-conditions
- [ ] Implement class `ClassName` with methods …
  - [ ] Add unit tests for `ClassName` per test-conditions

## Configuration Changes
- [ ] Update config at <key.path> — rationale and exact key/value
  - [ ] Verify config via small smoke check

## Documentation Updates
- [ ] Update in-repo docs/comments affected by this change

## Verification Steps
- [ ] Run unit tests for this module
- [ ] Run integration tests relevant to this module
- [ ] Confirm acceptance criteria in `overview.md` are met

## Completion
- [ ] All boxes above checked
- [ ] Test summary recorded below

### Test Summary
- Unit:
- Integration:
- Acceptance:
---

---
File: implementation/{module-name}/test-conditions.md

# {module-name}: Test Conditions

## Unit Tests
- Component A: inputs → outputs → assertions
- Component B: inputs → outputs → assertions

## Integration Scenarios
- Scenario 1: setup, actions, expected outcomes
- Scenario 2: …

## Acceptance Criteria (map to user stories)
- AC-1: … (linked to verification steps)
- AC-2: …

## Edge Cases / Error Handling
- Invalid inputs:
- Timeouts / retries:
- Partial failures:
- Concurrency/contention:

## Environment Considerations
- Data preconditions:
- External services/interfaces:

## Test Data and Fixtures
- Fixture list and sample values

## Exit Criteria
- All unit and integration tests pass locally
- Acceptance criteria verified against `overview.md`
---

