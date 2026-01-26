FSM-Based AI Agent Workflow: Core Concepts

This document defines a minimal and explicit conceptual model for managing an AI development agent using a Finite State Machine (FSM) and four strictly separated elements.

The primary objective of this architecture is control, reproducibility, and debuggability. Apparent intelligence or autonomous behavior is explicitly treated as a non-goal.


---

Overview

The agent is modeled as a deterministic system built on top of an LLM, structured around the following four elements:

1. Workflow


2. State


3. Detail Process


4. Contract (including Rulebook and Tools)



Each element has a single responsibility. Overlap or ambiguity between elements is considered a design error.


---

1. Workflow

Workflow manages the overall flow of the agent.

It defines the high-level progression of the task: which stages exist, how they are ordered, and how transitions occur.

Workflow answers the question:

> “What is the overall flow, and how does it advance?”



Scope

High-level stages of execution

Valid transition paths between stages

Start and termination points


Notes

Workflow focuses on structure, not detail

In simple systems, Workflow and Detail Process may be merged

In complex systems, separating them helps prevent context dilution



---

2. State

State records the current progress of the workflow in a concise form.

It represents where the agent is within the workflow, without embedding detailed execution data.

State answers the question:

> “Which stage of the workflow is currently active?”



Characteristics

Minimal and explicit

Tracks progress, not internal reasoning

Designed for easy inspection and transition control


Extended State

When detailed progress, artifacts, or intermediate results must be recorded, an extended state can be introduced.

This typically takes the form of:

An output file

A structured state artifact


Such extensions are conceptually still part of the state model, but are separated to keep the core state lightweight.


---

3. Detail Process

Detail Process defines the concrete execution details for each workflow stage.

It specifies what must be done within a given workflow step, including ordered actions and checks.

Detail Process answers the question:

> “What exactly should be executed at this stage?”



Scope

Step-by-step actions per workflow stage

Validation and consistency checks

Criteria for completing the current stage


Notes

Tightly coupled to Workflow stages

Can be merged with Workflow for simple flows

Separation is recommended for complex or multi-branch processes to maintain clarity and control



---

4. Contract (Rulebook, Templates, Tools)

Contract defines immutable rules and resources that govern the agent.

Unlike Workflow and Detail Process, Contract contains elements that are not naturally expressed as flow steps, but must be consistently enforced.

Contract answers the question:

> “What fixed rules, templates, or tools apply regardless of flow?”



Scope

Templates and scripts

Behavioral and structural rules

Tool definitions and invocation constraints


Notes

Treated as stable and invariant

Requires careful and conservative context referencing

Exists independently of specific workflows or processes



---

Design Principles

Each element must remain conceptually isolated

State transitions must be explicit, observable, and externally verifiable

Contracts take precedence over reasoning and process

The architecture favors predictable behavior over adaptive autonomy



---

Summary

This model treats an AI agent as a controlled execution system, not an autonomous actor.

The FSM provides structural determinism, while the four elements ensure that:

Behavior is reproducible

Failures are diagnosable

Responsibility boundaries are clear


This architecture is intended for engineering-grade reliability, not the simulation of intelligence.