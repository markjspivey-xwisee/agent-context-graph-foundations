# Abstract Agent Types (AAT)

## Overview

An **Abstract Agent Type** is a behavioral contract for an agent, defined by its capabilities, perceptions, actions, and invariants—independent of its internal architecture.

The relationship is:
```
ADT : Data :: AAT : Agency
```

Just as Abstract Data Types define what operations can be performed on data without specifying implementation, Abstract Agent Types define what an agent can do without specifying how it does it.

## Formal Definition

An Abstract Agent Type specifies:

### 1. Percept Space
What the agent can observe:
- Messages
- Sensory inputs
- State deltas
- Documents
- Events

### 2. Action Space
What the agent is allowed (and forbidden) to do:
- Allowed actions with their required capabilities
- Forbidden actions with rationale

### 3. Internal State (Abstract)
What information the agent may carry:
- Knowledge
- Goals
- Commitments
- Context

### 4. Transition Semantics
The mapping from percepts and state to actions:
```
(percept, state) → (action*, new_state)
```

### 5. Behavioral Invariants
Laws the agent must preserve:
- Safety properties ("never do X")
- Integrity properties ("always maintain Y")

## Standard Agent Types

### Observer
- **Purpose**: Watch and report; no side effects
- **Allowed**: Report, Summarize, Flag, Query
- **Forbidden**: Actuate, ModifyState, EmitPlan
- **Key Invariant**: No external side effects

### Planner
- **Purpose**: Produce plans, not execute
- **Allowed**: EmitPlan, RequestInfo, RevisePlan, ValidatePlan
- **Forbidden**: Actuate, WriteExternal
- **Key Invariant**: Planner must never directly actuate

### Executor
- **Purpose**: Perform actions based on authorized plans
- **Allowed**: Act, ReportOutcome, RequestAuthorization, RollbackAction
- **Forbidden**: EmitPlan, BypassAuthorization, SilentAction
- **Key Invariant**: All actions require explicit authorization

### Arbiter
- **Purpose**: Approve/deny/transform proposed actions
- **Allowed**: Approve, Deny, Modify, Escalate, RequestClarification
- **Forbidden**: Execute, SilentApproval, ForbiddenActionPass
- **Key Invariant**: Never approve forbidden actions

### Archivist
- **Purpose**: Store and retrieve traces/knowledge
- **Allowed**: Store, Retrieve, Summarize, Index, Version
- **Forbidden**: Delete, Mutate, FabricateProvenance
- **Key Invariant**: Traces are append-only

## AAT Enforcement

The critical invariant:

> **Every affordance surfaced in a Context Graph MUST be type-safe with respect to the agent's AAT.**

This means:
- If an affordance violates the AAT, it must not exist
- Not "exist but be blocked later"
- Not "exist but agent is expected to behave"
- **Absence = safety**

## Files

- `../spec/aat/*.json` - AAT definitions
- `../spec/shacl/aat-safety.ttl` - SHACL shapes for AAT enforcement
