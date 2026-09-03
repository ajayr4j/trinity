---
name: exhaustive-entity-coverage
description: Use when a task requires the same action repeated across multiple named entities (every flagged account, every open issue, every team member). Prevents skipping the last one or two entities under session/time pressure.
---

# Exhaustive Entity Coverage

When a task implies a fixed, enumerable set of entities to process identically (e.g. "for each account with an open ticket," "for each High/Critical issue"), track that set explicitly and verify every member was actually processed before submitting — not just the ones processed early in the session.

## Rule

1. At the start, enumerate the full set of entities the task requires (via the query itself, not estimation) and keep that list visible/checked-off through to the end.
2. Required per-entity steps (e.g. reading a transcript, computing a per-account figure) must be completed for every entity in the set, including the last one — the last entity is the one most often skipped when a session runs long or context gets tight.
3. Before final submission, re-count: does the output cover every entity from the enumerated set? If the count is short, go back and complete the missing entity rather than submitting a partial answer as if it were complete.
