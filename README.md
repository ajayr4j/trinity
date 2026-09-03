# Trinity

**An open framework for building the knowledge layer that enterprise AI agents actually run on.**

Trinity is not a model, and not an agent. It is the layer in between: the accumulated,
written-down knowledge of how a specific enterprise system misleads an agent, and how the
entities inside it actually relate to each other. That layer is what separates a demo from
something that survives contact with production data.

Today, that layer is almost always someone else's property. Either it lives inside a model
lab's closed harness, or it lives inside a vendor's "platform" that is, on inspection,
a group of forward deployed engineers carrying the knowledge in their heads. In both cases
the knowledge compounds on the vendor's side of the wall, not yours.

Trinity is an attempt to build that same layer in the open, and to measure honestly whether
it closes the gap.

---

## The thesis

1. A "platform" in the AI-native services market is, in practice, an internal knowledge base
   built up over time by forward deployed engineers.
2. If a vendor's FDE headcount scales roughly linearly with their customer count, the platform
   is thin and the developer time is doing the work. Less platform, more developer time.
3. Enterprises that try to build in-house and fall short usually conclude the gap is
   proprietary technology, and buy. In our measurements the gap is mostly **skill**, meaning
   a knowledge layer nobody wrote down, not IP.
4. A skill gap is closeable in the open. That is the claim this repo is here to test.

---

## The two methods

Trinity contains two independent methods. Each targets a different reason an agent gets an
enterprise question wrong.

### 1. [Empirical Failure Cataloguing](methods/empirical-failure-cataloguing.md)

An engineering discipline for finding the places where a system returns a **wrong answer with
a 200 OK**, and encoding each one as a small, routable, reusable rule.

You go around the agent entirely, probe the live API yourself, and catalogue where the
response is internally consistent and confidently wrong. Enum values a `describe` tool refuses
to enumerate. A display label that does not round-trip as a filter. A parent-level filter that
silently under-counts against the true rollup. A pagination token shaped like a cursor that is
actually a static literal. A snippet that reads as complete prose but is truncated.

This is a methodology, not a leaderboard tactic. The output is a catalogue of how a system
lies to you. The benchmark is simply where it could be measured.

### 2. [Fractional Knowledge Graph](methods/fractional-knowledge-graph.md)

A cross-silo knowledge graph built over a **bounded slice** of the enterprise estate and
exposed to the agent as a single recall tool.

Real enterprise questions span systems that share no foreign key. "What is blocking this deal"
lives across a CRM opportunity, a PM issue, and a wiki page, joined only by meaning. Chaining
three searches by hand is slow and lossy. A graph that already holds those relationships makes
it one call.

Fractional is the important word. It indexes the slice that matters rather than mirroring the
whole estate, which is what makes it buildable at all.

### Status: independent today, combined next

Each method has been benchmarked **on its own**, against the same tasks, the same models, and
the same 140-trial protocol. Every number below is one method alone.

**Combining them is in the pipeline.** The joint run is the next thing on the roadmap, and both
methods will keep being extended as more failure classes and more relationship types are found.
Until that run exists, no combined figure is claimed here.

---

## Measured results

Enterprise-Bench `l1-l2-bench`. 14 tasks, 10 trials each, 140 trials per configuration,
LLM judged, single submission per trial. All jobs are public on Harbor Hub.

| Configuration | Harness | Model | Accuracy |
|---|---|---|---|
| DevRev Computer | closed | Opus 4.6 (closed) | 95.7% |
| **Trinity, Empirical Failure Cataloguing** | **open** | **GLM-5.2 (open)** | **94.3%** |
| DevRev Computer | closed | Opus 4.8 (closed) | 94.3% |
| **Trinity, Empirical Failure Cataloguing** | **open** | **DeepSeek V4 Pro (open)** | **78.6%** |
| Claude Code, bare | open | Opus 4.8 (closed) | 63.6% |
| Claude Code, bare | open | Opus 4.6 (closed) | 61.4% |
| **Trinity, Fractional Knowledge Graph** | **open** | **DeepSeek V4 Pro (open)** | **57.1%** |
| Claude Code, bare | open | DeepSeek V4 Pro (open) | 51.4% |
| OpenCode, bare | open | DeepSeek V4 Pro (open) | 40.7% |

Read it as three findings:

- **Parity.** An open harness on an open model reaches the same score as a closed proprietary
  harness on a frontier closed model.
- **The layer carries the delta, not the model.** Empirical Failure Cataloguing moves the same
  model from 51.4% to 78.6%, a gain of 27.1 points with the model, the tools and the task set
  all held constant. Fractional Knowledge Graph moves its own baseline from 40.7% to 57.1%,
  a gain of 16.4 points on the same terms.
- **A frontier model with no knowledge layer still loses** to an open model that has one.

Full per-run detail, including the negative results, is in [benchmarks/RESULTS.md](benchmarks/RESULTS.md).

---

## What is in this repo

```
methods/
  empirical-failure-cataloguing.md   the discipline, step by step, with hard gates
  fractional-knowledge-graph.md      the graph method, build-out and wiring
skills/
  trial-derived/                     the pack behind every headline number below
  probe-only/                        the pack that regressed, kept as the negative result
benchmarks/RESULTS.md                every measured run, including the ones that regressed
```

Both directories are outputs of Empirical Failure Cataloguing. They are kept apart because they
were built differently and they scored differently, and that difference is the whole lesson of
the method.

**`skills/trial-derived/`** is the pack that produced the 94.3% and 78.6% results in the table
above. Router `using-benchmark-skills`, ten atomic leaves. Every leaf closes one silent-failure
mode that was observed in real judged trial data, then fixed and re-measured.

**`skills/probe-only/`** is the pack built entirely from exhaustive API probing, with no
trial-and-fix loop behind it. Router `using-trinity-skills`, six atomic leaves. It was smoke
tested at a single trial, looked complete, and then scored 0.543 on a full 140-trial run against
0.79 for the trial-derived pack on the same model the same week. It is published here because
the failure is the point. Probing finds the traps a system sets. It does not find the traps an
agent sets for itself, and a green checklist says nothing about the second kind.

If you are reusing one of these, reuse `trial-derived/`.

---

## Using it

Trinity attaches to any harness that can inject standing instructions before the first tool
call. There is no runtime to install.

```bash
harbor run -p tasks -a claude-code -m <model> \
  --mcp-config mcp.json \
  --skill ./skills/trial-derived \
  -k 10 -n 3 --yes
```

The router must be guaranteed to load before the first tool call. If it loads after, the agent
has already made the mistake the router exists to prevent.

---

## Contributing

The useful contribution is a **finding**, not a feature. If you probed a real system and found
a place where it hands back a confidently wrong answer with a 200, that is worth more than any
amount of framework code. Open an issue with the exact call, the exact wrong response, and the
exact correct one.

## License

MIT.
