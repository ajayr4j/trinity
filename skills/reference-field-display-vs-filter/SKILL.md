---
name: reference-field-display-vs-filter
description: Use when a task names a person, priority, or other reference-shaped field by its human-readable label (a name, not an id) in pm (and, unconfirmed but likely, crm) — the rendered display value and the value the filter actually accepts are different strings.
metadata:
  type: atomic
  systems: [pm]
  systems-unconfirmed: [crm]
  evidence: ../../step1-findings.md#finding-5
---

# reference-field-display-vs-filter

## Trigger

Any task that filters on a reference-shaped field using a human-readable name
pulled from a task description or a prior tool result's rendered output —
most commonly `assignee`, `reporter`, or any other person/lookup field in
`pm`. Also applies to picklist fields rendered as `{name, id}` pairs (see
[[enum-value-verification]] for the pure-vocabulary version of this trap;
this skill is about id-vs-label specifically, which persists even once you
know the right vocabulary word).

## The trap, with evidence

A real record's read/get response renders reference fields as a human name,
but the filter/query layer only accepts the internal id:

- `pm_get_issue(issue_key="ISS-001")` shows
  `assignee: {"accountId": "USER-062", "displayName": "Rahul Nair", ...}`.
- `pm_search_issues(jql="assignee = 'Rahul Nair'")` → `{"total": 0}` (silent —
  the exact string a task or a prior tool response would naturally hand you).
- `pm_search_issues(jql="assignee = 'USER-062'")` → `{"total": 1695}` (correct).

Same shape recurs for `priority`: rendered as `{"name": "High", "id":
"high"}` — the `name` form is closer to natural task language, the `id` form
is what the filter needs (also see [[enum-value-verification]]).

Not yet directly confirmed on `crm` (`OwnerId`, `CreatedById`, `Contact`
lookups are structurally the same shape) — treat as a live hypothesis there
until tested, not a confirmed rule.

## Corrective rule

1. Before filtering on any field that references another entity (a person, a
   user, an owner) or any picklist rendered as a `{name, id}`-shaped object,
   fetch one real record and look at exactly how that field is structured in
   the response.
2. If the field is an object with more than one representation (a display
   name and an internal id/key), use the **id/key** form for filtering, never
   the display name — even though the display name is what a task's wording
   will naturally give you.
3. If you only have a display name (from the task, not from a tool response),
   resolve it to an id first via a lookup/search call before filtering — don't
   filter on the name directly and trust an empty result.
