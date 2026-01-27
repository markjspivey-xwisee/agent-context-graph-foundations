# ADR 0001: Split Foundations from Implementation

## Status
Accepted

## Context

The original repository mixed principles, specs, and implementation code. This made contract governance difficult, slowed reviews, and blurred responsibilities between specification changes and runtime behavior.

## Decision

Create two public repositories:

- agent-context-graph-foundations: principles, architecture, protocol, and specs
- agent-context-graph-implementation: runtime code, demos, tests, and examples

Implementations consume the foundations contract via a resolved spec path (ACG_SPEC_DIR) and CI clones the foundations repo for validation.

## Consequences

- Contract changes are isolated and reviewable.
- Implementation can move faster while staying conformant.
- CI requires spec wiring for tests and schema validation.

## Alternatives Considered

- Keep monorepo and use folders only: rejected due to governance ambiguity.
- Git submodules: rejected due to friction for contributors.

## References

- spec/context-graph.schema.json
- spec/prov-trace.schema.json
- protocol/API.md
