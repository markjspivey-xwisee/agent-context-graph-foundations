# Policy Framework

## Overview

The policy framework provides governance over agent actions through two complementary mechanisms:

1. **Deontic Policies**: Permit/deny rules based on identity and credentials
2. **Outcome Policies**: Constraints based on predicted effects

## Deontic Policies

Deontic policies control **what agents are allowed to do** based on their identity and authority.

### Structure
```json
{
  "id": "policy:require-planner-credential",
  "type": "deontic",
  "rule": "EmitPlan requires credential: PlannerCapability",
  "appliesTo": ["EmitPlan", "RevisePlan"]
}
```

### Enforcement
Deontic policies affect **affordance existence**:
- If policy denies an action, the affordance does not appear
- Agent cannot even see the denied action
- This is "absence = enforcement"

### Common Patterns
- Require credentials for sensitive actions
- Restrict actions to specific agent types
- Time-based restrictions (business hours only)
- Resource-based restrictions (own resources only)

## Outcome Policies

Outcome policies control actions based on **predicted effects** using causal models.

### Structure
```json
{
  "type": "outcome",
  "rule": "Predicted downtime must be less than 5 minutes",
  "enforcementLevel": "strict"
}
```

### Enforcement
Outcome policies are evaluated at traversal time:
1. Agent requests traversal
2. System evaluates causal model for predicted outcomes
3. System checks outcomes against constraints
4. If constraints violated, traversal is denied

### Integration with Causal Models

```
Affordance → CausalSemantics → CausalModel → Predictions → PolicyCheck
```

The flow:
1. Affordance includes `causalSemantics` with model reference
2. At traversal, `ICausalEvaluator` runs the model
3. Predictions are checked against outcome constraints
4. Violations result in denial

## Policy Evaluation Order

1. **AAT Check**: Is action forbidden by agent type? (structural)
2. **Credential Check**: Does agent have required credentials? (deontic)
3. **Deontic Policy**: Do explicit policies permit? (deontic)
4. **Outcome Policy**: Do predicted outcomes satisfy constraints? (outcome)

Each step can result in denial. All must pass for permission.

## Escalation

When policy cannot make a clear decision:
```json
{
  "decision": "escalate",
  "reason": "Action requires human approval",
  "escalateTo": "did:web:approver.example.com"
}
```

Escalation affordances may be provided for the agent to request approval.

## Policy Registration

### Per-Agent Policies
```typescript
policyEngine.registerPolicy("did:key:agent123", {
  id: "policy:agent-specific",
  type: "deontic",
  rule: "...",
  appliesTo: ["*"]
});
```

### Global Policies
```typescript
policyEngine.registerGlobalPolicy({
  id: "policy:trace-required",
  type: "deontic",
  rule: "All actions must emit PROV trace",
  appliesTo: ["*"]
});
```

## Enforcement Levels

| Level | Behavior |
|-------|----------|
| strict | Violation blocks action |
| advisory | Violation logged, action proceeds |
| audit-only | Violation recorded for later review |

## Best Practices

1. **Default Deny**: Start with restrictive policies, add permissions
2. **Explicit Reasons**: Always provide clear denial reasons
3. **Audit Everything**: Even advisory violations should be traced
4. **Separate Concerns**: AAT for type safety, policy for governance
5. **Version Policies**: Track policy changes over time
