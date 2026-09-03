---
name: paginate-dont-truncate
description: Use whenever a query tool result includes a total/count field larger than the records actually returned (crm_query's totalSize vs records, pm_search_issues's total vs startAt/maxResults, or any similarly paginated response). Prevents abandoning or estimating from a truncated first page.
---

# Paginate, Don't Truncate

Tool results in this environment are frequently paginated or capped (e.g. `pm_search_issues` caps at 100 results per page and reports `total`/`startAt`; `crm_query` reports `totalSize` which may exceed the returned `records`). A truncated first page is not the full answer.

## Rule

1. Always compare the reported total/count field against the number of records actually returned. If they differ, you have more pages to fetch.
2. Use the tool's own pagination parameters (`start_at`, `OFFSET`, etc.) to retrieve every remaining page before aggregating or answering — do not extrapolate a total from a partial page, and do not silently drop the excess.
3. If a full paginated fetch is impractical (very large total), narrow the query with additional WHERE/JQL predicates so the true qualifying set fits within a page, rather than truncating an unfiltered query and treating the partial result as complete. A count of 32,768 candidate records is a sign the filter is too broad, not a paging problem to work around.
4. Never abandon a query mid-page and substitute a smaller, differently-sourced number (e.g. giving up on a paginated `crm_query` and instead guessing from memory) — this produces answers that look plausible but are disconnected from the real data.
