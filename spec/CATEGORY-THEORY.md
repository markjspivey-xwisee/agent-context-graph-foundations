# Context Category Specification

This document defines the optional category-theoretic view of a Context Graph.
It is normative for any payload that includes a `category` field.

## Data Model

- `category.objects` MUST be an array of category objects.
- `category.morphisms` MUST be an array of morphisms.
- Each morphism MUST declare `domain` and `codomain`.
- `morphism.affordanceRef`, when present, MUST reference an affordance id in the same context.

## Composition

- `category.composition` MAY record explicit compositions of morphisms.
- Composition records MUST provide an ordered `of` list and a `composite` id.

## Mapping

- `acg:ContextCategory` maps to `category`.
- `acg:CategoryObject` maps to entries in `category.objects`.
- `acg:Morphism` maps to entries in `category.morphisms`.
- `acg:Composition` maps to entries in `category.composition`.

## Notes

The category view is a semantic layer for formal verification and composition.
It does not encode causal reasoning, which remains external.
