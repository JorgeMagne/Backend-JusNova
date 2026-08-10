# Fase 0 - Final Decision Pack

**Estado documental:** Accepted
**Fecha:** 2026-08-10
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Cierre y congelamiento de Fase 0

## Proposito

Este documento es el indice final y congelado de las decisiones aceptadas en Fase 0. Fase 1 debe implementarlas sin crear rutas, enums, contratos o policies paralelas.

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

Desde el cierre de Fase 0, este documento funciona como paquete rector. Cualquier cambio posterior en una decision critica debe registrarse mediante ADR nuevo o documento `Superseded`.

## Estado de Subfase 0.0

| Entregable | Estado | Responsable |
|---|---|---|
| Estructura `docs/` | Accepted | Codex / JusNova Chief Backend Architect |
| Plantilla ADR | Accepted | Codex / JusNova Chief Backend Architect |
| Plantilla JSON Schema | Accepted | Codex / JusNova Chief Backend Architect |
| Plantilla Policy | Accepted | Codex / JusNova Chief Backend Architect |
| Plantilla Checklist | Accepted | Codex / JusNova Chief Backend Architect |
| `phase-0-status.md` | Accepted | Codex / JusNova Chief Backend Architect |
| `open-questions.md` | Draft vivo sin blockers; aceptado como evidencia de cierre | Codex / JusNova Chief Backend Architect |
| `risk-register.md` | Draft vivo actualizado | Codex / JusNova Chief Backend Architect |
| Responsables de revision propuestos | Draft | Codex / JusNova Chief Backend Architect; aceptacion requerida antes de market |

## Cierre final

Las subfases 0.0 a 0.14 estan `Accepted`. La auditoria final se registra en `docs/quality/phase-0-final-review.md` y confirma resultado `PASS` para los lentes tecnico, juridico y producto/operacion.

**Fecha de revision y freeze:** 2026-08-10

El cierre documental habilita el inicio de Fase 1. No habilita beta ni mercado, que permanecen sujetos a `beta-readiness-gates.md` y `market-readiness-gates.md`.

## Principios aceptados en Subfase 0.1

Los principios no negociables de `docs/policies/non-negotiable-principles.md` quedan `Accepted` como restricciones rectoras de arquitectura, implementacion y gates. Fase 1 y las fases posteriores deben materializarlos mediante los controles y owners definidos en su matriz de enforcement; no pueden degradarlos mediante prompts, configuracion local o contratos paralelos.

| Entregable | Estado | Archivo |
|---|---|---|
| Principios no negociables | Accepted | `docs/policies/non-negotiable-principles.md` |

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
| ADR-001 | High-Assurance Modular Core + Distributed Execution Layer | Accepted | Subfase 0.11 domain model/ownership accepted; Fase 1 estructura modular; workers por fases. |
| ADR-002 | Stack Backend And Infrastructure | Accepted | Subfase 0.11 API draft/ErrorEnvelope accepted; Fase 1 scaffold, settings, DB, Redis, OpenSearch, observabilidad. |
| ADR-003 | JusNova Live Legal Search Engine | Accepted | Subfase 0.5 contratos; Fases 4-8 implementacion. |
| ADR-004 | Launch Without Own Legal RAG | Accepted | Subfase 0.6 no-rag policy accepted; Fase 8 cache/snapshots. |
| ADR-005 | AI Provider And Model Policy | Accepted | Fase 1 ModelProvider; Subfase 0.10 provider policy. |
| ADR-006 | Document OCR Policy | Accepted | Subfase 0.9 document/OCR contracts and policies accepted; Subfase 0.10 completa seguridad; Fase 9 implementacion. |
| ADR-007 | Evidence, Answer, Citation And Claim Contracts | Accepted | Subfase 0.4 schemas; Subfase 0.7 citation audit contract; Fase 2 auditor. |
| ADR-008 | Cost Governor And Commercial Budgets | Accepted | Subfase 0.8 budgets/contracts/policies accepted; Fase 1 CostGovernor/UsageLedger. |
| ADR-009 | Source Registry And Validity Policy | Accepted | Subfase 0.6 source/validity/conflict/uncertainty policies accepted; Fase 4/5 registry/adapters. |
| ADR-010 | Traceability And Answer Versioning | Accepted | Subfase 0.7 trace/audit/version schemas accepted; Subfase 0.9 message refs accepted; Fase 3 versionado basico. |
| ADR-011 | Security, Privacy And Provider Boundaries | Accepted | Subfase 0.9 document security minima; Subfase 0.10 policies; Subfase 0.11 ownership matrix/API safe views; Fase 1 ownership. |
| ADR-012 | Evaluation And Quality Gates | Accepted | Subfase 0.12 eval plan, dataset and gates. |

La cobertura trazable entre requisitos, ADRs y fases de implementacion queda aceptada en `docs/adr/adr-requirements-coverage.md`. Esa matriz complementa este indice y no crea decisiones paralelas.

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

## Contratos aceptados en Subfase 0.7

Estos contratos quedan aceptados como contratos documentales de trazabilidad, auditoria y versionado. Esta lista no implica que persistencia, endpoints, Citation Auditor real, UI de soporte, permisos finales o retencion completa ya esten implementados.

| Contrato | Estado | Archivo |
|---|---|---|
| Trace Object | Accepted | `docs/contracts/trace-object.schema.json` |
| Model Call | Accepted | `docs/contracts/model-call.schema.json` |
| Tool Call | Accepted | `docs/contracts/tool-call.schema.json` |
| Citation Audit | Accepted | `docs/contracts/citation-audit.schema.json` |
| Answer Version | Accepted | `docs/contracts/answer-version.schema.json` |
| Abstention Render | Accepted | `docs/contracts/abstention-render.schema.json` |
| Cost Report | Accepted | `docs/contracts/cost-report.schema.json` |

## Politicas aceptadas en Subfase 0.7

| Politica | Estado | Archivo |
|---|---|---|
| Answer Versioning Policy | Accepted | `docs/policies/answer-versioning-policy.md` |
| Trace Visibility Policy | Accepted | `docs/policies/trace-visibility-policy.md` |

## Taxonomias aceptadas en Subfase 0.8

| Taxonomia | Estado | Archivo |
|---|---|---|
| Budgets por complejidad | Accepted | `docs/schemas/budgets.yaml` |

## Contratos aceptados en Subfase 0.8

Estos contratos quedan aceptados como contratos documentales de Cost Governor y Usage Ledger. Esta lista no implica que runtime, billing, Stripe, DB ni CostGovernor ya esten implementados.

| Contrato | Estado | Archivo |
|---|---|---|
| Cost Budget | Accepted | `docs/contracts/cost-budget.schema.json` |
| Usage Event | Accepted | `docs/contracts/usage-event.schema.json` |
| Trace Object Budget Amendment | Accepted | `docs/contracts/trace-object.schema.json` |
| Answer Version Reference Amendment | Accepted | `docs/contracts/answer-version.schema.json` |
| Abstention Render Reference Amendment | Accepted | `docs/contracts/abstention-render.schema.json` |

## Politicas aceptadas en Subfase 0.8

| Politica | Estado | Archivo |
|---|---|---|
| Commercial Plans v0 | Accepted | `docs/policies/commercial-plans-v0.md` |
| Cost Governor Policy | Accepted | `docs/policies/cost-governor-policy.md` |
| Trace Visibility Policy Amendment | Accepted | `docs/policies/trace-visibility-policy.md` |

## Contratos aceptados en Subfase 0.9

Estos contratos quedan aceptados como contratos documentales de conversacion, mensajes, memoria de caso, evidencia documental y OCR. Esta lista no implica que DB, storage, OCR worker, busqueda documental ni memoria persistente ya esten implementados.

| Contrato | Estado | Archivo |
|---|---|---|
| Conversation | Accepted | `docs/contracts/conversation.schema.json` |
| Message | Accepted | `docs/contracts/message.schema.json` |
| Case Memory | Accepted | `docs/contracts/case-memory.schema.json` |
| Document Evidence | Accepted | `docs/contracts/document-evidence.schema.json` |
| Trace Object Message Reference Amendment | Accepted | `docs/contracts/trace-object.schema.json` |

## Politicas aceptadas en Subfase 0.9

| Politica | Estado | Archivo |
|---|---|---|
| OCR Policy | Accepted | `docs/policies/ocr-policy.md` |
| Document Security Policy minima | Accepted | `docs/policies/document-security-policy.md` |
| Memory Policy | Accepted | `docs/policies/memory-policy.md` |
| Trace Visibility Policy Message Reference Amendment | Accepted | `docs/policies/trace-visibility-policy.md` |

## Taxonomias aceptadas en Subfase 0.10

| Taxonomia | Estado | Archivo |
|---|---|---|
| Data Classification | Accepted | `docs/schemas/data-classification.yaml` |
| Provider Registry | Accepted | `docs/schemas/provider-registry.yaml` |

## Contratos aceptados en Subfase 0.10

Estos contratos quedan aceptados como contratos documentales de seguridad, privacidad, boundaries de proveedores, auditoria de llamadas externas, auditoria raw y prompt injection. Esta lista no implica que auth, storage, provider SDKs, SIEM, DB ni permisos runtime ya esten implementados.

| Contrato | Estado | Archivo |
|---|---|---|
| Provider Call Audit | Accepted | `docs/contracts/provider-call-audit.schema.json` |
| Raw Access Event | Accepted | `docs/contracts/raw-access-event.schema.json` |
| Prompt Injection Risk | Accepted | `docs/contracts/prompt-injection-risk.schema.json` |
| Document Evidence Security Amendment | Accepted | `docs/contracts/document-evidence.schema.json` |
| Legal Search Query Classification Amendment | Accepted | `docs/contracts/legal-search-query.schema.json` |
| Model Call Provider Audit Amendment | Accepted | `docs/contracts/model-call.schema.json` |
| Tool Call Provider Audit Amendment | Accepted | `docs/contracts/tool-call.schema.json` |
| Retrieval Run Prompt Injection Amendment | Accepted | `docs/contracts/retrieval-run.schema.json` |
| Trace Object Security Amendment | Accepted | `docs/contracts/trace-object.schema.json` |

## Politicas aceptadas en Subfase 0.10

| Politica | Estado | Archivo |
|---|---|---|
| Privacy Security Policy | Accepted | `docs/policies/privacy-security-policy.md` |
| Provider Policy | Accepted | `docs/policies/provider-policy.md` |
| Prompt Injection Policy | Accepted | `docs/policies/prompt-injection-policy.md` |
| Document Security Policy Amendment | Accepted | `docs/policies/document-security-policy.md` |
| Trace Visibility Policy Security Amendment | Accepted | `docs/policies/trace-visibility-policy.md` |
| Security Checklist Phase 1 | Accepted | `docs/quality/security-checklist-phase-1.md` |

## Arquitectura y contratos aceptados en Subfase 0.11

Estos documentos quedan aceptados como base conceptual para migraciones iniciales, ownership y endpoints de Fase 1. Esta lista no implica que existan runtime, routers, migraciones, DB ni OpenAPI formal.

| Entregable | Estado | Archivo |
|---|---|---|
| Domain Model | Accepted | `docs/architecture/domain-model.md` |
| Entity Ownership Matrix | Accepted | `docs/architecture/entity-ownership-matrix.md` |
| API Draft v0 | Accepted | `docs/architecture/api-draft-v0.md` |
| Error Envelope | Accepted | `docs/contracts/error-envelope.schema.json` |
| Module Boundaries Amendment | Accepted | `docs/architecture/module-boundaries.md` |
| Architecture Overview Amendment | Accepted | `docs/architecture/architecture-overview.md` |

## Calidad aceptada en Subfase 0.12

Estos documentos quedan aceptados como criterios vinculantes para evaluar beta y mercado. Esta lista no implica que exista harness runtime, CI de evaluacion, dashboard, dataset completo ni reportes ejecutables.

| Entregable | Estado | Archivo |
|---|---|---|
| Evaluation Plan v0 | Accepted | `docs/quality/evaluation-plan-v0.md` |
| Initial Golden Dataset Spec | Accepted | `docs/quality/initial-golden-dataset-spec.md` |
| Beta Readiness Gates | Accepted | `docs/quality/beta-readiness-gates.md` |
| Market Readiness Gates | Accepted | `docs/quality/market-readiness-gates.md` |
| Phase 0 Acceptance Checklist update | Accepted | `docs/quality/phase-0-acceptance-checklist.md` |
| ADR-012 Quality Gate Amendment | Accepted | `docs/adr/ADR-012-evaluation-and-quality-gates.md` |

## Handoff aceptado en Subfase 0.13

Estos documentos quedan aceptados como handoff vinculante para Fase 1. La condicion de cierre formal de 0.14/Fase 0 quedo satisfecha el 2026-08-10. Esta lista no implica que el backend, migraciones, routers, workers, CI o eval runner ya existan; fija que debe implementarse primero y que queda fuera de alcance.

| Entregable | Estado | Archivo |
|---|---|---|
| Phase 1 Implementation Brief | Accepted | `docs/handoff/phase-1-implementation-brief.md` |
| Sprint 1 Backlog | Accepted | `docs/handoff/sprint-1-backlog.md` |
| Phase 1 Development Plan | Accepted | `docs/phases/phase-1-development-plan.md` |
| Phase 0 Acceptance Checklist update | Accepted | `docs/quality/phase-0-acceptance-checklist.md` |
| Phase 0 Status update | Accepted | `docs/phase-0-status.md` |

## Cierre aceptado en Subfase 0.14

| Entregable | Estado | Archivo |
|---|---|---|
| Final Review por tres lentes | Accepted | `docs/quality/phase-0-final-review.md` |
| Final Decision Pack congelado | Accepted | `docs/architecture/phase-0-final-decision-pack.md` |
| Phase 0 Acceptance Checklist final | Accepted | `docs/quality/phase-0-acceptance-checklist.md` |
| Phase 0 Status final | Accepted | `docs/phase-0-status.md` |
| Open Questions sin blockers | Draft vivo sin blockers; aceptado como evidencia de cierre | `docs/handoff/open-questions.md` |
| Risk Register vivo actualizado | Accepted como evidencia viva | `docs/handoff/risk-register.md` |

## Resultado de la revision final

| Lente | Resultado | Evidencia |
|---|---|---|
| Tecnico | PASS | Arquitectura coherente, contratos compilables, stack proporcionado, providers encapsulados y budgets versionados. |
| Juridico | PASS | Evidencia/citas obligatorias, vigencia conservadora, tiers visibles, abstencion cerrada y Bolivia-first. |
| Producto/operacion | PASS | Plan base 400 Bs, limites explicables, trazabilidad segura, gates verificables y handoff ejecutable. |

El detalle de preguntas, pruebas y conclusiones esta en `docs/quality/phase-0-final-review.md`.

## Open questions no bloqueantes al freeze

Permanecen abiertas o diferidas `OQ-001`, `OQ-002`, `OQ-003`, `OQ-004`, `OQ-017`, `OQ-018`, `OQ-019` y `OQ-020`. Ninguna bloquea Fase 0; `OQ-020` si bloquea beta hasta seleccionar e implementar el adapter productivo de auth. Su owner y momento de resolucion viven en `docs/handoff/open-questions.md`.

## Riesgos vivos al freeze

`docs/handoff/risk-register.md` es la fuente canonica y permanece como documento vivo. Los riesgos abiertos o en mitigacion tienen owner y tratamiento y pasan a Fase 1 o a gates pre-beta/pre-market. El cierre de 0.14 no representa aceptacion silenciosa de riesgo ni elimina controles posteriores.

## Orden vinculante de Fase 1

1. Aplicar el entry gate de `docs/phases/phase-1-development-plan.md`.
2. Ejecutar P0-01 a P0-13 de `docs/handoff/sprint-1-backlog.md` respetando sus dependencias y estrategia de PR.
3. Implementar P1 aplicable antes de cerrar Fase 1, incluidos contratos Evidence/Citation/Claim y beta blocker foundations; P1-09 es obligatorio antes de beta.
4. Ejecutar P2 sin convertir stubs opcionales en dependencias runtime duras.
5. Cerrar Fase 1 solo con los tests y criterios globales de `docs/handoff/phase-1-implementation-brief.md`.

## Dependencias posteriores preservadas

### Reconciliacion de artefactos historicos no canonicos

- No existe un contrato standalone `validity-status.schema.json`. La decision aceptada lo reemplaza por la taxonomia `docs/schemas/validity-statuses.yaml` y por los campos cerrados `validity_status` de `Source` y `LegalSearchResult`; crear otro schema produciria una segunda fuente de verdad.
- No existe un contrato standalone `conflict-report.schema.json`. El conflicto se representa mediante `EvidenceQuality.overall=CONFLICTIVE`, `conflicts_detected`, `Source.validity_status=CONFLICTIVA`, warnings/abstencion y la traza terminal. Un objeto runtime `ConflictReport` futuro requiere ADR o enmienda contractual explicita.
- `UNKNOWN` no pertenece al vocabulario de `Source.tier`. Solo puede usarse en taxonomias donde esta declarado, como estado de host/dependencia, o como estado interno no publicable antes de clasificar o rechazar una fuente.
- Planes padre, chats y archivos externos a `docs/` son referencia historica no vinculante cuando contradicen este paquete, conforme a `docs/phase-0-status.md`.

- Runtime de API, routers, OpenAPI formal, migraciones y servicios queda delegado a Fase 1.
- Source Registry schema completo queda delegado a subfases posteriores y Fase 4.
- Auth productiva minima queda delegada a P1-09 y es blocker pre-beta; permisos avanzados, retention automation y SIEM quedan para Fase 1 o fases posteriores. Storage real y provider SDKs solo se habilitan cuando su workstream pasa policy, auditoria y checklist.
- Runtime de conversacion, storage documental, OCR worker, busqueda documental y memoria persistente quedan delegados a Fase 1 y fases posteriores.
- Runtime de CostGovernor, billing y UsageLedger quedan delegados a Fase 1; los contratos y policies documentales quedaron aceptados en Subfase 0.8.
- Evaluation harness, eval runner, CI de scoring, regression suite y dataset completo quedan delegados a Fase 1.
- No hay beta sin eval report y sin cumplimiento de blockers de `beta-readiness-gates.md`.
- No hay mercado sin beta gates, market gates y revision juridica humana.
- Fase 0 global esta `Accepted` y congelada desde 2026-08-10; las dependencias anteriores permanecen explicitamente delegadas.
