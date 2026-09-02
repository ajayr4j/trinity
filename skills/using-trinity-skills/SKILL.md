---
name: using-trinity-skills
description: Router for Enterprise-Bench (pm / crm / file-server) failure-mode skills. Load before every trial. Routes to atomic skills; contains no task-solving logic itself.
metadata:
  type: router
  methodology: ../../methods/empirical-failure-cataloguing.md
  evidence: ../../benchmarks/RESULTS.md
---

# using-trinity-skills

## Failure thesis

Against `pm`, `crm`, and `file-server`, this agent's failures are almost
never loud. Every trap found so far returns a well-formed `200` with a
plausible-looking payload — an empty result set, an undercounted rollup, a
truncated snippet mistaken for full text, a static token mistaken for a
cursor, a display label that silently matches nothing when used as a filter,
a filename that doesn't describe what the file actually contains. The
dominant failure shape is: **the value or count returned is internally
consistent and confidently wrong**, and nothing in the response itself flags
that. Six such patterns have been confirmed empirically (see
`../../step1-findings.md`) and are each routed to below as one atomic skill.

## Systems in scope

| System | MCP endpoint | Style |
|---|---|---|
| `pm` | `http://host.docker.internal:8011/mcp` | Jira-style issue tracker |
| `crm` | `http://host.docker.internal:8012/mcp` | Salesforce-style CRM |
| `file-server` | `http://host.docker.internal:8013/mcp` | Google Drive-style file store |

## Decision tree

1. **About to filter/query on a status, priority, type, stage, or other
   closed-vocabulary field in `pm` or `crm`?**
   → [[enum-value-verification]]. Sample real values first; don't trust a
   plausible label or a `describe` tool's type-only output.

2. **Task is phrased at a product/category/parent level (not a specific leaf
   feature) in `pm` or `crm`, and involves a `component`/`Components__c`/tag
   field?**
   → [[hierarchy-rollup-filtering]]. Resolve the full descendant leaf-id set
   before filtering; a parent-id equality filter silently undercounts.

3. **About to filter on a person/reference field (`assignee`, `reporter`,
   owner-shaped fields) or use a rendered `{name, id}` picklist's `name`
   form, in `pm` (or, unconfirmed, `crm`)?**
   → [[reference-field-display-vs-filter]]. Use the id/key form, not the
   display name, even if the display name is what the task's wording gives
   you.

4. **About to answer a task from a `file-server` `search_files`/
   `list_recent_files` `contentSnippet` field?**
   → [[full-content-over-snippet]]. Call `read_file_content` first; the
   snippet is a ~200-char truncation that reads as complete prose but isn't.

5. **`file-server` `search_files` response includes a `nextPageToken` and
   more results are needed?**
   → [[pagination-token-verification]]. It's a static literal, not a real
   cursor — re-issue the search with a larger `page_size` instead of trying
   to pass the token back in.

6. **About to pick a `file-server` file to answer a task based on how well
   its `title` matches the task's wording (especially with multiple
   similarly-named candidates)?**
   → [[filename-content-mismatch]]. Read the actual content before trusting
   the filename; a title match doesn't guarantee the content matches.

## Near-universal defaults

- **Never trust a schema/describe tool (`crm_describe`, etc.) as the source
  of truth for a field's accepted values.** It reports type, not vocabulary,
  in both `pm` and `crm`. This underlies condition 1 but is worth stating as
  a standing default: when in doubt about *any* field's real values, sample
  records, don't assume from the schema.
- **A `total: 0` / `totalSize: 0` result is not evidence that no matching
  records exist** until you've verified the filter value itself against a
  real sampled record. Treat empty results on these three systems as
  "unverified," not "confirmed absent," by default.
- **A `200` response with well-formed JSON is not evidence of correctness.**
  All six confirmed failure modes on this pack look completely valid at the
  response-shape level. Correctness has to be checked against what the field
  actually means, not just that the call succeeded.

## Quick-reference table

| Condition | System(s) | Skill | Evidence |
|---|---|---|---|
| Filtering enum/picklist field | pm, crm | [[enum-value-verification]] | Finding 1 |
| Filtering parent/category-level component/tag | pm, crm | [[hierarchy-rollup-filtering]] | Finding 2 |
| Filtering person/reference field by display name | pm (crm unconfirmed) | [[reference-field-display-vs-filter]] | Finding 5 |
| Answering from a search snippet | file-server | [[full-content-over-snippet]] | Finding 3 |
| Paginating past `nextPageToken` | file-server | [[pagination-token-verification]] | Finding 4 |
| Selecting a file by title match | file-server | [[filename-content-mismatch]] | Finding 6 |

## Relationship to ontology-engineering.md

`../../../ontology-engineering.md` is a separate, complementary discipline
(formal domain modeling — classes/properties/constraints validated via
competency questions), referenced from the methodology doc itself
(`Empirical Failure-Mode Cataloguing v1.md`, section 1e-bis) as the place to
build a structural model of pm/crm/file-server's entities and relationships
if the hierarchy/multi-hop checks above ever get too tangled to track by
memory. It is not merged into this router's decision tree.

## How this router is loaded

```bash
cd enterprise-bench/l1-l2-bench   # after make install/setup/build-image/start-servers
harbor run -p tasks -a claude-code -m claude-opus-4-8 \
  --mcp-config mcp.json \
  --skill ../../trinity/trinity-harness/skills \
  -k 10 -n 3 --yes
```

## What this pack does not cover yet

- Schema drift detection — this is a snapshot from one probe pass; if the
  underlying data or field vocabulary changes, the pack goes stale silently.
- Write-action safety (create/update/delete) — only read/query/search paths
  have been probed.
- `crm` Case `OwnerId`/`CreatedById`/lookup fields for the
  display-vs-filter trap (condition 3) — flagged as likely-recurring by
  shape, not directly confirmed.
- `pm` wiki-page content traps beyond a basic list (only 5 pages exist; no
  snippet-truncation or content-mismatch behavior found there, unlike
  `file-server`).
- No real trial run against this pack yet (Step 4) — every rule here traces
  to a direct API probe, not a judged trial failure. First real trial run is
  the next step, per the methodology's own definition of done.
