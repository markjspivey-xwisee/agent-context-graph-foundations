# Causal Affordance Specification

This document defines the causal semantics extension for affordances.
It specifies how `causalSemantics` is represented and used.

## Data Model

An affordance MAY include a `causalSemantics` object with:

- `interventionLabel` (string, required)
  - do-calculus notation, e.g., `do(Action, {params})`
- `outcomeVariables` (string[], required)
  - outcome variables the evaluator will predict
- `causalModelRef` (string, required)
  - URI or URN referencing the causal model
- `evaluatorEndpoint` (string, optional)
  - endpoint for causal evaluation

## Requirements

- The broker MUST NOT embed causal reasoning in the LLM.
- The broker MAY call an external evaluator via `evaluatorEndpoint`.
- Outcome constraints MUST be evaluated against predicted outcomes.

## Mapping

- JSON Schema: `spec/context-graph.schema.json#/\$defs/CausalSemantics`
- Ontology: `acg:CausalSemantics` in `spec/ontology/acg-core.ttl`
