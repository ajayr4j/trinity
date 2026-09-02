# Method 2: Fractional Knowledge Graph

**A cross-silo knowledge graph built over a bounded slice of the enterprise estate, exposed to
the agent as a single recall tool.**

Where [Empirical Failure Cataloguing](empirical-failure-cataloguing.md) fixes what an agent gets
*wrong*, this method fixes what an agent cannot *reach*. They target different failures and are
measured separately.

---

## 0. The problem

Real enterprise questions span systems that share no foreign key.

> "What is blocking this deal?"

The answer lives across a CRM opportunity, a PM issue, and a wiki page. Nothing joins them
except meaning. A support ticket names a symptom, an engineering issue names a component, an
account record names the revenue at risk, and no column in any of the three points at the other
two.

An agent handed three separate search tools will chain them by hand: search CRM, guess a term,
search PM, guess again, search files. Each hop loses information and each guess is a chance to
retrieve confidently irrelevant results. The failure is not that the agent reasons badly. It is
that the relationship it needs was never written down anywhere it can query.

---

## 1. Why fractional

The obvious response is "index everything". That is the response that never ships.

A production estate is too large to mirror, too fast-moving to keep in sync, and mostly
irrelevant to any given question. In the benchmark this method was measured on, roughly **0.16%
of the corpus is relevant to any single task**. Indexing the other 99.84% buys nothing and costs
a sync problem forever.

Fractional means: **index the slice where cross-silo relationships actually carry answers, and
leave the rest to the live tools.**

Practically, this means the graph is:

- **Bounded.** Built over a sampled mirror, sized to the relationship density you need, not to
  the size of the source systems.
- **Relationship-first.** Its job is edges between silos. Within-silo lookups stay with the
  native tools, which are faster and always current.
- **Advisory, not authoritative.** The graph tells you *where to look* and *what connects to
  what*. Specific IDs, amounts and dates get confirmed against the live systems before they are
  reported. This constraint is not a limitation to apologise for, it is what lets the graph stay
  small enough to be buildable.

---

## 2. Build-out

### 2a. The graph

Ingest the sampled slice of each silo into a graph store and expose exactly one tool:

```python
cognee_recall(query: str)
```

backed by a graph-completion style query against the bounded dataset. One tool, natural language
in, related entities across silos out. The agent does not need to know the graph's schema, which
is the entire point. If the agent has to learn a query language to use it, you have replaced
three search tools with four.

Keep the surface at one tool. Every additional tool is another thing the agent has to decide
between, and tool-selection error is the failure mode you are trying to remove.

### 2b. Getting the agent to actually use it

This is the part that is easy to underestimate.

In the first single-trial tests, the tool was **ignored entirely**. The agent defaulted to the
native searches even on cross-silo tasks where recall was obviously the right call, because a
tool's own MCP description is not a strong enough signal to change a model's habits.

The fix is a router-style instruction loaded before the first tool call, describing:

- **When to reach for it.** Cross-silo relationship questions with no explicit join key.
- **When not to.** Single-silo lookups, anything needing a current exact value.
- **Its limitations.** Bounded and sampled, so always cross-check specific IDs, amounts and
  dates against the live tools.

Measured effect of adding that instruction, same task, same model:

| | `recall` invocations | Score |
|---|---|---|
| Tool present, no instruction | 0 | 0 |
| Tool present, instruction loaded | 4 | 1.0 |

The trial that passed correctly identified a deal blocker via a PM-issue to CRM-opportunity join
with no explicit foreign key. **A tool the agent does not reach for is worth exactly nothing.**
Budget real effort for the instruction layer, not just the graph.

---

## 3. Measured result

Enterprise-Bench `l1-l2-bench`, 14 tasks by 10 trials, 140 trials, DeepSeek V4 Pro, OpenCode
harness. Identical setup in both columns except for the recall tool and its instruction.

| Metric | Fractional Knowledge Graph | Baseline |
|---|---|---|
| Pass rate | **57.14%** | 40.71% |
| Errors | 3 | 9 |
| pass@2 | 70.5% | 56.7% |
| pass@5 | 81.3% | 73.1% |
| pass@10 | 85.7% | 85.7% |

**+16.4 percentage points**, with a third of the exception count.

The pass@10 convergence is the honest and interesting part. Both configurations reach the same
ceiling given ten attempts. The graph mainly improves **first-attempt reliability** on
cross-silo tasks rather than expanding the absolute capability ceiling. For production that is
the metric that matters, because production gets one attempt. But it should not be sold as
making the agent capable of things it otherwise could not do.

---

## 4. Operational notes

- **Never background a long-running job as a plain child of an interactive session.** One run
  died silently at 32/140 when its parent session was torn down, leaving orphaned containers
  idling for eleven hours with no crash signal anywhere. Launch detached, verify the process is
  its own session leader, and treat that as mandatory for anything unattended.
- **Do not conclude a crash from a missing process match alone.** Corroborate with log mtime,
  directory growth over a re-check window, and container liveness. A real stall has an
  unambiguous signature: zero write activity anywhere in the tree for thirty-plus minutes.
- **Watch small-n convergence before reacting.** That same run read 66.7% at n=9 and settled at
  57.1% by n=140. Early readings are noise. Do not tune against them.

---

## 5. Anti-patterns

- **Mirroring the whole estate.** You will spend all your time on sync and never on
  relationships.
- **Exposing the graph's query language to the agent.** One natural-language tool, or it will
  not get used.
- **Shipping the graph without the instruction layer.** Measured: zero invocations, zero gain.
- **Treating graph output as authoritative.** It is bounded and sampled. Confirm exact values
  against the live systems before reporting them.
- **Reading pass@k improvement as capability improvement.** It is reliability improvement, which
  is valuable and different.

---

## 6. Relationship to Method 1

These are independent. Method 1 removes wrong answers that look right. Method 2 removes missing
answers that were never reachable. Both are measured separately, against their own baselines,
with the model and tools held constant.

**Combining them is in the pipeline.** The joint configuration is the next run on the roadmap,
and both methods continue to be extended as new failure classes and new relationship types
surface. No combined number is claimed here until that run exists.
