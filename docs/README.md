# JusNova Backend Docs

**Estado documental:** Accepted
**Fecha:** 2026-08-10
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Fase 0.0 - Preparacion y gobierno documental

## Proposito

Esta carpeta contiene el paquete rector tecnico de JusNova Backend. Las decisiones vinculantes de Fase 0 viven aqui, versionadas en git, no en conversaciones, archivos externos o entendimiento verbal.

## Estructura

```txt
docs/
  architecture/  Arquitectura, limites modulares, modelo conceptual y decision pack.
  adr/           Architecture Decision Records.
  contracts/     JSON Schemas y contratos internos.
  policies/      Politicas operativas y juridicas del sistema.
  schemas/       Taxonomias YAML/JSON compartidas.
  quality/       Gates, evaluacion, checklists y criterios de aceptacion.
  handoff/       Backlog, riesgos, preguntas y transferencia a Fase 1.
  phases/        Planes de ejecucion por fase.
```

## Regla de gobierno

Una decision critica no es vinculante hasta que este documentada en esta carpeta, tenga responsable, estado documental y criterios de aceptacion cuando correspondan.

## Entregables de calidad 0.12

Los quality gates de 0.12 son vinculantes para Fase 1 y para cualquier decision de beta o mercado:

- `quality/evaluation-plan-v0.md`
- `quality/initial-golden-dataset-spec.md`
- `quality/beta-readiness-gates.md`
- `quality/market-readiness-gates.md`
- `quality/phase-0-acceptance-checklist.md`
- `adr/ADR-012-evaluation-and-quality-gates.md`

Fase 1 debe implementar harness/eval runner y reportes sin debilitar los blockers aceptados en estos documentos.

## Entregables de handoff 0.13

El handoff de 0.13 es vinculante para ejecutar Fase 1; la condicion de cierre formal de 0.14/Fase 0 quedo satisfecha el 2026-08-10:

- `handoff/phase-1-implementation-brief.md`
- `handoff/sprint-1-backlog.md`
- `phases/phase-1-development-plan.md`

Estos documentos deben leerse junto con `architecture/domain-model.md`, `architecture/api-draft-v0.md`, `contracts/error-envelope.schema.json` y `schemas/budgets.yaml`; no autorizan rutas, enums o contratos paralelos.

## Freeze de Fase 0 - Subfase 0.14

Fase 0 esta `Accepted` y congelada desde 2026-08-10. Los artefactos de cierre son:

- `quality/phase-0-final-review.md`
- `architecture/phase-0-final-decision-pack.md`
- `quality/phase-0-acceptance-checklist.md`
- `phase-0-status.md`

Fase 1 puede iniciar con el handoff 0.13. Cualquier cambio posterior a una decision critica de Fase 0 requiere ADR nuevo o documento `Superseded`. Beta y mercado siguen bloqueados por sus readiness gates.
