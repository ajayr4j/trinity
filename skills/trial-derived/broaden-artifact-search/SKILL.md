---
name: broaden-artifact-search
description: Use when a task requires finding supporting documents (call/QBR transcripts, meeting notes, enhancement docs) via search_files or similar. Prevents missing real files because the search query assumed a keyword that isn't actually in the title.
---

# Broaden Artifact Search

A literal keyword search on `title` can return zero results even when the file exists, if the actual filename doesn't contain the word you assumed (e.g. searching `title contains 'transcript'` when files are actually named after the account, date, or meeting type rather than the word "transcript").

## Rule

1. If an obvious keyword search (`title contains 'transcript'`, `'call'`, `'QBR'`) returns no results, don't conclude the artifact doesn't exist. Retry with `fullText contains` instead of `title contains`, and try account/deal/person names or date fragments actually relevant to the task instead of generic document-type words.
2. Cross-check with `list_recent_files` or a broader unfiltered listing scoped to the relevant parent folder/account, then filter visually, rather than relying solely on a single keyword query.
3. A search result's content snippet is never sufficient — always follow up with the actual read/get-content call to pull full text before citing anything from the document, since snippets can omit the specific fact the task is grading on.
4. Treat an empty search result as a signal to broaden the query (see [[verify-field-values]] for the analogous principle on structured filters), not as proof of absence.
