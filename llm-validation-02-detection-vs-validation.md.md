Detection vs Validation in LLM Evaluation

> LLM-based free-form judging can be useful for large-scale behavior elicitation, but without an explicit rubric it cannot serve as a reliable validation mechanism.



This document builds upon the previous discussion of rubric-based evaluation and clarifies a critical but often conflated distinction in LLM evaluation systems: detection versus validation. Using Anthropic's Bloom framework as a reference point, we articulate why certain evaluation approaches appear imprecise by design, and how rubric-based methods complement rather than replace them.


---

1. Two Different Problems, One Overloaded Word: "Evaluation"

In practice, the term evaluation is used to describe two fundamentally different objectives:

Detection (Elicitation): Discovering whether a model can exhibit a certain behavior under some conditions.

Validation (Verification): Determining whether a system meets a predefined standard or requirement.


These objectives impose very different constraints on how judgment should be performed.


---

2. Bloom as a Detection-Oriented Framework

Bloom is best understood as a behavior elicitation and exploration framework, not a validation tool.

Its core characteristics are:

Loose, prompt-defined judgment criteria

LLM-based free-form judging

High-volume sampling and statistical aggregation

Tolerance for noisy, imperfect individual judgments


The primary signal Bloom extracts is not an absolute score, but a rate:

Elicitation rate: how often a behavior emerges across many generated scenarios


In this context, individual judgment errors are acceptable as long as aggregate trends remain informative.


---

3. Why Free-Form LLM Judging Fails as Validation

Free-form LLM judging typically follows a pattern such as:

> "Consider intent, severity, and persistence, then assign a score."



While convenient, this approach has structural limitations when used for validation:

Implicit criteria: Evaluation factors are named but not operationally defined

No scoring semantics: Scores lack explicit meaning or thresholds

Low reproducibility: Small prompt or temperature changes alter outcomes

Poor explainability: Decisions cannot be reliably traced to specific causes


As a result, the judgment itself becomes a generative artifact rather than a measurement.


---

4. Rubrics as the Missing Structural Layer

A rubric externalizes judgment criteria into explicit, inspectable components.

Typical rubric elements include:

Discrete evaluation criteria

Binary or bounded scoring rules per criterion

Defined aggregation logic (e.g., sums, weights, thresholds)

Clear pass/fail or grading semantics


When a rubric is introduced, the role of the LLM shifts:

From holistic judge

To criterion-level signal extractor


This constrains model freedom while preserving its semantic understanding capabilities.


---

5. Layering Rubrics on Top of Bloom

Bloom’s architecture implicitly invites rubric augmentation.

Conceptually:

1. Bloom generates diverse scenarios


2. The target model produces rollouts


3. Rubric-defined criteria are evaluated per rollout


4. Deterministic code aggregates criterion-level scores



In this hybrid approach:

Bloom handles coverage and exploration

Rubrics handle objectivity and validation


This separation preserves Bloom’s strengths while making the results actionable in QA, CI, and compliance contexts.


---

6. Choosing the Right Tool for the Right Goal

Dimension	Detection (Bloom-style)	Validation (Rubric-based)

Goal	Discover risky behaviors	Verify requirements
Judgment	Noisy, probabilistic	Constrained, structured
Trust unit	Aggregate statistics	Individual evaluations
Reproducibility	Low per-sample	High by design
Use cases	Safety research, exploration	QA, CI, release gating



---

7. Conclusion

Free-form LLM judging is not inherently flawed; it is simply optimized for a different problem.

Bloom demonstrates how LLMs can act as powerful sensors for large-scale behavior elicitation. However, without explicit rubrics, such systems cannot provide the determinism, explainability, or guarantees required for validation.

A robust evaluation strategy recognizes this distinction and deliberately combines both approaches where appropriate.