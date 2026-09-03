---
name: using-benchmark-skills
description: Use at the start of every task in this environment (pm/crm/file-server MCP benchmark). This is the meta-skill that routes to which other skills apply, based on what the task is actually asking for. Read this first, before writing any query.
---

# Using Benchmark Skills

This benchmark grades tool-using agents against a mock CRM (`crm_query`, Salesforce-like SOQL over Account/Opportunity/Case/User), a mock PM tracker (`pm_search_issues`/`pm_list_components`, Jira-like JQL over a 3-level PART hierarchy), and a file server (`search_files` over transcripts/notes/docs). Trials fail almost entirely from silent, non-erroring mistakes: wrong enum literal, rolled-up grouping, unfiltered ranking, truncated pagination, incomplete join chains, narrated-but-unsent answers — not from crashes. Every atomic skill below exists to close one specific silent-failure mode observed in real trial data.

## Rule

1. Before writing the first query, read the task's instruction and criteria file (if present) and classify it against the decision tree below. Attach every skill that matches — most tasks match more than one.
2. Two skills are near-universal — apply them by default unless you're certain they don't apply:
   - [[verify-field-values]] — any task that filters on status/priority/stage/severity.
   - [[real-submission-call]] — every task, at the very end.
3. Do not substitute assumption or training-data convention for a live check. If a skill says "verify," that means call the tool, not recall what a typical Salesforce/Jira instance looks like.
4. If mid-task a query returns an empty, zero, or implausibly large result, stop and re-check the relevant skill's trap list before concluding "there is no data" or fabricating an answer.

## Decision tree

**Does the task filter or branch on a status/priority/stage/severity/type field?**
→ [[verify-field-values]]. Sample the field's literal values before filtering. (Confirmed traps: CRM `Case.Status` is only `"open"`/`"solved"`; PM `status` needs snake_case ids like `in_progress`, not display labels like "In progress".)

**Does the task ask to group, count, or summarize by product area / component / category?**
→ [[leaf-level-grouping]]. The PART hierarchy is product → product_area → feature; tickets/issues tag the leaf `feature`. Report at that leaf level — collapsing to 3-6 broad rows is a rollup bug, not a legitimate summary.

**Does the task ask to rank, prioritize, or identify "most impactful" / "at risk" / "top N" items?**
→ [[filter-before-rank]]. Apply the priority/severity predicate before or during ranking, not after — at scale, lower-priority items share the same components/ARR as high-priority ones and will silently dominate a naive join-only ranking.

**Does answering require following a chain of relationships (issue → component → ticket → account → revenue, or similar multi-hop join)?**
→ [[complete-join-chain]]. Walk every hop the question implies; don't stop at the first join and call it done, and don't fabricate a hop that returned nothing.

**Does the task specify a date range, a specific week, a specific team, or a specific reporting chain?**
→ [[exact-scope-boundary]]. Filter on the field that matches what's actually being scoped (event date vs. created/updated date; direct + indirect reports vs. adjacent team). Drop anything outside the literal boundary.

**Does the task require repeating the same lookup/action across a set of named entities (every flagged account, every open issue, every team member)?**
→ [[exhaustive-entity-coverage]]. Enumerate the full set up front; verify every member — especially the last one — was actually processed before submitting.

**Does the task ask to name, identify, or list entities at any level of a hierarchy or grouping (directors in a reporting chain, accounts in a segment, components with escalations)?**
→ [[resolve-to-named-records]]. A count or category ("5 directors exist") is not an answer when the task asked "who" — resolve every such level to actual named records before finalizing, even if levels above or below it were already named correctly.

**Does the task require finding a supporting document (transcript, QBR notes, meeting notes) via `search_files`?**
→ [[broaden-artifact-search]]. If an obvious title keyword returns nothing, retry with `fullText contains` and account/date/person names, and always pull full content before citing — never cite a snippet.

**Could the query's result set be larger than one page (`crm_query` `totalSize`, `pm_search_issues` `total`)?**
→ [[paginate-dont-truncate]]. Compare returned records against the reported total; page through or narrow the filter rather than extrapolating from a partial page.

**Every task, no exceptions:**
→ [[verify-field-values]] if any filtering is involved, and [[real-submission-call]] before finishing — the task is only complete when the actual submission tool call has been made and its response observed, not when a response describing the submission has been written.

## Quick reference

| Task signal | Skill(s) to attach |
|---|---|
| Filters on status/priority/stage | verify-field-values |
| Groups/summarizes by product area or component | leaf-level-grouping (+ verify-field-values) |
| Ranks/prioritizes/"top N"/"most at risk" | filter-before-rank (+ complete-join-chain) |
| Multi-hop relationship (issue→ticket→account→revenue) | complete-join-chain |
| Date window / team / reporting-chain scoped | exact-scope-boundary |
| "For each" account/issue/person | exhaustive-entity-coverage |
| "Name/identify/list" at any hierarchy level | resolve-to-named-records |
| Needs a transcript/note/document | broaden-artifact-search |
| Large or unfiltered query | paginate-dont-truncate |
| Always | verify-field-values, real-submission-call |
