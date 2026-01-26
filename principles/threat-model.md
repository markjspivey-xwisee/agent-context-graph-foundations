# Threat Model & Safety Invariants

## Overview

This document identifies potential attack vectors and how the system defends against them. Safety is enforced **structurally**, not through heuristics or "best effort" filtering.

## Threat Categories

### T1: Stale Context Replay

**Attack**: Agent captures a valid context and replays it later when conditions have changed.

**Defense**:
- Contexts include `expiresAt` timestamp (default 5 minutes)
- Contexts include unique `nonce` for each request
- Broker validates context freshness before traversal

**Enforcement**: Structural - expired contexts cannot be used

### T2: Credential Replay

**Attack**: Agent presents a credential that was valid but has been revoked.

**Defense**:
- Credentials verified at context generation time
- Credential expiration checked
- (Production) Revocation status checked

**Enforcement**: Structural - invalid credentials don't produce affordances

### T3: Confused Deputy

**Attack**: Planner agent tricks executor into performing forbidden action.

**Defense**:
- Each agent type has distinct AAT with clear boundaries
- Executors verify their own authorization independently
- Cross-type actions require explicit credentials

**Enforcement**: Structural - AAT boundaries prevent type confusion

### T4: Proof Stripping

**Attack**: Remove or forge proof to bypass authorization.

**Defense**:
- Proofs verified cryptographically
- DID proof-of-control required for identity
- VC signatures verified against issuer keys

**Enforcement**: Cryptographic - invalid proofs rejected

### T5: Trace Omission

**Attack**: Execute action without generating audit trace.

**Defense**:
- Traversal atomically includes trace generation
- No execution path bypasses trace store
- Trace emission is part of the traversal contract

**Enforcement**: Structural - untraceable traversal is impossible

### T6: Capability Escalation

**Attack**: Mislabel affordance to grant forbidden capabilities.

**Defense**:
- Broker generates all affordances (agents cannot inject)
- Affordances validated against AAT before inclusion
- Action types explicitly enumerated per AAT

**Enforcement**: Structural - broker controls affordance generation

### T7: Policy Bypass

**Attack**: Circumvent policy engine to perform denied action.

**Defense**:
- Policy evaluation mandatory before traversal
- Denial results in error, not affordance hiding
- All evaluations traced in PROV record

**Enforcement**: Structural - no execution without policy check

### T8: Side-Channel Actions

**Attack**: Use affordance for unintended purpose.

**Defense**:
- Affordance effects explicitly declared
- Parameters validated against schema
- Outcomes compared against predictions (causal)

**Enforcement**: Validation + audit

## Safety Invariants

### S1: No Forbidden Actions
```
∀ context, ∀ affordance ∈ context.affordances:
  affordance.actionType ∉ agent.AAT.forbiddenActions
```

### S2: Credential Required
```
∀ affordance with requiresCredential:
  ∃ vc ∈ context.verifiedCredentials:
    vc.type includes requiresCredential.schema
```

### S3: Every Action Traced
```
∀ traversal:
  ∃ trace: trace.used.affordance.id = traversal.affordanceId
```

### S4: Context Bound
```
∀ traversal:
  now() < context.expiresAt ∧
  traversal.contextId = valid_context.id
```

### S5: Policy Evaluated
```
∀ traversal:
  policyEngine.evaluateAction(traversal) ≠ skip
```

## Negative Test Cases

The following tests verify threat defenses:

| Test | Threat | Expected Result |
|------|--------|-----------------|
| test_expired_context | T1 | Reject with "Context has expired" |
| test_missing_credential | T2 | Affordance not present |
| test_planner_cannot_actuate | T3 | No Actuate affordance in Planner context |
| test_invalid_did_proof | T4 | Reject with "DID verification failed" |
| test_traverse_emits_trace | T5 | Trace exists after traversal |
| test_forbidden_action_not_afforded | T6 | Forbidden actions never appear |
| test_policy_denial_blocks | T7 | Traversal fails on policy deny |

## Security Audit Checklist

- [ ] All contexts have expiration and nonce
- [ ] All credentials verified against trusted issuers
- [ ] All affordances validated against AAT
- [ ] All traversals emit PROV traces
- [ ] All policy evaluations recorded
- [ ] No hardcoded action lists in agents
- [ ] No execution paths bypass broker
