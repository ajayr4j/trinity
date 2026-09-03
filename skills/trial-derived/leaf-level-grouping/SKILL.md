---
name: leaf-level-grouping
description: Use when a task asks you to group, aggregate, count, or sum records "by product area", "by component", "by category", "by part" or similar, and the schema exposes a hierarchical taxonomy (parent_id, product → product_area → feature trees). Prevents silently rolling child records up into a coarser parent grouping when the task wants the literal tagged level.
---

# Leaf-Level Grouping

The PM component hierarchy in this environment is three levels deep: `product` (e.g. PART-001) → `product_area` (e.g. PART-003 "Billing & Subscription Management") → `feature` (leaf, e.g. PART-010 "Subscription Lifecycle Management"). Confirmed via `pm_list_components`: every record's `metadata.parent_id` links a leaf feature up through its product_area to the root product.

Support tickets and PM issues are tagged directly at the **leaf feature level** (`Components__c` on Case, `components` on an issue). Grading always wants output grouped at that exact tagged leaf, never rolled up to the parent `product_area`.

## Rule

1. Group by the exact value each record is tagged with. If a ticket's `Components__c` is `PART-010`, report it under `PART-010` / "Subscription Lifecycle Management" — never under its parent `PART-003` / "Billing & Subscription Management" — unless the task explicitly asks for a rollup view.
2. It is fine to call `pm_list_components` to resolve a leaf id to a human-readable name. It is not fine to use that hierarchy to merge multiple distinct leaf ids into one parent output row.
3. Do the group-by as a single query directly on the raw tagged field (`GROUP BY Components__c`), not as a two-step "fetch hierarchy, then decide what level to report at" process — the two-step process is where rollups creep in.
4. Sanity check before finalizing: count how many distinct leaf-level ids exist in the raw data versus how many groups your output has. If your output collapsed many distinct tagged values into 3-6 broad rows, you rolled up incorrectly — re-group at the literal tagged level.
5. This rule applies to every join hop, not just the top-level group-by — including cross-entity matching like "which accounts have tickets on the same product area/component as this issue." Even when the task's own wording says "same product area" or "same category," treat that as loose/colloquial phrasing, not a license to widen the join: if the issue and the ticket are both tagged with a specific leaf id (`Components__c` / `components`), the match is on that exact leaf id being equal, not on both leaves sharing a common parent. Confirmed failure mode: an agent reads "same product area" in the question, resolves each leaf up to its parent product_area, and joins there — this produces a fabricated/inflated account list, because it now includes every account with a ticket anywhere under that parent, not just the same leaf. If the question's wording and the tagging granularity conflict, the tagging granularity wins.
