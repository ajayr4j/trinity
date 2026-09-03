---
name: complete-join-chain
description: Use when a task asks "which X are at risk / affect Y" and answering requires joining across more than one entity type (issue → component → ticket → account → ARR/opportunity). Prevents stopping the join one hop short of the entity the question actually asked about.
---

# Complete the Join Chain

Trace the full chain of entities the question implies — not just the first join that returns non-empty results.

## Rule

1. "Which issues affect paying customers" is not answered by issue → component → account. It requires issue → component → open ticket on that component → account → the account's ARR (or active opportunities), because "affect" and "revenue impact" are the actual asked-for output, not an intermediate account name.
2. If the question mentions "opportunities," "revenue," or "ARR," you are not done until you've joined all the way to that record. A ticket or account name alone is an intermediate result.
3. Before finalizing, re-read the question and check: does the output contain every entity type the question named (issue, component, account, AND opportunity/ARR), or did the join stop at the first step that "worked"?
4. Don't fabricate a join path when the underlying query returns nothing — this is the same failure the fields in [[verify-field-values]] guard against. A missing intermediate join result is a signal to re-check the query, not to fill in a plausible-sounding account/ticket pair from memory.
