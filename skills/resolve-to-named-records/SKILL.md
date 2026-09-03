---
name: resolve-to-named-records
description: Use when a task asks to identify, name, or list entities (people, accounts, components, escalations) at any level of a hierarchy or grouping. Prevents stopping at a count or category once its existence is confirmed, when the question actually requires the individual named records.
---

# Resolve to Named Records

A count or category is an intermediate result, not an answer, whenever the task's wording implies identity (name, identify, list, who) rather than quantity (how many, what proportion).

## Rule

1. If a query or a prior step establishes that N entities exist at some level (N directors in a reporting chain, N accounts in a segment, N escalations on a component), that count alone never satisfies a task asking to name, identify, or list them. Treat the count as a pointer to a follow-up query, not as the deliverable.
2. This applies at every level of a traversal, not just the last hop. A reporting chain resolved to "VP → 5 Directors → 20 AEs" is incomplete if the task requires naming people at the Director level, even if AEs and the VP were correctly named — silently downgrading one level of the chain to an aggregate is the same failure as omitting it.
3. Before finalizing, re-read the task's exact verbs ("name," "identify," "list," "who") against every level/group/category mentioned in your draft answer. For each one still expressed only as a count or generic label, run the query needed to resolve it to actual records, then substitute the names in.
4. Do not accept "role exists but is unnamed" as a stopping point because the individual query for that level returned harder-to-parse or larger results than the levels above or below it — this is the same failure this skill exists to prevent, at whatever level it recurs at.
