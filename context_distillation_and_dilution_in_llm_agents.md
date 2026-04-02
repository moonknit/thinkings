# Context Distillation and Dilution in LLM Agents

## Append-Only History vs. Summary Compaction: A Practical Technical Comparison

**Document Type**: Technical Analysis  
**Domain**: LLM Agent Engineering  
**Version**: 2026

## 1. Overview

This document compares two broad approaches to context management in LLM-based ReAct (Reasoning + Acting) agents:

- `append-only history`: keep replaying the growing transcript
- `summary compaction`: periodically replace accumulated history with a bounded state summary

The central claim is intentionally narrower than "compaction is always better." For long-horizon, tool-heavy agent loops, append-only history tends to suffer from **context dilution**: as more observations accumulate, the ratio of useful state to irrelevant residue gets worse. In these settings, some form of summary compaction is often the more robust default.

The corresponding positive idea is **context distillation**: after each step or compaction interval, preserve the state the next step actually needs rather than the full transcript of how the current step got there.

## 2. Scope and Evidence Level

This document combines three types of claims:

- `Observed behavior`: patterns commonly seen in long-horizon, tool-using agent loops
- `Literature-backed claims`: conclusions supported by the cited papers or vendor documentation
- `Reasoned design guidance`: architectural conclusions inferred from those observations and sources

It is not a proof that append-only history is universally inferior. The comparison is most applicable to:

- multi-step ReAct-style agents
- tasks with large tool outputs or web/page observations
- sessions long enough for context growth and attention dilution to matter

It is less applicable to:

- short chats
- shallow Q&A tasks
- systems that prioritize full transcript fidelity over bounded inference state

## 3. Background: The ReAct Loop

ReAct (Yao et al., 2023) is a common agentic pattern in which an LLM repeatedly cycles through:

| Step | Phase | Agent Action | Context Impact |
| --- | --- | --- | --- |
| 1 | Thought | Generate reasoning from the current context | Adds reasoning tokens |
| 2 | Action | Execute a tool call such as search, code, or API access | Adds tool call tokens |
| 3 | Observation | Receive environment feedback | Adds observation tokens, often the largest component |

In many practical agents, observations dominate context growth. Search results, crawled pages, code execution output, logs, and raw documents often dwarf the model's own reasoning tokens within only a few iterations.

## 4. Append-Only History

### 4.1 Mechanics

In an append-only design, every new API call replays the prior trajectory:

```text
Turn N context: [System Prompt] + [Tools] + [Q1][A1][Q2][A2]...[Qn-1][An-1][Qn]
```

The main appeal is simplicity. The agent does not need a separate state representation because the transcript itself is the state.

### 4.2 Strengths

- Minimal implementation complexity
- Natural auditability because the full trajectory is preserved
- Strong intra-session cache locality when the provider reuses a stable prefix
- Easier debugging because failures can often be reconstructed from raw history alone

### 4.3 Failure Modes

The main weakness is not just token overflow. It is **context dilution**: relevant state becomes harder to recover from a transcript increasingly filled with stale tool output, failed attempts, resolved subproblems, and repeated observations.

Typical symptoms include:

- the model revisits already-resolved branches
- failed tool attempts continue to bias later reasoning
- earlier bulky observations crowd out current decision-making
- quality drops before the hard context window is reached

This makes append-only especially fragile in tool-heavy, long-horizon loops.

### 4.4 Cache Behavior

Append-only often benefits from stable-prefix caching within a session. That benefit is real, but it does not eliminate the cost of replaying a large context, and it can disappear when:

- session state is resumed imperfectly
- the prompt prefix changes
- the agent must switch models or execution modes
- large observations dominate the replayed context

In other words, cache locality helps, but it does not fully solve dilution.

## 5. Summary Compaction

### 5.1 Mechanics

Summary compaction replaces accumulated transcript history with a bounded state summary:

```text
Compaction cycle: [System Prompt] + [Tools] + [Compressed State Summary] + [Current Observation] + [New Query]
```

The compacted state is not meant to preserve every intermediate detail. It is meant to preserve what the next step needs:

- goals and constraints
- decisions already made
- critical evidence that must carry forward
- unresolved questions
- the next likely action

This is where **context distillation** matters: the state becomes more semantically dense even as it becomes shorter.

### 5.2 Why It Often Helps

Summary compaction can improve long-horizon behavior because it changes the shape of the prompt the model sees:

- less irrelevant residue competes for attention
- important evidence can be made explicit instead of buried in logs
- cost growth becomes policy-bounded rather than transcript-bounded
- the next step operates on a state representation rather than a raw transcript

This does not guarantee better performance. It means the system has a better chance of remaining legible to the model as the task continues.

### 5.3 Working Memory vs. Persistent State

A useful design distinction is to separate temporary working material from cross-step state:

| Memory Layer | Content | Lifetime |
| --- | --- | --- |
| Compacted State | Goals, decisions, essential constraints, critical evidence, current task state | Persisted across iterations |
| Temporary Working Memory | Scratch reasoning, transient observations, intermediate calculations | Current iteration only |
| Raw Tool Results | Full search/code/API output | Logged externally or discarded after compaction |

This distinction helps explain why compaction is not identical to deletion. The question is not "keep or remove everything." The question is "what belongs in persistent state versus local working context?"

## 6. Limits and Failure Modes of Summary Compaction

Summary compaction is not a free win. It introduces its own risks:

- `summary drift`: the carried state becomes gradually less faithful to the underlying evidence
- `evidence loss`: critical details may be omitted during compression
- `premature abstraction`: the summary keeps conclusions but drops the supporting facts needed later
- `state poisoning`: a mistaken intermediate summary can bias many later steps
- `debugging overhead`: the compacted state is easier to run on, but harder to audit unless full traces are logged separately

Because of these failure modes, summary compaction should be evaluated not only on token savings, but also on:

- final answer quality
- recovery after wrong turns
- robustness to tool errors
- ability to resume correctly from compacted state alone

## 7. Head-to-Head Comparison

| Dimension | Append-Only (Full History) | Summary Compaction |
| --- | --- | --- |
| Context Growth | `O(n)` with iteration count | Policy-bounded if the summary is actively capped and refreshed |
| Cache Efficiency | Often strong within a stable session | Typically weaker stable-prefix reuse because compacted state changes |
| Attention Profile | Tends to dilute as history accumulates | Can remain more focused if summary quality stays high |
| Token Cost | Usually grows with transcript length | Usually more stable, though not literally constant |
| Output Quality | Can degrade in long, noisy trajectories | Can improve in long-horizon settings, but depends on compaction quality |
| Error Carry-Over | Raw failed attempts remain visible | Mistakes can be filtered, but bad summaries can propagate |
| Scalability | Bound by transcript growth and context window limits | Bound more by compaction policy and summary quality than by raw history length |
| Debugging | Easier if the raw transcript is the source of truth | Requires separate trace logging for reliable post-hoc inspection |
| Implementation | Simple | More complex; requires state design and validation |

## 8. When Append-Only Is Still Reasonable

Append-only is still a sensible choice in some scenarios:

| Scenario | Why It Can Be Reasonable |
| --- | --- |
| Short conversations | Context never grows large enough for dilution to dominate |
| Debugging and reproducibility | Full raw history is useful as the primary artifact |
| Audit-heavy workflows | Systems may need transcript fidelity for oversight or compliance |
| Shallow user support flows | The overhead of building compacted state may not be worth it |

The practical takeaway is not "never use append-only." It is closer to this:

- use append-only when the task is short, bounded, or audit-first
- prefer summary compaction when the task is long, tool-heavy, and stateful

## 9. Multi-Turn User Interaction

Summary compaction also applies to user-facing conversation systems, but the case is less clear-cut than in inner ReAct loops.

Potential advantages:

- session cost can remain more stable over time
- old dialogue can be replaced with a bounded session state
- long-running assistants can avoid replaying large inactive histories

Practical tradeoffs:

- users often expect transcript continuity, not just state continuity
- summarizing user conversation introduces fidelity and trust issues
- debugging and compliance may require the original transcript to remain first-class

So for multi-turn user interaction, the common production pattern is often hybrid:

- preserve the canonical transcript externally
- provide the model with a bounded session summary plus the most recent turns

## 10. Design Recommendations

### 10.1 Architectural Defaults

- Treat summary compaction as the default option for long-horizon, tool-heavy agent loops.
- Treat append-only replay as the default option only when transcript fidelity is the primary requirement.
- Separate persistent state from temporary working context.
- Use deterministic trimming only as a pre-processing step for obviously low-value bulk such as repeated logs, oversized raw HTML, or verbose raw traces.
- Keep full raw traces outside the model context when possible, even if the live prompt uses compacted state.

### 10.2 Compaction Quality Guidelines

- Preserve goals, constraints, critical identifiers, decisions made, and evidence that later steps may need to cite.
- Discard redundant observations, resolved branches, and bulky raw output that no longer changes the decision state.
- Prefer evidence-preserving summaries over purely abstract summaries when tool results matter.
- Regularly test whether the agent can continue correctly from the compacted state alone.

### 10.3 Monitoring

- Track how quickly raw observations grow relative to compacted state.
- Measure quality over session length instead of assuming compaction helps automatically.
- Compare cost, latency, and answer quality together.
- Inspect failure cases for summary drift, not only token overflow.

## 11. Conclusion

For short, bounded, or audit-first workflows, append-only history remains a valid and often simpler design. But for long-horizon, tool-heavy agent loops, the more important problem is usually not raw context window size alone. It is **context dilution**: too much stale transcript and too little high-value state.

That is why summary compaction is often a strong default for agent systems that must reason across many steps. Its value is not that it magically removes all tradeoffs. Its value is that it turns an ever-growing transcript into a maintained inference state. In that sense, the practical goal is not merely shorter prompts. It is better **context distillation**.

## References

- Yao, S. et al. (2023). *ReAct: Synergizing Reasoning and Acting in Language Models*. ICLR 2023.
- Zhou, et al. (2025). *MEM1: Learning to Compress Context for Long-Horizon LLM Agent Tasks*.
- Lindenbauer, T. et al. (2025). *Efficient Context Management for Software Engineering Agents*. NeurIPS 2025 Deep Learning 4 Code Workshop.
- *ReSum: Unlocking Long-Horizon Search Intelligence via Context Summarization*. arXiv:2509.13313.
- *ACON: Agent Context Optimization*. arXiv (October 2025).
- *BATS: Budget-Aware Tool Scheduling for LLM Agents* (November 2025).
