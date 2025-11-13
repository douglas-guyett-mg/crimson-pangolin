# Prompt: Review and Edit Plan (Stage 4)

Objective
- Review `{module-name}` implementation against its checklist and tests; capture learnings; propose and, upon approval, edit the architecture plan; await approval.

Inputs
- `implementation/{module-name}/overview.md`
- `implementation/{module-name}/implementation-checklist.md`
- `implementation/{module-name}/test-conditions.md`
- Current `implementation/implementation-architecture.md`

Guardrails
- Do not start the next module until human approval of any proposed architecture updates
- Keep proposals concise and focused; one change per proposal when possible

Steps
1) Verification
   - Run all tests referenced by `test-conditions.md`
   - Ensure all checklist items are checked; if not, return to Stage 3
2) Learnings
   - Document discoveries: constraints, patterns, pitfalls, simplifications
3) Architecture Update Proposal (if warranted)
   - Create: `implementation/proposals/{YYYY-MM-DD}-{module-slug}-architecture-update.md`
   - Use the template below
4) Approval Gate
   - Request human review and explicit approval/rejection
   - If approved, update `implementation/implementation-architecture.md` accordingly (edit in place)
   - If rejected or deferred, record reasoning and adjust backlog/sequence

Acceptance Criteria
- All tests pass; acceptance criteria met
- Learnings captured succinctly
- Proposal (if any) includes problem, evidence, change, impact, alternatives, test implications

Proposal Template (copy and fill)

---
# Architecture Update Proposal: {short-title}

## Problem / Evidence
- What was observed; links to code, tests, or logs

## Proposed Change
- Description of the change to modules/sequence/structure

## Impact
- Affected modules and dependencies
- Risks and mitigations

## Alternatives Considered
- Option A/B with trade-offs

## Test Implications
- New/updated tests required

## Approval
- Status: Pending | Approved | Rejected
- Reviewer notes:
---

