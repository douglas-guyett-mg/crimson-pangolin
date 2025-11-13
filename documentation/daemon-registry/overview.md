# Daemon Registry

**Status:** Specification v1.0  
**Date:** 2025-11-06  
**Purpose:** Provide a centralized registry for daemon type discovery and daemon instance lifecycle management

---

## Overview

The Daemon Registry is an **informational system** that maintains two separate registries:

1. **Daemon Type Registry** - Catalog of available daemon types (fixed, code-defined)
2. **Daemon Instance Registry** - Runtime instances of daemons with their status and relationships

The registry enables:
- **Discovery**: Find available daemon types and running daemon instances
- **Lifecycle Management**: Track daemon creation, status, and shutdown
- **Access Control**: Enforce turn scoping and executor hierarchy rules
- **Monitoring**: Track daemon health and resource usage

**Key Principle**: The registry is **informational only**. The Executor daemon is responsible for deciding which daemons to create, spawning them, and managing their lifecycle. The registry stores and provides information about those decisions.

---

## Daemon Type Registry

A static catalog of daemon types available in the system. Defined in code and updated only when new daemon types are added to the system.

### Data Model

```
DaemonType {
  daemon_type_id: string          # Unique identifier (e.g., "Router", "FC", "Working Memory")
  description: string             # What this daemon does
  version: string                 # Version (e.g., "1.0")
}
```

### Available Daemon Types

- **Router** - Analyzes incoming input and decides urgency level
- **Executor** - Orchestrates turn execution and spawns sub-turns
- **Frontal Cortex (FC)** - Builds execution plans
- **Working Memory** - Manages context assembly and memory operations
- **Consciousness** - Provides Si's self-concept, mandates, and governance rules
- **Episodic Memory** - Stores and retrieves turn events and observations
- **Semantic Memory** - Stores and retrieves distilled facts and skills
- **Pattern Recognition** - Identifies patterns in observations
- **Reflector** - Learns from turn outcomes and updates knowledge
- **Responder** - Generates user-facing messages
- **Blank Agent** - Flexible daemon that Executor can write custom prompts for

---

## Daemon Instance Registry

Runtime registry of daemon instances that have been created. Each instance tracks its status, resource usage, and relationships.

### Data Model

```
DaemonInstance {
  daemon_id: string               # Unique identifier for this instance
  daemon_type: string             # Which type (from Daemon Type Registry)
  turn_id: string                 # Which turn this daemon belongs to
  status: string                  # "running" | "idle" | "error" | "shutdown"
  created_at: timestamp           # When this instance was created
  last_heartbeat: timestamp       # Last time daemon reported health
  resource_usage: {
    tokens: number                # Tokens consumed
    memory_mb: number             # Memory used in MB
  }
  error_message: string           # Error details (if status="error")
  reference: object               # How to call this daemon (implementation-specific)
}
```

### Executor Relationships

For Executor daemon instances, the registry also tracks recruitment relationships:

```
ExecutorRelationship {
  executor_id: string             # The executor
  boss_executor_id: string        # Who recruited this executor (null if root)
  recruited_executor_ids: string[]  # Executors this one recruited
  level: number                   # Hierarchy level (1 = root, 2 = recruited by root, etc.)
}
```

---

## Turn Scoping Rules

The registry enforces strict turn scoping for daemon communication:

### Rule 1: Turn Isolation
- Daemons can only communicate with other daemons in the same turn
- A daemon in Turn A cannot directly query a daemon in Turn B

### Rule 2: Executor Exception
- Only Executor daemons can communicate across turns
- Non-Executor daemons cannot cross turn boundaries

### Rule 3: Executor Hierarchy
- Executors form a hierarchical tree based on recruitment relationships
- An Executor can communicate with:
  - Its boss (the Executor that recruited it)
  - Its siblings (other Executors recruited by the same boss)
  - Its recruits (Executors it recruited)
- An Executor CANNOT communicate with:
  - Unrelated Executors in other branches
  - Executors at levels it has no direct relationship with

---

## Registry Operations

### Daemon Type Queries

**list_daemon_types() → DaemonType[]**
- Returns all available daemon types
- Used by Executor to understand what daemons can be created

**get_daemon_type(daemon_type_id) → DaemonType**
- Returns details about a specific daemon type
- Used by Executor to get description and version

### Daemon Instance Management

**register_daemon_instance(daemon_id, daemon_type, turn_id, status, created_at, reference) → void**
- Called by Executor when creating a new daemon instance
- Stores instance information in registry

**update_daemon_status(daemon_id, status, last_heartbeat, resource_usage, error_message) → void**
- Called by daemon or monitoring system to update status
- Tracks health, resource usage, and errors

**deregister_daemon_instance(daemon_id) → void**
- Called by Executor when daemon shuts down
- Removes instance from registry

### Daemon Instance Queries

**list_daemons_in_turn(turn_id) → DaemonInstance[]**
- Returns all daemons in a specific turn
- Used for monitoring and debugging

**get_daemon_instance(daemon_id) → DaemonInstance**
- Returns details about a specific daemon instance
- Used to get reference for calling the daemon

**can_communicate(daemon_id_1, daemon_id_2) → boolean**
- Checks if two daemons can communicate
- Enforces turn scoping and executor hierarchy rules

### Executor Relationships

**register_executor_relationship(boss_executor_id, recruited_executor_id) → void**
- Called when Executor recruits a new Executor
- Tracks the recruitment relationship

**list_communicable_executors(executor_id) → string[]**
- Returns list of executor IDs this executor can communicate with
- Includes boss, siblings, and recruits

---

## Integration Points

- **Executor Daemon**: Queries registry to understand available daemon types, registers instances when creating daemons, updates status during execution
- **Monitoring Systems**: Query registry to track daemon health and resource usage
- **Debugging Tools**: Query registry to understand daemon relationships and turn structure
- **Turn Trace**: May log registry operations for audit trail

