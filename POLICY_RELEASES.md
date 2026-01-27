# Release and Change Policy (Foundations)

This policy governs changes to the ACG foundations contract (principles, specs, protocol, ontology, and SHACL).

## Versioning

- Semantic versioning for contract artifacts.
- PATCH: clarifications or non-breaking constraints.
- MINOR: additive fields, optional endpoints, new shapes.
- MAJOR: breaking changes, removals, or changed semantics.

## Compatibility

- Additive schema changes should preserve backward compatibility.
- Breaking changes require a MAJOR version bump and a migration note.
- Deprecated fields must remain for at least one MINOR cycle before removal.

## Required Updates per Change

- Update relevant JSON Schema and SHACL shapes.
- Update ontology (acg-core.ttl) for new classes/properties.
- Update protocol/API.md for new endpoints or payloads.
- Update examples to reflect the change.

## Review Checklist

- Schema changes include examples and tests.
- Policy/semantics changes are documented in principles.
- Trace implications are covered in prov-trace.schema.json.

## Migration Notes

- Every breaking change must ship a migration guide.
- Include detection guidance and remediation steps.

## Approval

- Contract-breaking changes require principal engineer approval.
- Protocol changes require security review.
