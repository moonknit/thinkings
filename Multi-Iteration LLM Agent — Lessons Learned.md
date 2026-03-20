# Multi-Iteration LLM Agent — Lessons Learned (Design Notes)

This document summarizes key insights discovered while building a multi-iteration LLM agent for development / workflow automation.
These are practical notes from implementation experience.

---

## 1. Natural language goals do NOT give stable termination

Naive loop:

goal: "Fix the problem"
loop:
  think
  act
  observe
  judge(done?)

This is unstable.

Termination must be predicate-based.

Example:

done = tests_passed && file_generated && errors == 0

Not:

done = "looks correct"

---

## 2. Quality evaluation is unstable — coverage evaluation is stable

Bad:

Is the result good?
Is it complete?
Is this enough?

Good:

Does result contain required fields?
Did we collect required data?
Did all plan steps finish?

Use coverage, not quality.

success = required_items ⊆ result

---

## 3. Separate PLAN and NON-PLAN queries

Not all queries need planning.

Types:

REPORT
ANSWER
SEARCH
PLAN
VERIFY
GENERATE
FIX
ANALYZE

Planner should not run for everything.

---

## 4. Intent classification must happen BEFORE iteration

Termination depends on intent.

REPORT -> enough data
ANSWER -> answer exists
PLAN -> steps done
FIX -> tests pass
VERIFY -> condition checked

Without intent, termination becomes unstable.

---

## 5. Use STATE, not raw history

Bad:

history += output

Good:

history -> reducer -> state

Example state:

state = {
  intent,
  plan,
  collected_data,
  required_data,
  status
}

---

## 6. Use reducer every iteration

Pattern:

raw_output
-> reducer
-> state
-> reasoning

Reducer tasks:

- summarize
- filter
- update state
- track coverage
- normalize output

Use cheap model if possible.

---

## 7. Do NOT let LLM decide termination

Bad:

LLM: should we stop?

Good:

if state.status == DONE:
    stop

Termination must be rule-based.

---

## 8. Reduce context every iteration

Bad:

turn1 + turn2 + turn3 + turn4

Good:

history
-> reduce
-> state
-> next iteration

This reduces cost and drift.

---

## 9. Cheap model for reducer, expensive for reasoning

Cheap:

summarize
extract
classify
normalize

Expensive:

plan
solve
decide
generate

Split roles.

---

## 10. Final architecture

query
-> intent classifier
-> plan or direct

loop:
    execute
    reducer
    update state
    coverage check
    termination predicate

Design like a state machine, not a conversation.