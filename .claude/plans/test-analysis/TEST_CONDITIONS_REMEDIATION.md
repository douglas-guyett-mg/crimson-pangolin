# Test Conditions Remediation Plan

## Priority-Based Action Items

---

## TIER 1: Critical (Must Complete This Sprint)

### 1.1 Create Missing Test-Conditions Files

#### 1.1.1 Twilio Conversations Channel
**File:** `documentation/first-mile-communications/channels/twilio-conversations/test-conditions.md`
**Estimated size:** 120-150 tests
**Test categories needed:**

1. **SMS/MMS Specific**
   - SMS character encoding and length limits
   - MMS attachment handling
   - Multimedia fallback scenarios
   
2. **Twilio API Integration**
   - Credential management
   - Request/response parsing
   - Webhook handling for inbound messages
   - Status callbacks

3. **Error Handling**
   - Invalid phone numbers
   - Account suspension
   - Rate limiting
   - API quota exceeded

4. **Session Management**
   - Session creation from SMS
   - Multi-recipient conversations
   - Conversation state during Twilio outages

5. **Compliance**
   - Message opt-out (STOP)
   - Consent tracking
   - Message archival

#### 1.1.2 React Native Streaming Channel
**File:** `documentation/first-mile-communications/channels/react-native-streaming/test-conditions.md`
**Estimated size:** 100-130 tests
**Test categories needed:**

1. **Platform-Specific (iOS/Android)**
   - Permission requests (microphone, camera)
   - Background execution limits
   - Process lifecycle (foreground/background/suspended)
   - Platform-specific audio codecs

2. **Audio/Video Handling**
   - Audio routing (speaker, earpiece, Bluetooth)
   - Video quality adaptation
   - Camera switching
   - Screen sharing (if applicable)

3. **Network Adaptation**
   - WiFi to cellular handoff
   - Connection quality detection
   - Automatic bitrate adjustment

4. **Device Integration**
   - Battery optimization
   - Thermal throttling handling
   - Storage space constraints

5. **App Lifecycle**
   - App backgrounding/foregrounding
   - System interrupts (calls, notifications)
   - App termination and recovery

#### 1.1.3 Media Pipeline
**File:** `documentation/first-mile-communications/shared-services/media-pipeline/test-conditions.md`
**Estimated size:** 80-100 tests
**Test categories needed:**

1. **Codec Handling**
   - Supported codecs (opus, h264, vp9, etc.)
   - Codec negotiation
   - Codec error recovery
   - Graceful degradation

2. **Stream Processing**
   - Bitrate control
   - Frame rate adaptation
   - Sample rate conversion
   - Audio mixing

3. **Quality Management**
   - Audio levels normalization
   - Echo cancellation validation
   - Noise suppression verification
   - Dynamic range compression

4. **Buffer Management**
   - Jitter buffer behavior
   - Buffer overflow handling
   - Buffer underflow recovery
   - Adaptive buffer sizing

5. **Format Conversion**
   - Format transcoding
   - Resolution scaling
   - Frame interpolation
   - Sample rate conversion

#### 1.1.4 Observability Shared Service
**File:** `documentation/first-mile-communications/shared-services/observability/test-conditions.md`
**Estimated size:** 50-70 tests
**Test categories needed:**

1. **Metrics Collection**
   - Metric recording accuracy
   - Metric aggregation windows
   - Cardinality explosion prevention
   - Custom metric registration

2. **SLO Tracking**
   - Error budget calculation
   - Burn-down rate
   - Alert triggering on SLO breach
   - Recovery confirmation

3. **Alerting**
   - Alert deduplication
   - Alert routing
   - Escalation procedures
   - Alert context sufficiency

4. **Dashboarding**
   - Real-time updates
   - Historical data retention
   - Query performance
   - Chart rendering

5. **Trace Correlation**
   - Trace ID propagation
   - Multi-service tracing
   - Latency attribution
   - Error attribution

#### 1.1.5 Turn Trace System
**File:** `documentation/turn-trace/test-conditions.md`
**Estimated size:** 100-120 tests
**Test categories needed:**

1. **Event Recording**
   - Event types and schema
   - Timestamp precision
   - Event ordering guarantees
   - Concurrent event handling

2. **Trace Retrieval**
   - Filtering by daemon/service
   - Time range queries
   - Full-text search
   - Trace re-export

3. **Data Integrity**
   - No event loss
   - No event duplication
   - Immutability enforcement
   - Data corruption detection

4. **Performance**
   - Recording latency
   - Query latency (< 1s for typical)
   - Storage efficiency
   - Concurrent trace access

5. **Retention & Cleanup**
   - TTL enforcement
   - Deletion verification
   - Archive functionality
   - Garbage collection

---

### 1.2 Fix Vague Acceptance Criteria

#### Consciousness Tests

**Current:** "Returns relevant mandates"  
**Better:** "Query returns only items where: (a) type matches request, (b) applies_to is empty OR includes current_mode, (c) approval_status = 'approved', (d) effective_date <= current_time, (e) no sunset_date OR sunset_date > current_time"

**Current:** "Rationale is human-readable"  
**Better:** "Rationale includes: (a) reason why item was selected (e.g., 'Matches query: safety mandates'), (b) how item was ranked (e.g., 'Priority 9/10'), (c) any synthesis that occurred (e.g., 'Generated for customer_service mode')"

**Current:** "Results are sorted by priority"  
**Better:** "Results sorted by: (a) priority descending, (b) for equal priority: creation_date ascending, (c) is deterministic (same query always returns same order)"

#### Turn Architecture Tests

**Current:** "Executor always spawned"  
**Better:** "Executor spawned with: (a) unique daemon_id, (b) turn_id from current context, (c) parent_executor_id set if sub-turn, (d) registered in daemon registry before execution begins"

**Current:** "Plan Created"  
**Better:** "Plan is valid object with: (a) plan_id (unique), (b) steps: array of 1+ items, (c) each step has: operation, required_inputs[], optional_inputs[], outputs[], parallelizable(bool), (d) dependency graph acyclic"

**Current:** "Constraint compliance check"  
**Better:** "Executor checks ALL applicable constraints: (a) consciousness mandates, (b) working memory availability, (c) LLM budget remaining, (d) tool availability. Returns: {compliant: bool, violations: [{constraint, reason}]}"

#### Working Memory Tests

**Current:** "Token count is accurate"  
**Better:** "Token count calculation: (a) uses specified tokenizer, (b) accurate within ±2% for typical content, (c) includes markup tokens for formatting, (d) accounts for assembly overhead"

**Current:** "Context is properly formatted"  
**Better:** "Context includes: (a) header for each memory type (#### Episodic Memory, #### Semantic Memory, etc.), (b) source attribution for each item, (c) separator between items (--- or blank line), (d) total tokens at end"

---

### 1.3 Add Concurrent Scenario Tests

#### Consciousness: Mode Switching During Turn

**New Test:**
```
Test: Concurrent Mode Switch Ignored
Setup:
  - Turn T1 starts with Mode = "research"
  - Consciousness locked to "research" at turn start
  
Steps:
  1. T1 executes, queries consciousness 3 times
  2. Between query 1 and 2: User switches to Mode = "code_review"
  3. Queries 2 and 3 still use Mode = "research"
  4. T1 completes
  5. T2 starts, now uses Mode = "code_review"

Expected:
  - All queries in T1 return "research" mandates
  - Query 2 and 3 NOT affected by mode switch
  - T2 queries return "code_review" mandates
  - Turn Trace shows both mode locks

Verification:
  - T1 Turn Trace: all queries have mode="research"
  - T2 Turn Trace: all queries have mode="code_review"
```

#### Working Memory: Concurrent Evictions

**New Test:**
```
Test: Concurrent Eviction with Cascading Dependencies
Setup:
  - WM Store (max_items=10)
  - Item A (importance=0.9) - referenced by Items B,C
  - Items D-M (importance=0.3)
  - Budget triggers eviction of 5 items

Expected:
  - Evict Items D-H (lowest importance)
  - Item A protected (high importance, dependencies)
  - Items B,C protected (depend on A)
  
Verification:
  - Store contains: A,B,C,I,J,K,L,M (not D-H)
  - Eviction log shows reason for each
```

#### Frontal Cortex: Parallel Updates

**New Test:**
```
Test: Concurrent Updates to Different Fields (Merge)
Setup:
  - Goal v5: {progress: 50%, priority: high, status: active}
  - Turn A reads Goal v5, updates progress → 75%
  - Turn B reads Goal v5, updates priority → critical

Expected:
  - Turn A succeeds: Goal v6 {progress: 75%, priority: high}
  - Turn B succeeds: Goal v7 {progress: 75%, priority: critical}
  - Merge successful (different fields)

Verification:
  - Final state: progress=75%, priority=critical
  - Lineage: v5→v6→v7
  - No ConflictError
```

---

## TIER 2: High Priority (Within 2 Sprints)

### 2.1 Add Performance & Load Tests

#### Consciousness
- Query performance with 1000+ items
- Mode switching latency
- Dynamic synthesis generation time

#### Working Memory
- Context assembly with 100k episodic items
- Search performance across 50k semantic items
- Eviction time with 1000+ items

#### Frontal Cortex
- Query performance with 50k items
- Concurrent mutations from 100 turns
- Batch operations (100+ items)

#### First-Mile Communications
- Session creation latency (P99 < 500ms)
- Message throughput (events/second)
- Concurrent session limits

---

### 2.2 Improve Daemon Coordination Tests

#### New Test Category: Deadlock Prevention
- Circular daemon dependencies
- Timeout enforcement
- Escalation to Executor

#### New Test Category: Ordering Guarantees
- FIFO ordering verification
- Causal ordering (dependency-based)
- Ordering enforcement under load

#### New Test Category: Timeout Handling
- Per-query timeouts
- Default timeout application
- Timeout chain (query timeout < turn timeout)

---

### 2.3 Add Integration Test Matrix

Create matrix of:
- Consciousness + Working Memory
- Working Memory + LLM Budget System
- Turn + Error Handling
- All daemons together (10+ concurrent turns)

---

## TIER 3: Medium Priority (Next Quarter)

### 3.1 Expand Error Scenarios

For each subsystem:
- Network failures
- Resource exhaustion
- Timeout scenarios
- Cascade failures
- State corruption

### 3.2 Add Edge Case Coverage

- Empty collections
- Single item
- Maximum allowed values
- Temporal boundary cases (midnight, DST, leap seconds)
- Unicode and special characters
- Very large payloads

### 3.3 Backward Compatibility Tests

- Version migration scenarios
- API versioning
- Deprecated feature handling

---

## Implementation Checklist

### Week 1
- [ ] Create Twilio Conversations test-conditions (50 tests)
- [ ] Create React Native test-conditions (50 tests)
- [ ] Fix vague criteria in Consciousness (10 tests)
- [ ] Fix vague criteria in Turn (15 tests)

### Week 2
- [ ] Complete Twilio Conversations test-conditions
- [ ] Complete React Native test-conditions
- [ ] Create Media Pipeline test-conditions (50 tests)
- [ ] Add concurrent mode switching test
- [ ] Add concurrent eviction test

### Week 3
- [ ] Create Observability test-conditions (30 tests)
- [ ] Create Turn Trace test-conditions (50 tests)
- [ ] Add concurrent FC update test
- [ ] Fix vague criteria in Working Memory (15 tests)

### Week 4
- [ ] Validate all 31 test-conditions files
- [ ] Create integration test matrix
- [ ] Add performance targets to all tests
- [ ] Update turn trace to include all scenarios

### Ongoing
- [ ] Add load tests to performance tests
- [ ] Expand error scenarios
- [ ] Add edge case coverage
- [ ] Update as new features added

---

## Success Criteria

- All 31+ test-conditions files complete
- Zero vague acceptance criteria (all measurable)
- 80%+ happy path coverage
- 70%+ error scenario coverage
- 60%+ edge case coverage
- 70%+ concurrent scenario coverage
- All tests have clear, actionable steps
- All tests have specific expected outcomes

---

## Effort Estimate

| Task | Est. Hours | Priority |
|------|-----------|----------|
| Twilio Conversations | 16 | P0 |
| React Native | 16 | P0 |
| Media Pipeline | 10 | P0 |
| Observability | 6 | P0 |
| Turn Trace | 12 | P0 |
| Fix vague criteria | 20 | P0 |
| Concurrent tests | 12 | P0 |
| Integration tests | 16 | P1 |
| Performance tests | 12 | P1 |
| Edge cases | 12 | P2 |
| **Total** | **132 hours** | |

**Recommended:** 33 hours/week = 4 weeks to complete all Tier 1 and 2

