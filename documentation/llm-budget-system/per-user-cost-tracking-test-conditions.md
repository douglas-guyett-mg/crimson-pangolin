# Per-User Cost Tracking - Test Conditions

**Status**: Specification v1.0  
**Last Updated**: 2025-11-04  
**Scope**: Tests for lifetime budgets, period budgets, enforcement, and rollover

---

## Test Suite: Lifetime Budget Tracking

### Test 1.1: Initialize User with Lifetime Budget
**Objective**: Verify user is initialized with lifetime budget

**Setup**:
- Create user with lifetime_budget=1000000 tokens
- Initialize tracking

**Steps**:
1. Initialize user tracking
2. Query lifetime budget

**Expected Outcome**:
- User has lifetime_budget=1000000
- lifetime_tokens_used=0

**Verification**:
- `get_user_lifetime_budget(user_id)` returns 1000000
- `get_user_lifetime_used(user_id)` returns 0

---

### Test 1.2: Track Lifetime Token Usage
**Objective**: Verify lifetime token usage is accumulated

**Setup**:
- User with lifetime_budget=1000000
- Make 3 LLM calls: 5000, 3000, 2000 tokens

**Steps**:
1. Make first LLM call (5000 tokens)
2. Make second LLM call (3000 tokens)
3. Make third LLM call (2000 tokens)
4. Query lifetime usage

**Expected Outcome**:
- Lifetime usage = 10000 tokens
- All calls succeed

**Verification**:
- `get_user_lifetime_used(user_id)` returns 10000

---

### Test 1.3: Reject LLM Call When Lifetime Budget Exceeded
**Objective**: Verify LLM calls rejected when lifetime budget exceeded

**Setup**:
- User with lifetime_budget=10000
- Already used 9500 tokens
- Attempt call with 1000 tokens

**Steps**:
1. Set lifetime_tokens_used=9500
2. Attempt LLM call with 1000 tokens
3. Verify rejection

**Expected Outcome**:
- LLM call rejected
- Error indicates lifetime budget exceeded

**Verification**:
- `check_user_budget(user_id, tokens=1000)` returns {allowed: false, reason: "lifetime_budget_exceeded"}

---

## Test Suite: Period Budget Tracking

### Test 2.1: Initialize User with Period Budget
**Objective**: Verify user is initialized with period budget

**Setup**:
- Create user with:
  - plan_id="pro"
  - period_budget=100000 tokens
  - period_duration="1 month"
- Initialize tracking

**Steps**:
1. Initialize user tracking
2. Query period budget

**Expected Outcome**:
- User has period_budget=100000
- period_tokens_used=0
- period_start is current time

**Verification**:
- `get_user_period_budget(user_id)` returns 100000
- `get_user_period_used(user_id)` returns 0

---

### Test 2.2: Track Period Token Usage
**Objective**: Verify period token usage is accumulated

**Setup**:
- User with period_budget=100000
- Make 3 LLM calls: 5000, 3000, 2000 tokens

**Steps**:
1. Make first LLM call (5000 tokens)
2. Make second LLM call (3000 tokens)
3. Make third LLM call (2000 tokens)
4. Query period usage

**Expected Outcome**:
- Period usage = 10000 tokens
- All calls succeed

**Verification**:
- `get_user_period_used(user_id)` returns 10000

---

### Test 2.3: Reject LLM Call When Period Budget Exceeded
**Objective**: Verify LLM calls rejected when period budget exceeded

**Setup**:
- User with period_budget=10000
- Already used 9500 tokens in current period
- Attempt call with 1000 tokens

**Steps**:
1. Set period_tokens_used=9500
2. Attempt LLM call with 1000 tokens
3. Verify rejection

**Expected Outcome**:
- LLM call rejected
- Error indicates period budget exceeded

**Verification**:
- `check_user_budget(user_id, tokens=1000)` returns {allowed: false, reason: "period_budget_exceeded"}

---

## Test Suite: Period Rollover

### Test 3.1: Automatic Period Rollover
**Objective**: Verify period automatically rolls over when duration expires

**Setup**:
- User with period_duration="1 day"
- period_start = 2 days ago
- period_tokens_used = 50000

**Steps**:
1. Query user budget (triggers rollover check)
2. Verify period was reset

**Expected Outcome**:
- New period_start = current time
- New period_tokens_used = 0
- Lifetime tokens still accumulated

**Verification**:
- `get_user_period_used(user_id)` returns 0
- `get_user_lifetime_used(user_id)` includes previous period

---

### Test 3.2: Archive Previous Period Data
**Objective**: Verify previous period data is archived

**Setup**:
- User with period_duration="1 day"
- period_start = 2 days ago
- period_tokens_used = 50000

**Steps**:
1. Query user budget (triggers rollover)
2. Query period history

**Expected Outcome**:
- Previous period archived with 50000 tokens
- New period initialized with 0 tokens

**Verification**:
- `get_user_period_history(user_id)` includes archived period
- Archived period shows 50000 tokens used

---

## Test Suite: Hard Enforcement

### Test 4.1: Stop All LLM Calls When Lifetime Exceeded
**Objective**: Verify all LLM calls stop when lifetime budget exceeded

**Setup**:
- User with lifetime_budget=10000
- Already used 10000 tokens
- Attempt multiple LLM calls

**Steps**:
1. Set lifetime_tokens_used=10000
2. Attempt 3 LLM calls
3. Verify all rejected

**Expected Outcome**:
- All 3 LLM calls rejected
- User notified of status

**Verification**:
- All calls return {allowed: false, reason: "lifetime_budget_exceeded"}

---

### Test 4.2: Stop All LLM Calls When Period Exceeded
**Objective**: Verify all LLM calls stop when period budget exceeded

**Setup**:
- User with period_budget=10000
- Already used 10000 tokens in current period
- Attempt multiple LLM calls

**Steps**:
1. Set period_tokens_used=10000
2. Attempt 3 LLM calls
3. Verify all rejected

**Expected Outcome**:
- All 3 LLM calls rejected
- User notified of status

**Verification**:
- All calls return {allowed: false, reason: "period_budget_exceeded"}

---

## Test Suite: Plan-Based Configuration

### Test 5.1: Different Plans Have Different Periods
**Objective**: Verify period duration varies by plan

**Setup**:
- Create 3 users with different plans:
  - free: period="1 day"
  - pro: period="1 month"
  - enterprise: period="1 quarter"

**Steps**:
1. Initialize all users
2. Query period duration for each

**Expected Outcome**:
- Free user: period_duration="1 day"
- Pro user: period_duration="1 month"
- Enterprise user: period_duration="1 quarter"

**Verification**:
- `get_user_period_duration(user_id)` returns correct duration per plan

---

### Test 5.2: Different Plans Have Different Budgets
**Objective**: Verify budget limits vary by plan

**Setup**:
- Create 3 users with different plans:
  - free: lifetime=100k, period=10k
  - pro: lifetime=1M, period=100k
  - enterprise: lifetime=10M, period=1M

**Steps**:
1. Initialize all users
2. Query budgets for each

**Expected Outcome**:
- Each user has correct budgets per plan

**Verification**:
- `get_user_lifetime_budget(user_id)` returns correct value per plan
- `get_user_period_budget(user_id)` returns correct value per plan

---

## Test Suite: Logging and Auditing

### Test 6.1: Log All Cost Tracking Decisions
**Objective**: Verify all cost tracking decisions are logged

**Setup**:
- Configure system with log_all_tracking=true
- Make multiple LLM calls (some allowed, some rejected)

**Steps**:
1. Make 5 LLM calls (3 allowed, 2 rejected)
2. Query cost tracking log

**Expected Outcome**:
- All 5 decisions logged
- Log includes: user_id, tokens, decision, reason, timestamp

**Verification**:
- `get_cost_tracking_log(user_id)` returns 5 entries
- Each entry has required fields

---

### Test 6.2: Track Cost Per User
**Objective**: Verify costs are tracked separately per user

**Setup**:
- Create 2 users
- User A makes calls totaling 5000 tokens
- User B makes calls totaling 3000 tokens

**Steps**:
1. Make calls from both users
2. Query usage for each user

**Expected Outcome**:
- User A usage: 5000 tokens
- User B usage: 3000 tokens
- Tracked separately

**Verification**:
- `get_user_lifetime_used("user_a")` returns 5000
- `get_user_lifetime_used("user_b")` returns 3000

