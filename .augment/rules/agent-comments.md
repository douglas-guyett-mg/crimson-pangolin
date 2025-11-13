---
type: "always_apply"
---

# Agent Comments Directory

## Purpose

The `agent-comments/` directory is a local development workspace for agent-generated summaries, analysis notes, and change records. These files are **NOT part of the official documentation** and are excluded from version control via `.gitignore`.

## When to Create Files in agent-comments/

Create files in `agent-comments/` when you:
- Generate a summary of changes you've made
- Create analysis or planning documents during development
- Document your reasoning or decision-making process
- Create temporary notes or working documents
- Generate reports on work completed

## When NOT to Create Files in agent-comments/

Do NOT create files in `agent-comments/` when:
- The content should be part of official documentation (use `documentation/` instead)
- The content is a specification or design document (use `documentation/` or `agent-plans/` instead)
- The content is a test file (use appropriate test directory)
- The content is implementation code (use appropriate source directory)

## File Naming Convention

Use descriptive names with timestamps or context:
- `summary-issue-3-integration-guide-2025-11-05.md`
- `analysis-consciousness-integration-2025-11-05.md`
- `changes-made-turn-architecture-2025-11-05.md`

## Examples of Appropriate Content

✅ **DO create in agent-comments/:**
- "Summary of changes made to 11 integration documents"
- "Analysis of test scenario improvements"
- "Record of files archived from documentation-questions/"
- "Notes on decision-making for documentation structure"
- "Completion report for Issue 3"

❌ **DO NOT create in agent-comments/:**
- Official documentation files (use `documentation/`)
- Test specifications (use `documentation/` with test-conditions.md)
- Implementation guides (use `documentation/`)
- Code files (use appropriate source directory)

## Version Control

- Files in `agent-comments/` are **NOT tracked by git**
- They are local development artifacts only
- They provide a record for the developer but don't clutter the repository
- Safe to delete without affecting the project

## Integration with Task Management

When completing complex tasks, consider creating a summary file in `agent-comments/` that documents:
- What was completed
- How many files were modified
- Key decisions made
- Related files and their changes
- Next steps or follow-up items

This creates a useful record while keeping the repository clean.

