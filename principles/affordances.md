# Affordance Theory

## Overview

Affordances are **relational properties** that describe what actions are possible given:
1. An agent's capabilities
2. The environment's state
3. Governing norms and policies

An affordance is not intrinsic to either the agent or the environment—it emerges from their relationship.

## Key Principles

### 1. Actions Are Not Intrinsic
An agent cannot simply "have" the ability to perform an action. Whether an action is possible depends on:
- What the agent is (AAT)
- What credentials the agent has (VCs)
- What the current situation allows (context)
- What policies permit (deontic constraints)

### 2. Affordances Are Discovered, Not Assumed
Agents MUST NOT assume what actions are available. They must:
1. Request a Context Graph from the broker
2. Examine the affordances provided
3. Act only through those affordances

### 3. Absence = Enforcement
If an affordance is not present in the Context Graph, the action is not possible. This is the fundamental safety property:
- Illegal actions are not "blocked"—they never exist
- The agent cannot attempt what is not afforded

## Affordance Structure

```json
{
  "id": "aff-emit-plan",
  "rel": "emit-plan",
  "relVersion": "1.0.0",
  "actionType": "EmitPlan",
  "target": {
    "type": "HTTP",
    "href": "https://broker.example.com/plans",
    "method": "POST"
  },
  "params": {
    "type": "json-schema",
    "schema": { ... }
  },
  "requiresCredential": [
    { "schema": "PlannerCapability" }
  ],
  "effects": [
    { "type": "resource-create", "description": "Creates a plan" }
  ],
  "causalSemantics": {
    "interventionLabel": "do(EmitPlan, params)",
    "outcomeVariables": ["plan_quality", "time_to_complete"]
  },
  "enabled": true
}
```

In the hypergraph view of a Context Graph, each affordance is represented as a **hyperedge** that binds
the agent, target, credentials, constraints, and effects in a single relation.

### Components

- **id**: Unique identifier within the context
- **rel**: Semantic relation (hypermedia link relation)
- **relVersion**: Version of the relation semantics
- **actionType**: The AAT action type
- **target**: Where/how to perform the action
- **params**: Schema for required parameters
- **requiresCredential**: VCs needed to traverse
- **effects**: Declared effects of traversal
- **causalSemantics**: For causal reasoning (optional)
- **usageSemantics**: Usage-based semantics signals (optional)
- **enabled**: Whether currently traversable

## Target Types

| Type | Description |
|------|-------------|
| HTTP | Standard HTTP request |
| DIDComm | DIDComm messaging |
| OID4VCI | OpenID for Verifiable Credential Issuance |
| Internal | Internal system operation |
| EventEmit | Event emission |

## Credential Gating

Affordances may require specific credentials:

```json
{
  "requiresCredential": [
    {
      "schema": "PlannerCapability",
      "issuer": "did:web:authority.example.com"
    }
  ]
}
```

If the agent lacks the required credential:
1. The affordance does not appear
2. Instead, a `request-credential` affordance may appear
3. Agent must obtain credential before the action becomes available

## Causal Semantics

For actions with predictable outcomes, affordances may include causal semantics:

```json
{
  "causalSemantics": {
    "interventionLabel": "do(MigrateSchema, {targetVersion: $params.targetVersion})",
    "outcomeVariables": ["downtime_minutes", "data_loss_probability"],
    "causalModelRef": "urn:causal-model:db-migration-v2",
    "evaluatorEndpoint": "https://causal.example.com/evaluate"
  }
}
```

This enables:
- Outcome prediction before action
- Policy evaluation based on predicted effects
- Counterfactual analysis after action
