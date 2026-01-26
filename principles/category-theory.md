# Category-Theoretic Semantics

## Overview

Context Graphs admit a **category-theoretic** view that formalizes how affordances compose.
This is a semantic layer for specification and verification, not a mandate for in-model reasoning.

## Core Principle

**Affordance traversals are morphisms.**
Objects are context state slices (resources, scopes, or invariants), and morphisms map those objects.

## Structural Requirements

1. **Objects are explicit**
   - Objects must reference state slices or resources in the context.
2. **Morphisms are explicit**
   - Each morphism should reference the affordance (or hyperedge) it represents.
3. **Composition is explicit**
   - Workflows are modeled as morphism composition; no hidden control flow.
4. **Causality is external**
   - Causal reasoning is delegated to evaluators; the category view only records structure.

## Representation

When the explicit category view is included, the Context Graph contains a `category` field with:

- `objects`: category objects referencing state/resources
- `morphisms`: affordance/hyperedge morphisms
- `composition`: explicit composition records

Example:

```json
{
  "category": {
    "objects": [
      { "id": "obj-scope", "ref": "urn:resource:project-alpha" },
      { "id": "obj-plan", "ref": "https://broker.example.com/plans" }
    ],
    "morphisms": [
      {
        "id": "m-emit-plan",
        "name": "EmitPlan",
        "domain": ["obj-scope"],
        "codomain": ["obj-plan"],
        "affordanceRef": "aff-emit-plan"
      }
    ],
    "composition": []
  }
}
```

## Why Category Theory

- **Composition**: makes multi-step workflows composable and reviewable
- **Verification**: enables formal checks of invariants across chains of actions
- **Federation**: supports functor-like mappings between domains (e.g., trust filtering)
