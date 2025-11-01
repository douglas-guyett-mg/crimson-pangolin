# Clarification Questions — 2025-10-31
**Purpose**: Resolve outstanding ambiguities surfaced during the latest documentation review before moving into remediation work.

---

## Turn System

1. **Required Test Artifacts**  
   `.augment/rules/documentation.md:60-70` requires every testable component to ship an `overview.md` and `test-conditions.md`. Should we produce:
   - A single `documentation/turn/test-conditions.md`, or
   - Separate specs per hobgoblin (for example `documentation/turn/hobgoblins/clarifier/test-conditions.md`), or both?

2. **Missing Hobgoblin Specs**  
   `documentation/turn/overview.md:79-90` enumerates `error-handler`, `evaluator`, `reflector`, and `responder`, but no documents exist for them. Are these roles still part of the canonical architecture, and if so what scope and test expectations should their specs cover?

## Working Memory

3. **Redaction & Sampling Subsystems**  
   Budget enforcement references “Sampling Engine” and “Redaction Engine” (`documentation/working-memory/budget-enforcement/overview.md:141-155`), and write gating relies on `Redaction`/`Scoring Engine` services (`documentation/working-memory/write-gating-algorithm/overview.md:151-156`). Should these engines live as standalone components (with their own overview/test docs), or should their logic be documented inline within existing files?

4. **Budget Allocation Strategies**  
   Multiple specs expect strategies like “equal, weighted, priority-based” (`documentation/working-memory/test-conditions.md:95-109`, `documentation/working-memory/memory-orchestrator/overview.md:193-205`). Can you provide the authoritative list of strategies, configuration knobs, and defaults we should encode?

5. **Integration Guide Expectations**  
   The working memory overview references numerous integration points (`documentation/working-memory/overview.md:175-212`). Do we need a dedicated `documentation/working-memory/integration.md` capturing end-to-end data flows, or is another location preferred for those diagrams/contracts?

6. **Redaction Level Taxonomy**  
   The write-gating pipeline summarizes redaction levels (`documentation/working-memory/write-gating-algorithm/overview.md:19-34`). Should we align with an existing organization-wide taxonomy (PII classifications, data sensitivity tiers), or define one specific to Si here?

## Tooling Architecture

7. **Canonical Location for Tool Schemas**  
   The architecture plan still points to `agent-plans/tools/*` (`documentation/tools/si_tools_architecture.md:28-63`), but schemas live under `documentation/tools/`. Should we relocate the files back to `agent-plans/`, or update the plan to reference the new location?

8. **Test Matrix Artifact**  
   The same diagram references a `test_matrix.md` that does not exist. Is a consolidated test matrix still desired for tooling, and if so what scope/format should it take?

## Cross-System

9. **Hobgoblin Extensibility**  
   Are hobgoblins intended to be registered/extensible (similar to memory plugins), and if so where should that registry and its governance be documented?

10. **Default Governance for Budget Strategy Selection**  
    Who decides which budget allocation strategy applies to a given agent or tenant, and should that governance process live in working memory docs or a higher-level operations guide?

