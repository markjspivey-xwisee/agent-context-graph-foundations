# Decentralized Identity (DIDs / VCs)

## Overview

The Agent Context Graph uses W3C Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs) for:
- **Agent Identity**: Each agent has a unique, verifiable DID
- **Capability Proof**: Agents prove capabilities through VCs
- **Authority Verification**: Credentials are verified, not assumed

## DIDs (Decentralized Identifiers)

### What is a DID?
A DID is a globally unique identifier that the subject controls:
```
did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK
```

Format: `did:<method>:<identifier>`

### Supported Methods
| Method | Use Case |
|--------|----------|
| did:key | Self-generated, no registration needed |
| did:web | Organization-controlled, DNS-based |
| did:ion | Bitcoin-anchored, highly decentralized |

### Proof of Control
Agents prove they control their DID through cryptographic proofs:
```json
{
  "agentDID": "did:key:z6Mk...",
  "proof": {
    "type": "Ed25519Signature2020",
    "challenge": "random-challenge",
    "signature": "..."
  }
}
```

## Verifiable Credentials (VCs)

### What is a VC?
A VC is a tamper-evident claim about a subject:
```json
{
  "@context": ["https://www.w3.org/2018/credentials/v1"],
  "type": ["VerifiableCredential", "PlannerCapability"],
  "issuer": "did:web:authority.example.com",
  "issuanceDate": "2024-01-01T00:00:00Z",
  "expirationDate": "2025-01-01T00:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6Mk...",
    "capability": "PlannerCapability"
  },
  "proof": { ... }
}
```

### Capability Credentials
Standard capability types:
- `PlannerCapability`: Can emit plans
- `ExecutorCapability`: Can execute actions
- `ObserverCapability`: Can observe and report
- `ArbiterCapability`: Can approve/deny actions
- `ArchivistCapability`: Can store/retrieve records

### Trusted Issuers
The system maintains a list of trusted credential issuers:
```typescript
const verifier = new StubVerifier([
  'did:web:authority.example.com',
  'did:web:issuer.example.com'
]);
```

Only credentials from trusted issuers are accepted.

## Credential Gating

### How It Works
1. Affordance specifies required credentials
2. Broker checks agent's verified credentials
3. If credential missing, affordance not included
4. Instead, `request-credential` affordance may appear

### Example: Missing Credential
Agent without `PlannerCapability` requests context:
```json
{
  "affordances": [
    {
      "id": "aff-request-credential",
      "rel": "request-credential",
      "actionType": "RequestCredential",
      "target": {
        "type": "OID4VCI",
        "href": "https://issuer.example.com/.well-known/openid-credential-issuer"
      }
    }
  ]
}
```

### Example: With Credential
Same agent after obtaining credential:
```json
{
  "affordances": [
    {
      "id": "aff-emit-plan",
      "rel": "emit-plan",
      "actionType": "EmitPlan",
      "requiresCredential": [{ "schema": "PlannerCapability" }]
    }
  ]
}
```

## OID4VCI (Credential Issuance)

OpenID for Verifiable Credential Issuance is supported as a target type:
```json
{
  "target": {
    "type": "OID4VCI",
    "href": "https://issuer.example.com/.well-known/openid-credential-issuer",
    "serviceEndpoint": "https://issuer.example.com/credential"
  }
}
```

This enables standard credential issuance flows.

## DIDComm (Messaging)

DIDComm is supported for agent-to-agent messaging:
```json
{
  "target": {
    "type": "DIDComm",
    "didcommType": "https://didcomm.org/message/1.0",
    "serviceEndpoint": "did:web:receiver.example.com#messaging"
  }
}
```

## Security Considerations

1. **Never Infer Authority**: Always verify credentials
2. **Check Expiration**: Reject expired credentials
3. **Verify Issuer**: Only trust configured issuers
4. **Proof Freshness**: Require recent proofs to prevent replay
5. **Credential Scope**: Check credential scope matches action
