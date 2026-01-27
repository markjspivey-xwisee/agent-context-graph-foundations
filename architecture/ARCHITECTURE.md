# Agent Context Graph Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENTS                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │   Web    │  │   CLI    │  │   MCP    │  │ WebSocket│  │ External │      │
│  │Dashboard │  │  Tools   │  │  Clients │  │  Clients │  │  Agents  │      │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘      │
└───────┼─────────────┼─────────────┼─────────────┼─────────────┼────────────┘
        │             │             │             │             │
        └─────────────┴─────────────┼─────────────┴─────────────┘
                                    │
┌───────────────────────────────────┼─────────────────────────────────────────┐
│                              API LAYER                                       │
│  ┌────────────────────────────────┴────────────────────────────────────┐    │
│  │                         Hapi.js Server                               │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐  │    │
│  │  │Rate Limiter │  │   Logger    │  │    CORS     │  │   Auth     │  │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘  │    │
│  └──────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│  ┌─────────────────────────────────┼───────────────────────────────────┐    │
│  │                          ENDPOINTS                                   │    │
│  │  /context  /traverse  /broker/*  /social/*  /contexts/*  /sparql    │    │
│  └─────────────────────────────────┼───────────────────────────────────┘    │
└────────────────────────────────────┼────────────────────────────────────────┘
                                     │
┌────────────────────────────────────┼────────────────────────────────────────┐
│                           CORE SERVICES                                      │
│                                    │                                         │
│  ┌─────────────────────────────────┴──────────────────────────────────┐     │
│  │                        CONTEXT BROKER                               │     │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐       │     │
│  │  │    AAT    │  │  Policy   │  │   SHACL   │  │  Causal   │       │     │
│  │  │ Registry  │  │  Engine   │  │ Validator │  │ Evaluator │       │     │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘       │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                    │                                         │
│  ┌────────────────┬────────────────┼────────────────┬────────────────┐      │
│  │                │                │                │                │      │
│  ▼                ▼                ▼                ▼                ▼      │
│ ┌────────┐  ┌──────────┐  ┌──────────────┐  ┌──────────┐  ┌─────────────┐  │
│ │Personal│  │  Social  │  │   Shared     │  │ Realtime │  │  Channel    │  │
│ │ Broker │  │Federation│  │  Context     │  │   Sync   │  │   Bridge    │  │
│ └───┬────┘  └────┬─────┘  └──────┬───────┘  └────┬─────┘  └──────┬──────┘  │
│     │            │               │               │               │         │
│     │  ┌─────────┴───────────────┴───────────────┴───────────────┘         │
│     │  │                                                                    │
│     │  ▼                                                                    │
│     │ ┌──────────────────────────────────────────────────────────────┐     │
│     │ │                    FEDERATION LAYER                           │     │
│     │ │  ┌────────────┐  ┌────────────┐  ┌────────────┐              │     │
│     │ │  │ ActivityPub│  │  DIDComm   │  │  DID/VC    │              │     │
│     │ │  │   Bridge   │  │ Messaging  │  │   Auth     │              │     │
│     │ │  └────────────┘  └────────────┘  └────────────┘              │     │
│     │ └──────────────────────────────────────────────────────────────┘     │
│     │                                                                       │
└─────┼───────────────────────────────────────────────────────────────────────┘
      │
┌─────┼───────────────────────────────────────────────────────────────────────┐
│     │                       STORAGE LAYER                                    │
│     ▼                                                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         PERSISTENCE                                   │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐     │   │
│  │  │  RDF Store │  │   SQLite   │  │ Federation │  │ Checkpoint │     │   │
│  │  │  (Traces)  │  │(Trace Meta)│  │Persistence │  │   Store    │     │   │
│  │  └────────────┘  └────────────┘  └────────────┘  └────────────┘     │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Component Details

### API Layer

- **Hapi.js Server**: HTTP/REST API server
- **Rate Limiter**: Token bucket rate limiting per route
- **Logger**: Structured JSON logging with correlation IDs
- **Auth**: DID-based authentication with challenge-response

### Core Services

#### Context Broker
The central orchestrator that generates Context Graphs for agents:
- **AAT Registry**: Manages Abstract Agent Type definitions
- **Policy Engine**: Evaluates access policies (OPA-compatible)
- **SHACL Validator**: Validates context graphs against shapes
- **Causal Evaluator**: do-calculus intervention tracking
- **Semiotics Telemetry**: records usage events for stability/drift analysis

#### Knowledge Graph Service
Persistent, ontology-driven memory used alongside ephemeral Context Graphs:
- Register knowledge graph metadata and endpoints
- Accept mappings (e.g., R2RML) from source systems
- Provide query/update APIs
- Emit provenance for graph updates

#### Semantic Virtualization Layer
Virtual, zero-copy semantic layer that exposes a SPARQL interface over heterogeneous sources:
- RDF Knowledge Graph (OWL + SHACL)
- Federated query via explicit R2RML/OBDA mappings
- HyprCat-aligned DCAT + DPROD metadata for datasets and data products
- Hydra catalog with hypermedia operations for browsing/querying
- SHACL data contracts linked to products
- Adapters are implementation details

#### Tool Registry
Allow agents to register and use tools through explicit affordances:
- Store tool definitions and schemas
- Enforce policy/credential gating on registration
- Expose registered tools via broker endpoints

#### Hypergraph + Category Semantics
Context Graphs can be rendered as hypergraphs and categories for formal reasoning:
- **Hypergraph View**: affordances are hyperedges binding agent, target, credentials, and constraints in one relation.
- **Category View**: objects represent state/resource slices; morphisms represent affordance traversals.
- **Composition**: multi-step workflows are explicit morphism composition records.
These views are optional payloads but are considered canonical when present.

#### Personal Broker
User's unified AI assistant interface:
- Conversations with multi-channel history
- Memory store (semantic, episodic, procedural, preference)
- Contacts with DID-based identity
- Presence and availability status

#### Social Federation
Broker-to-broker social networking:
- Connection requests and acceptance
- Groups with role-based membership
- Invite links with usage limits
- Discovery via DID, WebID, or search

#### Shared Context
Collaborative context graphs:
- CRDT-based conflict-free replication
- Access control (read/write/admin)
- Vector clocks for causal ordering
- Real-time presence tracking

#### Realtime Sync
WebSocket-based live collaboration:
- Channel subscriptions (context, presence, notifications)
- Broadcast of changes to participants
- Reconnection handling

### Federation Layer

#### ActivityPub Bridge
Fediverse interoperability:
- Actor creation and management
- WebFinger discovery
- HTTP Signatures for delivery
- Inbox/Outbox handling

#### DIDComm Messaging
Secure agent-to-agent communication:
- DIDComm v2 protocol
- Ed25519/X25519 key pairs
- Encrypted message envelopes
- Thread management

#### DID/VC Auth
Decentralized identity and credentials:
- did:key generation
- Challenge-response authentication
- Verifiable Credential issuance
- Capability-based access control

### Storage Layer

#### RDF Store
Native triplestore for PROV traces:
- N3.js quad storage
- SPARQL query support
- Turtle/JSON-LD export

#### SQLite
Relational storage for trace metadata:
- Full-text search
- Efficient queries
- Transaction support

#### Federation Persistence
SQLite storage for federation data:
- Profiles and connections
- Groups and memberships
- Shared contexts
- Notifications

## Data Flow

### Context Graph Generation

```
1. Agent requests context
   │
   ▼
2. Broker validates DID and credentials
   │
   ▼
3. AAT Registry provides allowed actions
   │
   ▼
4. Policy Engine filters based on policies
   │
   ▼
5. SHACL validates result
   │
   ▼
6. Context Graph returned with affordances
```

### Federation Message Flow

```
Broker A                    Broker B
   │                           │
   │ 1. Create DIDComm message │
   │──────────────────────────>│
   │                           │
   │ 2. Encrypt with B's key   │
   │                           │
   │ 3. Deliver to B's endpoint│
   │──────────────────────────>│
   │                           │
   │                    4. Decrypt
   │                           │
   │                    5. Process
   │                           │
   │<──────────────────────────│
   │ 6. Acknowledgement        │
```

### Real-time Sync Flow

```
Client A          Server          Client B
   │                │                 │
   │ 1. Connect WS  │                 │
   │───────────────>│                 │
   │                │                 │
   │ 2. Subscribe   │                 │
   │   to context   │                 │
   │───────────────>│                 │
   │                │                 │
   │ 3. Update node │                 │
   │───────────────>│                 │
   │                │                 │
   │                │ 4. Broadcast    │
   │                │────────────────>│
   │                │                 │
   │                │ 5. Conflict?    │
   │                │   CRDT merge    │
   │                │                 │
```

## Security Model

### Authentication
- DID-based identity (did:key, did:web)
- Challenge-response authentication
- Session tokens with expiration

### Authorization
- Verifiable Credentials for capabilities
- Policy-based access control
- Per-context access levels

### Encryption
- DIDComm for message encryption
- TLS for transport
- Ed25519 signatures

### Isolation
- Execution Enclaves for agents
- DID-bound git worktrees
- Resource limits per agent type

## Scalability Considerations

### Horizontal Scaling
- Stateless API servers (share sessions via store)
- WebSocket sticky sessions
- Distributed CRDT sync

### Caching
- Context Graph caching with TTL
- SPARQL query result caching
- Rate limiter bucket distribution

### Performance
- Connection pooling for SQLite
- Batch operations for RDF writes
- Async trace storage
