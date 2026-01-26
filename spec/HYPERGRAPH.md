# Context Hypergraph Specification

This document defines the optional hypergraph view of a Context Graph.
It is normative for any payload that includes a `hypergraph` field.

## Data Model

- `hypergraph.nodes` MUST be an array of hypernodes.
- `hypergraph.hyperedges` MUST be an array of hyperedges.
- Each hyperedge MUST connect two or more hypernodes.
- `hyperedge.affordanceRef`, when present, MUST reference an affordance id in the same context.

## Mapping

- `acg:Hypergraph` maps to `hypergraph`.
- `acg:Hypernode` maps to entries in `hypergraph.nodes`.
- `acg:Hyperedge` maps to entries in `hypergraph.hyperedges`.
- `acg:connectsNode` maps to `hyperedge.connects`.

## Notes

The hypergraph view is a lossless projection of the JSON-LD Context Graph when role bindings are included.
