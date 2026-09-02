# Method 1: Empirical Failure Cataloguing

**A discipline for finding where a real system returns a wrong answer with a 200 OK, and
encoding each occurrence as an atomic, routable rule.**

This is an approach, not a benchmark tactic. The artifact it produces is a catalogue of how a
specific system misleads a competent reader. That catalogue is portable across tasks, across
agents, and across models. A benchmark is only where its effect can be measured.

---

## 0. The problem

Most agent harnesses trust their own tool calls. If a call returns 200 with a well-formed
payload, the harness treats the content of that payload as ground truth. It has no native
mechanism to ask: did I just get the right answer, or a wrong answer that merely looks
complete?

That gap is where enterprise agents quietly fail. Not with a stack trace, but with a confident,
fully formed, wrong answer:

- A count off by an order of magnitude because a status filter matched nothing.
- "5 items found" that never gets resolved to the 5 actual names.
- A join at the wrong level of a hierarchy, because the question's wording did not match the
  data's tagging granularity.
- One page of results silently mistaken for all of them.

If your agent's failures are loud, exceptions and 4xx and stack traces, you do not need this.
Fix the error handling instead. This is for the failures that do not tell you they happened.

---

## 1. Two failure classes, two discovery mechanisms

Conflating these is the single most expensive mistake in this method.

### Class (a): system-level silent traps

Traps in the shape of the API or the data itself, independent of any task wording or any agent.
An enum field whose schema tool reports `type: string` without enumerating accepted values. A
display label that does not round-trip as a filter value. A parent-node filter that under-counts
against the true rollup. A pagination token shaped like a cursor that the tool will not accept
as input. A two-hop join that a naive one-call query silently shortcuts.

**Found by: direct API probing, done exhaustively.** No trial and no judge required.

### Class (b): agent-reasoning traps

Traps that exist only at the intersection of a task's phrasing, an agent's reasoning path, and
a grader's expectation. A task saying "same product area" leading to a coarser join than the
rubric wanted, even though a correct rule was already in context. An agent correctly finding
"5 directors exist" but never resolving that to the 5 names the rubric required.

**Found by: real trials against a real judge. There is no other way.** No API response tells you
a model will override a correct rule because of colloquial phrasing. No schema dump tells you a
grader wants named records rather than a count.

### The consequence

Class (a) coverage, however complete, says nothing about class (b) coverage. A pack can have a
perfect probing checklist and still miss every rule a real workload needs.

This is not theoretical. A pack that was exhaustively probed but validated only at n=1 scored
**0.543** at full scale, against **0.79** for a prior pack that had been through a real
trial-and-fix loop. Same model, same benchmark, same week. The regression was not "pack versus
no pack". It was a trial-validated pack swapped for a probe-only one.

---

## 2. Prerequisites

- A harness that can inject standing instructions before every run, guaranteed to be read
  before the first tool call.
- Credentials to query the target system **directly, outside the agent**.
- A way to run repeatable trials and inspect the actual pass/fail judgment, not just the score.
  You need the failure explanation, not the number.

---

## 3. The steps

### Step 0.5: seed from real failure data if it exists

If a baseline run already exists, aggregate its judge results, read the actual explanation on
each failure, and let that list drive where probing focuses first. Probing targeted at
explaining an observed failure is far cheaper than a blind sweep. The blind sweep still has to
happen in full, but it should not be your first move.

If no baseline exists, run one. A first unassisted run is itself data.

### Step 1: probe the live system directly, exhaustively, by protocol

Go around the agent entirely. Raw HTTP, JSON-RPC, direct queries, whatever the transport is.
Never substitute "what a system like this typically looks like". That assumption is exactly what
produces the bugs you are trying to prevent.

The failure mode inside this step is stopping at the first interesting trap on the first entity
you happened to check. That produces a thin, plausible-looking catalogue.

**1a. Inventory first.** Enumerate every tool and every entity type before probing anything in
depth. A PM-shaped system has issues *and* a wiki *and* comments *and* users *and* components.
If you cannot yet name them all, you are not ready to move on.

**1b. Enum-like fields: sample records, do not trust the schema.** For anything that looks like
a status, category, priority or type: check whether the describe tool actually enumerates values
or only reports a type. Pull real records and record distinct values verbatim, including case
and delimiters. Then try two to four plausible-but-wrong values, the kind a competent person
would guess, and record whether they error or return empty silently.

**1c. Display form versus filter form.** Fetch a record through a normal read and note how the
field renders. Then filter using exactly that rendered value. If it does not round-trip, that is
a trap of its own, separate from 1b. Record both forms.

**1d. Hierarchies.** Get the full tree. Identify which level records are actually tagged at.
Filter by a non-leaf node and compare that count against the true rollup computed independently.
If the numbers differ, record both as evidence.

**1e. Foreign keys: perform the join, do not reason about it.** Walk every reference field at
least one hop, ideally two. Then ask explicitly: is there a naive one-call query that looks like
it answers a two-hop question but stops one join short? Record the exact naive query and why it
is wrong.

**1f. Lists and search: test truncation and pagination legitimacy.** If a continuation token is
returned, actually round-trip it. Confirm the input schema even accepts such a parameter. A
token that mimics a well-known convention but is a static literal is easy to miss. If a snippet
field exists alongside a full-content fetch, compare lengths on a large record.

**1g. Close each system out against a checklist** before moving to the next. Every tool called.
Every entity sampled. Every enum's real values recorded. Every display form round-tripped. Every
hierarchy rollup compared. Every foreign key walked. Every pagination token tested.

**1h. Pool findings across systems before writing anything.** Recurring classes become one
atomic rule referenced by several routing conditions, not one rule per system.

### Step 2: borrow structure, not content, from a working router

One root router with zero task-solving logic. Atomic, single-responsibility leaves. Routing
rules stated as plain checkable conditions. Near-universal defaults called out separately.
A failure thesis stated up front.

### Step 3: write the atomic rules first, the router last

A good atomic rule has one clear trigger condition, states the trap with concrete evidence
(the exact wrong value, the exact silent behaviour, the exact before and after counts), gives an
unambiguous correction, and is reusable across many tasks rather than scoped to one task's
wording. If you are about to name a rule after a specific failing task, find the general failure
class underneath it first.

Tag each rule with the class it addresses, (a) or (b). This is what makes the gate below
checkable: at any moment you can count how many class (b) rules exist, and know honestly whether
the catalogue has ever been exposed to real trial failures.

### Step 4: test against real trials, in sequence, with checkpoints

Not optional, not deferrable.

1. **Single exploratory trial.** Confirm it loads and routes. Read the whole transcript.
2. **3-trial confirmation.** Consistency, not one lucky pass. Keep these physically separate
   from anything already trusted.
3. **Small-batch validation**, roughly one trial per task type. This is where class (b) findings
   actually appear. Read every judge explanation on every failure.
4. **Only then, full scale.** Promotion is an explicit, disclosed step.

When something fails: read the actual explanation, never guess. Then check whether an existing
rule should have covered it. If it should have but was not invoked, that is a routing gap, fix
the router. If it was invoked but missed an edge case, extend that rule. Only create a new rule
for a genuinely new class. Re-test after every change and confirm the number moved.

### Step 4.5: distrust anomalous numbers before believing them

Check exception logs for infra noise before treating any delta as real. Credit exhaustion,
transient gateway bugs, timeout misconfiguration. In one session a run came back 3/10 and looked
like a regression until inspection showed 9 of those 10 trials hit a credit-exhaustion error
unrelated to the change. Treat every surprising delta as guilty until the logs clear it.

### Step 5: force the router to load first, every run

Wire it into whatever mechanism guarantees it is read before the first tool call.

### Step 6: keep the loop open

Stop revising a rule when repeated testing stops surfacing new instances of its failure mode.
Start a new one the moment a genuinely new class appears. There is no scheduled done.

---

## 4. Hard gates

- No full-scale run without clearing the small-batch checkpoint. A probe-only catalogue has
  zero class (b) coverage by construction.
- A catalogue must state out loud which classes it covers. If it has not faced real trials, it
  says "no class (b) coverage yet" rather than implying completeness from a green checklist.
- Testing and production locations must be physically separate, not a convention in someone's
  head.
- Every anomalous delta gets an infra-noise check before being trusted.
- Seed from an existing baseline's judge failures when one exists.

---

## 5. Anti-patterns

- Treating probe completeness as catalogue completeness.
- Writing rules from assumed or textbook behaviour instead of the live system.
- Declaring probing done after the first interesting trap on the first entity.
- Trusting a describe tool as the source of truth for enum values.
- Assuming a pagination token is legitimate because it is shaped like one.
- One rule per failing task instead of one rule per failure mode.
- Writing the router before the atomic rules exist.
- Patching a symptom narrowly enough that it fixes only the one case you saw.
- Declaring victory without a verified before and after.
- Running at production scale because the probing "felt thorough".
- Treating this as a substitute for a data or sync layer. It produces a reasoning layer, not
  schema-drift detection.

---

## 6. A worked output

`../skills/` is a real catalogue produced by this method against a Jira-shaped PM system, a
Salesforce-shaped CRM, and a Drive-shaped file server. One router, six atomic rules:

| Rule | Trap |
|---|---|
| `enum-value-verification` | describe reports a type, not a vocabulary |
| `hierarchy-rollup-filtering` | parent-level filter silently under-counts |
| `reference-field-display-vs-filter` | display name does not round-trip as a filter |
| `full-content-over-snippet` | snippet reads as complete prose, is truncated |
| `pagination-token-verification` | token is a static literal, not a cursor |
| `filename-content-mismatch` | title match does not guarantee content match |

Every one of the six returns a well-formed 200. None of them announce themselves.
