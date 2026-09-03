---
name: full-content-over-snippet
description: Use whenever a file-server search result's contentSnippet is about to be treated as the answer — it's a truncated ~200-character excerpt that reads as coherent prose, not the full document.
metadata:
  type: atomic
  systems: [file-server]
  evidence: ../../step1-findings.md#finding-3
---

# full-content-over-snippet

## Trigger

Any task answered from `search_files` or `list_recent_files` output where the
answer is drawn from the `contentSnippet` field, for any file whose
`fileSize` is larger than the snippet itself.

## The trap, with evidence

`contentSnippet` is consistently truncated to roughly 200 characters,
regardless of the file's real size, and it's cut off mid-sentence — but it
reads as ordinary prose, not obviously partial:

- `MAPLE_FULL_MSA.md`, `fileSize: 61468` bytes → snippet is 203 chars, ends
  mid-sentence.
- `maple_arch_spec.md`, `fileSize: 73297` bytes → same pattern.

Nothing in the snippet itself signals "this is 0.3% of the file."

## Corrective rule

1. Never answer a task from `contentSnippet` alone once you know (or can
   check) that `fileSize` is meaningfully larger than the snippet length.
2. Call `read_file_content(file_id)` to get the full text before extracting
   any fact, quote, or answer from a file's contents.
3. Use `search_files`/`list_recent_files` only to *locate* the right file
   (see also [[filename-content-mismatch]] — locating by title alone isn't
   enough either) — never to *read* it.
