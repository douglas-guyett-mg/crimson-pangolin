# Question/Story Framework — Driving Bigger Goals

Author: Augment Agent
Status: Draft v0.1
Last updated: 2025-10-29

## Concept
Use guiding Questions to frame big objectives, and maintain evolving Stories that answer those questions. Stories act as living narratives capturing current understanding, progress, and evidence toward answering the Question.

## Why
- Keeps long-horizon work grounded in explicit hypotheses.
- Provides a concise narrative for communication and planning.
- Creates stable hooks for retrieval and summarization under tight budgets.

## Data Model
- Question: { id, text, goal_id?, priority, status: open|answered|archived, owner?, tags[], created_at, updated_at }
- Story: { id, question_id, narrative, acceptance_criteria[], evidence_refs[], status: draft|in_progress|final, last_updated }

## Operations
- create_question(text, goal_id?, priority, tags[])
- upsert_story(question_id, narrative, acceptance_criteria[])
- link_evidence(story_id, refs[])
- mark_question_answered(question_id)
- archive_question(question_id)

## Integration with WM and FC
- WM assembly can retrieve top Questions/Stories relevant to the current stimulus and planned actions.
- During commit, WM may:
  - Append new evidence to Stories.
  - Update narratives and acceptance criteria.
  - Mark Questions answered if criteria are met.

## Summarization Patterns
- Micro: 1–2 sentence “story beat” to include in WM.
- Meso: paragraph summary of progress and blockers.
- Macro: consolidated narrative across episodes; include acceptance_criteria status table.

## Retrieval and Ranking
- Rank Questions by goal alignment, recency of updates, and priority.
- Rank Stories by proximity to the current query and presence of unmet acceptance_criteria.

## Testing Requirements
- Idempotent updates to Stories; merging concurrent edits results in deterministic ordering.
- Acceptance criteria gating required before marking a Question answered.
- Evidence linkage persists and is searchable by story/question.

## Acceptance Criteria
- Coding agent can implement Question/Story interfaces, integrate with FC and WM, and pass tests.
- Stories meaningfully guide planning and improve retrieval utility without exceeding budgets.

