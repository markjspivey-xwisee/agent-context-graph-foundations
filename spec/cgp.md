# Context Graph Profile (CGP)

This document defines the normative profile for Context Graph payloads.
It is the semantic contract between brokers and agents.

## Scope

CGP applies to all JSON-LD Context Graphs that represent affordances for an agent.
It is complementary to:
- `spec/context-graph.schema.json` (JSON Schema)
- `spec/shacl/context.ttl` (SHACL shapes)
- `spec/ontology/acg-core.ttl` (OWL ontology)

## Required Fields

A Context Graph MUST include:
- `@context`
- `id`
- `agentDID`
- `timestamp`
- `affordances`

## Affordances

- Every affordance MUST include `id`, `rel`, `actionType`, and `target`.
- Affordances MAY include `requiresCredential`, `effects`, and `causalSemantics`.
- Affordances MUST be valid against the agent's AAT.

## Hypergraph View (Optional)

If the `hypergraph` field is present, it MUST conform to `spec/HYPERGRAPH.md`.
Affordances SHOULD be represented as hyperedges binding agent, target,
credentials, and constraints.

## Category View (Optional)

If the `category` field is present, it MUST conform to `spec/CATEGORY-THEORY.md`.
Morphism composition MUST be explicit.

## Provenance

Every affordance traversal MUST emit a PROV trace that includes:
- the context snapshot
- the affordance used
- parameters provided
- outcomes observed
