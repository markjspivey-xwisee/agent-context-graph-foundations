# Agent Context Graph API Reference

Complete API documentation for the ACG federated personal assistant system.

## Table of Contents

- [Core Endpoints](#core-endpoints)
- [Knowledge Graphs](#knowledge-graphs)
- [Personal Broker](#personal-broker)
- [Social Federation](#social-federation)
- [Shared Contexts](#shared-contexts)
- [Real-time Sync (WebSocket)](#real-time-sync-websocket)
- [Semantic Web](#semantic-web)
- [Authentication](#authentication)

---

## Core Endpoints

### GET /
API entry point with Hydra hypermedia controls.

**Response:**
```json
{
  "@context": "https://www.w3.org/ns/hydra/context.jsonld",
  "@type": "EntryPoint",
  "@id": "/",
  "contexts": "/context",
  "traverse": "/traverse",
  "traces": "/traces",
  "knowledgeGraphs": "/knowledge-graphs",
  "dataCatalog": "/data/catalog",
  "dataProducts": "/data/products",
  "dataContracts": "/data/contracts",
  "dataQuery": "/data/query",
  "tools": "/broker/tools",
  "social": { ... },
  "sharedContexts": "/contexts"
}
```

### GET /health
Health check endpoint.

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### POST /context
Generate a Context Graph for an agent based on goal and capabilities.

**Request:**
```json
{
  "agentDID": "did:key:z6Mk...",
  "agentType": "planner",
  "goal": "Research AI safety papers",
  "capabilities": ["web_search", "document_analysis"],
  "currentContext": {}
}
```

**Response:** JSON-LD Context Graph with affordances.

**Optional structural views:**
- `hypergraph`: explicit hypergraph representation (hypernodes + hyperedges)
- `category`: category-theoretic view (objects + morphisms + composition)
- `usageSemantics`: usage-based semantics on affordances (stability, drift, polysemy)
- `knowledgeGraphRef`: reference to persistent knowledge graph
- `knowledgeGraphSnapshot`: lightweight KG summary for the context

Example (abridged):
```json
{
  "id": "urn:uuid:ctx-123",
  "knowledgeGraphRef": { "id": "urn:kg:default", "version": "2026.01" },
  "affordances": [ ... ],
  "hypergraph": { "nodes": [ ... ], "hyperedges": [ ... ] },
  "category": { "objects": [ ... ], "morphisms": [ ... ], "composition": [] }
}
```

### POST /traverse
Execute an affordance from a Context Graph.

**Request:**
```json
{
  "contextId": "ctx_123",
  "affordanceId": "aff_456",
  "parameters": { "query": "RLHF papers 2024" }
}
```

**Usage-based semantics (protocol-level):**

Affordance traversals SHOULD emit usage telemetry embedded in the trace:
```json
{
  "usageEvent": {
    "usageRel": "emit-plan",
    "usageRelVersion": "1.0.0",
    "usageActionType": "EmitPlan",
    "usageOutcomeStatus": "success",
    "usageTimestamp": "2026-01-24T15:00:05Z",
    "contextId": "urn:uuid:ctx-123",
    "traceId": "urn:uuid:trace-123"
  }
}
```

If an affordance updates the knowledge graph, the trace SHOULD also include:
```json
{
  "generated": {
    "knowledgeGraphUpdate": {
      "graphId": "urn:kg:enterprise",
      "updateType": "mapping",
      "updateRef": "https://broker.example.com/knowledge-graphs/enterprise/mappings"
    }
  }
}
```

---

## Knowledge Graphs

Persistent, ontology-driven knowledge graphs used as long-term memory alongside Context Graphs.

### GET /knowledge-graphs
List known knowledge graphs.

### POST /knowledge-graphs
Register a knowledge graph (metadata, endpoints, ontology refs).

### POST /knowledge-graphs/{id}/query
Query a knowledge graph (SPARQL or broker-defined query payload).

### POST /knowledge-graphs/{id}/mappings
Register or update mappings (e.g., R2RML) for a source system.

Example metadata:
```json
{
  "id": "urn:kg:enterprise",
  "label": "Enterprise Knowledge Graph",
  "version": "2026.01",
  "ontologyRefs": [
    "https://www.w3.org/ns/dcat#",
    "https://www.omg.org/spec/DPROD/",
    "https://hyprcat.io/vocab#",
    "https://www.w3.org/ns/r2rml#"
  ],
  "queryEndpoint": "https://broker.example.com/knowledge-graphs/enterprise/query",
  "updateEndpoint": "https://broker.example.com/knowledge-graphs/enterprise/update"
}
```

---

## Semantic Layer

Hydra-enabled virtual semantic layer for federated data access (HyprCat-aligned: DCAT + DPROD + Hydra + SHACL).

### GET /data/catalog
Retrieve the Hydra-enabled semantic catalog (datasets, products, contracts).

**Response (JSON-LD):**
```json
{
  "@context": {
    "dcat": "http://www.w3.org/ns/dcat#",
    "dcterms": "http://purl.org/dc/terms/",
    "dprod": "https://ekgf.github.io/data-product-spec/dprod#",
    "hydra": "http://www.w3.org/ns/hydra/core#",
    "hyprcat": "https://hyprcat.io/vocab#",
    "sl": "https://agentcontextgraph.dev/semantic-layer#"
  },
  "@id": "/data/catalog",
  "@type": ["dcat:Catalog", "hyprcat:Catalog", "sl:SemanticCatalog"],
  "dcterms:title": "ACG Semantic Catalog",
  "dcterms:description": "Hydra-enabled catalog of datasets, products, and contracts.",
  "dcat:dataset": [{ "@id": "urn:acg:dataset:sales-orders" }],
  "sl:hasDataProduct": [{ "@id": "urn:acg:data-product:sales-insights" }]
}
```

### GET /data/products
List data products in the catalog.

### GET /data/products/{id}
Fetch a specific data product by ID.

### GET /data/contracts
List data contracts in the catalog.

### GET /data/contracts/{id}
Fetch a specific data contract by ID (SHACL-based).

### GET /data/contracts/{id}/shape
Fetch the SHACL shape for a contract.

**Notes:**
- Data contracts MUST reference SHACL shapes (OWL/SHACL are canonical; JSON Schema is non-normative).

### POST /data/query
Execute a semantic query against the virtual RDF layer. SPARQL is canonical; adapters MAY support additional query languages.

**Request (SPARQL):**
```json
{
  "query": "SELECT ?s ?p ?o WHERE { ?s ?p ?o } LIMIT 5",
  "queryLanguage": "sparql",
  "semanticLayerRef": "https://semantic-layer.example.com/sparql",
  "sourceRef": "urn:dcat:dataset:example",
  "mappingRef": "urn:r2rml:mapping:example",
  "timeoutSeconds": 60,
  "resultFormat": "application/sparql-results+json"
}
```

---

## Personal Broker

### GET /broker
Get broker information and status.

**Response:**
```json
{
  "id": "broker_abc123",
  "ownerDID": "did:key:z6Mk...",
  "displayName": "My Personal Assistant",
  "createdAt": "2024-01-15T10:00:00Z",
  "stats": {
    "conversations": 5,
    "channels": 2,
    "memories": 42,
    "contacts": 10
  }
}
```

### GET /broker/tools
List registered tools for the broker.

### POST /broker/tools
Register a new tool definition for agents to use (policy-gated).

### GET /broker/conversations
List all conversations.

**Query Parameters:**
- `limit` (optional): Max results (default: 50)
- `channel` (optional): Filter by channel ID

**Response:**
```json
{
  "conversations": [
    {
      "id": "conv_123",
      "title": "Research Discussion",
      "channelId": "ch_slack_001",
      "messageCount": 15,
      "createdAt": "2024-01-15T09:00:00Z",
      "updatedAt": "2024-01-15T10:30:00Z"
    }
  ]
}
```

### POST /broker/conversations
Start a new conversation.

**Request:**
```json
{
  "title": "New Research Topic",
  "channelId": "ch_slack_001"
}
```

### GET /broker/conversations/{id}
Get a specific conversation with messages.

### POST /broker/conversations/{id}/messages
Send a message in a conversation.

**Request:**
```json
{
  "role": "user",
  "content": "What are the latest AI safety papers?"
}
```

### GET /broker/memory
Query the broker's memory store.

**Query Parameters:**
- `type` (optional): Filter by memory type
- `tags` (optional): Comma-separated tags
- `limit` (optional): Max results
- `minImportance` (optional): Minimum importance score

**Response:**
```json
{
  "memories": [
    {
      "id": "mem_123",
      "memoryType": "fact",
      "content": "User prefers morning meetings",
      "importance": 0.8,
      "tags": ["preference", "schedule"],
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ]
}
```

### POST /broker/memory
Store a new memory.

**Request:**
```json
{
  "memoryType": "fact",
  "content": "User's research focus is AI alignment",
  "importance": 0.9,
  "tags": ["research", "ai-safety"]
}
```

### GET /broker/contacts
List all contacts.

### POST /broker/contacts
Add a new contact.

**Request:**
```json
{
  "name": "Alice Researcher",
  "did": "did:key:z6Mk...",
  "channels": [
    { "platform": "slack", "handle": "@alice" }
  ]
}
```

### GET /broker/presence
Get broker's presence status.

### PUT /broker/presence
Update presence status.

**Request:**
```json
{
  "state": "active",
  "statusMessage": "Working on research"
}
```

---

## Social Federation

### GET /social/profile
Get current broker's social profile.

**Response:**
```json
{
  "id": "profile_123",
  "brokerId": "broker_abc",
  "displayName": "Research Assistant",
  "bio": "AI Safety focused personal assistant",
  "avatar": "https://example.com/avatar.png",
  "visibility": "connections",
  "verifiedCredentials": []
}
```

### PUT /social/profile
Update social profile.

**Request:**
```json
{
  "displayName": "Updated Name",
  "bio": "New bio text",
  "visibility": "public"
}
```

### GET /social/connections
List all social connections.

**Query Parameters:**
- `state` (optional): Filter by state (pending, accepted, blocked)

**Response:**
```json
{
  "connections": [
    {
      "id": "conn_123",
      "fromBrokerId": "broker_abc",
      "toBrokerId": "broker_xyz",
      "state": "accepted",
      "protocol": "native_acg",
      "establishedAt": "2024-01-15T10:00:00Z"
    }
  ]
}
```

### POST /social/connections/request
Request a new connection.

**Request:**
```json
{
  "toBrokerId": "broker_xyz",
  "message": "Hi, would like to collaborate on research!"
}
```

### POST /social/connections/{id}/accept
Accept a pending connection request.

### POST /social/connections/{id}/reject
Reject a pending connection request.

### GET /social/invites
List active invite links.

### POST /social/invites
Create a new invite link.

**Request:**
```json
{
  "maxUses": 5,
  "expiresInHours": 24,
  "label": "Research Team Invite"
}
```

**Response:**
```json
{
  "id": "inv_123",
  "code": "ABC123XYZ",
  "creatorBrokerId": "broker_abc",
  "maxUses": 5,
  "useCount": 0,
  "expiresAt": "2024-01-16T10:00:00Z"
}
```

### POST /social/invites/{code}/use
Use an invite link to connect.

### GET /social/notifications
Get notifications.

**Query Parameters:**
- `unreadOnly` (optional): Only unread (default: false)
- `limit` (optional): Max results

### PUT /social/notifications/{id}/read
Mark a notification as read.

### GET /social/groups
List groups the broker belongs to.

### POST /social/groups
Create a new group.

**Request:**
```json
{
  "name": "AI Safety Researchers",
  "description": "Collaborative research group",
  "isPublic": false
}
```

### POST /social/groups/{id}/members
Add a member to a group.

**Request:**
```json
{
  "brokerId": "broker_xyz",
  "role": "member"
}
```

### GET /social/discovery
Discover other brokers.

**Query Parameters:**
- `method`: Discovery method (did, webid, search)
- `query`: Search query

### GET /social/stats
Get federation statistics.

---

## Shared Contexts

### GET /contexts
List shared contexts accessible to the broker.

**Response:**
```json
{
  "contexts": [
    {
      "id": "ctx_123",
      "name": "Research Project",
      "ownerBrokerId": "broker_abc",
      "syncStrategy": "crdt",
      "version": 5,
      "nodeCount": 42,
      "edgeCount": 38
    }
  ]
}
```

### POST /contexts
Create a new shared context.

**Request:**
```json
{
  "name": "Collaborative Research",
  "description": "Team research context",
  "syncStrategy": "crdt",
  "conflictResolution": "auto_merge",
  "isPublic": false
}
```

### GET /contexts/{id}
Get a shared context with full graph data.

### DELETE /contexts/{id}
Delete a shared context (owner only).

### POST /contexts/{id}/join
Join a shared context as a participant.

**Response:**
```json
{
  "id": "replica_123",
  "contextId": "ctx_123",
  "brokerId": "broker_abc",
  "localVersion": 0,
  "status": "syncing"
}
```

### POST /contexts/{id}/leave
Leave a shared context.

### POST /contexts/{id}/access
Grant access to another broker.

**Request:**
```json
{
  "brokerId": "broker_xyz",
  "level": "write"
}
```

### DELETE /contexts/{id}/access/{brokerId}
Revoke access from a broker.

### GET /contexts/{id}/nodes
Get all nodes in a context.

**Query Parameters:**
- `type` (optional): Filter by node type

### POST /contexts/{id}/nodes
Add a node to the context.

**Request:**
```json
{
  "type": "Finding",
  "data": {
    "title": "Research Finding",
    "content": "Important discovery...",
    "confidence": 0.85
  }
}
```

### PUT /contexts/{id}/nodes/{nodeId}
Update a node.

**Request:**
```json
{
  "confidence": 0.90,
  "status": "verified"
}
```

### DELETE /contexts/{id}/nodes/{nodeId}
Delete a node.

### GET /contexts/{id}/edges
Get all edges in a context.

**Query Parameters:**
- `type` (optional): Filter by edge type
- `sourceId` (optional): Filter by source node
- `targetId` (optional): Filter by target node

### POST /contexts/{id}/edges
Add an edge between nodes.

**Request:**
```json
{
  "sourceId": "node_123",
  "targetId": "node_456",
  "type": "relatesTo",
  "data": { "weight": 0.8 }
}
```

### DELETE /contexts/{id}/edges/{edgeId}
Delete an edge.

### GET /contexts/{id}/participants
Get active participants in a context.

### PUT /contexts/{id}/presence
Update presence in a context.

**Request:**
```json
{
  "state": "active",
  "cursor": {
    "nodeId": "node_123",
    "field": "content",
    "offset": 42
  }
}
```

### GET /contexts/stats
Get shared context statistics.

---

## Real-time Sync (WebSocket)

Connect to `ws://localhost:3000/ws` for real-time updates.

### Authentication

After connecting, send an auth message:

```json
{
  "type": "auth",
  "id": "msg_123",
  "timestamp": "2024-01-15T10:00:00Z",
  "payload": {
    "brokerId": "broker_abc"
  }
}
```

### Subscriptions

Subscribe to channels:

```json
{
  "type": "subscribe",
  "id": "msg_124",
  "timestamp": "2024-01-15T10:00:00Z",
  "payload": {
    "channel": "context",
    "contextId": "ctx_123"
  }
}
```

Available channels:
- `context` - Context graph changes
- `presence` - Presence updates
- `notifications` - Social notifications
- `federation` - Federation events

### Message Types

**Incoming:**
- `context_change` - Node/edge added, updated, or deleted
- `presence_update` - Participant presence changed
- `notification` - New notification
- `federation_message` - Federation event

**Outgoing:**
- `auth` - Authenticate
- `subscribe` / `unsubscribe` - Manage subscriptions
- `ping` - Keep-alive
- `presence_update` - Update own presence

### GET /ws/stats
Get WebSocket server statistics.

---

## Semantic Web

### POST /sparql
Execute a SPARQL query.

**Request:**
```json
{
  "query": "SELECT ?s ?p ?o WHERE { ?s ?p ?o } LIMIT 10"
}
```

### GET /sparql/queries
List predefined SPARQL queries.

### POST /sparql/queries/{name}
Execute a named query.

### GET /rdf
Export all traces as Turtle RDF.

### GET /rdf/stats
Get RDF store statistics.

### GET /ontology
Get the ACG ontology.

### GET /shacl
Get SHACL shapes for validation.

### POST /shacl/validate
Validate data against SHACL shapes.

---

## Authentication

The ACG system supports multiple authentication methods:

### DID-based Authentication

Agents authenticate using Decentralized Identifiers:

```json
{
  "did": "did:key:z6Mk...",
  "signature": "base64-encoded-signature",
  "timestamp": "2024-01-15T10:00:00Z"
}
```

### Verifiable Credentials

Access control using W3C Verifiable Credentials:

```json
{
  "@context": ["https://www.w3.org/2018/credentials/v1"],
  "type": ["VerifiableCredential", "ACGCapabilityCredential"],
  "issuer": "did:key:z6Mk...",
  "credentialSubject": {
    "id": "did:key:z6Mk...",
    "capability": "PlannerCapability"
  }
}
```

### API Keys (Development)

For development, simple API key authentication:

```
Authorization: Bearer <api-key>
```

---

## Error Responses

All endpoints return consistent error responses:

```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": {}
}
```

Common HTTP status codes:
- `400` - Bad Request (invalid input)
- `401` - Unauthorized (missing/invalid auth)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `409` - Conflict (CRDT conflict requiring manual resolution)
- `500` - Internal Server Error

---

## Rate Limiting

API endpoints are rate limited:
- Default: 100 requests/minute per broker
- WebSocket: 50 messages/second
- Bulk operations: 10 requests/minute

Rate limit headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705315200
```

---

## Pagination

List endpoints support pagination:

```
GET /contexts?page=2&limit=20
```

Response includes pagination metadata:
```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "pages": 8
  }
}
```
