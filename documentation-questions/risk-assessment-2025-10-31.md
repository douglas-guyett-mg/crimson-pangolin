# Risk Assessment: Si Documentation Post-Updates
**Date**: 2025-10-31  
**Reviewer**: Codex Agent  
**Scope**: Critical gaps and implementation blockers after recent documentation updates

---

## Executive Summary

The documentation set is **extensive and well-structured** (20+ working memory sub-components, complete turn architecture, consciousness framework). However, **8 significant risks** remain that will block or confuse implementation. **3 are critical** and require immediate resolution.

---

## CRITICAL RISKS (Blocking Implementation)

### 🔴 Risk #1: Budget Allocation Strategies Undefined
**Severity**: CRITICAL  
**Status**: NOT STARTED  
**Files Affected**:
- `documentation/working-memory/memory-orchestrator/overview.md:60-72` (references strategy but doesn't define it)
- `documentation/working-memory/test-conditions.md:91-109` (tests expect strategies)
- `documentation/working-memory/overview.md:91-92` (mentions budget allocation)

**Problem**: Multiple components reference `budget_allocation_strategy` (equal, weighted, priority-based) but no specification exists for:
- Available strategies and their names
- Configuration parameters for each strategy
- Default strategy selection logic
- How strategies interact with memory type priorities

**Impact**: 
- Implementers cannot build reproducible orchestrator behavior
- Tests cannot verify correct budget allocation
- Different implementations will use different strategies

**Recommended Action**:
- Create `documentation/working-memory/budget-enforcement/budget-allocation-strategies.md`
- Document: equal, weighted, priority-based, dynamic strategies
- Include: parameters, defaults, examples, test scenarios

---

### 🔴 Risk #2: Sampling and Scoring Engines Not Documented
**Severity**: CRITICAL  
**Status**: NOT STARTED  
**Files Affected**:
- `documentation/working-memory/budget-enforcement/overview.md:50-65` (mentions Sampling Engine)
- `documentation/working-memory/write-gating-algorithm/overview.md:38-49` (mentions Scoring Engine)

**Problem**: Two critical subsystems are referenced but never documented as standalone components:
- **Sampling Engine**: How does it decide what to sample? What algorithms? What parameters?
- **Scoring Engine**: How does it score items? What factors? What formula?

**Impact**:
- Unclear responsibilities and interfaces
- No test requirements defined
- Implementers must guess at algorithms and parameters

**Recommended Action**:
- Create `documentation/working-memory/sampling-engine/` with overview.md + test-conditions.md
- Create `documentation/working-memory/scoring-engine/` with overview.md + test-conditions.md
- Document algorithms, parameters, scoring factors, test scenarios

---

### 🔴 Risk #3: Tooling Architecture Path References Wrong
**Severity**: CRITICAL  
**Status**: NOT STARTED  
**Files Affected**:
- `documentation/tools/si_tools_architecture.md:30-35` (references agent-plans/tools/)
- `documentation/tools/si_tools_architecture.md:46, 62` (more old path references)

**Problem**: The tooling architecture document is the authoritative spec but references old directory structure:
- Says: `agent-plans/tools/tool_manifest_schema.yaml`
- Actually: `documentation/tools/tool_manifest_schema.yaml`
- This breaks the "documentation-as-code" principle

**Impact**:
- Broken references confuse implementers
- Unclear where authoritative specs live
- Violates documentation-as-code principle

**Recommended Action**:
- Update all path references in `si_tools_architecture.md` from `agent-plans/tools/` to `documentation/tools/`
- Verify all referenced files exist in correct location
- Add cross-reference validation to documentation review process

---

## MODERATE RISKS (Needs Clarification)

### 🟡 Risk #4: test_matrix.md Missing for Tools
**Severity**: MEDIUM  
**Status**: NOT STARTED  
**Files Affected**:
- `documentation/tools/si_tools_architecture.md:35` (marked as "future work")

**Problem**: Tool testing requirements are not specified. Line 35 says `test_matrix.md` is "future work" but this is critical for:
- Tool invocation error handling
- Tool observability verification
- Tool contract compliance

**Impact**:
- Unclear what tests are required for tools
- No standard test scenarios
- Tool implementations may lack critical tests

**Recommended Action**:
- Create `documentation/tools/test_matrix.md`
- Document: invocation scenarios, error cases, observability tests, contract validation

---

### 🟡 Risk #5: Consciousness Integration Mechanism Unclear
**Severity**: MEDIUM  
**Status**: PARTIALLY DOCUMENTED  
**Files Affected**:
- `documentation/working-memory/memory-orchestrator/overview.md:25-52` (mentions consciousness alignment)
- `documentation/consciousness/` (no integration spec)

**Problem**: Memory Orchestrator says it "aligns with consciousness" but the mechanism is vague:
- How does WM retrieve consciousness objects?
- What is the `.to_text()` method contract?
- How are consciousness fragments injected into prompts?
- What is the lifecycle of consciousness objects during a turn?

**Impact**:
- Unclear how to implement consciousness integration
- Unclear error handling for missing consciousness objects
- Unclear performance implications

**Recommended Action**:
- Create `documentation/consciousness/working-memory-integration.md`
- Document: retrieval API, object contract, injection mechanism, error handling

---

### 🟡 Risk #6: Working Memory Integration Guide Missing
**Severity**: MEDIUM  
**Status**: NOT STARTED  
**Files Affected**:
- `documentation/working-memory/overview.md:175-212` (references integration points)

**Problem**: No `documentation/working-memory/integration.md` exists to document:
- Data flows between WM and other systems
- API contracts for memory operations
- Error handling expectations
- Lifecycle of memory items across turns

**Impact**:
- Unclear how WM integrates with turn architecture
- Unclear how WM integrates with frontal cortex
- Unclear how WM integrates with consciousness

**Recommended Action**:
- Create `documentation/working-memory/integration.md`
- Document: data flows, API contracts, error handling, lifecycle

---

### 🟡 Risk #7: Memory Daemon Interface Unclear
**Severity**: MEDIUM  
**Status**: NOT STARTED  
**Files Affected**:
- `documentation/scratch-page/overview.md` (mentions "memory daemons")
- `documentation/working-memory/memory-daemon-integration.md` (exists but unclear)

**Problem**: Multiple references to "memory daemons" but no clear specification of:
- What is a memory daemon?
- What is the daemon interface?
- How do daemons interact with scratch page?
- What is the daemon lifecycle?

**Impact**:
- Unclear how to implement memory daemons
- Unclear how daemons interact with other systems
- Unclear error handling

**Recommended Action**:
- Review and clarify `documentation/working-memory/memory-daemon-integration.md`
- Document: daemon interface, lifecycle, communication patterns

---

## LOW RISKS (Confusing But Not Blocking)

### 🟠 Risk #8: Redaction Engine Removed But References Remain
**Severity**: LOW  
**Status**: PARTIALLY RESOLVED  
**Files Affected**:
- `documentation/working-memory/budget-enforcement/overview.md:58-65` (mentions Redaction Policies)

**Problem**: Architectural decision made (no redaction engine, privacy via access control) but budget enforcement still mentions "Redaction Policies" as if they exist.

**Impact**:
- Confusion about whether redaction is supported
- Unclear what "Redaction Policies" means in context of access control

**Recommended Action**:
- Update budget enforcement documentation to clarify: sampling is supported, redaction is not (privacy via access control)
- Remove or clarify "Redaction Policies" section

---

## Summary Table

| Risk | Severity | Status | Blocker | Est. Effort |
|------|----------|--------|---------|------------|
| Budget Allocation Strategies | CRITICAL | NOT STARTED | YES | 4-6 hours |
| Sampling/Scoring Engines | CRITICAL | NOT STARTED | YES | 6-8 hours |
| Tooling Path References | CRITICAL | NOT STARTED | YES | 1-2 hours |
| test_matrix.md | MEDIUM | NOT STARTED | PARTIAL | 3-4 hours |
| Consciousness Integration | MEDIUM | PARTIAL | PARTIAL | 2-3 hours |
| WM Integration Guide | MEDIUM | NOT STARTED | PARTIAL | 3-4 hours |
| Memory Daemon Interface | MEDIUM | PARTIAL | PARTIAL | 2-3 hours |
| Redaction References | LOW | PARTIAL | NO | 1 hour |

**Total Estimated Effort**: 22-31 hours

---

## Recommended Priority Order

1. **Fix tooling path references** (1-2 hours) - Quick win, unblocks tool implementation
2. **Document budget allocation strategies** (4-6 hours) - Unblocks orchestrator implementation
3. **Document sampling/scoring engines** (6-8 hours) - Unblocks budget enforcement and write gating
4. **Create test_matrix.md** (3-4 hours) - Unblocks tool testing
5. **Clarify consciousness integration** (2-3 hours) - Unblocks consciousness integration
6. **Create WM integration guide** (3-4 hours) - Unblocks system-level testing
7. **Clarify memory daemon interface** (2-3 hours) - Unblocks daemon implementation
8. **Update redaction references** (1 hour) - Clarifies budget enforcement

