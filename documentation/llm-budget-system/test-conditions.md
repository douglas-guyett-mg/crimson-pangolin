# LLM Budget System - Test Conditions

**Status**: Specification v1.0  
**Last Updated**: 2025-11-04  
**Scope**: Tests for global LLM budgets, daemon-specific overrides, and budget enforcement

---

## Test Suite: Global LLM Budget Configuration

### Test 1.1: System Initializes with Global LLM Budget
**Objective**: Verify system initializes with configured global LLM budget

**Setup**:
- Configure system with global_llm_budget=4000
- Initialize system

**Steps**:
1. Initialize system
2. Query global LLM budget configuration

**Expected Outcome**:
- System has global_llm_budget=4000 configured
- All daemons use this budget unless overridden

**Verification**:
- `get_global_llm_budget()` returns 4000
- Daemons without specific overrides use 4000

---

### Test 1.2: System Initializes with Daemon-Specific Overrides
**Objective**: Verify daemon-specific budget overrides are applied

**Setup**:
- Configure system with:
  - global_llm_budget=4000
  - daemon_budgets: {router: 2000, plan_generator: 6000}
- Initialize system

**Steps**:
1. Initialize system
2. Query budget for each daemon

**Expected Outcome**:
- Router daemon has budget=2000
- Plan Generator daemon has budget=6000
- Other daemons use global budget=4000

**Verification**:
- `get_daemon_budget("router")` returns 2000
- `get_daemon_budget("plan_generator")` returns 6000
- `get_daemon_budget("executor")` returns 4000

---

## Test Suite: Budget Enforcement

### Test 2.1: Accept LLM Call Within Budget
**Objective**: Verify LLM calls within budget are accepted

**Setup**:
- Configure daemon with budget=4000
- Prepare prompt with 3500 tokens

**Steps**:
1. Call `check_budget(daemon="router", prompt_tokens=3500)`
2. Verify result

**Expected Outcome**:
- Budget check passes
- LLM call is allowed

**Verification**:
- `check_budget()` returns {allowed: true, reason: "within_budget"}

---

### Test 2.2: Reject LLM Call Over Budget
**Objective**: Verify LLM calls over budget are rejected

**Setup**:
- Configure daemon with budget=4000
- Prepare prompt with 4500 tokens

**Steps**:
1. Call `check_budget(daemon="router", prompt_tokens=4500)`
2. Verify result

**Expected Outcome**:
- Budget check fails
- LLM call is rejected

**Verification**:
- `check_budget()` returns {allowed: false, reason: "exceeds_budget"}
- Error is logged

---

### Test 2.3: Reject LLM Call Exactly at Budget Limit
**Objective**: Verify behavior when prompt exactly equals budget

**Setup**:
- Configure daemon with budget=4000
- Prepare prompt with exactly 4000 tokens

**Steps**:
1. Call `check_budget(daemon="router", prompt_tokens=4000)`
2. Verify result

**Expected Outcome**:
- Budget check passes (at limit is acceptable)
- LLM call is allowed

**Verification**:
- `check_budget()` returns {allowed: true, reason: "at_budget_limit"}

---

## Test Suite: Budget Logging and Auditing

### Test 3.1: Log Budget Decisions
**Objective**: Verify all budget decisions are logged

**Setup**:
- Configure system with log_budget_decisions=true
- Make multiple LLM calls (some within, some over budget)

**Steps**:
1. Make 3 LLM calls: 2 within budget, 1 over budget
2. Query budget decision log

**Expected Outcome**:
- All 3 decisions are logged
- Log includes: daemon, prompt_tokens, budget, decision, timestamp

**Verification**:
- `get_budget_log()` returns 3 entries
- Each entry has required fields

---

### Test 3.2: Track Budget Usage Per Daemon
**Objective**: Verify budget usage is tracked per daemon

**Setup**:
- Configure system with multiple daemons
- Make LLM calls from different daemons

**Steps**:
1. Router makes call with 2000 tokens
2. Plan Generator makes call with 3000 tokens
3. Query usage per daemon

**Expected Outcome**:
- Router usage: 2000 tokens
- Plan Generator usage: 3000 tokens
- Tracked separately

**Verification**:
- `get_daemon_usage("router")` returns 2000
- `get_daemon_usage("plan_generator")` returns 3000

---

## Test Suite: Configuration Changes

### Test 4.1: Update Global LLM Budget at Runtime
**Objective**: Verify global budget can be updated at runtime

**Setup**:
- Initialize with global_llm_budget=4000
- Update to global_llm_budget=5000

**Steps**:
1. Initialize system
2. Call `set_global_llm_budget(5000)`
3. Query new budget

**Expected Outcome**:
- Global budget updated to 5000
- Daemons without overrides use new budget

**Verification**:
- `get_global_llm_budget()` returns 5000
- New LLM calls use 5000 budget

---

### Test 4.2: Update Daemon-Specific Budget at Runtime
**Objective**: Verify daemon budgets can be updated at runtime

**Setup**:
- Initialize with router budget=2000
- Update to router budget=3000

**Steps**:
1. Initialize system
2. Call `set_daemon_budget("router", 3000)`
3. Query new budget

**Expected Outcome**:
- Router budget updated to 3000
- Other daemons unaffected

**Verification**:
- `get_daemon_budget("router")` returns 3000
- `get_daemon_budget("plan_generator")` unchanged

---

## Test Suite: Edge Cases

### Test 5.1: Zero Budget Rejects All Calls
**Objective**: Verify zero budget rejects all LLM calls

**Setup**:
- Configure daemon with budget=0

**Steps**:
1. Attempt LLM call with any prompt size
2. Verify rejection

**Expected Outcome**:
- All LLM calls rejected
- Error message indicates zero budget

**Verification**:
- `check_budget()` returns {allowed: false}

---

### Test 5.2: Negative Budget Treated as Unlimited
**Objective**: Verify negative budget means unlimited

**Setup**:
- Configure daemon with budget=-1

**Steps**:
1. Attempt LLM call with very large prompt (100k tokens)
2. Verify acceptance

**Expected Outcome**:
- LLM call accepted
- No budget enforcement

**Verification**:
- `check_budget()` returns {allowed: true, reason: "unlimited_budget"}

