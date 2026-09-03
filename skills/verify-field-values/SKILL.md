---
name: verify-field-values
description: Use before writing any WHERE/filter clause on status, priority, severity, or stage fields (crm_query SOQL, pm_search_issues JQL, or any similar query tool). Prevents silently wrong or empty result sets caused by assuming standard/common enum values instead of the literal ones the schema actually stores.
---

# Verify Field Values

Never assume a field's literal values from convention (Salesforce/Zendesk/Jira defaults, prior tasks, training-data habits). Different mock backends in this environment use different literal strings for the same concept, and a wrong literal does not error — it silently returns 0 or an inflated/wrong count.

Confirmed traps in this environment (verified by direct query, not assumption):
- CRM `Case.Status` only has two literal values: `"open"` and `"solved"`. There is no `"Closed"`, `"Resolved"`, or `"In Progress"`. A filter like `Status NOT IN ('Closed','Resolved')` matches everything, including all `"solved"` (closed-equivalent) tickets — inflating an "open P0/P1 tickets" count from the true 31 to 11,089+.
- PM `pm_search_issues` JQL `status` filter requires the internal snake_case id (`in_progress`, `done`), not the human-readable label the API itself displays (`"In progress"`, `"Done"`) in every returned record. `status = 'In Progress'` returns `total: 0` — silently, with no error.
- `Case.Priority` and `Components__c` equality/membership checks in this CRM ARE case-insensitive and tolerate `=` vs `INCLUDES` — those two are not traps. Do not over-generalize; verify each field independently rather than assuming all fields in a tool behave the same way.

## Rule

1. Before filtering on any status/priority/stage/severity-like field, call the tool's describe/list method (`crm_describe`, `pm_list_components`, etc.) AND run one unfiltered sample query (`SELECT <field> FROM <Object> LIMIT 20` or equivalent) to see the literal values actually present. Do not proceed straight to a filtered query on a field you haven't sampled.
2. If a filtered query returns 0 results, or a count that looks implausibly large relative to the story the task tells (e.g. "how many escalated tickets" returning thousands), treat that as a signal your filter literal is wrong — go back and re-sample the field. Do not report "there are none" and do not fabricate a plausible-looking answer to fill the gap.
3. Do this per-field, per-tool. A behavior confirmed on one object/tool (e.g. case-insensitive matching in CRM) does not transfer to another (e.g. PM's JQL, which is strict about internal ids).
