# Results

Every measured run, including the ones that went backwards.

## The benchmark

[Enterprise-Bench](https://github.com/devrev/enterprise-bench) `l1-l2-bench`, built and open
sourced by DevRev. Credit where it is due: it is a genuinely hard and honestly constructed
benchmark, which is exactly why the numbers below mean anything.

**Shape.** 14 tasks across engineering, sales and support, at two autonomy levels. L1 is
reactive retrieval, L2 requires synthesis and judgment. 10 trials per task, 140 trials per
configuration.

**Needle in a haystack.** This is the design decision that makes it hard. The dataset is a
synthetic payments company at production scale: 32,768 tickets, 8,448 issues, 8,704
opportunities, 42 accounts, 40 product components. The correct answer is held constant while the
surrounding noise scales up, so roughly **0.16% of the corpus is relevant to any given task**.

The consequence is that broad retrieval is punished twice. An agent that filters correctly at
the data-access layer pulls back signal. An agent that retrieves broadly pays in tokens and in
error surface, and the wrong answer it constructs will look perfectly well formed. This is the
same failure shape Method 1 exists to catch, which is why the benchmark is a fair test of it.

**Grading.** One submission per trial, no retries within a trial. An LLM judge scores against a
rubric and a reference answer that are injected into the container only at verify time, after
the agent has finished, so the agent can never read them. Scoring is binary per trial and
ID-agnostic, since display IDs are renumbered per org at load time. Grading is on semantics, not
string match.

---

## Main table

| Configuration | Harness | Model | Accuracy | Trials |
|---|---|---|---|---|
| DevRev Computer | closed | Opus 4.6 (closed) | 95.7% | 140 |
| **Trinity, Empirical Failure Cataloguing** | **open** | **GLM-5.2 (open)** | **94.3%** | 140 |
| DevRev Computer | closed | Opus 4.8 (closed) | 94.3% | 140 |
| **Trinity, Empirical Failure Cataloguing** | **open** | **DeepSeek V4 Pro (open)** | **78.6%** | 140 |
| Claude Code, bare | open | Opus 4.8 (closed) | 63.6% | 140 |
| Claude Code, bare | open | Opus 4.6 (closed) | 61.4% | 140 |
| **Trinity, Fractional Knowledge Graph** | **open** | **DeepSeek V4 Pro (open)** | **57.1%** | 140 |
| Claude Code, bare | open | DeepSeek V4 Pro (open) | 51.4% | 140 |
| OpenCode, bare | open | DeepSeek V4 Pro (open) | 40.7% | 140 |

Every Empirical Failure Cataloguing row above was run with the trial-derived pack,
`skills/trial-derived/`, router `using-benchmark-skills`. The probe-only pack shipped alongside it
in `skills/probe-only/` produced none of these numbers, and its own result is in the negative
results below.

## Controlled deltas

Each method is compared only against its own baseline, with harness family, model, tools and
task set held constant. The only variable is the knowledge layer.

| Method | Baseline | With method | Delta |
|---|---|---|---|
| Empirical Failure Cataloguing (DeepSeek V4 Pro) | 51.4% | 78.6% | **+27.1 pts** |
| Fractional Knowledge Graph (DeepSeek V4 Pro) | 40.7% | 57.1% | **+16.4 pts** |

These two deltas are **not additive and have not been measured together.** The combined
configuration is in the pipeline.

## Parity

Trinity with Empirical Failure Cataloguing on GLM-5.2, an open model, scores 94.3%. DevRev
Computer, a closed proprietary harness, on Opus 4.8, scores 94.3%. On Opus 4.6 it scores 95.7%.

An open harness plus an open model lands inside the same band as a closed harness on a frontier
closed model.

Separately, a frontier closed model with no knowledge layer at all (Claude Code on Opus 4.8,
63.6%) scores well below an open model that has one (78.6%). The layer moves the number more
than the model does.

---

## Negative results

These matter more than the headline, because they are what the method's gates were written from.

**A probe-only catalogue regressed at full scale.** Both packs in this repo are published, and
this is the run that separates them.

`skills/probe-only/` (router `using-trinity-skills`, six leaves) was built entirely from
exhaustive API probing against the benchmark's own MCP servers, and smoke tested only at a single
trial. It scored **0.543** on a full 140-trial run on DeepSeek V4 Pro, with 8 exceptions.

`skills/trial-derived/` (router `using-benchmark-skills`, ten leaves) had been through a real
trial-and-fix loop. It scored **0.79** on the same benchmark, the same model and the same week,
with 2 exceptions. It is also the pack behind every headline number in the main table above.

The probe-only catalogue had complete class (a) coverage, the traps the system sets, and zero
class (b) coverage, the traps the agent sets for itself. The checklist being green said nothing
about the second kind, and a single-trial smoke test cannot distinguish them. This is why
Method 1 makes the small-batch checkpoint a hard blocking gate rather than advice.

**A tool with no instruction layer scored zero.** With the recall tool present but no router
instruction, the agent invoked it **0 times** and the trial scored 0. With the instruction
loaded, same task and same model, it invoked it 4 times and scored 1.0. A tool's own description
is not enough to change a model's habits.

**A false regression that was infra noise.** A confirmation run came back 3/10 and read as a
regression until inspection showed 9 of the 10 trials had hit a credit-exhaustion error
unrelated to the change. The number was discarded and the run redone clean. Every anomalous
delta now gets an exception-log check before anyone draws a conclusion from it.

---

## Reproducibility notes

All jobs are public on Harbor Hub. Two caveats on the published figures:

- **Token counts are audit-reconstructed, not self-reported.** Gateway-reported usage is
  unreliable in both directions for non-Anthropic models, off by roughly 10x high in one run and
  near-zero in another. Published counts are reconstructed independently from the trial
  transcripts with a cl100k tokenizer.
- **Cost figures are suppressed where they are not trustworthy.** Where the gateway did not
  populate real usage, cost is shown as unavailable rather than as a plausible-looking wrong
  number. That is the same principle the rest of this repo is about.
