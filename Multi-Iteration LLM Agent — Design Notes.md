# Multi-Iteration LLM Agent — Design Notes

Practical lessons learned while building a stable multi-iteration LLM agent.

---

## 1. Termination must be explicit

Natural language goals do not produce stable termination.

Bad:

goal: "fix the problem"
loop:
  think
  act
  judge(done?)

Good:

done = tests_passed && file_generated && errors == 0

Use predicate-based termination, not semantic judgment.

Coverage checks are more stable than quality checks.

success = required_items ⊆ result

---

## 2. Intent must be classified before iteration

Different query types require different termination rules.

Examples:

REPORT
ANSWER
SEARCH
PLAN
VERIFY
GENERATE
FIX
ANALYZE

Planner should not run for all queries.

Termination depends on intent.

---

## 3. Use state, not raw history

History-based loops drift and become expensive.

Bad:

history += output

Good:

history -> reducer -> state

Example:

state = {
  intent,
  plan,
  collected_data,
  required_data,
  status
}

Use canonical state every iteration.

---

## 4. Reducer is required for multi-iteration

Each iteration should:

raw_output
-> reducer
-> state
-> reasoning

Reducer tasks:

- summarize
- filter
- update state
- track coverage
- normalize

Use cheap model if possible.

---

## 5. Reduce context every iteration

Without reduction:

turn1 + turn2 + turn3 + turn4

With reduction:

history
-> reduce
-> state
-> next iteration

This reduces cost, latency, and drift.

---

## 6. Separate cheap model and reasoning model

Cheap model:

summarize
extract
classify
normalize

Expensive model:

plan
solve
decide
generate

This improves cost and stability.

---

## 7. Design as a state machine

Stable multi-iteration agents behave like state machines.

Example:

query
-> intent classifier
-> planner or direct

loop:
    execute
    reducer
    update state
    coverage check
    termination predicate

Do not design as a conversation loop.

Design as a state-driven workflow.