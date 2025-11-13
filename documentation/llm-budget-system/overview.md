# LLM Budget System

**Status**: Specification v1.0  
**Last Updated**: 2025-11-04  
**Alignment**: Si Core Engineering Tenants (documentation-as-code, tests-first, modularity, technology-agnosticism)

## Overview

The LLM Budget System controls token usage when making calls to Large Language Models. It enforces limits on prompt size (including system instructions, context, and user input) to manage costs and latency. The system operates at two levels:

1. **Global LLM Budget**: A system-wide default limit for all LLM calls
2. **Daemon-Specific Budgets**: Per-daemon overrides that can increase or decrease the global limit

## Purpose

The LLM Budget System serves to:

1. **Control LLM Call Costs**: Limit tokens sent to LLMs to manage per-user and system costs
2. **Enforce Global Defaults**: Provide a configurable system-wide default limit
3. **Enable Daemon Customization**: Allow daemons to override limits based on their specific needs
4. **Prevent Runaway Costs**: Stop LLM calls when budgets are exceeded
5. **Track Usage**: Monitor token consumption per daemon and per user

## Scope

**In Scope**: LLM call budgets, prompt size limits, daemon-specific overrides, budget enforcement, cost tracking

**Out of Scope**: Storage limits (handled separately), tool execution budgets, time budgets, memory store capacity

## Key Concepts

### Global LLM Budget
A system-wide default limit (in tokens) for the maximum prompt size when calling an LLM. This applies to all daemons unless overridden.

**Example**: Global limit = 4000 tokens

### Daemon-Specific Budget
A per-daemon override that can increase or decrease the global limit. Each daemon that makes LLM calls can have its own budget.

**Example**:
- Router: 2000 tokens (lower, quick decisions)
- Plan Generator: 6000 tokens (higher, complex planning)
- Executor: 3000 tokens (medium, tool execution)

### Prompt Simplification
When a daemon's prompt exceeds its budget, the daemon applies its own simplification strategy to reduce the prompt size while preserving essential information.

## Architecture

### Budget Configuration

Budgets are configured at system initialization:

```
global_llm_budget: 4000  # Default for all daemons

daemon_budgets:
  router: 2000           # Override: lower limit
  plan_generator: 6000   # Override: higher limit
  executor: 3000         # Override: medium limit
  # Other daemons use global_llm_budget if not specified
```

### Budget Enforcement Flow

1. **Daemon prepares LLM call**: Assembles system instructions, context, and user input
2. **Calculate prompt size**: Count tokens in the assembled prompt
3. **Check budget**: Compare against daemon-specific budget (or global if not specified)
4. **If over budget**:
   - Daemon applies its simplification strategy
   - Recalculate prompt size
   - If still over budget, reject the LLM call
5. **If within budget**: Proceed with LLM call
6. **Track usage**: Log tokens used for per-user cost tracking

## Design Principles

1. **Global Default with Local Override**: Simple default, flexible customization
2. **Daemon-Specific Simplification**: Each daemon knows best how to simplify its prompts
3. **Fail-Safe**: Reject LLM calls rather than silently truncate
4. **Auditable**: All budget decisions logged for debugging and learning
5. **Technology-Agnostic**: Works with any LLM provider

## Interaction with Other Systems

- **Per-User Cost Tracking**: LLM Budget System reports token usage to cost tracking system
- **Daemons**: Each daemon implements its own prompt simplification strategy
- **Turn Trace**: All budget decisions logged for auditability
- **Error Handler**: Handles LLM call rejections due to budget exceeded

## Configuration

The LLM Budget System can be configured with:

- **global_llm_budget**: Default token limit for all LLM calls (default: 4000)
- **daemon_budgets**: Per-daemon overrides (optional)
- **enforce_hard_limit**: Whether to reject calls over budget (default: true)
- **log_budget_decisions**: Whether to log all budget checks (default: true)

## Example Scenario

1. System initializes with global_llm_budget=4000, router budget=2000
2. Router daemon prepares LLM call with 1800 tokens → Within budget → Proceeds
3. Plan Generator daemon prepares LLM call with 5500 tokens → Over global budget (4000) → Applies simplification
4. After simplification: 3800 tokens → Within budget → Proceeds
5. Executor daemon prepares LLM call with 4200 tokens → Over global budget (4000) → Applies simplification
6. After simplification: 4100 tokens → Still over budget → Rejects call, logs error

