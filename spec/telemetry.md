# Telemetry Specification

This document defines the telemetry events required for measuring affordance label stability,
drift, and polysemy.

## Event Types

### 1. AffordanceOffered
Emitted when an affordance is included in a context graph.

Required fields:
- `contextId`
- `affordanceId`
- `rel`
- `relVersion`
- `actionType`
- `agentType`
- `timestamp`

### 2. AffordanceTraversed
Emitted when an affordance is traversed. This maps to the `usageEvent`
object embedded in the PROV trace.

Required fields:
- `traceId`
- `contextId`
- `affordanceId`
- `usageRel`
- `usageRelVersion`
- `usageActionType`
- `usageOutcomeStatus`
- `usageTimestamp`

### 3. AffordanceDisabled
Emitted when an affordance is present but disabled.

Required fields:
- `contextId`
- `affordanceId`
- `disabledReason`
- `timestamp`

## Storage Requirements

- Telemetry MUST be append-only.
- Telemetry MUST be queryable by `rel`, `relVersion`, and `actionType`.

## Notes

Telemetry is used for emergent semiotics analysis and does not grant authority.

## Aggregation

Usage telemetry MAY be aggregated into a `usageSemantics` object on affordances,
including stability, drift, and polysemy metrics.
