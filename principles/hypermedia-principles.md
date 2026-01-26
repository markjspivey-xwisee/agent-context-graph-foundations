# Hypermedia Principles (HATEOAS)

## The Core Principle

**Agents must discover actions at runtime through broker-provided affordances.**

This is not optional. This is the fundamental design principle that makes the system safe.

## What This Means

### NO Global Tool Lists
Agents do not have a pre-configured list of available actions. They cannot:
- Assume an endpoint exists
- Hardcode API calls
- Pre-program workflows

### YES Dynamic Discovery
Agents receive their available actions from the Context Graph:
1. Agent requests context from broker
2. Broker evaluates agent identity, credentials, and policy
3. Broker returns only valid affordances
4. Agent acts through those affordances

## Why This Matters

### Safety Through Absence
If an agent could assume actions exist, it could attempt forbidden actions. By making all actions discoverable:
- Forbidden actions never appear
- The agent cannot even conceptualize illegal moves
- Safety is structural, not behavioral

### Adaptability
When policies change, affordances change. Agents automatically adapt:
- No code changes required
- No redeployment needed
- Policy updates are immediately effective

### Auditability
Every action goes through an affordance:
- Complete trace of what was offered
- Complete trace of what was chosen
- No hidden side channels

## Implementation Requirements

### Broker Requirements
1. Never expose forbidden actions as affordances
2. Gate affordances by credentials
3. Include all necessary metadata in affordances
4. Ensure affordances are self-describing

### Agent Requirements
1. Never assume actions—always request context first
2. Only traverse affordances from current context
3. Re-request context when situation changes
4. Never cache affordances beyond their expiration

### Anti-Patterns to Avoid
- Hardcoded endpoints
- Action lists in agent configuration
- "Fallback" actions when affordance missing
- Guessing at API structure

## Comparison to REST HATEOAS

This system applies HATEOAS principles to agent systems:

| REST HATEOAS | Agent Context Graph |
|--------------|---------------------|
| Links in responses | Affordances in Context Graph |
| Client follows links | Agent traverses affordances |
| Server controls navigation | Broker controls action space |
| Stateless requests | Context-bound actions |

The key extension: We add **semantic constraints** (AAT), **credential requirements** (VCs), and **causal semantics** to the hypermedia controls.
