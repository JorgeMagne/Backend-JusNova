# Shared Schemas And Taxonomies

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Estructura para taxonomias base de Fase 0

## Proposito

Esta carpeta contendra taxonomias YAML/JSON compartidas por contratos, policies, planners, evaluadores y documentacion de arquitectura.

## Taxonomias esperadas

```txt
budgets.yaml
source-tiers.yaml
legal-intents.yaml
legal-areas.yaml
response-modes.yaml
host-statuses.yaml
validity-statuses.yaml
complexity-levels.yaml
data-classification.yaml
```

## Regla

Cuando una taxonomia exista en esta carpeta, los contratos deben referenciar sus nombres canonicos y no inventar variantes informales.

## Taxonomias aceptadas en Subfase 0.2

| Taxonomia | Estado | Archivo |
|---|---|---|
| Intents juridicos | Accepted | `legal-intents.yaml` |
| Complejidad | Accepted | `complexity-levels.yaml` |
| Areas juridicas | Accepted | `legal-areas.yaml` |
| Tiers de fuente | Accepted | `source-tiers.yaml` |
| Estados de vigencia | Accepted | `validity-statuses.yaml` |
| Estados de host externo | Accepted | `host-statuses.yaml` |
| Modos de respuesta | Accepted | `response-modes.yaml` |
