# Remediation Checklist - Si Documentation Risks
**Date**: 2025-10-31
**Total Items**: 5 risks across 2 severity levels

---

## ✅ RESOLVED - Decentralized to Memory Type Plugins

### [x] Risk #1: Budget Allocation Strategies
- **Status**: RESOLVED
- **Reason**: Logic decentralized to individual memory type plugins
- **Details**: Each memory type (Working Memory, Episodic, Semantic, Conversation History) handles its own capacity management and eviction policies
- **Files Removed**: `documentation/working-memory/budget-enforcement/overview.md`

### [x] Risk #2: Sampling Engine Documentation
- **Status**: RESOLVED
- **Reason**: Logic decentralized to individual memory type plugins
- **Details**: Each memory type handles its own sampling/reduction when capacity is exceeded
- **Files Removed**: Centralized sampling engine references

### [x] Risk #3: Scoring Engine Documentation
- **Status**: RESOLVED
- **Reason**: Logic decentralized to individual memory type plugins
- **Details**: Each memory type scores items according to its own semantics (importance, recency, etc.)
- **Files Removed**: `documentation/working-memory/write-gating-algorithm/overview.md`

---

## 🔴 CRITICAL - Must Fix Before Implementation

### [ ] Risk #4: Fix Tooling Architecture Path References
- **File to update**: `documentation/tools/si_tools_architecture.md`
- **Changes needed**:
  - [ ] Line 30-35: Change `agent-plans/tools/` to `documentation/tools/`
  - [ ] Line 46: Update tool manifest schema path
  - [ ] Line 62: Update tool IO contract path
  - [ ] Verify all referenced files exist in correct location
  - [ ] Add validation note about path consistency
- **Blocks**: Tool Discovery, Tool Invocation
- **Effort**: 1-2 hours
- **Status**: NOT STARTED

---

## 🟡 MODERATE - Should Fix Before Testing

### [ ] Risk #5: Create test_matrix.md for Tools
- **File to create**: `documentation/tools/test_matrix.md`
- **Content needed**:
  - [ ] Tool invocation scenarios (success, error, timeout, partial)
  - [ ] Error handling test cases
  - [ ] Observability verification tests
  - [ ] Contract compliance tests
  - [ ] Performance benchmarks
  - [ ] Integration test scenarios
- **Blocks**: Tool Testing Framework
- **Effort**: 3-4 hours
- **Status**: NOT STARTED

### [ ] Risk #6: Clarify Consciousness Integration
- **File to create**: `documentation/consciousness/working-memory-integration.md`
- **Content needed**:
  - [ ] Define consciousness object retrieval API
  - [ ] Document `.to_text()` method contract
  - [ ] Specify consciousness fragment injection mechanism
  - [ ] Define lifecycle of consciousness objects during turn
  - [ ] Document error handling for missing objects
  - [ ] Include integration examples
- **Blocks**: Consciousness Integration Implementation
- **Effort**: 2-3 hours
- **Status**: NOT STARTED

### [ ] Risk #7: Create WM Integration Guide
- **File to create**: `documentation/working-memory/integration.md`
- **Content needed**:
  - [ ] Data flows between WM and Turn Architecture
  - [ ] Data flows between WM and Frontal Cortex
  - [ ] Data flows between WM and Consciousness
  - [ ] API contracts for memory operations
  - [ ] Error handling expectations
  - [ ] Lifecycle of memory items across turns
  - [ ] Include sequence diagrams
- **Blocks**: System-Level Testing
- **Effort**: 3-4 hours
- **Status**: NOT STARTED

### [ ] Risk #8: Clarify Memory Daemon Interface
- **File to review/update**: `documentation/working-memory/memory-daemon-integration.md`
- **Content needed**:
  - [ ] Define what a memory daemon is
  - [ ] Document daemon interface/contract
  - [ ] Specify daemon lifecycle (creation, activation, deactivation)
  - [ ] Document communication patterns with scratch page
  - [ ] Document communication patterns with memory orchestrator
  - [ ] Include error handling
  - [ ] Provide implementation examples
- **Blocks**: Daemon Implementation
- **Effort**: 2-3 hours
- **Status**: NOT STARTED

---

## Summary by Phase

### Phase 1: Quick Wins (1-2 hours) ✅ COMPLETE
- [x] Fix tooling path references

### Phase 2: Moderate Issues (10-14 hours)
- [ ] Create test_matrix.md
- [ ] Clarify consciousness integration
- [ ] Create WM integration guide
- [ ] Clarify memory daemon interface

---

## Verification Checklist

After completing each item, verify:

- [ ] File created in correct location
- [ ] Follows documentation structure (overview.md + test-conditions.md for components)
- [ ] Aligns with Si Core Engineering Tenants
- [ ] All referenced files/paths are correct
- [ ] Cross-references updated in related files
- [ ] Test conditions are measurable and verifiable
- [ ] Examples provided where helpful
- [ ] No broken links or references

---

## Sign-Off

- [ ] All critical risks resolved
- [ ] All moderate risks resolved
- [ ] All low risks resolved
- [ ] Documentation review passed
- [ ] Ready for implementation

