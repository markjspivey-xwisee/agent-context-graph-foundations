# ACG Dereferenceable URI Specification

**Version:** 1.0.0
**Status:** Normative
**Created:** 2026-01-25

## Abstract

This specification defines requirements for dereferenceable URIs in the Agent Context Graph (ACG) system. All ACG resource URIs MUST be dereferenceable via HTTP GET, following Linked Data principles.

## 1. Introduction

Dereferenceable URIs are a core principle of the Semantic Web and Linked Data. They enable "follow your nose" navigation, where any URI can be resolved to retrieve information about that resource.

ACG adopts this principle to ensure:
- **Interoperability**: ACG resources can be integrated with other Linked Data systems
- **Discoverability**: Agents can discover related resources by following links
- **Verifiability**: Resource descriptions can be independently verified

## 2. Conformance Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 3. URI Requirements

### 3.1 HTTP URIs

All ACG resource URIs MUST use the `https` scheme (or `http` for development/testing).

**Compliant:**
```
https://broker.example.com/context/ctx-123
https://broker.example.com/trace/urn:uuid:abc-123
```

**Non-compliant:**
```
urn:uuid:abc-123  (not directly dereferenceable)
file:///path/to/resource  (not web-accessible)
```

### 3.2 URI Patterns

ACG defines the following URI patterns:

| Resource Type | Pattern | Example |
|---------------|---------|---------|
| Context Graph | `{broker}/context/{id}` | `https://broker.example.com/context/ctx-123` |
| Affordance | `{broker}/affordance/{id}` | `https://broker.example.com/affordance/aff-456` |
| PROV Trace | `{broker}/trace/{id}` | `https://broker.example.com/trace/trace-789` |
| AAT | `{broker}/aat/{id}` | `https://broker.example.com/aat/planner-v1` |
| Agent | `{broker}/agent/{did}` | `https://broker.example.com/agent/did:web:...` |
| Data Space | `{space}/` | `https://pod.example.com/alice/` |

### 3.3 Fragment Identifiers

Fragment identifiers (`#fragment`) MAY be used for sub-resources within a document:

```
https://broker.example.com/context/ctx-123#affordance-1
https://broker.example.com/agent/alice#profile
```

When a URI with a fragment is dereferenced, the server returns the containing document, and the client extracts the fragment-identified resource.

## 4. Content Negotiation

### 4.1 Required Media Types

ACG servers MUST support the following media types via HTTP content negotiation:

| Media Type | Format | Use Case |
|------------|--------|----------|
| `application/ld+json` | JSON-LD | **Default** - Machine-readable, broad client support |
| `text/turtle` | Turtle | Human-readable RDF, SPARQL tooling |
| `application/n-triples` | N-Triples | Simple RDF serialization |

### 4.2 Accept Header Processing

Servers MUST respect the `Accept` header. If no `Accept` header is provided, servers SHOULD return `application/ld+json`.

**Example Request:**
```http
GET /context/ctx-123 HTTP/1.1
Host: broker.example.com
Accept: text/turtle
```

**Example Response:**
```http
HTTP/1.1 200 OK
Content-Type: text/turtle

@prefix acg: <https://agentcontextgraph.dev/ontology#> .
@prefix prov: <http://www.w3.org/ns/prov#> .

<https://broker.example.com/context/ctx-123>
    a acg:ContextGraph ;
    acg:forAgent <did:web:broker.example.com:agents:alice> ;
    acg:hasTimestamp "2026-01-25T12:00:00Z"^^xsd:dateTime .
```

### 4.3 Quality Values

Servers SHOULD respect quality values in `Accept` headers:

```http
Accept: application/ld+json;q=1.0, text/turtle;q=0.8, */*;q=0.1
```

## 5. HTTP Response Requirements

### 5.1 Success Responses

For existing resources, servers MUST return:
- **200 OK**: Resource exists and is returned
- **Content-Type**: Appropriate media type
- **Link**: Header with related resources (RECOMMENDED)

**Example:**
```http
HTTP/1.1 200 OK
Content-Type: application/ld+json
Link: <https://broker.example.com/context/ctx-123/affordances>; rel="affordances"
Link: <https://broker.example.com/hydra>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation"

{
  "@context": ["https://www.w3.org/ns/hydra/core", "https://agentcontextgraph.dev/ontology"],
  "@id": "https://broker.example.com/context/ctx-123",
  "@type": ["acg:ContextGraph", "hydra:Resource"],
  ...
}
```

### 5.2 Error Responses

For non-existent resources:
- **404 Not Found**: Resource does not exist
- **410 Gone**: Resource existed but was deleted (for traces, this should not happen due to immutability)

For access control:
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Authenticated but not authorized

### 5.3 CORS Headers

For browser-based clients, servers SHOULD include CORS headers:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, HEAD, OPTIONS
Access-Control-Allow-Headers: Accept, Authorization
Access-Control-Expose-Headers: Link, Content-Type
```

## 6. Redirect Handling

### 6.1 Canonical URIs

If a resource has a canonical URI different from the requested URI, servers SHOULD return:
- **301 Moved Permanently**: For permanent redirects
- **303 See Other**: For redirecting to a related resource

### 6.2 DID to HTTP Resolution

For `did:web` identifiers, the resolution follows the DID:web specification:

```
did:web:broker.example.com:agent:alice
  → https://broker.example.com/agent/alice/did.json
```

For other DID methods, the broker MAY proxy the resolution and return:
```http
HTTP/1.1 303 See Other
Location: https://broker.example.com/agent/did:key:z6Mk.../resolved
```

## 7. Link Relations

### 7.1 Standard Link Relations

ACG uses standard IANA link relations where applicable:

| Relation | Usage |
|----------|-------|
| `self` | Canonical URI of this resource |
| `describedby` | Link to the ontology/schema |
| `collection` | Link to parent collection |
| `item` | Link to individual items |
| `prev`, `next` | Pagination |

### 7.2 ACG-Specific Link Relations

| Relation | URI | Usage |
|----------|-----|-------|
| `acg:affordances` | `https://agentcontextgraph.dev/rel/affordances` | Link to affordances in a context |
| `acg:traces` | `https://agentcontextgraph.dev/rel/traces` | Link to traces for an agent/context |
| `acg:dataSpace` | `https://agentcontextgraph.dev/rel/dataSpace` | Link to an agent's data space |

## 8. Caching

### 8.1 Cache-Control

Context Graphs are ephemeral and SHOULD NOT be cached:
```http
Cache-Control: no-store
```

Ontologies and shapes MAY be cached:
```http
Cache-Control: public, max-age=86400
ETag: "v1.0.0"
```

PROV Traces are immutable and MAY be cached indefinitely:
```http
Cache-Control: public, max-age=31536000, immutable
```

## 9. Security Considerations

### 9.1 HTTPS Requirement

All production deployments MUST use HTTPS to prevent:
- Man-in-the-middle attacks on identity assertions
- Tampering with context graphs
- Credential interception

### 9.2 Access Control

Dereferenceable URIs do not imply public access. Servers MUST implement appropriate access control:
- Verify agent identity (DID proof or WebID-TLS)
- Check data space permissions
- Enforce deontic policies

### 9.3 Information Disclosure

Servers SHOULD NOT reveal the existence of resources through different error codes (e.g., returning 404 for both non-existent and unauthorized resources).

## 10. Implementation Notes

### 10.1 Broker Implementation

ACG brokers implementing this specification MUST:
1. Route all resource URIs to appropriate handlers
2. Implement content negotiation
3. Return proper RDF representations
4. Include Hydra hypermedia controls

### 10.2 Client Implementation

ACG clients SHOULD:
1. Set appropriate `Accept` headers
2. Follow redirects (up to a reasonable limit)
3. Cache responses according to `Cache-Control` headers
4. Handle content negotiation failures gracefully

## 11. Examples

### 11.1 Dereferencing a Context Graph

**Request:**
```http
GET /context/ctx-abc123 HTTP/1.1
Host: broker.example.com
Accept: application/ld+json
Authorization: Bearer <agent-token>
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/ld+json
Link: </ontology>; rel="describedby"
Link: </hydra>; rel="http://www.w3.org/ns/hydra/core#apiDocumentation"

{
  "@context": [
    "https://www.w3.org/ns/hydra/core",
    "https://agentcontextgraph.dev/ontology"
  ],
  "@id": "https://broker.example.com/context/ctx-abc123",
  "@type": "acg:ContextGraph",
  "acg:forAgent": "did:web:broker.example.com:agents:planner-1",
  "acg:hasTimestamp": "2026-01-25T12:00:00Z",
  "acg:expiresAt": "2026-01-25T12:05:00Z",
  "acg:hasAffordance": [
    {
      "@id": "https://broker.example.com/context/ctx-abc123#aff-1",
      "@type": ["acg:Affordance", "hydra:Operation"],
      "acg:hasActionType": "aat:EmitPlan"
    }
  ]
}
```

### 11.2 Dereferencing a PROV Trace

**Request:**
```http
GET /trace/urn:uuid:550e8400-e29b-41d4-a716-446655440000 HTTP/1.1
Host: broker.example.com
Accept: text/turtle
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: text/turtle
Cache-Control: public, max-age=31536000, immutable

@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix acg: <https://agentcontextgraph.dev/ontology#> .

<https://broker.example.com/trace/urn:uuid:550e8400-e29b-41d4-a716-446655440000>
    a prov:Activity, acg:ProvTrace ;
    prov:wasAssociatedWith <did:web:broker.example.com:agents:planner-1> ;
    prov:startedAtTime "2026-01-25T12:00:00Z"^^xsd:dateTime ;
    prov:endedAtTime "2026-01-25T12:00:01Z"^^xsd:dateTime ;
    acg:interventionLabel "do(EmitPlan, {goal: 'analyze data'})" .
```

## 12. References

- [RFC 2119](https://tools.ietf.org/html/rfc2119) - Key words for use in RFCs
- [Linked Data Principles](https://www.w3.org/DesignIssues/LinkedData.html) - Tim Berners-Lee
- [Cool URIs for the Semantic Web](https://www.w3.org/TR/cooluris/) - W3C Interest Group Note
- [JSON-LD 1.1](https://www.w3.org/TR/json-ld11/) - W3C Recommendation
- [Hydra Core Vocabulary](https://www.hydra-cg.com/spec/latest/core/) - Hydra CG
- [DID Core](https://www.w3.org/TR/did-core/) - W3C Recommendation
