---
type: "always_apply"
---

# Documentation Writing Guidelines for Si

## Purpose
Ensure all Si documentation is consistent, comprehensive, and aligns with the project's core engineering tenants defined in `documentation/si_core_engineering_tennants.md`: documentation-as-code, tests-first, modularity, and technology-agnosticism.

## Workflow

### 1. Review Existing Documentation
Before writing new documentation:
- Search the `documentation/` directory for existing content on the topic
- Identify any related specifications, architecture documents, or implementation guides
- Flag any conflicts, inconsistencies, or poorly-written sections as questions to resolve before proceeding

### 2. Create a Documentation Plan
- First if there is no documentation plan for this part of the documentation then create and save a step-by-step plan in `documentation-plans/` (use descriptive filename, e.g., `plan-{topic}.md`)
- Include: A checklist on what what documentation needs to be written so that we can check that off as we progress
- This plan can change as we actually write the documenation and clear up steps
- This plan becomes the blueprint for writing the documentation
- You will use this plan 
- Note: The actual documentation will be saved in the `documentation/` folder

### 3. Research Best Practices
- Review how similar concepts are documented in existing Si documentation
- Ensure alignment with Si's core tenants (especially documentation-as-code and technology-agnosticism)
- Identify any patterns or conventions already established in the project

### 4. Ask Clarifying Questions
- Ask one question at a time before implementation
- Focus on: scope boundaries, audience assumptions, integration points with existing systems, and test requirements
- Wait for answers before proceeding to implementation

### 5. Implement Documentation
- Save all documentation files in the `documentation/` folder
- Write in natural language only—no code snippets or implementation details
- Focus on "what" and "why," not "how to code it"
- Ensure the documentation is detailed enough to rebuild the system from it using an agentic process
- Include explicit test requirements and verification criteria for each section

## Documentation Structure for Testable Components

Every testable part of the system must be organized in its own directory or subdirectory with the following structure:

```
documentation/
  {component-name}/
    overview.md          # What it does, why it exists, key concepts
    test-conditions.md   # Detailed test requirements and verification criteria
    integration.md       # (optional) How it connects to other components
```

For systems with multiple sub-components:

```
documentation/
  {system-name}/
    overview.md
    {sub-component-1}/
      overview.md
      test-conditions.md
    {sub-component-2}/
      overview.md
      test-conditions.md
```

**Requirement**: Every testable component MUST have both an `overview.md` and `test-conditions.md` file. No exceptions.

## Quality Assurance and Critical Review

The agent should be critical and rigorous when writing or reviewing documentation:

- **Challenge vague language**: If a description could be interpreted multiple ways, flag it and ask for clarification
- **Verify completeness**: Ensure test conditions cover all major behaviors, edge cases, and failure modes
- **Check consistency**: Verify terminology, concepts, and patterns match existing documentation
- **Validate testability**: Confirm that test conditions are actually measurable and verifiable—not aspirational
- **Question assumptions**: If documentation makes implicit assumptions about the reader's knowledge, make them explicit or simplify
- **Ensure modularity**: Verify that component boundaries are clear and that integration points are well-defined
- **Assess alignment**: Confirm that the documentation aligns with Si's core engineering tenants and project goals

When in doubt, ask clarifying questions rather than making assumptions. Better to spend time upfront getting it right than to have ambiguous documentation that leads to misaligned implementations.

## Key Principles
- **Documentation is code**: treat it as the authoritative specification
- **Technology-agnostic**: avoid language or vendor-specific details unless necessary
- **Tests-first**: every section should include clear test requirements
- **Modular**: structure documentation to reflect the modular nature of Si
- **Consistency**: match existing documentation style, terminology, and structure

## Minimal File Creation
- Unless specifically asked, do not create documentation or files
