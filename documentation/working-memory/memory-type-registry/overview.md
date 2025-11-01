# MemoryType Registry

## Overview

The MemoryType Registry is the foundational component that enables the working memory system to support pluggable, extensible memory types without requiring code modifications. It implements a registry pattern that allows memory types to be registered, discovered, and instantiated dynamically at runtime.

## Purpose

The registry serves three critical functions:

1. **Registration**: Allow memory type implementations to register themselves with the system
2. **Discovery**: Enable the system to discover available memory types and their capabilities
3. **Factory**: Provide a factory mechanism to instantiate memory types on demand

## Key Concepts

### Memory Type
A memory type is a named, versioned implementation of the MemoryPlugin interface. Examples include:
- Working Memory (bounded queue for immediate context)
- Episodic Memory (append-only log for experiences)
- Semantic Memory (upsert-based store for facts and patterns)
- Conversation History (rolling buffer of utterances)

### Registry
The registry maintains a mapping of memory type names to their implementations. It supports:
- Adding new memory types at startup or runtime
- Removing memory types without affecting other components
- Querying available types and their metadata
- Creating instances of registered types

### Factory Pattern
The registry uses a factory pattern to instantiate memory types. When a component needs a memory type, it requests an instance from the registry rather than directly instantiating the type. This decouples consumers from implementations.

## Design Principles

1. **No Hard-Coded Types**: Memory types are not hard-coded into the system. All types are registered dynamically.
2. **Uniform Interface**: All memory types implement the MemoryPlugin interface, allowing the orchestrator to treat them uniformly.
3. **Metadata-Driven**: Each registered type includes metadata (name, version, capabilities) that describes its behavior.
4. **Fail-Safe**: If a memory type is not registered, the system gracefully handles the error rather than crashing.

## Interaction with Other Components

- **Memory Orchestrator**: Queries the registry to discover available memory types and instantiate them
- **Memory Plugins**: Register themselves with the registry at startup
- **Configuration**: May be initialized with a set of memory types to register

## Success Criteria

- Registry can add memory types without code changes
- Registry can remove memory types without affecting other registered types
- Factory creates correct instances of registered types
- Registry provides accurate metadata about registered types
- System handles attempts to instantiate unregistered types gracefully

