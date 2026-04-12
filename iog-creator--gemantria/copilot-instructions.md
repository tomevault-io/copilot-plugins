## gemantria

> Ensure JSON-LD ontology extensions are add-only for forward compatibility


# Ontology Forward Compatibility Rule

## Core Requirements

### Add-Only Extensions
- **Any new JSON-LD context terms must be add-only**
- **Never remove or rename existing context keys**
- **Backwards compatibility must be maintained**

### Documentation Requirements
- **New context terms require ADR documentation**
- **SSOT entry must be created for new terms**
- **Namespace consistency must be maintained**

### Schema Evolution
- **New properties must have default null/omitted behavior**
- **Consumers must gracefully handle unknown properties**
- **Version negotiation should use content negotiation, not schema changes**

## Validation Checklist

- [ ] New JSON-LD keys are additive only
- [ ] ADR created for ontology changes (ADR-016 for metrics)
- [ ] SSOT documentation updated with new terms
- [ ] Namespace URIs are consistent (gemantria.ai/concept/)
- [ ] Optional properties are properly omitted when null

## Namespace Management

### Current Namespaces
- **https://gemantria.ai/concept/**: Core concept properties
- **http://schema.org/**: General schema.org terms
- **http://www.w3.org/2000/01/rdf-schema#**: RDF Schema terms

### Property Classification
- **Required**: @id, @type, label
- **Recommended**: cluster, degree, betweenness, eigenvector
- **Optional**: All metric properties (semanticCohesion, etc.)

## Failure Consequences

If ontology compatibility is broken:
- Existing JSON-LD consumers fail to parse
- Semantic web tools lose compatibility
- Knowledge graph integrations break
- API stability guarantees are violated

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iog-creator) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
