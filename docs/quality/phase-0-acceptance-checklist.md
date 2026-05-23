# Fase 0 - Acceptance Checklist

**Estado documental:** Draft  
**Fecha:** 2026-05-22  
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
| Source Registry Contract aprobado | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/contracts/source-registry-entry.schema.json` |
| Validity Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/validity-policy.md` |
| Source Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/source-policy.md` |
| Citation Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/citation-policy.md` |
| Abstention Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/abstention-policy.md` |
| Conflict Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/conflict-policy.md` |
| Uncertainty Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/uncertainty-policy.md` |
| No RAG Launch Policy aprobada | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/policies/no-rag-launch-policy.md` |
| Cost Governor Policy aprobada | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/policies/cost-governor-policy.md` |
| `budgets.yaml` aprobado | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/schemas/budgets.yaml` |
| Plan base 400 Bs reflejado | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/policies/commercial-plans-v0.md` |
| TraceObject aprobado | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/contracts/trace-object.schema.json` |
| Provider Policy aprobada | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/policies/provider-policy.md` |
| Security/Privacy Policy aprobada | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/policies/privacy-security-policy.md` |
| Evaluation Plan v0 aprobado | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/quality/evaluation-plan-v0.md` |
| Beta Readiness Gates definidos | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/quality/beta-readiness-gates.md` |
| Market Readiness Gates definidos | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/quality/market-readiness-gates.md` |
| Phase 1 Implementation Brief listo | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/handoff/phase-1-implementation-brief.md` |
| Sprint 1 Backlog listo | Si | Pending | Codex / JusNova Chief Backend Architect | `docs/handoff/sprint-1-backlog.md` |
| Risk Register actualizado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/risk-register.md` |
| Open Questions no tiene bloqueantes | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/open-questions.md` |

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
| `risk-register.md` creado | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/risk-register.md` |
| Responsables de revision definidos | Si | Accepted | Codex / JusNova Chief Backend Architect | `docs/handoff/review-responsibilities.md` |

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
