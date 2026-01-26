# ACG Linked Data Principles

**Version:** 1.0.0
**Status:** Normative
**Created:** 2026-01-25

## Abstract

This document establishes the foundational Linked Data principles for the Agent Context Graph (ACG) system. ACG adheres to Tim Berners-Lee's Linked Data principles and Kingsley Idehen's vision of URIs as universal "superkeys" for entity identification.

## 1. URIs as Superkeys

### 1.1 Foundational Principle

**Every entity in ACG is identified by a URI that serves as its universal superkey.**

A URI in ACG is not merely an address—it is:
- **An identifier**: Uniquely names the entity across all systems
- **A key**: Can be used to retrieve information about the entity
- **A link**: Connects to related entities via typed relationships
- **Verifiable**: Can be dereferenced to obtain authoritative descriptions

### 1.2 URI-Identified Entities

All ACG entities MUST have URI identifiers:

| Entity Type | URI Pattern | Example |
|-------------|-------------|---------|
| Agent | `{broker}/agent/{did}` | `https://broker.example.com/agent/did:web:alice.example.com` |
| Context Graph | `{broker}/context/{id}` | `https://broker.example.com/context/ctx-abc123` |
| Affordance | `{context}#aff-{id}` | `https://broker.example.com/context/ctx-abc123#aff-1` |
| PROV Trace | `{broker}/trace/{uuid}` | `https://broker.example.com/trace/urn:uuid:550e8400...` |
| Data Space | `{provider}/{owner}/` | `https://pod.example.com/alice/` |
| AAT | `{broker}/aat/{id}` | `https://broker.example.com/aat/planner-v1` |

### 1.3 URI Properties

ACG URIs MUST be:

1. **Persistent**: URIs SHOULD NOT change once assigned
2. **Dereferenceable**: HTTP GET returns an RDF description
3. **Content-negotiable**: Supports multiple RDF serializations
4. **Secure**: Uses HTTPS in production

### 1.4 URI as Identity Anchor

The URI serves as the anchor for all statements about an entity:

```turtle
# The URI IS the entity's identity
<https://broker.example.com/agent/did:web:alice.example.com>
    a acg:Agent, foaf:Agent ;
    acg:hasDID "did:web:alice.example.com" ;
    acg:hasAgentType aat:PlannerAgentType ;
    foaf:name "Alice Planning Agent" ;
    id:hasIdentity <#did-identity>, <#webid-identity> .
```

When you dereference the URI, you get **everything known about that entity**.

## 2. The Four Linked Data Principles

ACG implements Tim Berners-Lee's four Linked Data principles:

### 2.1 Use URIs as Names for Things

**Principle**: Use URIs to identify things.

**ACG Implementation**:
- Every agent has a URI (derived from or linked to its DID)
- Every context graph has a URI
- Every affordance has a URI
- Every trace has a URI

```
✓ https://broker.example.com/agent/did:web:alice
✗ "alice" (string identifier without URI)
✗ 12345 (numeric ID)
```

### 2.2 Use HTTP URIs

**Principle**: Use HTTP URIs so people can look up those names.

**ACG Implementation**:
- All resource URIs use `https://` scheme
- DIDs are bridged to HTTP URIs via the broker
- WebIDs are natively HTTP URIs

```
✓ https://broker.example.com/context/ctx-123
✗ urn:uuid:550e8400-e29b-41d4-a716-446655440000 (not directly dereferenceable)
✗ did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK (requires resolution)
```

Note: Non-HTTP identifiers (URNs, DIDs) are still used internally but are exposed via HTTP URIs for dereferencing.

### 2.3 Provide Useful Information

**Principle**: When someone looks up a URI, provide useful information using standards (RDF, SPARQL).

**ACG Implementation**:
- All URIs return RDF descriptions (JSON-LD, Turtle, N-Triples)
- Descriptions include type, properties, and relationships
- SPARQL endpoints enable complex queries

```http
GET /agent/did:web:alice HTTP/1.1
Accept: text/turtle

# Returns:
@prefix acg: <https://agentcontextgraph.dev/ontology#> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

<https://broker.example.com/agent/did:web:alice>
    a acg:Agent, foaf:Agent ;
    acg:hasDID "did:web:alice.example.com" ;
    acg:hasAgentType <https://broker.example.com/aat/planner-v1> ;
    acg:hasDataSpace <https://pod.example.com/alice/> ;
    foaf:name "Alice Planning Agent" .
```

### 2.4 Include Links to Other URIs

**Principle**: Include links to other URIs so they can discover more things.

**ACG Implementation**:
- Agents link to their AAT, data space, and credentials
- Contexts link to their affordances, constraints, and traces
- Traces link to contexts, agents, and generated artifacts
- Data spaces link to containers and access controls

```turtle
<https://broker.example.com/context/ctx-123>
    a acg:ContextGraph ;
    acg:forAgent <https://broker.example.com/agent/did:web:alice> ;      # Link to agent
    acg:hasAffordance <#aff-1>, <#aff-2> ;                                # Links to affordances
    acg:hasTracePolicy <https://broker.example.com/policy/default> ;     # Link to policy
    prov:wasGeneratedBy <https://broker.example.com/trace/trace-001> .   # Link to trace
```

## 3. Five-Star Linked Data

ACG targets **5-star Linked Data** quality:

| Stars | Requirement | ACG Status |
|-------|-------------|------------|
| ★ | Available on the web (any format) with open license | ✓ All resources web-accessible |
| ★★ | Available as machine-readable structured data | ✓ JSON-LD, Turtle, N-Triples |
| ★★★ | Non-proprietary format (e.g., CSV, not Excel) | ✓ W3C standard RDF formats |
| ★★★★ | Use URIs to denote things | ✓ All entities have URIs |
| ★★★★★ | Link your data to other data (context) | ✓ Extensive linking via ontology |

### 3.1 Achieving 5-Star Quality

To achieve 5-star quality, ACG data:

1. **Links to standard vocabularies**:
   - PROV-O for provenance
   - FOAF for agents
   - Schema.org for actions
   - Hydra for hypermedia
   - ODRL for policies

2. **Links to external datasets** (where applicable):
   - DBpedia for entity disambiguation
   - Wikidata for knowledge enrichment
   - Domain-specific ontologies

3. **Provides bidirectional links**:
   - Contexts link to traces, traces link back to contexts
   - Agents link to data spaces, data spaces link to owners

## 4. WebACL Integration

ACG uses W3C Web Access Control (WebACL) for fine-grained authorization:

### 4.1 ACL Documents

Every ACG resource can have an associated ACL document:

```
Resource: https://broker.example.com/context/ctx-123
ACL:      https://broker.example.com/context/ctx-123.acl
```

### 4.2 Access Modes

Standard WebACL modes plus ACG extensions:

| Mode | Description |
|------|-------------|
| `acl:Read` | Read resource content |
| `acl:Write` | Modify resource |
| `acl:Append` | Add to resource (traces) |
| `acl:Control` | Modify ACL |
| `wacl:TraverseAffordance` | Invoke affordance action |
| `wacl:RequestContext` | Request context from broker |
| `wacl:EmitTrace` | Emit provenance trace |
| `wacl:QuerySPARQL` | Execute SPARQL queries |

### 4.3 Example ACL

```turtle
@prefix acl: <http://www.w3.org/ns/auth/acl#> .
@prefix wacl: <https://agentcontextgraph.dev/webacl#> .

# Owner has full control
<#owner>
    a acl:Authorization ;
    acl:agent <https://broker.example.com/agent/did:web:alice> ;
    acl:accessTo <https://broker.example.com/context/ctx-123> ;
    acl:mode acl:Read, acl:Write, acl:Control .

# Planner agents can read and traverse
<#plannerAccess>
    a wacl:ACGAuthorization ;
    wacl:agentType aat:PlannerAgentType ;
    acl:accessTo <https://broker.example.com/context/ctx-123> ;
    acl:mode acl:Read, wacl:TraverseAffordance .

# Anyone can read traces (audit transparency)
<#publicTraceRead>
    a acl:Authorization ;
    acl:agentClass foaf:Agent ;
    acl:accessTo <https://broker.example.com/traces/> ;
    acl:mode acl:Read .
```

## 5. Implementation Requirements

### 5.1 Broker Requirements

ACG brokers MUST:
1. Assign HTTP URIs to all managed resources
2. Support content negotiation (JSON-LD, Turtle, N-Triples)
3. Return RDF descriptions for all resource URIs
4. Include typed links to related resources
5. Support WebACL for access control

### 5.2 Agent Requirements

ACG agents SHOULD:
1. Have at least one HTTP-dereferenceable identity (WebID or bridged DID)
2. Follow links to discover related resources
3. Respect WebACL authorization decisions
4. Include provenance links in generated artifacts

### 5.3 Data Space Requirements

ACG data spaces MUST:
1. Expose LDP Containers for standard CRUD
2. Provide `.acl` documents for access control
3. Support SPARQL queries where applicable
4. Link to owner identity and related spaces

## 6. Vocabulary Alignment

ACG aligns with established vocabularies:

| ACG Concept | Aligned Vocabulary | Relationship |
|-------------|-------------------|--------------|
| `acg:Agent` | `foaf:Agent` | `owl:equivalentClass` |
| `acg:ContextGraph` | `prov:Entity` | `rdfs:subClassOf` |
| `acg:ProvTrace` | `prov:Activity` | `rdfs:subClassOf` |
| `acg:Affordance` | `hydra:Operation` | `rdfs:subClassOf` |
| `acg:Affordance` | `schema:Action` | `rdfs:subClassOf` |
| `ds:DataSpace` | `ldp:Container` | `rdfs:subClassOf` |
| `acg:DeonticConstraint` | `odrl:Policy` | `rdfs:subClassOf` |

## 7. References

- [Linked Data Principles](https://www.w3.org/DesignIssues/LinkedData.html) - Tim Berners-Lee
- [5-Star Linked Data](https://5stardata.info/) - Tim Berners-Lee
- [WebACL Specification](https://www.w3.org/wiki/WebAccessControl) - W3C
- [Solid WebACL](https://solidproject.org/TR/wac) - Solid Project
- [Linked Data Platform](https://www.w3.org/TR/ldp/) - W3C Recommendation
- [URI Persistence](https://www.w3.org/Provider/Style/URI) - W3C
- [Cool URIs for the Semantic Web](https://www.w3.org/TR/cooluris/) - W3C

## 8. Acknowledgments

These principles are heavily influenced by:
- Tim Berners-Lee's vision of the Semantic Web
- Kingsley Uyi Idehen's work on Linked Data and Virtuoso
- The Solid Project's approach to user data sovereignty
- The broader semantic web community
