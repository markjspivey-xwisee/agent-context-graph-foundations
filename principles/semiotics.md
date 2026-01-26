# Emergent Semiotics & Usage-Based Meaning

## Overview

Meaning in the Agent Context Graph emerges through **use**, not through hard-coded definitions. This is based on usage-based linguistics: the meaning of a term stabilizes through patterns of successful communication.

## Core Principles

### 1. Labels Are Conventions
Affordance relation types (`rel`) are conventions that agents and systems agree upon through use:
```json
{
  "rel": "emit-plan",
  "relVersion": "1.0.0"
}
```

The meaning of "emit-plan" is not defined by a dictionary—it's defined by:
- What agents do when they traverse it
- What outcomes result
- How other agents interpret those outcomes

### 2. Meaning Drifts
Over time, how affordances are used may shift:
- New use cases emerge
- Edge cases are discovered
- Interpretations diverge

The system must track this drift to maintain coherence.

### 3. Versioning Preserves Compatibility
Relation types include versions:
```json
{
  "rel": "emit-plan",
  "relVersion": "1.0.0"
}
```

When meaning changes significantly, version increments signal the change.

## Telemetry Requirements

To support semiotic analysis, the system logs:

### Usage Events
Every affordance traversal records:
- Relation type and version
- Context at time of use
- Parameters provided
- Outcome achieved

The canonical payload is `usageEvent` embedded in the PROV trace, and aggregated
into `usageSemantics` on affordances in future contexts.

### Outcome Correlation
Track relationship between:
- What agent intended (parameters)
- What system did (effects)
- What resulted (outcomes)

### Interpretation Variance
When multiple agents use the same relation:
- Do they provide similar parameters?
- Do they expect similar outcomes?
- Do interpretations converge or diverge?

## Metrics

### Stability
How consistently is a relation used?
```
stability(rel) = 1 - variance(usage_patterns)
```

High stability = shared understanding
Low stability = meaning contested

### Drift
How much has usage changed over time?
```
drift(rel, t1, t2) = distance(usage_pattern(t1), usage_pattern(t2))
```

Monitor drift to detect semantic evolution.

### Polysemy
Does a relation have multiple distinct meanings?
```
polysemy(rel) = count(distinct_usage_clusters)
```

High polysemy = term is overloaded, may need splitting.

## No Hard-Coded Semantics

### What This Means
- Don't embed meaning in code: `if (rel === 'emit-plan') { assumePlannerBehavior() }`
- Don't assume effects: check the affordance's declared effects
- Don't guess parameters: use the provided schema

### Why This Matters
If semantics are hard-coded:
- System becomes brittle to evolution
- Agents may exploit assumed meanings
- Cross-system interoperability breaks

## Semantics Observatory (Future)

A planned component for analyzing semantic health:

```typescript
interface SemanticsObservatory {
  // Compute stability metric for a relation
  computeStability(rel: string): Promise<number>;

  // Detect semantic drift over time
  detectDrift(rel: string, window: TimeRange): Promise<DriftReport>;

  // Identify polysemous relations
  identifyPolysemy(threshold: number): Promise<string[]>;

  // Suggest version increment when drift exceeds threshold
  suggestVersionBump(rel: string): Promise<VersionSuggestion | null>;
}
```

## Current Implementation

For MVP, the system:
1. Records relation types and versions in traces
2. Logs usage outcomes
3. Can optionally include `usageSemantics` on affordances (stability/drift/polysemy)
4. Does not yet compute metrics automatically

This provides the data foundation for future semiotic analysis.

## Best Practices

1. **Version from the start**: Even if meaning is stable, version enables future evolution
2. **Declare effects**: Don't leave effects implicit
3. **Log outcomes**: Every traversal should record what actually happened
4. **Monitor usage patterns**: Watch for divergent interpretations
5. **Communicate changes**: When relation meaning evolves, bump version and document

## UsageSemantics in Context Graphs

When present, `usageSemantics` provides usage-derived signals:

```json
{
  "usageSemantics": {
    "stability": 0.86,
    "drift": 0.07,
    "polysemy": 1,
    "evidenceWindow": "P30D",
    "lastObservedAt": "2026-01-24T15:00:05Z"
  }
}
```

These signals are advisory. They do not grant authority or alter affordance validity.
