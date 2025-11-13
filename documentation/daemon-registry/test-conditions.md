# Daemon Registry - Test Conditions

**Status:** Specification v1.0  
**Date:** 2025-11-06

---

## Test Suite: Daemon Type Registry

### Test: List All Daemon Types
- **Given:** Daemon Type Registry is initialized
- **When:** `list_daemon_types()` is called
- **Then:**
  - Returns list of all available daemon types
  - Includes at least: Router, Executor, FC, Working Memory, Consciousness, Episodic Memory, Semantic Memory, Pattern Recognition, Reflector, Responder, Blank Agent
  - Each type has daemon_type_id, description, version

### Test: Get Specific Daemon Type
- **Given:** Daemon Type Registry contains "Router" type
- **When:** `get_daemon_type("Router")` is called
- **Then:**
  - Returns DaemonType with daemon_type_id="Router"
  - description is non-empty
  - version is non-empty

### Test: Daemon Type Not Found
- **Given:** Daemon Type Registry is initialized
- **When:** `get_daemon_type("NonExistentType")` is called
- **Then:**
  - Returns error or null (implementation-specific)

---

## Test Suite: Daemon Instance Registration

### Test: Register Daemon Instance
- **Given:** Executor wants to create a new daemon
- **When:** `register_daemon_instance(daemon_id, daemon_type, turn_id, status, created_at, reference)` is called
- **Then:**
  - Instance is stored in registry
  - Instance can be retrieved with `get_daemon_instance(daemon_id)`
  - Instance has all provided fields

### Test: Register Multiple Instances in Same Turn
- **Given:** Turn 1 has multiple daemons
- **When:** Multiple instances are registered with same turn_id
- **Then:**
  - All instances are stored
  - `list_daemons_in_turn(turn_id)` returns all of them

### Test: Register Instances in Different Turns
- **Given:** Two turns are executing in parallel
- **When:** Instances are registered with different turn_ids
- **Then:**
  - Instances are stored separately
  - `list_daemons_in_turn(turn_1)` returns only turn 1 instances
  - `list_daemons_in_turn(turn_2)` returns only turn 2 instances

---

## Test Suite: Daemon Instance Lifecycle

### Test: Update Daemon Status
- **Given:** Daemon instance is registered with status="running"
- **When:** `update_daemon_status(daemon_id, "idle", last_heartbeat, resource_usage, null)` is called
- **Then:**
  - Instance status is updated to "idle"
  - last_heartbeat is updated
  - resource_usage is updated
  - error_message is cleared

### Test: Update Daemon Error Status
- **Given:** Daemon instance is registered with status="running"
- **When:** `update_daemon_status(daemon_id, "error", last_heartbeat, resource_usage, "Connection timeout")` is called
- **Then:**
  - Instance status is updated to "error"
  - error_message is set to "Connection timeout"
  - last_heartbeat is updated

### Test: Deregister Daemon Instance
- **Given:** Daemon instance is registered
- **When:** `deregister_daemon_instance(daemon_id)` is called
- **Then:**
  - Instance is removed from registry
  - `get_daemon_instance(daemon_id)` returns error or null
  - Instance no longer appears in `list_daemons_in_turn(turn_id)`

---

## Test Suite: Turn Scoping - Non-Executor Daemons

### Test: Non-Executor Daemons Cannot Cross Turns
- **Given:** FC daemon in Turn 1 and Working Memory daemon in Turn 2
- **When:** `can_communicate(fc_daemon_id, wm_daemon_id)` is called
- **Then:**
  - Returns false (communication not allowed)

### Test: Non-Executor Daemons Can Communicate Within Turn
- **Given:** FC daemon and Working Memory daemon in same Turn 1
- **When:** `can_communicate(fc_daemon_id, wm_daemon_id)` is called
- **Then:**
  - Returns true (communication allowed)

### Test: Non-Executor Cannot Query Executor in Different Turn
- **Given:** FC daemon in Turn 1 and Executor daemon in Turn 2
- **When:** `can_communicate(fc_daemon_id, executor_daemon_id)` is called
- **Then:**
  - Returns false (non-executor cannot cross turns)

---

## Test Suite: Executor Hierarchy and Cross-Turn Communication

### Test: Root Executor Can Communicate With Recruited Executors
- **Given:** Root Executor (Level 1) recruits Executor A and B (Level 2)
- **When:** `can_communicate(root_executor_id, executor_a_id)` is called
- **Then:**
  - Returns true (root can communicate with recruits)

### Test: Sibling Executors Can Communicate
- **Given:** Root Executor recruits Executor A and B (both Level 2)
- **When:** `can_communicate(executor_a_id, executor_b_id)` is called
- **Then:**
  - Returns true (siblings can communicate)

### Test: Recruited Executor Can Communicate With Boss
- **Given:** Root Executor recruits Executor A (Level 2)
- **When:** `can_communicate(executor_a_id, root_executor_id)` is called
- **Then:**
  - Returns true (recruit can communicate with boss)

### Test: Executor Can Communicate With Its Recruits
- **Given:** Executor A (Level 2) recruits Executor C (Level 3)
- **When:** `can_communicate(executor_a_id, executor_c_id)` is called
- **Then:**
  - Returns true (executor can communicate with recruits)

### Test: Unrelated Executors Cannot Communicate
- **Given:** Root recruits A and B (Level 2). A recruits C, B recruits D (Level 3)
- **When:** `can_communicate(executor_c_id, executor_d_id)` is called
- **Then:**
  - Returns false (no direct relationship)

### Test: Executor Cannot Skip Levels
- **Given:** Root recruits A (Level 2). A recruits C (Level 3). C recruits E (Level 4)
- **When:** `can_communicate(root_executor_id, executor_c_id)` is called
- **Then:**
  - Returns false (no direct relationship, skips level)

---

## Test Suite: Executor Relationships

### Test: Register Executor Recruitment
- **Given:** Root Executor creates sub-turns
- **When:** `register_executor_relationship(root_executor_id, executor_a_id)` is called
- **Then:**
  - Relationship is stored
  - executor_a has boss_executor_id = root_executor_id
  - root_executor has recruited_executor_ids including executor_a_id

### Test: List Communicable Executors
- **Given:** Executor A (Level 2) with boss Root, siblings B, and recruits C, D
- **When:** `list_communicable_executors(executor_a_id)` is called
- **Then:**
  - Returns [root_executor_id, executor_b_id, executor_c_id, executor_d_id]
  - Does not include unrelated executors

---

## Test Suite: Integration

### Test: Full Executor Hierarchy Scenario
- **Given:** Root Executor spawns Turn 1 and Turn 2
- **When:** 
  - Executor A (Turn 1) recruits Executor C (Turn 3)
  - Executor B (Turn 2) recruits Executor D (Turn 4)
- **Then:**
  - A and B can communicate (siblings)
  - A and C can communicate (boss-recruit)
  - C and D cannot communicate (unrelated)
  - FC in Turn 1 cannot communicate with FC in Turn 3 (non-executor)
  - Executor A can communicate with Executor C (executor hierarchy)

### Test: Monitoring System Can Query Registry
- **Given:** Multiple daemons running across multiple turns
- **When:** Monitoring system calls `list_daemons_in_turn(turn_id)` for each turn
- **Then:**
  - Can see all daemons in each turn
  - Can see status, resource_usage, last_heartbeat for each
  - Can identify unhealthy daemons (status="error")

### Test: Executor Can Discover Available Daemon Types
- **Given:** Executor needs to decide which daemons to create
- **When:** Executor calls `list_daemon_types()`
- **Then:**
  - Gets list of all available types
  - Can use descriptions to decide which to create
  - Can create instances of any type

