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
provider-registry.yaml
```

## Regla

Cuando una taxonomia exista en esta carpeta, los contratos deben referenciar sus nombres canonicos y no inventar variantes informales.

Todos los archivos YAML de esta carpeta se cargan como UTF-8 mediante un loader seguro y estricto. El loader debe rechazar claves duplicadas en cualquier nivel, cualquier tag explicito, anchors, aliases y merge keys (`<<`) antes de construir el objeto; despues valida una forma tipada cerrada con campos desconocidos prohibidos. `document_status`, `date`, `responsible` y `related_decision` son metadata obligatoria y no pueden perderse al consumir o regenerar una taxonomia. Un `safe_load` que acepte silenciosamente la ultima clave duplicada no satisface esta regla.

`scripts/validate_schemas.py` valida tanto JSON Schemas como estas taxonomias/configuraciones YAML. Los loaders runtime de budgets y provider registry deben reutilizar la misma primitiva estricta; no se admiten parsers permisivos separados.

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

## Taxonomias aceptadas en Subfase 0.8

| Taxonomia | Estado | Archivo |
|---|---|---|
| Budgets por complejidad | Accepted | `budgets.yaml` |

## Reglas especificas de Subfase 0.8

- `budgets.yaml` define perfiles exactos para `SIMPLE`, `MEDIO`, `COMPLEJO` e `INVESTIGACION`.
- `budgets.yaml` declara `cost_budget_version` para sintetizar instancias `CostBudget` conciliables.
- Las keys de `budgets.yaml` deben coincidir exactamente con `complexity-levels.yaml`.
- El vocabulario canonico es `discovery_calls_max`; no se introducen contadores ligados a un proveedor especifico.

## Taxonomias aceptadas en Subfase 0.10

| Taxonomia | Estado | Archivo |
|---|---|---|
| Data Classification | Accepted | `data-classification.yaml` |
| Provider Registry | Accepted | `provider-registry.yaml` |

## Reglas especificas de Subfase 0.10

- `data-classification.yaml` define `sensitivity_order`, `sensitivity_rank`, visibilidad por defecto, logging raw y reglas por familia de provider.
- Las keys de `sensitivity_order` y `data_classification` deben coincidir exactamente, sin extras ni faltantes.
- Los ranks de sensibilidad son unicos y avanzan de 10 en 10 segun el orden declarado.
- Todo payload o artefacto derivado hereda la clase con mayor `sensitivity_rank` entre sus entradas; esto aplica a summaries, snapshots, embeddings, trazas, indices, queries reformuladas, provider payloads y derivados persistidos.
- Cada clase declara reglas para las 11 familias canonicas de provider.
- `provider-registry.yaml` declara providers permitidos, feature flags, kill switches, clases recibidas/devueltas, region, logs operativos y `training_use_allowed = false`.
- `provider-registry.reliability_policy.error_mapping_target=provider_call_audit_error_code` declara que los valores de cada `error_mapping` pertenecen al enum interno de `ProviderCallAudit.error_code`, no al enum publico de `ErrorEnvelope.error_code`.
- Un provider no puede declarar clases prohibidas por `provider_family_rules`.
