---
name: hierarchy-rollup-filtering
description: Use when a task is phrased at a product/category/parent level (e.g. "issues in Checkout & Customer Experience") in pm or crm — a direct equality filter on the parent id only matches records tagged with that exact id, silently missing every descendant leaf.
metadata:
  type: atomic
  systems: [pm, crm]
  evidence: ../../step1-findings.md#finding-2
---

# hierarchy-rollup-filtering

## Trigger

Any task in `pm` or `crm` phrased at a product/product-area/category level
rather than the leaf-feature level — e.g. "tickets under Billing &
Subscription Management," "issues in the Checkout product area." Also trigger
whenever a `component`/`Components__c`/tag-like field is being filtered and
you have not yet checked whether the target id is a leaf or a parent node.

## The trap, with evidence

The component/tag taxonomy is a real 3-4 level tree (`product` →
`product_area` → `feature` → nested `feature`). Records are tagged at
**leaf** level only, essentially never at the parent level directly.
Filtering on a parent id returns only records literally tagged with that
exact id — no error, no partial-result notice, just a badly undercounted but
entirely plausible-looking number:

- `pm`: `component = PART-002` → **208** issues. True count (`PART-002` +
  all 6 descendant feature ids) → **1489** issues. ~7.2x undercount.
- `crm`: `Components__c INCLUDES ('PART-002')` → **744** tickets. True count
  via descendants → **5784** tickets. ~7.8x undercount.

## Corrective rule

1. Call the component/hierarchy listing tool (e.g. `pm_list_components`) to
   get the full tree, including `parent_id` for every node.
2. Determine whether the id in the task maps to a leaf or a parent/category
   node in that tree.
3. If it's a parent/category node, resolve the **full set of descendant leaf
   ids** first (walk the tree, or filter `parent_id` recursively).
4. Filter/query using an OR-ed/IN-list across every descendant leaf id — never
   a single equality filter on the parent id.
5. If you must report a count or list for a "product-area"-level ask, the
   number should reflect the rolled-up total, and you should be able to name
   which leaf ids you unioned to get it.
