---
name: exact-scope-boundary
description: Use when a task specifies a date range ("the week of Feb 9-14"), a specific team/org, or a specific reporting chain. Prevents activity from just outside the boundary bleeding into the answer.
---

# Exact Scope Boundary

Every activity/event/person cited in the answer must fall strictly inside the stated boundary — temporal or organizational.

## Rule

1. Filter on the specific timestamp/date field for each activity type individually (deal close date, creation date, meeting date, ticket-open date) — a deal that closed just before or after the window is not "that week's" activity even if discussed during the window.
2. When a field has multiple date-like candidates (created, updated, resolved, activity/event date), pick the one that matches what the question is actually scoping on — e.g. "what happened during the week" means the event/activity date, not when the parent record was created or last modified.
3. When summarizing an org/team's activity, verify the *who* boundary too: a manager's direct AND indirect reports count; an adjacent team that looks related but isn't in the reporting chain does not. Attributing an adjacent team's activity to the wrong org is the same class of boundary violation as a date leak.
4. Before finalizing, scan every date and every person cited against the stated window/org boundary and drop or re-attribute anything that fails the check.
