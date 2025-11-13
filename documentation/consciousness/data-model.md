# Consciousness Data Model

**Status:** Specification v1.0  
**Date:** 2025-11-01  
**Purpose:** Define the structure, versioning, and query interface for consciousness items

---

## 1. Overview

Consciousness is a **daemon** that provides a versioned, immutable (at runtime) knowledge base storing SI's self-concept. It implements the Daemon Interface and is accessed via the `query()` method.

Consciousness consists of four types of items:
- **Mandates**: What SI believes and values
- **Capabilities**: What SI can do
- **Modes**: Operational states/contexts
- **Governance Rules**: Constraints and change controls

Each item is versioned, requires approval before deployment, and can be queried by other daemons and Working Memory through the consciousness daemon's `query()` interface.

---

## 2. Consciousness Item Data Model

### 2.1 Core Fields (All Items)

Every consciousness item has:

```
{
  id: string                    # Unique identifier (e.g., "mandate_user_autonomy_v1")
  type: enum                    # "mandate" | "capability" | "mode" | "governance_rule"
  version: string               # Semantic version (e.g., "1.0.0", "1.1.0")
  name: string                  # Human-readable name
  description: string           # What this item represents
  content: string               # The actual consciousness text/specification
  
  # Metadata
  tags: string[]                # For categorization/querying (e.g., ["user-autonomy", "core"])
  approval_status: enum         # "draft" | "approved" | "deprecated"
  effective_date: ISO8601       # When this version becomes active
  sunset_date: ISO8601 | null   # When this version expires (null = no expiration)
  priority: integer             # 1-10 (higher = more important if conflicts arise)
  
  # Audit Trail
  created_by: string            # User/system that created this
  created_at: ISO8601           # Creation timestamp
  approved_by: string | null    # User that approved (null if not approved)
  approved_at: ISO8601 | null   # Approval timestamp
  change_rationale: string      # Why this version was created
}
```

### 2.2 Item-Specific Fields

**Mandates:**
- `applies_to: string[]` - Which modes/contexts this mandate applies to (empty = all)
- `enforcement_level: enum` - "hard" (must follow) | "soft" (should follow)

**Capabilities:**
- `requires_approval: boolean` - Whether using this capability requires approval
- `risk_level: enum` - "low" | "medium" | "high"

**Modes:**
- `parent_mode: string | null` - If this mode inherits from another
- `activation_criteria: string` - When/how this mode is activated

**Governance Rules:**
- `applies_to: string[]` - Which item types this rule constrains
- `enforcement_mechanism: string` - How the rule is enforced

---

## 3. Query Interface

Consciousness is accessed through the Daemon Interface `query()` method. Daemons and Working Memory query consciousness using:

```
consciousness.query(
  prompt: string,               # Natural language query (e.g., "What mandates apply to my current mode?")
  metadata: object              # Context (e.g., {current_mode: "code_review", task_type: "implementation"})
) → Future<ConsciousnessResult>
```

**Returns:** Future that resolves to ConsciousnessResult containing:
- `items: ConsciousnessItem[]` - Array of matching consciousness items, sorted by priority (highest first)
- `rationale: string` - Explanation of why these items were selected
- `effective_at: ISO8601` - Timestamp when these items become effective

**Note:** The consciousness daemon internally handles filtering by approval_status (only approved items), effective_date, and other constraints.

---

## 4. Selection Logic (v1 - Initial Implementation)

When a daemon or Working Memory needs consciousness:

1. **Always include:**
   - Latest approved version of item with `id = "core_si_self"` (type: mandate)
   - Latest approved version of current mode (type: mode)

2. **Optional (future enhancement):**
   - Query for additional items based on context/task type
   - Apply relevance scoring
   - Respect budget constraints

**v1 Implementation:**
```
consciousness_for_turn = [
  query_consciousness("mandate", ["core_si_self"])[0],
  query_consciousness("mode", [current_mode_name])[0]
]
```

---

## 5. Versioning Strategy

- **Semantic Versioning:** MAJOR.MINOR.PATCH
  - MAJOR: Breaking changes to meaning/intent
  - MINOR: Clarifications, additions
  - PATCH: Typos, formatting

- **Active Version:** System always uses the latest approved version with `effective_date <= now`

- **Deprecation:** Set `approval_status = "deprecated"` and `sunset_date` to retire old versions

---

## 6. Change Management

All consciousness changes require:
1. Create new version (increment version number)
2. Set `approval_status = "draft"`
3. Document `change_rationale`
4. Submit for approval (external process)
5. Once approved: set `approval_status = "approved"` and `approved_at = now`
6. Changes take effect at next turn (no runtime modifications)

---

## 7. Storage Requirements

The system must support:
- Querying by `id`, `type`, `tags`, `approval_status`
- Filtering by `effective_date` and `sunset_date`
- Retrieving full history of an item (all versions)
- Audit trail (who approved, when, why)

---

## 8. Test Conditions

See `test-conditions.md` for comprehensive test requirements.

