# Semantic Layer Principle

The system exposes **data access through a virtual, zero-copy semantic layer**:

- **RDF-first**: the canonical representation is RDF/OWL, not ad-hoc JSON.
- **SPARQL-first**: all semantic queries are expressed in SPARQL.
- **Federated mapping**: relational or document sources are mapped via R2RML/OBDA.
- **Metadata-driven**: sources are described with DCAT (datasets) and DPROD (data products).
- **HyprCat upper ontology**: DCAT + DPROD + Hydra + SHACL are aligned through HyprCat for reuse.
- **Hypermedia-native catalogs**: Hydra affordances describe how to browse and query datasets/products.
- **Data contracts are SHACL**: contracts link to SHACL shapes that define expected structure.
- **No hardcoded sources**: adapters are implementation details, not protocol primitives.

This ensures agent queries remain stable as source systems change, while meaning is preserved
through explicit ontologies and SHACL-validated shapes.
