---
name: pagination-token-verification
description: Use whenever a file-server search result includes a nextPageToken and more results are needed — the token is a hardcoded literal, not a real cursor, and the tool's input schema has no parameter to pass it back in.
metadata:
  type: atomic
  systems: [file-server]
  evidence: ../../step1-findings.md#finding-4
---

# pagination-token-verification

## Trigger

Any task where `search_files` returns a `nextPageToken` (i.e. total matches
exceed the current `page_size`, default 50) and the task requires the full
result set, not just the first page.

## The trap, with evidence

- `nextPageToken` is always the literal string `"truncated_at_page_size"` —
  it carries no real position/cursor information.
- `search_files`'s input schema (`additionalProperties: false`) accepts only
  `query` and `page_size` — there is no parameter to pass a page token back
  in. A call that tries to pass one back will be schema-rejected or silently
  ignored.
- Confirmed fix: re-issuing the search with a larger `page_size` retrieves
  the rest (`page_size=5` → 5 files + token; `page_size=200` → all 12
  matching files, no token).

This mimics real Drive-API pagination closely enough that an agent familiar
with that convention will either get a rejected call trying to page with it,
or will stop silently, believing the search is exhausted when it isn't.

## Corrective rule

1. Treat `nextPageToken` on `search_files` as **not a real cursor** — do not
   attempt to pass it back into a subsequent call.
2. If a `nextPageToken` is present, re-issue the same query with a larger
   `page_size` instead, until no token is returned.
3. Before concluding "these are all the matching files," confirm the
   response has no `nextPageToken` at the `page_size` you actually used.
