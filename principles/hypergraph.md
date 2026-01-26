# Hypergraph Semantics

## Overview

Context Graphs are modeled as **hypergraphs**: a single action can relate many entities at once.
This makes multi-party constraints first-class and avoids flattening complex relations into pairwise edges.

## Core Principle

**Affordances are hyperedges.**
An affordance relates an agent, its credentials, target resources, and governing constraints in a single relation.

## Structural Requirements

1. **Multi-party relations are explicit**
   - A single affordance may bind agent, target, credentials, policies, and resources.
2. **Incidence roles are explicit**
   - Role bindings (agent, target, credential, constraint) must be available when a hyperedge is present.
3. **Hypergraph view is canonical (when provided)**
   - The hypergraph must be a faithful projection of the JSON-LD context graph.
4. **No inferred authority**
   - Hyperedges do not grant capabilities; they reflect already-validated affordances.

## Representation

When the explicit hypergraph view is included, the Context Graph contains a `hypergraph` field with:

- `nodes`: hypernodes referencing agents, affordances, credentials, constraints, and resources
- `hyperedges`: hyperedges connecting two or more hypernodes

Example:

```json
{
  "hypergraph": {
    "nodes": [
      { "id": "hn-agent", "kind": "Agent", "ref": "did:key:..." },
      { "id": "hn-aff-emit", "kind": "Affordance", "ref": "aff-emit-plan" }
    ],
    "hyperedges": [
      {
        "id": "he-emit-plan",
        "relation": "affordance",
        "affordanceRef": "aff-emit-plan",
        "connects": ["hn-agent", "hn-aff-emit"],
        "roles": { "agent": "hn-agent", "affordance": "hn-aff-emit" }
      }
    ]
  }
}
```

## Why Hypergraphs

- **Safety**: constraints, credentials, and targets are bound in one relation
- **Clarity**: no hidden edge semantics
- **Composition**: supports category-theoretic composition without ambiguity
