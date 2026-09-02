---
name: filename-content-mismatch
description: Use whenever a file-server task is about to be answered by picking a file based on how well its title/filename matches the task's wording — the title is not a reliable proxy for what the file actually contains.
metadata:
  type: atomic
  systems: [file-server]
  evidence: ../../step1-findings.md#finding-6
---

# filename-content-mismatch

## Trigger

Any task that asks for a specific document by description ("the full MSA,"
"the architecture spec," "the latest contract") and is answered by selecting
whichever `search_files`/`list_recent_files` result has the closest-matching
`title`, without reading its actual content first.

## The trap, with evidence

Two files in the same directory (`fileSize: 61468` and `22090` respectively):

- `MAPLE_FULL_MSA.md` — the name that best matches "the full MSA." Its actual
  content, confirmed via `read_file_content`, opens `"# COMPARISON: Generated
  MSA vs. Actual Maple T&Cs"` — a diff/analysis document, not the agreement
  itself.
- `MAPLE_FULL_MSA_DRAFT.md` — the name that reads like an incomplete draft.
  Its actual content opens `"# MASTER SERVICES AGREEMENT ... Between: Maple
  Software Inc. ..."` — this is the real MSA text.

Both return well-formed `200` responses with plausible content; nothing
signals the mismatch except reading past the title into the body.

## Corrective rule

1. Never select a file to answer a task based on title/filename match alone,
   especially when multiple candidate files have similar names.
2. Once a candidate is chosen, call `read_file_content` (not just the search
   snippet — see [[full-content-over-snippet]]) and verify the opening
   content actually matches what the task is asking for before using it as
   the answer.
3. If multiple similarly-named files exist, read enough of each to positively
   identify which one is the real target — don't assume the more literally
   matching name is correct.
