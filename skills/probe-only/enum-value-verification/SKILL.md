---
name: enum-value-verification
description: Use before filtering on any status/priority/category/type/stage-like field in pm or crm — the plausible label you'd guess is usually not the system's real value, and a wrong value returns a silent empty result, not an error.
metadata:
  type: atomic
  systems: [pm, crm]
  evidence: ../../step1-findings.md#finding-1
---

# enum-value-verification

## Trigger

Any task that filters/queries on a field that looks like a closed vocabulary:
`status`, `priority`, `issuetype`, `StageName`, or any other picklist-shaped
field in `pm` or `crm`.

## The trap, with evidence

A well-formed, human-plausible value for one of these fields silently returns
an empty result instead of erroring — no warning that the value itself might
be wrong:

- `pm.pm_search_issues(jql="status = open")` → `{"total": 0, "issues": []}`.
  Real values: `in_progress`, `done`. There is no `open`.
- `pm.pm_search_issues(jql="priority = low")` → `{"total": 0}`. Real values:
  `high`, `highest`, `medium`. There is no `low`.
- `crm.crm_query(soql="... WHERE Priority = 'High'")` → `{"totalSize": 0}`.
  Real values: `p1`, `p2`, `p3`.
- `crm.crm_query(soql="... WHERE StageName = 'Closed Won'")` (standard
  Salesforce label) → `{"totalSize": 0}`. Real value: `closed_won`.

`crm_describe(object_type=...)` reports these fields only as
`type: "string"`/`"multipicklist"` — it does **not** enumerate accepted
values. Calling `describe` first is necessary but not sufficient.

## Corrective rule

Before filtering on any enum-shaped field:
1. Call the schema/describe tool if one exists, but treat its output as
   "this field exists and has this type," not "these are its valid values."
2. Pull a small real sample of records (a narrow query, a search, a list
   call) and read the field's actual values verbatim — exact casing,
   delimiter, and id-vs-label form.
3. Filter using exactly those sampled values, not a guessed label.
4. If a filter returns `total: 0` / `totalSize: 0` and you have not yet
   sampled real values for that field in this task, treat the empty result as
   **unverified**, not as "no matching records exist" — go sample before
   concluding there's nothing there.

## Scope note

This does not cover the case where the *field name* used for filtering
differs from the *value* rendered on a record (e.g. `priority.name` vs
`priority.id`, or a person's display name vs their account id) — that's
[[reference-field-display-vs-filter]], a related but distinct trap.
