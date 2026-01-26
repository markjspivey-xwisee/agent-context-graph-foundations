# Provenance & Audit (PROV)

## Overview

Every affordance traversal produces an immutable PROV trace. These traces enable:
- **Accountability**: Who did what, when, and why
- **Auditing**: Complete history of agent actions
- **Counterfactual Analysis**: What would have happened if...
- **Debugging**: Understanding agent behavior

## PROV Trace Structure

```json
{
  "@context": ["https://www.w3.org/ns/prov#", "..."],
  "id": "urn:uuid:...",
  "type": ["prov:Activity", "aat:Decision"],

  "wasAssociatedWith": {
    "agentDID": "did:key:...",
    "agentType": "aat:PlannerAgentType"
  },

  "used": {
    "contextSnapshot": { ... },
    "affordance": { ... },
    "parameters": { ... },
    "credentials": [ ... ]
  },

  "generated": {
    "outcome": { "status": "success", ... },
    "stateChanges": [ ... ],
    "eventsEmitted": [ ... ],
    "newContext": { ... }
  },

  "interventionLabel": "do(EmitPlan, {...})",
  "startedAtTime": "...",
  "endedAtTime": "...",

  "policyEvaluations": [ ... ],
  "causalEvaluation": { ... },
  "signature": { ... }
}
```

## Key Components

### wasAssociatedWith
Links the trace to the agent that performed the action:
- Agent DID for identity
- Agent Type for AAT context
- Optional instance ID for multi-instance agents

### used (Inputs)
Everything that went into the decision:
- **contextSnapshot**: State of context at decision time
- **affordance**: The affordance that was traversed
- **parameters**: User-provided parameters
- **credentials**: Credentials used for authorization

### generated (Outputs)
Everything that resulted from the action:
- **outcome**: Success/failure status
- **stateChanges**: What changed in the system
- **eventsEmitted**: Events produced
- **newContext**: Reference to post-action context

### Causal Information
- **interventionLabel**: The do(action, params) label for causal analysis
- **causalEvaluation**: Predicted outcomes if causal model was used

## Invariants

### Append-Only
Traces are never modified or deleted:
```
once(stored(trace)) → always(stored(trace) ∧ unchanged(trace))
```

### Completeness
Every traversal produces a trace:
```
∀ traversal: emitsTrace(traversal) = true
```

### Provenance Integrity
All stored records have complete PROV links:
```
∀ trace: hasCompleteProvenance(trace)
```

## Use Cases

### Audit Query
"Show me all actions by agent X in the last 24 hours"
```
GET /traces?agentDID=did:key:...&fromTime=2024-01-22T00:00:00Z
```

### Debugging
"Why did this action fail?"
- Look up trace by ID
- Examine policyEvaluations for denials
- Check credentials for missing capabilities
- Review causalEvaluation for constraint violations

### Counterfactual Analysis
"What would have happened with different parameters?"
- Retrieve original trace
- Replay with modified parameters through causal model
- Compare predicted vs actual outcomes

## Storage Considerations

- **Immutability**: Use append-only stores (event sourcing, immutable DBs)
- **Integrity**: Sign traces for tamper detection
- **Retention**: Follow tracePolicy retention periods
- **Privacy**: Consider data minimization for sensitive parameters
