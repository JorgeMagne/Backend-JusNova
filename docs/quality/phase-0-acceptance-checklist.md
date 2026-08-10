# Fase 0 - Acceptance Checklist

**Estado documental:** Accepted
**Fecha:** 2026-08-10
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Gates de cierre de Fase 0

## Proposito

Este checklist define las condiciones minimas para declarar Fase 0 cerrada. Si falta un item bloqueante, Fase 0 no pasa a Fase 1.

## Estados permitidos

| Estado | Significado |
|---|---|
| Pending | No iniciado o incompleto. |
| Review | Listo para revision. |
| Accepted | Aprobado. |
| Blocked | Bloqueado por decision o informacion faltante. |

## Checklist final de Fase 0

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| ADR-001 a ADR-012 estan Accepted o Accepted with Review Date | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/` |
| Answer Contract aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/answer-contract.schema.json` |
| Evidence Pack Contract aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/evidence-pack.schema.json` |
| Citation Contract aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/citation.schema.json` |
| Claim Contract aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/claim.schema.json` |
| Live Legal Search Contract aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/legal-search-query.schema.json` |
| Source Registry conceptual/delegation documentada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-009-source-registry-and-validity-policy.md`, `docs/policies/source-policy.md`, `docs/architecture/domain-model.md` |
| Validity Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/validity-policy.md` |
| Source Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/source-policy.md` |
| Citation Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/citation-policy.md` |
| Abstention Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/abstention-policy.md` |
| Conflict Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/conflict-policy.md` |
| Uncertainty Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/uncertainty-policy.md` |
| No RAG Launch Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/no-rag-launch-policy.md` |
| Cost Governor Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/cost-governor-policy.md` |
| `budgets.yaml` aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/budgets.yaml` |
| Plan base 400 Bs reflejado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/commercial-plans-v0.md` |
| TraceObject aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/trace-object.schema.json` |
| Provider Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/provider-policy.md` |
| Security/Privacy Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/privacy-security-policy.md` |
| Prompt Injection Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/prompt-injection-policy.md` |
| Data Classification aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/data-classification.yaml` |
| Provider Registry aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/provider-registry.yaml` |
| Modelo conceptual de dominio aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/domain-model.md` |
| Matriz de ownership por entidad aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/entity-ownership-matrix.md` |
| API draft v0 aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/api-draft-v0.md` |
| Error Envelope aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/error-envelope.schema.json` |
| Evaluation Plan v0 aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/evaluation-plan-v0.md` |
| Initial Golden Dataset Spec aprobado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/initial-golden-dataset-spec.md` |
| Beta Readiness Gates definidos | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/beta-readiness-gates.md` |
| Market Readiness Gates definidos | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/market-readiness-gates.md` |
| Phase 1 Implementation Brief listo | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/phase-1-implementation-brief.md` |
| Sprint 1 Backlog listo | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/sprint-1-backlog.md` |
| Risk Register actualizado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/risk-register.md`; documento vivo `Draft` actualizado |
| Open Questions no tiene bloqueantes | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/open-questions.md`; registro vivo `Draft` sin preguntas `Blocking` |
| Revision final por lente tecnico aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/phase-0-final-review.md` |
| Revision final por lente juridico aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/phase-0-final-review.md` |
| Revision final por lente producto/operacion aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/phase-0-final-review.md` |
| Final Decision Pack congelado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |
| Subfase 0.14 y Fase 0 global aceptadas | Si | Accepted | Usuario / Owner del producto | `docs/phase-0-status.md` |

Los items historicos que dicen "Fase 0 global no se declara completa" validan que cada subfase anterior no adelanto el cierre. El estado global vigente despues de 0.14 es `Accepted`.

## Checklist especifico de Subfase 0.0

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| Estructura `docs/` creada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/` |
| Convencion de nombres ADR creada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/README.md` |
| Plantilla ADR creada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/_ADR_TEMPLATE.md` |
| Plantilla JSON Schema creada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/_SCHEMA_TEMPLATE.json` |
| Plantilla Policy creada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/_POLICY_TEMPLATE.md` |
| Plantilla Checklist creada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/_CHECKLIST_TEMPLATE.md` |
| `phase-0-status.md` creado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/phase-0-status.md` |
| `open-questions.md` creado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/open-questions.md` |
| `risk-register.md` creado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/risk-register.md`; documento vivo `Draft` |
| Responsables de revision propuestos | No | Pending | Codex / JusNova Chief Backend Architect | `docs/handoff/review-responsibilities.md`; aceptacion requerida antes de market |

## Checklist especifico de Subfase 0.1

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| Documento de principios no negociables creado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/non-negotiable-principles.md` |
| Cada principio tiene regla tecnica | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/non-negotiable-principles.md` |
| Cada regla tecnica tiene dueno futuro | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/non-negotiable-principles.md` |
| Ninguna regla depende unicamente del prompt | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/non-negotiable-principles.md` |
| Documento queda en estado Accepted | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/non-negotiable-principles.md` |

## Checklist especifico de Subfase 0.2

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `legal-intents.yaml` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/legal-intents.yaml` |
| `complexity-levels.yaml` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/complexity-levels.yaml` |
| `legal-areas.yaml` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/legal-areas.yaml` |
| `source-tiers.yaml` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/source-tiers.yaml` |
| `validity-statuses.yaml` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/validity-statuses.yaml` |
| `host-statuses.yaml` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/host-statuses.yaml` |
| `response-modes.yaml` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/response-modes.yaml` |
| Taxonomias base quedan como canon documental para contratos posteriores | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/README.md` |

## Checklist especifico de Subfase 0.3

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| ADR-001 creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-001-high-assurance-modular-core.md` |
| ADR-002 creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-002-stack-backend-and-infrastructure.md` |
| ADR-003 creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-003-live-legal-search-engine.md` |
| ADR-004 creado y aceptado segun numeracion canonica | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-004-launch-without-legal-rag.md` |
| ADR-005 creado y aceptado segun numeracion canonica | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-005-ai-provider-and-model-policy.md` |
| ADR-006 creado y aceptado segun numeracion canonica | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-006-document-ocr-policy.md` |
| ADR-007 creado y aceptado con dependencias posteriores explicitas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-007-evidence-answer-citation-contracts.md` |
| ADR-008 creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-008-cost-governor-and-commercial-budgets.md` |
| ADR-009 creado y aceptado con dependencias posteriores explicitas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-009-source-registry-and-validity-policy.md` |
| ADR-010 creado y aceptado con dependencias posteriores explicitas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-010-traceability-and-answer-versioning.md` |
| ADR-011 creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-011-security-privacy-and-provider-boundaries.md` |
| ADR-012 creado y aceptado con dependencias posteriores explicitas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-012-evaluation-and-quality-gates.md` |
| Decision matrix creada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/decision-matrix.md` |
| Coverage matrix creada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/adr-requirements-coverage.md` |
| ADR-001 respaldado por arquitectura | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/architecture-overview.md`, `docs/architecture/module-boundaries.md` |
| Open questions sin preguntas Blocking | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/open-questions.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.4

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `evidence-pack.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/evidence-pack.schema.json` |
| `source.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/source.schema.json` |
| `passage.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/passage.schema.json` |
| `citation.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/citation.schema.json` |
| `claim.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/claim.schema.json` |
| `answer-contract.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/answer-contract.schema.json` |
| `citation-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/citation-policy.md` |
| `abstention-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/abstention-policy.md` |
| Ejemplos validos demuestran `claim -> citation -> passage -> source` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/answer-contract.schema.json`, `docs/policies/citation-policy.md` |
| `citation-audit.schema.json` queda fuera de 0.4 y delegado a 0.7 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/README.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.5

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `legal-entity.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/legal-entity.schema.json` |
| `legal-search-query.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/legal-search-query.schema.json` |
| `legal-search-result.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/legal-search-result.schema.json` |
| `retrieval-plan.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/retrieval-plan.schema.json` |
| `retrieval-run.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/retrieval-run.schema.json` |
| `evidence-quality.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/evidence-quality.schema.json` |
| `provider-interfaces.md` creado y aceptado sin provider obligatorio | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/provider-interfaces.md` |
| `legal-search-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/legal-search-policy.md` |
| `source-routing-matrix.md` cubre los 11 intents canonicos | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/source-routing-matrix.md`, `docs/schemas/legal-intents.yaml` |
| `LegalSearchQuery` no referencia presupuesto comercial de Subfase 0.8 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/legal-search-query.schema.json` |
| Ningun resultado web crudo es citable | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/legal-search-policy.md`, `docs/contracts/legal-search-result.schema.json` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.6

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `source-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/source-policy.md` |
| `validity-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/validity-policy.md` |
| `conflict-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/conflict-policy.md` |
| `uncertainty-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/uncertainty-policy.md` |
| `no-rag-launch-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/no-rag-launch-policy.md` |
| `abstention-policy.md` extendido con historial 0.4/0.6 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/abstention-policy.md` |
| `source.schema.json` representa snapshot o razon cerrada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/source.schema.json` |
| `legal-search-result.schema.json` comparte enum de snapshot reason | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/legal-search-result.schema.json` |
| ADR-004 cierra dependencia de no-rag launch policy | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-004-launch-without-legal-rag.md` |
| ADR-009 cierra dependencias de source/validity/conflict/uncertainty policies | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-009-source-registry-and-validity-policy.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.7

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `trace-object.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/trace-object.schema.json` |
| `model-call.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/model-call.schema.json` |
| `tool-call.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/tool-call.schema.json` |
| `citation-audit.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/citation-audit.schema.json` |
| `answer-version.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/answer-version.schema.json` |
| `abstention-render.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/abstention-render.schema.json` |
| `cost-report.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/cost-report.schema.json` |
| `answer-versioning-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/answer-versioning-policy.md` |
| `trace-visibility-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/trace-visibility-policy.md` |
| Trazas no permiten prompts, salidas, documentos ni mensajes completos como propiedades validas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/trace-object.schema.json`, `docs/contracts/model-call.schema.json`, `docs/contracts/tool-call.schema.json` |
| AnswerVersion no embebe respuesta completa sensible | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/answer-version.schema.json` |
| ADR-010 cierra dependencias documentales de trace y answer version | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-010-traceability-and-answer-versioning.md` |
| ADR-007 registra que CitationAudit queda aceptado en 0.7 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-007-evidence-answer-citation-contracts.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.8

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `budgets.yaml` creado y aceptado con metadata documental | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/budgets.yaml` |
| `cost-budget.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/cost-budget.schema.json` |
| `usage-event.schema.json` creado, aceptado y validado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/usage-event.schema.json` |
| `commercial-plans-v0.md` creado y plan base 400 Bs reflejado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/commercial-plans-v0.md` |
| `cost-governor-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/cost-governor-policy.md` |
| `TraceObject` exige budget ref, budget version, plan code y complexity | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/trace-object.schema.json` |
| `TraceObject` no embebe `CostBudget` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/trace-object.schema.json` |
| Enmiendas 0.8 de referencias tecnicas en contratos 0.7 registradas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/answer-version.schema.json`, `docs/contracts/abstention-render.schema.json` |
| `trace-visibility-policy.md` registra enmienda 0.8 y visibilidad de plan/budget | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/trace-visibility-policy.md` |
| ADR-008 cierra dependencia documental de budgets y plans | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-008-cost-governor-and-commercial-budgets.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.9

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `conversation.schema.json` creado, aceptado y validado sin `user_id` crudo | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/conversation.schema.json` |
| `message.schema.json` creado, aceptado y validado con actor refs, content hash y attachments documentales procesados | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/message.schema.json` |
| `case-memory.schema.json` creado, aceptado y validado separando memoria de verdad juridica | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/case-memory.schema.json` |
| `document-evidence.schema.json` creado, aceptado y validado para fragmentos `D#:P#` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/document-evidence.schema.json` |
| `ocr-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/ocr-policy.md` |
| `document-security-policy.md` minima creada y aceptada sin reemplazar 0.10 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/document-security-policy.md` |
| `memory-policy.md` creado y aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/memory-policy.md` |
| `TraceObject` exige `input_message_ids` y `output_message_id` sin copiar contenido de mensajes | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/trace-object.schema.json` |
| Fixtures cross-contract con `trace_object` actualizados tras enmienda 0.9 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/answer-version.schema.json`, `docs/contracts/abstention-render.schema.json`, `docs/contracts/cost-budget.schema.json` |
| `trace-visibility-policy.md` registra visibilidad de refs de mensajes sin contenido | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/trace-visibility-policy.md` |
| ADR-006 registra document security minima en 0.9 y seguridad completa en 0.10 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-006-document-ocr-policy.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.10

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `data-classification.yaml` creado con ranks, visibilidad y reglas por provider family | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/data-classification.yaml` |
| `provider-registry.yaml` creado con providers canonicos, feature flags, kill switches y no training use | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/provider-registry.yaml` |
| `provider-call-audit.schema.json` creado y aceptado con `x-policy-invalid-examples` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/provider-call-audit.schema.json` |
| `raw-access-event.schema.json` creado y aceptado sin `support_operator` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/raw-access-event.schema.json` |
| `prompt-injection-risk.schema.json` creado como contrato compartido | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/prompt-injection-risk.schema.json` |
| `LegalSearchQuery` exige `query_classification` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/legal-search-query.schema.json` |
| `ModelCall` y `ToolCall` exigen `external_provider_call` y `provider_call_audit_id` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/model-call.schema.json`, `docs/contracts/tool-call.schema.json` |
| `DocumentEvidence` exige `document_evidence_id` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/document-evidence.schema.json` |
| `DocumentEvidence.prompt_injection_risks[]` es requerido y `[]` significa evaluado sin riesgos detectados | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/document-evidence.schema.json`, `docs/policies/document-security-policy.md`, `docs/policies/prompt-injection-policy.md` |
| `RetrievalRun` y `TraceObject` registran `prompt_injection_risks[]` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/retrieval-run.schema.json`, `docs/contracts/trace-object.schema.json` |
| `privacy-security-policy.md` define matriz de visibilidad por actor | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/privacy-security-policy.md` |
| `provider-policy.md`, `provider-interfaces.md` y `architecture-overview.md` comparten las 11 familias canonicas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/provider-policy.md`, `docs/contracts/provider-interfaces.md`, `docs/architecture/architecture-overview.md` |
| `prompt-injection-policy.md` trata documentos, HTML, snippets y OCR como datos no confiables | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/prompt-injection-policy.md` |
| `security-checklist-phase-1.md` creado para handoff minimo de seguridad | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/security-checklist-phase-1.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.11

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `domain-model.md` creado con entidades, IDs globales, refs locales y relaciones/cardinalidades | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/domain-model.md` |
| Cardinalidades permiten shells para Conversation, Document, Answer, Subscription y ResearchCredit | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/domain-model.md` |
| `run_id=tr_*` en progreso no se trata como `TraceObject` valido hasta finalizacion | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/domain-model.md`, `docs/architecture/api-draft-v0.md` |
| `entity-ownership-matrix.md` cubre toda entidad y no expande `RawAccessEvent.resource_type` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/entity-ownership-matrix.md` |
| Entidades sin raw access 0.10 explicitan `No aplica` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/entity-ownership-matrix.md` |
| `LegalSearchQuery`, `LegalEntity`, `RetrievalPlan` y `LegalSearchResult` tienen ownership e IDs canonicos | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/domain-model.md`, `docs/architecture/entity-ownership-matrix.md`, `docs/contracts/legal-search-query.schema.json`, `docs/contracts/retrieval-plan.schema.json` |
| `AnswerContract`, `AnswerVersion`, `AbstentionRender` y `EvidencePack` exigen tenant `org_*` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/answer-contract.schema.json`, `docs/contracts/answer-version.schema.json`, `docs/contracts/abstention-render.schema.json`, `docs/contracts/evidence-pack.schema.json` |
| `api-draft-v0.md` no expone memoria, documentos, OCR, trazas ni payloads de provider crudos | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/api-draft-v0.md` |
| `/cases/{case_id}/memory` devuelve `CaseMemorySafeSummary`, no `CaseMemory` crudo | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/api-draft-v0.md` |
| `/documents/{document_id}` devuelve metadata/status sin binario, OCR completo ni signed URL | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/api-draft-v0.md` |
| Stream final incluye `answer_id`, `answer_version_ref` y `run_id` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/api-draft-v0.md` |
| `error-envelope.schema.json` creado, cerrado y con mapping `error_code -> safe_message_code` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/error-envelope.schema.json` |
| `module-boundaries.md` y `architecture-overview.md` registran enmienda 0.11 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/module-boundaries.md`, `docs/architecture/architecture-overview.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.12

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| ADR-012 actualizado con metricas, gates y waiver policy | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-012-evaluation-and-quality-gates.md` |
| `evaluation-plan-v0.md` creado con 10 metricas canonicas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/evaluation-plan-v0.md` |
| Metricas beta tienen target, blocker, definicion, numerador, denominador y consecuencia | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/evaluation-plan-v0.md` |
| `initial-golden-dataset-spec.md` creado con buckets, tags, enums, refs locales, claims esperados, fuentes esperadas y grafo de citas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/initial-golden-dataset-spec.md` |
| Golden dataset usa `legal_intents[]` canonicos y `expected_validity_status` aceptado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/initial-golden-dataset-spec.md`, `docs/schemas/legal-intents.yaml`, `docs/schemas/validity-statuses.yaml` |
| Expected blocked outcomes usan `expected_block_reason` o `expected_error_code` cerrados | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/initial-golden-dataset-spec.md`, `docs/contracts/trace-object.schema.json`, `docs/contracts/error-envelope.schema.json` |
| Beta readiness gates definidos con evidencia, owner, blocker, fase y respaldo contractual | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/beta-readiness-gates.md` |
| Market readiness gates definidos con revision humana, seguridad, backups, soporte e incidentes | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/market-readiness-gates.md` |
| Source Registry/Snapshot schemas inexistentes no quedan como blockers de Fase 0 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/phase-0-acceptance-checklist.md`, `docs/architecture/phase-0-final-decision-pack.md` |
| No quedan menciones legacy de fase posterior de evaluacion en `docs/` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/`, `docs/policies/non-negotiable-principles.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/phase-0-status.md`, `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.13

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| `phase-1-implementation-brief.md` creado y alineado a contratos/rutas canonicas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/phase-1-implementation-brief.md` |
| `sprint-1-backlog.md` creado con prioridades P0/P1/P2 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/sprint-1-backlog.md` |
| `phase-1-development-plan.md` creado como plan operativo de Fase 1 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/phases/phase-1-development-plan.md` |
| Handoff usa JSON Schemas desde `docs/contracts/` y budgets desde `docs/schemas/budgets.yaml` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/phase-1-implementation-brief.md`, `docs/handoff/sprint-1-backlog.md` |
| `ErrorEnvelope` de Fase 1 respeta `error-envelope.schema.json` y `api-draft-v0.md` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/error-envelope.schema.json`, `docs/architecture/api-draft-v0.md`, `docs/handoff/phase-1-implementation-brief.md` |
| Runs/eventos operativos no crean entidades canonicas nuevas ni sustituyen `TraceObject` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/domain-model.md`, `docs/handoff/phase-1-implementation-brief.md`, `docs/phases/phase-1-development-plan.md` |
| Fase 1 no construye busqueda viva, OCR, RAG ni analisis juridico real | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/phase-1-implementation-brief.md`, `docs/handoff/sprint-1-backlog.md`, `docs/phases/phase-1-development-plan.md` |
| Open questions no contienen bloqueantes para preparar Fase 1; inicio condicionado a 0.14/Fase 0 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/open-questions.md` |
| Fase 0 global no se declara completa | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/phase-0-status.md`, `docs/architecture/phase-0-final-decision-pack.md` |

## Checklist especifico de Subfase 0.14

| Item | Bloqueante | Estado | Responsable | Evidencia |
|---|---:|---|---|---|
| Auditoria final ejecutada con lentes tecnico, juridico y producto/operacion | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/phase-0-final-review.md` |
| Los 12 ADRs tienen estado de decision y documental `Accepted` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/adr/ADR-001-high-assurance-modular-core.md` a `docs/adr/ADR-012-evaluation-and-quality-gates.md` |
| JSON Schemas contractuales compilan como Draft 2020-12 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/` |
| Taxonomias y budgets YAML parsean y conservan metadata de gobierno | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/schemas/` |
| Source Registry/Snapshot schemas completos permanecen delegados y no bloquean Fase 0 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/phase-0-final-review.md`, `docs/adr/ADR-009-source-registry-and-validity-policy.md` |
| `validity-status.schema.json` y `conflict-report.schema.json` historicos tienen reemplazo canonico explicito y no se crean como contratos paralelos | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md`, `docs/schemas/validity-statuses.yaml`, `docs/contracts/evidence-quality.schema.json` |
| `UNKNOWN` queda excluido de `Source.tier` y reservado a estados internos o taxonomias que lo declaran | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/source-policy.md`, `docs/schemas/source-tiers.yaml`, `docs/schemas/host-statuses.yaml` |
| Claims juridicos criticos no pueden degradar criticality ni quedar fuera del grafo visible; golden dataset define oracle semantico independiente | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/claim.schema.json`, `docs/policies/citation-policy.md`, `docs/quality/initial-golden-dataset-spec.md` |
| `EvidencePack` respeta RetrievalRun opcional y reutiliza `EvidenceQuality` como contrato unico | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/evidence-pack.schema.json`, `docs/contracts/evidence-quality.schema.json`, `docs/architecture/domain-model.md` |
| API draft define upload multipart y variantes de respuesta bloqueada/abstenida sin refs de evidencia inventadas | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/api-draft-v0.md` |
| Auth productiva y PRL determinista quedan como blockers pre-beta ejecutables en Fase 1 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/quality/beta-readiness-gates.md`, `docs/handoff/sprint-1-backlog.md`, `docs/policies/provider-policy.md` |
| Final Decision Pack indexa Subfase 0.1 y la matriz de cobertura de requisitos ADR | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md`, `docs/policies/non-negotiable-principles.md`, `docs/adr/adr-requirements-coverage.md` |
| PRL falla cerrada sin ruta productiva y ProviderCallAudit conserva contexto auditable por intento | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/contracts/provider-interfaces.md`, `docs/contracts/provider-call-audit.schema.json`, `docs/schemas/provider-registry.yaml`, `docs/policies/provider-policy.md` |
| ValidityResolver base queda asignado a Fase 4 y su integracion con adapters a Fase 5 | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/non-negotiable-principles.md`, `docs/adr/adr-requirements-coverage.md` |
| Dependencias P0-12/P0-13 distinguen dependencia dirigida de integracion atomica sin ciclo | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/sprint-1-backlog.md` |
| Open Questions tiene cero preguntas `Blocking` | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/open-questions.md` |
| Risk Register vivo fue revisado y actualizado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/risk-register.md` |
| Orden de Fase 1 esta definido sin contratos, rutas o enums paralelos | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/phase-1-implementation-brief.md`, `docs/handoff/sprint-1-backlog.md`, `docs/phases/phase-1-development-plan.md` |
| Final Decision Pack lista decisiones, preguntas, riesgos, orden y fecha de revision | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/architecture/phase-0-final-decision-pack.md` |
| Cambios criticos post-freeze requieren ADR nuevo o documento `Superseded` | Si | Accepted | Usuario / Owner del producto | `docs/architecture/phase-0-final-decision-pack.md` |
| Subfase 0.14 pasa a `Accepted` | Si | Accepted | Usuario / Owner del producto | `docs/phase-0-status.md` |
| Fase 0 global pasa a `Accepted` y Fase 1 queda habilitada | Si | Accepted | Usuario / Owner del producto | `docs/phase-0-status.md`, `docs/quality/phase-0-final-review.md` |
