# Dynamic State Graph: A Lightweight Cognitive Layer for LLM Agents

## Abstract

Current LLM-based agents primarily rely on two sources of context:

1.  The current conversation history.
2.  Retrieved long-term memory.

While this approach enables long-context reasoning, it lacks an explicit
representation of the agent's continuously evolving cognitive state.

This proposal introduces the concept of a **Dynamic State Graph (DSG)**,
a lightweight cognitive layer positioned between user interaction and a
large language model.

Instead of treating context as a sequence of text, DSG models the
conversation as a continuously evolving graph of active entities,
concepts, goals, and relationships.

The LLM focuses on reasoning, while the DSG continuously maintains what
is currently important.

------------------------------------------------------------------------

## Motivation

Human conversations are not driven directly by raw conversation history.

Instead, humans maintain an internal cognitive state consisting of:

-   Current goals
-   Active concepts
-   Conversation focus
-   Recently recalled information
-   Relationships between concepts

Conversation history merely acts as an event stream that updates this
internal state.

Current LLM agents largely skip this intermediate representation.

------------------------------------------------------------------------

## Core Idea

Replace:

``` text
Conversation History
        ↓
      LLM
```

with:

``` text
Conversation
      ↓
Dynamic State Graph
      ↓
Memory Retrieval
      ↓
LLM
```

The **Dynamic State Graph** becomes the primary working context.

Conversation history becomes only an event stream used to update the
graph.

------------------------------------------------------------------------

## Dynamic State Graph

Each entity inside the graph maintains multiple cognitive attributes.

``` text
Entity: "Working Memory"

activation = 0.96
importance = 0.98
confidence = 0.91
stability = 0.82
novelty = 0.35

last_updated_turn = 42

links:
    Memory (0.83)
    Agent (0.94)
    State (0.97)
```

The graph continuously evolves after every conversation turn.

------------------------------------------------------------------------

## Entity Activation

Activation increases when:

-   The entity is explicitly mentioned.
-   The entity is recalled.
-   Connected entities become active.
-   The current goal depends on it.

Activation naturally decays.

Rather than using a constant decay:

``` text
effective_decay =
base_decay × (1 - importance)
```

Highly important concepts remain active much longer.

------------------------------------------------------------------------

## Turn-Based State Evolution

Instead of using wall-clock time, DSG updates occur after every
conversation turn.

``` text
Turn 15
Working Memory

↓

Turn 16
Agent Architecture

↓

Turn 17
Dynamic State Graph
```

Conversation turns, rather than elapsed time, drive cognitive evolution.

------------------------------------------------------------------------

## Recall Mechanism

Humans rarely search raw conversation history.

Instead, dormant concepts become reactivated.

``` text
Recall
    ↓
activation += recall_boost
    ↓
linked entities become active
    ↓
optional memory retrieval
```

Memory retrieval becomes **state-driven** rather than **query-driven**.

------------------------------------------------------------------------

## Relationship Graph

Entities are connected through weighted relationships.

``` text
Working Memory
        │
        ▼
Dynamic State
        │
        ▼
Agent Architecture
```

Relationship strengths evolve continuously.

``` text
edge_weight += repeated_cooccurrence

edge_weight -= gradual_decay
```

The graph captures conceptual structure instead of simple conversation
order.

------------------------------------------------------------------------

## Multi-Dimensional Cognitive Scores

Each entity may maintain multiple dimensions:

-   Activation
-   Importance
-   Confidence
-   Stability
-   Novelty
-   Prediction Weight

Additional cognitive signals can be introduced as needed.

------------------------------------------------------------------------

## Working State vs Memory

### Long-Term Memory

Stores knowledge.

Examples:

-   User preferences
-   Previous projects
-   Facts
-   Persistent knowledge

### Dynamic State Graph

Represents:

> What is currently important?

It is continuously reconstructed from:

-   Conversation
-   Recalled memories
-   Current goals
-   Reasoning outcomes

------------------------------------------------------------------------

## Client-Side Micro Agent

One possible deployment architecture:

``` text
User
   │
   ▼
Local sLLM
(Dynamic State Manager)
   │
   ▼
Large Remote LLM
```

The lightweight local model continuously:

-   Updates the graph
-   Propagates activation
-   Applies decay
-   Predicts future topics
-   Generates compact state summaries

The remote LLM focuses purely on reasoning.

------------------------------------------------------------------------

## Potential Advantages

-   Reduced prompt length
-   Better long conversations
-   Human-like topic persistence
-   Efficient recall
-   Dynamic importance tracking
-   Improved personalization
-   Lower server computation
-   Better agent reasoning

------------------------------------------------------------------------

## Future Directions

Possible extensions include:

-   Hierarchical state graphs
-   Episodic subgraphs
-   Predictive activation
-   Emotional salience
-   Reinforcement-based importance updates
-   Graph neural representations
-   Multimodal state integration

------------------------------------------------------------------------

## Summary

The central hypothesis is:

> Large Language Models should reason over a continuously evolving
> cognitive state rather than raw conversation history.

Memory stores knowledge.

The **Dynamic State Graph** maintains the present.

Reasoning emerges from the interaction between both.


---

# Next Exploration

The current proposal focuses on introducing the **Dynamic State Graph (DSG)** as a cognitive layer between conversation history and LLM reasoning.

The next step is to formalize the architecture into a complete cognitive system.

## 1. Dynamic State Graph Data Model

Define the internal graph representation.

Possible node attributes:

- Activation
- Importance
- Confidence
- Stability
- Novelty
- Prediction Weight
- Emotional Salience (optional)

Possible edge attributes:

- Association Strength
- Dependency
- Causality
- Temporal Relation
- Decay Rate

---

## 2. State Evolution Algorithm

Formalize how the graph evolves after each conversation turn.

Potential components:

- Entity extraction
- Entity merging
- Activation propagation
- Importance update
- Relationship update
- Graph pruning
- Recall handling

---

## 3. Memory Integration

Rather than querying memory directly:

```
Current State
        ↓
State Analysis
        ↓
Memory Retrieval
        ↓
State Reconstruction
        ↓
LLM
```

Memory should become a supporting component instead of the primary conversational context.

---

## 4. Working State vs Long-Term Memory

Clearly separate:

- Working State
- Episodic Memory
- Semantic Memory
- Procedural Memory

and define interactions between them.

---

## 5. Hierarchical State Graph

Investigate whether multiple graph layers are beneficial.

Example:

```
Conversation Layer

↓

Task Layer

↓

Project Layer

↓

Long-Term User Model
```

This enables persistent reasoning across long-running conversations and projects.

---

## 6. Predictive State

Introduce future-oriented cognition.

Each state estimates:

- probable next topics
- expected user intent
- likely future recalls

Prediction can proactively activate relevant concepts before they are explicitly requested.

---

## 7. Client-Side Cognitive Agent

Investigate whether a lightweight local sLLM can continuously manage the Dynamic State Graph.

Responsibilities may include:

- State updates
- Activation propagation
- Decay management
- Recall handling
- Prediction
- Prompt state generation

The large remote LLM would focus exclusively on reasoning.

---

## 8. Evaluation

Potential evaluation metrics:

- Long conversation consistency
- Topic continuity
- Recall accuracy
- Prompt compression ratio
- Token efficiency
- Response latency
- User satisfaction
- Adaptive reasoning quality

---

## Long-Term Vision

Replace the traditional pipeline:

```
Conversation History
        ↓
Prompt
        ↓
LLM
```

with a cognitive architecture:

```
Conversation
        ↓
Dynamic State Graph
        ↓
Memory
        ↓
Reasoning
        ↓
LLM
```

The ultimate goal is to shift from **history-driven reasoning** to **state-driven reasoning**, allowing LLM agents to maintain a continuously evolving internal cognitive model rather than relying solely on conversation history.