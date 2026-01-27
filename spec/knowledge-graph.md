# Knowledge Graph Specification (Persistent Memory)

## Scope

This document defines how ACG distinguishes ephemeral Context Graphs from persistent, ontology-driven Knowledge Graphs.

## Definitions

- **Context Graph**: short-lived, bounded affordance view for a specific agent request.
- **Knowledge Graph**: long-lived, ontology-driven representation of the world; versioned and auditable.

## Required Properties

Knowledge Graphs MUST:
- be versioned and persist across sessions,
- declare ontology alignment (e.g., HyprCat-aligned DCAT/DPROD),
- allow mapping links (e.g., R2RML definitions),
- emit provenance on updates.

Context Graphs MAY reference a Knowledge Graph using `knowledgeGraphRef` and MAY include a lightweight `knowledgeGraphSnapshot`.

## Ontology Alignment

Common ontologies include:
- **HyprCat** as the upper ontology aligning DCAT, DPROD, Hydra, and SHACL.
- **DCAT** for catalogs, datasets, distributions, and access services.
- **DPROD** for data product concepts and lifecycle.
- **R2RML** for relational-to-RDF mappings.

## Knowledge Graph Reference (JSON shape)

```json
{
  "id": "urn:kg:default",
  "label": "Enterprise Knowledge Graph",
  "version": "2026.01",
  "ontologyRefs": [
    "https://www.w3.org/ns/dcat#",
    "https://www.omg.org/spec/DPROD/",
    "https://hyprcat.io/vocab#",
    "https://www.w3.org/ns/r2rml#"
  ],
  "queryEndpoint": "https://broker.example.com/kg/query",
  "updateEndpoint": "https://broker.example.com/kg/update"
}
```

## Knowledge Graph Snapshot (JSON shape)

```json
{
  "graphId": "urn:kg:default",
  "version": "2026.01",
  "lastUpdated": "2026-01-26T12:00:00Z",
  "summary": {
    "nodes": 15420,
    "edges": 40211,
    "datasets": 58,
    "dataProducts": 12
  }
}
```

## Provenance

Knowledge Graph updates MUST be traceable and tied to an affordance traversal.
Updates should appear as part of the PROV trace or reference a trace ID.

## Relationship to Context Graphs

- Context Graphs are computed per request and expire quickly.
- Knowledge Graphs persist, aggregate, and evolve.
- Agents should consult both: the Context Graph for what actions exist, and the Knowledge Graph for world-state.
