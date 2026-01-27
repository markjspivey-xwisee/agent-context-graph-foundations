# Knowledge Graphs vs Context Graphs

## Overview

ACG differentiates between **ephemeral Context Graphs** and **persistent Knowledge Graphs**:

- **Context Graphs** are short-lived, session-bound views of current affordances.
- **Knowledge Graphs** are long-lived, ontology-driven representations of the world that agents and humans build over time.

Both are required. Agents should use a Context Graph for *what can I do now* and a Knowledge Graph for *what do we know*.

## Ephemeral Context Graphs

**Purpose:** bounded, time-limited action surfaces.

Properties:
- Short TTL and nonces for replay protection.
- Affordances are gated by credentials and policy.
- Context is a *view* that can change per request.
- All traversals are traced via PROV.

## Persistent Knowledge Graphs

**Purpose:** durable, ontology-driven memory and shared understanding.

Properties:
- Versioned, long-lived graphs with explicit ontology alignment.
- Updated by agent actions and human stewardship.
- Supports semantic queries, lineage, and governance.
- Mappings (e.g., R2RML) connect source systems to RDF.

## Interaction between the two

- Context Graphs MAY include references to Knowledge Graphs.
- Traversals MAY emit Knowledge Graph updates.
- Usage semantics can be derived from both runtime traces and persistent graph state.

## Governance Expectations

- Context Graphs are generated dynamically and must be reproducible.
- Knowledge Graphs should be auditable with traceable provenance for updates.
- Ontology changes must be versioned and documented with migration notes.

## Tool Authoring as Knowledge Work

Agents may author tools as first-class artifacts. Tools should be:
- registered through explicit affordances,
- policy-gated and credentialed,
- versioned and discoverable,
- traceable via provenance.

This ensures tool creation remains accountable and does not violate the "no global tool list" invariant.
