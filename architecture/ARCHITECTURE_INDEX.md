# Architecture Index — Source of Truth

This document defines the **non-negotiable architectural pillars** of the system.
Every implementation artifact MUST map to one or more pillars below.
No feature is considered complete unless its pillar mapping is explicit.

---

## Pillar A — Abstract Agent Types (AAT)

**Purpose:** Agent type system (behavioral contracts)

### Requirements
- Define agent types independently of runtime context
- Specify:
  - allowed action types
  - forbidden action types
  - behavioral invariants
  - trace / accountability requirements
- Context Graph affordances MUST be a subtype of the agent's AAT

### Artifacts
- `../spec/aat/*.json`
- `../spec/shacl/aat-safety.ttl`
- `../principles/aat.md`

---

## Pillar B — Context Graphs (Situational Awareness)

**Purpose:** Runtime, bounded action surface

### Requirements
- Contexts are generated dynamically
- Context includes:
  - agent DID
  - verified capabilities
  - constraints / policy
  - timestamps / scope
- Affordances appear or disappear based on:
  - credentials
  - policy
  - situation
- No global tool list exists

### Artifacts
- `../spec/context-graph.schema.json`
- `../spec/shacl/context.ttl`
- `../spec/cgp.md`
- `../examples/golden-path/context-fragment.json`

---

## Pillar C — Hypermedia / HATEOAS Principle

**Purpose:** Late-bound action discovery

### Requirements
- Agents MUST NOT assume actions
- Agents may only act via broker-provided affordances
- Workflows are not pre-programmed

### Artifacts
- `../principles/hypermedia-principles.md`
- `../AGENTS.md`

---

## Pillar D — Affordance Theory

**Purpose:** Actions are relational, not intrinsic

### Requirements
- Affordances depend on:
  - agent identity
  - environment
  - norms / policy
- Parameters are typed (schema + SHACL)
- Capability requirements are explicit

### Artifacts
- `../principles/affordances.md`
- `../spec/shacl/*ParamsShape.ttl`

---

## Pillar E — Causality (SCM / Interventions)

**Purpose:** Predict effects of actions

### Requirements
- Affordances MAY reference causal semantics
- Each causal affordance defines:
  - an intervention label (`do(action, params)`)
  - outcome variables
- No causal reasoning is embedded in the LLM

### Artifacts
- `../spec/causal-affordance.md`
- `./interfaces.md` (ICausalEvaluator)
- `../examples/golden-path/*causal*`

---

## Pillar F — Policy (Deontic + Outcome Constraints)

**Purpose:** Governance and safety

### Requirements
- Deontic policy controls affordance existence
- Outcome constraints may block traversal
- Escalation paths must be explicit

### Artifacts
- `../principles/policy.md`
- `./interfaces.md` (IPolicyEngine)

---

## Pillar G — Provenance & Audit (PROV)

**Purpose:** Accountability and counterfactual analysis

### Requirements
- Every affordance traversal emits a PROV trace
- Trace binds:
  - context snapshot
  - affordance
  - parameters
  - proofs
  - intervention label
  - observed outcomes
- Traces are immutable

### Artifacts
- `../spec/prov-trace.schema.json`
- `../principles/prov.md`
- `../examples/golden-path/prov-trace.json`

---

## Pillar H — Decentralized Identity (DIDs / VCs)

**Purpose:** Verifiable authority

### Requirements
- Agent identity = DID
- Capabilities proven via VCs / VPs
- Affordances are VC-gated
- Credential issuance appears as an affordance
- DIDComm is a valid action target type

### Artifacts
- `../principles/decentralized-identity.md`
- `./interfaces.md` (IVerifier)
- `../examples/golden-path/request-credential.json`

---

## Pillar I — Emergent Semiotics & Usage-Based Meaning

**Purpose:** Meaning stabilizes through use

### Requirements
- Affordance labels are conventions, not truth
- Labels must be versioned
- System logs enough telemetry to measure:
  - stability
  - drift
  - polysemy
- No hard-coded semantics in code

### Artifacts
- `../principles/semiotics.md`
- `../spec/telemetry.md`

---

## Pillar J — Threat Model & Safety Invariants

**Purpose:** Prevent structural failure modes

### Requirements
- Address:
  - stale context replay
  - credential replay
  - confused deputy
  - trace omission
- Safety is enforced structurally, not heuristically

### Artifacts
- `../principles/threat-model.md`
- `../tests/negative/*`

---

## Pillar K — Hypergraph Semantics

**Purpose:** Multi-party relations as first-class structure

### Requirements
- Context Graphs MAY provide an explicit hypergraph view
- Affordances SHOULD map to hyperedges
- Hyperedges MUST bind agents, targets, credentials, and constraints explicitly
- No inferred authority: hyperedges only represent validated affordances

### Artifacts
- `../principles/hypergraph.md`
- `../spec/HYPERGRAPH.md`
- `../spec/context-graph.schema.json`
- `../spec/ontology/acg-core.ttl`
- `../spec/shacl/context.ttl`
- `../examples/golden-path/context-fragment.json`

---

## Pillar L — Category-Theoretic Composition

**Purpose:** Formal composition of affordances and workflows

### Requirements
- Context Graphs MAY provide a category view (objects + morphisms)
- Morphisms SHOULD reference affordances or hyperedges
- Composition MUST be explicit (no hidden workflows)
- Causality remains external (no causal reasoning in LLM logic)

### Artifacts
- `../principles/category-theory.md`
- `../spec/CATEGORY-THEORY.md`
- `../spec/context-graph.schema.json`
- `../spec/ontology/acg-core.ttl`
- `../examples/golden-path/context-fragment.json`

---

## Coverage Table (Required)

| Pillar | Spec | Schema | Example | Test |
|--------|------|--------|---------|------|
| AAT | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| Context Graph | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| Hypermedia | :white_check_mark: | — | :white_check_mark: | :white_check_mark: |
| Affordances | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| Causality | :white_check_mark: | — | :white_check_mark: | :white_check_mark: |
| Policy | :white_check_mark: | — | :white_check_mark: | :white_check_mark: |
| Provenance | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| Identity | :white_check_mark: | — | :white_check_mark: | :white_check_mark: |
| Semiotics | :white_check_mark: | — | :white_check_mark: | — |
| Threat Model | :white_check_mark: | — | — | :white_check_mark: |
| Hypergraph | :white_check_mark: | :white_check_mark: | :white_check_mark: | — |
| Category Theory | :white_check_mark: | :white_check_mark: | :white_check_mark: | — |

**No pull request may be merged unless this table is updated.**
