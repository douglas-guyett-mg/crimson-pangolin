# File Creation Discipline

## Core Principle
**Do not create files unless explicitly requested.** This rule prevents unnecessary clutter, scope creep, and unsolicited documentation that the user did not ask for.

## When NOT to Create Files

### Documentation Files (*.md)
- **NEVER** create README files proactively
- **NEVER** create summary documents unless explicitly requested
- **NEVER** create overview files, guides, or tutorials unless the user asks for them
- **NEVER** create documentation plans unless the user specifically says "create a plan"
- **NEVER** create implementation guides or how-to documents unless requested

### Code Files
- **NEVER** create new test files unless explicitly requested
- **NEVER** create utility files "for future use"
- **NEVER** create helper modules or scaffolding code that wasn't asked for
- **NEVER** create configuration files unless necessary to complete the requested task

### Other Files
- **NEVER** create `.env` files, `.gitignore` updates, or other configuration unless necessary
- **NEVER** create changelog entries or version files unless requested
- **NEVER** create example files or template files unless asked

## When TO Create Files

### Explicit Requests
- User says: "Create a plan for..."
- User says: "Write documentation for..."
- User says: "Create a new file..."
- User says: "Add a test file..."
- User says: "Generate a config file..."

### Necessary for Task Completion
- A file is **absolutely required** to complete the requested task
- Example: User asks to "add a new feature" and you need to create the feature file itself
- Example: User asks to "set up a new service" and configuration files are essential

### Clarification First
When uncertain whether a file is necessary:
1. Ask the user before creating it
2. Explain why you think it's needed
3. Wait for confirmation

## Editing vs. Creating

**Always prefer editing existing files to creating new ones.**

- If a file already exists that could contain the content, edit it
- If documentation already exists on a topic, update it rather than creating a new file
- If a configuration file exists, modify it rather than creating a new one

## The "Scope Creep" Test

Before creating any file, ask yourself:
- Did the user explicitly ask for this file?
- Is this file absolutely necessary to complete what the user asked?
- Could this content go in an existing file instead?

If you answer "no" to the first two questions, **do not create the file**.

## Examples

### ❌ DO NOT DO THIS
- User: "Help me fix this bug"
  - You create: `bug-fix-summary.md`, `implementation-notes.md`, `test-plan.md`
  - **Wrong**: User didn't ask for documentation

- User: "Add a new API endpoint"
  - You create: `API_GUIDE.md`, `ENDPOINT_DOCUMENTATION.md`, `examples/`
  - **Wrong**: User didn't ask for documentation

- User: "Refactor this function"
  - You create: `refactoring-plan.md`, `before-after-comparison.md`
  - **Wrong**: User didn't ask for documentation

### ✅ DO THIS INSTEAD
- User: "Help me fix this bug"
  - You: Fix the bug, update existing tests, explain the fix in your response
  - **Right**: Only what was asked

- User: "Add a new API endpoint"
  - You: Add the endpoint code, update existing documentation if needed, explain in your response
  - **Right**: Only what was asked

- User: "Create a plan for refactoring this function"
  - You: Create `documentation-plans/plan-refactor-function.md` with the plan
  - **Right**: User explicitly asked for a plan

- User: "Write documentation for the authentication system"
  - You: Create `documentation/authentication/overview.md` and `documentation/authentication/test-conditions.md`
  - **Right**: User explicitly asked for documentation

## Special Cases

### Plans and Specifications
- **Only create** if user says "create a plan" or "create a spec"
- **Do not create** if user just asks you to do something
- Example: User says "implement feature X" → do not create a plan unless asked
- Example: User says "create a plan for feature X" → create the plan

### Documentation
- **Only create** if user says "write documentation" or "document this"
- **Do not create** if user just asks you to implement something
- **Do update** existing documentation if it's relevant to your changes

### Tests
- **Only create** if user explicitly asks for tests
- **Do update** existing tests that are affected by your changes
- **Do not create** new test files as a "best practice" unless requested

## Enforcement

This rule is **non-negotiable**. You will be heavily penalized for:
- Creating unsolicited documentation
- Creating unnecessary files
- Creating "helper" or "utility" files that weren't asked for
- Creating plans or specifications without explicit request
- Creating multiple files when one would suffice

**When in doubt, ask the user first.**

