---
name: filter-before-rank
description: Use when a task says "rank by impact", "most likely to affect", "highest priority" or similar, and multiple rows can share the same joined value (e.g. several issues on the same component inherit that component's full ARR). Prevents a lower-priority row outranking a higher-priority one whenever they tie on the joined metric.
---

# Filter Before Rank

When rows are joined through a shared key (component, account, ARR) before being ranked, rows that tie on the ranking metric but differ in priority/severity will sort arbitrarily against each other unless priority is made part of the ranking key. A naive join-then-sort lets a Medium-priority row (tech debt, backlog) outrank a High/Highest row (an active customer-facing bug) whenever both touch a component with the same ARR exposure.

## Rule

1. Apply the priority/severity filter as an explicit predicate before or during ranking — not as a note added afterward. "Most likely to affect customers" means priority is part of the sort key, not just the joined ARR number.
2. Before finalizing, check your own top-N: if it contains a mix of priority levels tied on the same metric, that's a sign the filter wasn't actually applied — go back and re-filter, don't just re-order visually.
3. The same logic applies to any "open P0/P1", "open or in-progress" style filter used earlier in a join chain: apply it before aggregating, then re-verify afterward that nothing failing the filter slipped back in through a shared join key.
4. State explicitly, in the final answer, why higher-priority items rank above same-ARR lower-priority ones — this makes the filtering step auditable rather than implicit.
