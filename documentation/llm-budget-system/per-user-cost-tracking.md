# Per-User Cost Tracking

**Status**: Specification v1.0  
**Last Updated**: 2025-11-04  
**Alignment**: Si Core Engineering Tenants (documentation-as-code, tests-first, modularity, technology-agnosticism)

## Overview

The Per-User Cost Tracking system monitors token consumption across all LLM calls for each user. It tracks both lifetime usage and usage within configurable periods, and enforces hard limits by stopping LLM calls when users exceed their budgets.

## Purpose

The Per-User Cost Tracking system serves to:

1. **Track Lifetime Usage**: Monitor total tokens consumed by each user across all time
2. **Track Period Usage**: Monitor tokens consumed within configurable time periods
3. **Enforce Hard Limits**: Stop LLM calls when users exceed their budget
4. **Manage Costs**: Enable cost management and billing based on actual usage
5. **Provide Visibility**: Log all cost tracking decisions for auditing

## Scope

**In Scope**: Per-user token tracking, lifetime and period budgets, cost enforcement, usage logging

**Out of Scope**: Billing implementation, payment processing, cost allocation across teams

## Key Concepts

### Lifetime Budget
Total tokens a user is allowed to consume across their entire lifetime using the system.

**Example**: User has lifetime budget of 1,000,000 tokens

### Period Budget
Tokens a user is allowed to consume within a specific time period. The period duration is configurable based on the user's plan.

**Example**: User has monthly budget of 100,000 tokens (period = 1 month)

### Period Duration
The length of time for which a period budget applies. This is configurable per user based on their plan.

**Example**:
- Free plan: 1 day period
- Pro plan: 1 month period
- Enterprise plan: 1 quarter period

### Cost Enforcement
When a user exceeds either their lifetime or period budget, all LLM calls are stopped and rejected.

## Architecture

### Cost Tracking Data Model

For each user, track:

```
user_cost_tracking:
  user_id: string
  lifetime_tokens_used: integer
  lifetime_budget: integer
  
  current_period:
    period_start: ISO8601 timestamp
    period_duration: duration (e.g., "1 month")
    period_tokens_used: integer
    period_budget: integer
  
  plan_id: string  # Determines period duration
  
  cost_tracking_enabled: boolean
  enforcement_enabled: boolean
```

### Cost Tracking Flow

1. **LLM call completes**: Daemon receives token count from LLM provider
2. **Update tracking**: Add tokens to user's lifetime and period usage
3. **Check limits**:
   - If lifetime_tokens_used >= lifetime_budget → STOP all LLM calls
   - If period_tokens_used >= period_budget → STOP all LLM calls
4. **Log decision**: Record in Turn Trace for auditing
5. **Notify user**: If limit exceeded, inform user of status

### Period Rollover

When a period ends:

1. **Check period end time**: If current_time >= period_start + period_duration
2. **Archive period**: Save completed period data for historical analysis
3. **Reset period**: Create new period with:
   - period_start = current_time
   - period_tokens_used = 0
4. **Preserve lifetime**: lifetime_tokens_used continues accumulating

## Design Principles

1. **Dual Tracking**: Both lifetime and period budgets provide flexibility
2. **Configurable Periods**: Period duration matches user's plan
3. **Hard Enforcement**: Reject LLM calls, don't silently degrade
4. **Auditable**: All tracking decisions logged
5. **Technology-Agnostic**: Works with any LLM provider

## Interaction with Other Systems

- **LLM Budget System**: Reports token usage from each LLM call
- **Daemons**: Receive rejection when user budget exceeded
- **Turn Trace**: All cost tracking decisions logged
- **Error Handler**: Handles LLM call rejections due to cost limits
- **User Management**: Provides user plan information for period duration

## Configuration

The Per-User Cost Tracking system can be configured with:

- **tracking_enabled**: Whether to track per-user costs (default: true)
- **enforcement_enabled**: Whether to enforce hard limits (default: true)
- **default_lifetime_budget**: Default lifetime tokens per user (default: 1,000,000)
- **plan_period_mapping**: Maps plan_id to period duration and budget
- **log_all_tracking**: Whether to log every tracking decision (default: true)

## Example Scenario

1. User "alice" has Pro plan: lifetime_budget=1M, period_budget=100k, period=1 month
2. Turn 1: LLM call uses 5000 tokens → lifetime: 5k, period: 5k → Within budget
3. Turn 2: LLM call uses 3000 tokens → lifetime: 8k, period: 8k → Within budget
4. ... (many turns later)
5. Period end: 30 days pass → Reset period, lifetime still 98k
6. Turn N: LLM call would use 5000 tokens → period would be 105k → EXCEEDS period budget
7. LLM call rejected, user notified: "Monthly budget exceeded"
8. Lifetime tracking continues for next period

