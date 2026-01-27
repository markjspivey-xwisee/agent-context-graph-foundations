# Semantic Layer Specification

This document defines the **virtual semantic layer** used for data access.
It is **RDF/OWL-based** and queried using **SPARQL**. Source systems are federated
via explicit mappings; no source system is a protocol primitive.

## Normative Requirements

1. **Canonical representation is RDF** (OWL + SHACL).
2. **Canonical query language is SPARQL**.
3. **Federation is explicit** via R2RML/OBDA mappings (no implicit SQL).
4. **Sources are described semantically** using DCAT (datasets) and DPROD (data products).
5. **Upper ontology is HyprCat** (DCAT + DPROD + Hydra + SHACL alignment).
6. **Catalogs are hypermedia-driven**: a Hydra-enabled catalog MUST expose datasets, products, and contracts.
7. **Data contracts are SHACL shapes** linked from data products (no JSON Schema as canonical).
8. **Adapters are implementation details**, not part of the protocol/spec layer.

## Catalog + Contracts (Normative)

The semantic layer MUST publish a **Hydra-enabled catalog** aligned to **HyprCat** and combining:

- **DCAT** for datasets and services.
- **DPROD** for data products and ports.
- **SHACL** for data contracts (shape constraints).
- **Hydra** for hypermedia affordances on catalogs/products/contracts.

Contracts MUST reference SHACL shapes (e.g., `sl:contractShape` or `dcterms:conformsTo`)
and MAY be versioned and status-tagged.

## QueryData Action (Normative)

`QueryData` is the canonical action for semantic data access.

### Required Parameters
- `query` (string): SPARQL query

### Optional Parameters
- `queryLanguage` (string): `sparql` (canonical); other values are adapter extensions
- `semanticLayerRef` (string): SPARQL endpoint reference
- `sourceRef` (string): HyprCat-aligned DCAT dataset or DPROD product reference
- `mappingRef` (string): R2RML/OBDA mapping reference
- `federationProfileRef` (string): federation plan/profile reference
- `resultFormat` (string): preferred result format (e.g., `application/sparql-results+json`)
- `timeoutSeconds` (integer): query timeout
- `maxRows` (integer): adapter-specific row limit

## Execution Model

1. Agent traverses `QueryData` affordance.
2. Broker routes to the configured **semantic layer** SPARQL endpoint.
3. Semantic layer resolves mappings and federates to sources (e.g., SQL warehouses), translating SPARQL to source queries.
4. Results returned as SPARQL result sets (or mapped formats).

## Zero-copy federation example (Non-normative)

- A Databricks warehouse is described as a DCAT dataset / DPROD data product in the HyprCat catalog.
- R2RML/OBDA mappings define how relational tables map to RDF classes and predicates.
- The semantic layer translates SPARQL to Databricks SQL at query time.
- No data is copied into ACG; Databricks remains the source of truth.

## Implementation Notes (Non-normative)

Implementations MAY include adapter-specific SQL execution as an **extension**,
but **protocol and spec remain SPARQL-first**.
