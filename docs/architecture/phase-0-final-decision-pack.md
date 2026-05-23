# Fase 0 - Final Decision Pack

**Estado documental:** Draft  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Cierre y congelamiento de Fase 0

## Proposito

Este documento sera el indice final de decisiones aceptadas al cerrar Fase 0. Mientras Fase 0 este en ejecucion, permanece en `Draft`.

## Criterio de uso

Fase 0 solo puede cerrarse cuando este documento liste, como minimo:

- ADRs aceptados.
- Contratos aceptados.
- Politicas aceptadas.
- Taxonomias aceptadas.
- Budgets aceptados.
- Preguntas abiertas no bloqueantes.
- Riesgos vivos.
- Orden de ejecucion de Fase 1.
- Fecha de revision.

## Regla de congelamiento

Cuando Fase 0 pase a `Accepted`, este documento funcionara como paquete rector. Cualquier cambio posterior en una decision critica debe registrarse mediante ADR nuevo o documento `Superseded`.

## Estado de Subfase 0.0

| Entregable | Estado | Responsable |
|---|---|---|
| Estructura `docs/` | Accepted | Codex / JusNova Chief Backend Architect |
| Plantilla ADR | Accepted | Codex / JusNova Chief Backend Architect |
| Plantilla JSON Schema | Accepted | Codex / JusNova Chief Backend Architect |
| Plantilla Policy | Accepted | Codex / JusNova Chief Backend Architect |
| Plantilla Checklist | Accepted | Codex / JusNova Chief Backend Architect |
| `phase-0-status.md` | Accepted | Codex / JusNova Chief Backend Architect |
| `open-questions.md` | Accepted | Codex / JusNova Chief Backend Architect |
| `risk-register.md` | Accepted | Codex / JusNova Chief Backend Architect |
| Responsables de revision | Accepted | Codex / JusNova Chief Backend Architect |

## Pendiente para cierre final

Este documento no debe marcarse como `Accepted` hasta completar subfases 0.1 a 0.14.

## Taxonomias aceptadas en Subfase 0.2

Estas taxonomias quedan aceptadas como vocabulario canonico de Fase 0. Su aceptacion no implica que los contratos posteriores ya esten implementados; implica que deberan referenciar estos nombres canonicos.

| Taxonomia | Estado | Archivo |
|---|---|---|
| Intents juridicos | Accepted | `docs/schemas/legal-intents.yaml` |
| Complejidad | Accepted | `docs/schemas/complexity-levels.yaml` |
| Areas juridicas | Accepted | `docs/schemas/legal-areas.yaml` |
| Tiers de fuente | Accepted | `docs/schemas/source-tiers.yaml` |
| Estados de vigencia | Accepted | `docs/schemas/validity-statuses.yaml` |
| Estados de host externo | Accepted | `docs/schemas/host-statuses.yaml` |
| Modos de respuesta | Accepted | `docs/schemas/response-modes.yaml` |

## ADRs aceptados en Subfase 0.3

Estos ADRs quedan aceptados como decisiones arquitectonicas. Esta lista no implica que Fase 0 completa este cerrada ni que los contracts, policies, schemas o evals posteriores ya existan.

| ADR | Decision | Estado | Dependencias posteriores |
|---|---|---|---|
| ADR-001 | High-Assurance Modular Core + Distributed Execution Layer | Accepted | Fase 1 estructura modular; workers por fases. |
| ADR-002 | Stack Backend And Infrastructure | Accepted | Fase 1 scaffold, settings, DB, Redis, OpenSearch, observabilidad. |
| ADR-003 | JusNova Live Legal Search Engine | Accepted | Subfase 0.5 contratos; Fases 4-8 implementacion. |
| ADR-004 | Launch Without Own Legal RAG | Accepted | Subfase 0.6 no-rag policy accepted; Fase 8 cache/snapshots. |
| ADR-005 | AI Provider And Model Policy | Accepted | Fase 1 ModelProvider; Subfase 0.10 provider policy. |
| ADR-006 | Document OCR Policy | Accepted | Subfase 0.9/0.10 policies; Fase 9 implementacion. |
| ADR-007 | Evidence, Answer, Citation And Claim Contracts | Accepted | Subfase 0.4 schemas; Fase 2 auditor. |
| ADR-008 | Cost Governor And Commercial Budgets | Accepted | Subfase 0.8 budgets; Fase 1 CostGovernor/UsageLedger. |
| ADR-009 | Source Registry And Validity Policy | Accepted | Subfase 0.6 source/validity/conflict/uncertainty policies accepted; Fase 4/5 registry/adapters. |
| ADR-010 | Traceability And Answer Versioning | Accepted | Subfase 0.7 trace/answer version schemas. |
| ADR-011 | Security, Privacy And Provider Boundaries | Accepted | Subfase 0.10 policies; Fase 1 ownership. |
| ADR-012 | Evaluation And Quality Gates | Accepted | Subfase 0.12 eval plan, dataset and gates. |

## Canon de numeracion ADR

La lista ADR-001 a ADR-012 definida en `docs/adr/decision-matrix.md` es la referencia canonica de Fase 0. Cualquier orden anterior del plan padre es referencia historica si contradice esta lista.

## Contratos aceptados en Subfase 0.4

Estos contratos quedan aceptados como contratos documentales. Esta lista no implica que el backend funcional, Citation Auditor, Claim Verifier o TraceObject ya esten implementados.

| Contrato | Estado | Archivo |
|---|---|---|
| Evidence Pack | Accepted | `docs/contracts/evidence-pack.schema.json` |
| Evidence Source | Accepted | `docs/contracts/source.schema.json` |
| Evidence Passage | Accepted | `docs/contracts/passage.schema.json` |
| Citation | Accepted | `docs/contracts/citation.schema.json` |
| Claim | Accepted | `docs/contracts/claim.schema.json` |
| Answer Contract | Accepted | `docs/contracts/answer-contract.schema.json` |

## Politicas aceptadas en Subfase 0.4

| Politica | Estado | Archivo |
|---|---|---|
| Citation Policy | Accepted | `docs/policies/citation-policy.md` |
| Abstention Policy | Accepted | `docs/policies/abstention-policy.md` |

## Contratos aceptados en Subfase 0.5

Estos contratos quedan aceptados como contratos documentales del JusNova Live Legal Search Engine. Esta lista no implica que adaptadores oficiales, discovery providers, fetchers, extractors, snapshots o ranking ya esten implementados.

| Contrato | Estado | Archivo |
|---|---|---|
| Legal Entity | Accepted | `docs/contracts/legal-entity.schema.json` |
| Legal Search Query | Accepted | `docs/contracts/legal-search-query.schema.json` |
| Legal Search Result | Accepted | `docs/contracts/legal-search-result.schema.json` |
| Retrieval Plan | Accepted | `docs/contracts/retrieval-plan.schema.json` |
| Retrieval Run | Accepted | `docs/contracts/retrieval-run.schema.json` |
| Evidence Quality | Accepted | `docs/contracts/evidence-quality.schema.json` |
| Provider Interfaces | Accepted | `docs/contracts/provider-interfaces.md` |

## Politicas aceptadas en Subfase 0.5

| Politica | Estado | Archivo |
|---|---|---|
| Legal Search Policy | Accepted | `docs/policies/legal-search-policy.md` |
| Source Routing Matrix | Accepted | `docs/policies/source-routing-matrix.md` |

## Politicas aceptadas en Subfase 0.6

| Politica | Estado | Archivo |
|---|---|---|
| Source Policy | Accepted | `docs/policies/source-policy.md` |
| Validity Policy | Accepted | `docs/policies/validity-policy.md` |
| Conflict Policy | Accepted | `docs/policies/conflict-policy.md` |
| Uncertainty Policy | Accepted | `docs/policies/uncertainty-policy.md` |
| No RAG Launch Policy | Accepted | `docs/policies/no-rag-launch-policy.md` |
| Abstention Policy Extension | Accepted | `docs/policies/abstention-policy.md` |

## Contratos endurecidos en Subfase 0.6

| Contrato | Estado | Ajuste |
|---|---|---|
| Evidence Source | Accepted | `snapshot_unavailable_reason`, reglas de snapshot, vigencia no aplicable para norma bloqueada, confirmacion/derogacion solo en TIER1 y warnings obligatorios en TIER2/TIER3. |
| Legal Search Result | Accepted | Enum compartido de razon de snapshot, restriccion de razon privada a `USER_DOCUMENT`, confirmacion/derogacion solo en TIER1 con pasaje extraido y warnings obligatorios en TIER2/TIER3. |

## Dependencias posteriores preservadas

- `citation-audit.schema.json` queda delegado a Subfase 0.7.
- Source Registry schema completo queda delegado a subfases posteriores y Fase 4.
- Fase 0 global permanece en `Draft` hasta completar 0.1 a 0.14.
