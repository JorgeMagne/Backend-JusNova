# ADR Governance

**Estado documental:** Accepted
**Fecha:** 2026-08-10
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Convencion de nombres y gobierno de ADRs

## Convencion de nombres

Los ADRs deben seguir el formato:

```txt
ADR-XXX-slug-descriptivo.md
```

Reglas:

- `XXX` usa tres digitos consecutivos: `001`, `002`, `003`.
- El slug va en minusculas.
- El slug usa guiones, no espacios.
- El titulo dentro del documento puede estar en espanol natural.
- Un ADR aceptado no se edita para cambiar la decision historica; se crea un ADR nuevo y el anterior pasa a `Superseded`.

## Estados ADR permitidos

| Estado | Significado |
|---|---|
| Proposed | Decision propuesta, no vinculante. |
| Accepted | Decision aprobada y vinculante. |
| Superseded | Decision reemplazada por otra. |
| Rejected | Decision considerada y descartada. |

## Estados documentales permitidos

Los archivos ADR tambien deben declarar estado documental:

- Draft
- Review
- Accepted
- Superseded

## ADRs fundacionales esperados

```txt
ADR-001-high-assurance-modular-core.md
ADR-002-stack-backend-and-infrastructure.md
ADR-003-live-legal-search-engine.md
ADR-004-launch-without-legal-rag.md
ADR-005-ai-provider-and-model-policy.md
ADR-006-document-ocr-policy.md
ADR-007-evidence-answer-citation-contracts.md
ADR-008-cost-governor-and-commercial-budgets.md
ADR-009-source-registry-and-validity-policy.md
ADR-010-traceability-and-answer-versioning.md
ADR-011-security-privacy-and-provider-boundaries.md
ADR-012-evaluation-and-quality-gates.md
```
