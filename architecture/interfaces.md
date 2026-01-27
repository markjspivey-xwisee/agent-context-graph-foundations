# System Interfaces

## Overview

The Agent Context Graph system is built around clean interface contracts. This document describes the core interfaces that implementations must satisfy.

## IVerifier

Handles DID and Verifiable Credential verification.

```typescript
interface IVerifier {
  // Verify DID proof of control
  verifyDIDProof(did: string, proof: unknown): Promise<VerificationResult>;

  // Verify a Verifiable Credential
  verifyVC(credential: unknown): Promise<VCVerificationResult>;

  // Verify a Verifiable Presentation
  verifyVP(presentation: unknown): Promise<VPVerificationResult>;
}
```

### Responsibilities
- Validate DID format and proof signatures
- Check credential structure and expiration
- Verify issuer signatures
- Check trusted issuer lists

### Implementation Notes
- Production implementations should use `did-resolver` for DID resolution
- Credential verification should follow W3C VC Data Model
- Support multiple DID methods (did:key, did:web, etc.)

## ICausalEvaluator

Evaluates causal models for outcome prediction.

```typescript
interface ICausalEvaluator {
  // Evaluate causal model for predicted outcomes
  evaluate(
    modelRef: string,
    intervention: string,
    context: Record<string, unknown>
  ): Promise<CausalEvaluationResult>;

  // Check if outcomes meet constraints
  checkConstraints(
    outcomes: Record<string, unknown>,
    constraints: OutcomeConstraint[]
  ): Promise<ConstraintCheckResult>;
}
```

### Responsibilities
- Load and execute causal models
- Predict outcome variables given interventions
- Check predicted outcomes against policy constraints

## IKnowledgeGraphService

Provides persistent, ontology-driven knowledge graphs alongside ephemeral Context Graphs.

```typescript
interface IKnowledgeGraphService {
  listGraphs(): Promise<KnowledgeGraphRef[]>;
  registerGraph(graph: KnowledgeGraphRef): Promise<KnowledgeGraphRef>;
  getGraph(id: string): Promise<KnowledgeGraphRef | null>;
  queryGraph(id: string, query: KnowledgeGraphQuery): Promise<unknown>;
  registerMapping(id: string, mappingRef: string): Promise<KnowledgeGraphUpdate>;
  updateGraph(id: string, update: KnowledgeGraphUpdate): Promise<KnowledgeGraphUpdate>;
}
```

### Responsibilities
- Register knowledge graph metadata and ontology refs
- Provide query access (SPARQL or broker-defined queries)
- Accept mapping definitions (e.g., R2RML)
- Emit provenance-aware update records

### Implementation Notes
- Prefer persistent storage (RDF store) behind this interface
- Keep update semantics auditable and append-only
- Treat mapping registration as a policy-gated affordance

## IPolicyEngine

Evaluates deontic and outcome-based policies.

```typescript
interface IPolicyEngine {
  // Evaluate whether an action is permitted
  evaluateAction(
    agentDID: string,
    actionType: string,
    context: PolicyContext
  ): Promise<PolicyDecision>;

  // Get active policies for an agent
  getActivePolicies(agentDID: string): Promise<Policy[]>;
}
```

### Responsibilities
- Evaluate deontic policies (permit/deny/duty)
- Evaluate outcome constraints using causal predictions
- Return clear decisions with reasons

### Implementation Notes
- Consider ODRL for policy representation
- Support policy composition and priority
- All decisions must be traceable

## ITraceStore

Handles PROV trace storage and retrieval.

```typescript
interface ITraceStore {
  // Store a PROV trace (append-only)
  store(trace: ProvTrace): Promise<StoreResult>;

  // Retrieve traces by query
  query(query: TraceQuery): Promise<ProvTrace[]>;

  // Get a specific trace by ID
  getById(traceId: string): Promise<ProvTrace | null>;
}
```

### Responsibilities
- Append-only storage (no updates or deletes)
- Efficient querying by agent, action type, time range
- Maintain data integrity

### Implementation Notes
- Consider event sourcing patterns
- Support for cryptographic integrity (signatures)
- Must handle high write volumes

## IAATRegistry

Manages Abstract Agent Type definitions.

```typescript
interface IAATRegistry {
  // Get an AAT by ID
  getAAT(aatId: string): Promise<AbstractAgentType | null>;

  // Check if an action type is allowed for an AAT
  isActionAllowed(aatId: string, actionType: string): Promise<boolean>;

  // Check if an action type is forbidden for an AAT
  isActionForbidden(aatId: string, actionType: string): Promise<boolean>;

  // Get required capabilities for an action
  getRequiredCapability(aatId: string, actionType: string): Promise<string | null>;
}
```

### Responsibilities
- Load and manage AAT definitions
- Fast lookup for action validation
- Support AAT versioning

### Implementation Notes
- AATs should be loaded from spec files at startup
- Consider caching for performance
- Support hot-reload for development
