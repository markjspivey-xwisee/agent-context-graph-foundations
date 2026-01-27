# Agent Context Graph Foundations

Canonical principles, architecture, protocol, and specification layers for Agent Context Graph (ACG).

This repo is the source of truth for the theory and formal artifacts that the implementation consumes.

## What's here

- principles/  
  Core principles: AAT, affordances, hypermedia, hypergraph, category theory, semiotics, policy, PROV, DI/VC, threat model, knowledge graphs, semantic layer.
- architecture/  
  System architecture, interfaces, and inspiration notes.
- protocol/  
  API-level protocol documentation.
- spec/  
  JSON Schema, SHACL shapes, Hydra docs, ontologies, and formal specs.

## Start here

- OVERVIEW.md
- POLICY_RELEASES.md
- adr/README.md
- principles/README.md
- principles/knowledge-graphs.md
- principles/semantic-layer.md
- architecture/ARCHITECTURE_INDEX.md
- protocol/API.md
- spec/context-graph.schema.json
- spec/prov-trace.schema.json
- spec/knowledge-graph.md
- spec/knowledge-graph.schema.json
- spec/semantic-layer.md
- spec/ontology/acg-core.ttl
- spec/ontology/semantic-layer.ttl
- spec/shacl/context.ttl
- spec/shacl/semantic-layer.ttl
- spec/hydra/api.ttl
- spec/HYPERGRAPH.md
- spec/CATEGORY-THEORY.md
- spec/telemetry.md
- spec/causal-affordance.md
- spec/cgp.md

## Foundations highlighted

- Hypergraph semantics for multi-party affordances
- Category-theoretic composition of affordance workflows
- Usage-based semiotics (meaning emerges from usage)
- Ephemeral Context Graphs + persistent Knowledge Graphs
- HyprCat upper ontology for semantic catalogs (DCAT + DPROD + Hydra + SHACL)
- Tool authoring through explicit affordances
- PROV tracing, DID/VC, and causal affordance semantics

## Related repositories

- Foundations: https://github.com/markjspivey-xwisee/agent-context-graph-foundations
- Implementation: https://github.com/markjspivey-xwisee/agent-context-graph-implementation

## How this maps to implementation

The implementation repo reads specs from this repo. If you have both repos cloned side by side:

  devstuff/
    agent-context-graph-foundations/
    agent-context-graph-implementation/

The implementation will resolve specs from the foundations repo automatically. You can also set
ACG_SPEC_DIR to point at this repo's spec directory.

## Agent rules

See AGENTS.md for non-negotiable agent constraints.
