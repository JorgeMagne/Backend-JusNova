# Open Questions

**Estado documental:** Draft
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Registro de dudas abiertas de Fase 0

## Regla

Fase 0 puede cerrar con preguntas no bloqueantes, pero no puede cerrar con preguntas bloqueantes sobre arquitectura, evidencia, citas, vigencia, trazabilidad, costos, seguridad o lanzamiento sin RAG.

## Clasificacion

| Tipo | Significado |
|---|---|
| Blocking | Impide cerrar Fase 0 o iniciar Fase 1. |
| Non-blocking | Puede resolverse en Fase 1 o durante evaluacion real. |
| Resolved | Ya fue respondida por ADR, policy, schema o decision documentada. |
| Deferred | Se resolvera con metricas o decisiones comerciales futuras sin bloquear Fase 0. |

## Preguntas abiertas iniciales

| ID | Pregunta | Tipo | Estado | Responsable | Resolucion esperada |
|---|---|---|---|---|---|
| OQ-001 | Proveedor final de object storage S3-compatible para produccion inicial. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Resolver tras ADR-002 con criterios de operacion, costo y licencia. |
| OQ-002 | Temporal desde Fase 1 o cola durable inicial si operacion local bloquea avance. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Cerrar al completar [P2-06](sprint-1-backlog.md#p2-06-workflowgateway-y-localworkflowgateway): Temporal queda como meta de workflow, no P0 de Sprint 1; Fase 1 debe crear `WorkflowGateway`/`LocalWorkflowGateway` sin acoplar el core a Temporal. |
| OQ-003 | Proveedor secundario de web discovery si adaptadores oficiales y proveedor inicial no alcanzan recall. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Resolver despues de benchmarks iniciales de retrieval. |
| OQ-004 | Limites exactos de OCR por plan despues de medir costos reales. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Ajustar tras pruebas de OCR y Cost Governor. |
| OQ-005 | Lista canonica ADR-001 a ADR-012 de Fase 0 detallada. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto por `docs/adr/decision-matrix.md`. |
| OQ-006 | Si Evidence Cache puede tratarse como corpus juridico completo. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por ADR-004. |
| OQ-007 | Si una pregunta de Fase 0.3 queda bloqueando aceptacion. | Resolved | Closed | Codex / JusNova Chief Backend Architect | No hay preguntas `Blocking` al cierre de Subfase 0.3. |
| OQ-008 | Donde vive la razon de no snapshot para una fuente usada. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto por `source.schema.json` con `snapshot_unavailable_reason` en Subfase 0.6. |
| OQ-009 | Si cache o snapshot confirman vigencia. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `validity-policy.md` y `no-rag-launch-policy.md`. |
| OQ-010 | Si `Conversation.title` puede usarse como memoria probatoria o fuente juridica. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `conversation.schema.json` y `memory-policy.md` en Subfase 0.9. |
| OQ-011 | Si seguridad documental completa queda cerrada en 0.9. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto: 0.9 solo define minimo documental; Subfase 0.10 acepta privacy/security, provider boundaries, raw access y prompt injection contracts. |
| OQ-012 | Si soporte general puede acceder a prompts, documentos, mensajes o salidas raw. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `privacy-security-policy.md`, `trace-visibility-policy.md` y `raw-access-event.schema.json`; soporte normal usa `SUPPORT_VIEW` redacted y `support_operator` no existe en `RawAccessEvent`. |
| OQ-013 | Si llamadas externas de provider pueden quedar sin auditoria. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `provider-call-audit.schema.json`, `model-call.schema.json`, `tool-call.schema.json` y `provider-policy.md`. |
| OQ-014 | Si un `run_id` en progreso es ya un `TraceObject` valido. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `domain-model.md` y `api-draft-v0.md`: `run_id=tr_*` es reservado operacional hasta finalizacion con `answer_id` y `answer_version_ref`. |
| OQ-015 | Si `/v1/cases/{case_id}/memory` devuelve `CaseMemory` crudo. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `api-draft-v0.md`: el endpoint devuelve `CaseMemorySafeSummary`; raw/elevated access requiere `RawAccessEvent`. |
| OQ-016 | Si evaluacion puede ser opcional antes de beta. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `evaluation-plan-v0.md` y `beta-readiness-gates.md`: blockers no medibles o fallidos bloquean beta. |
| OQ-017 | Herramienta concreta para eval runner y formato ejecutable de reportes. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Definir en Fase 1 sin cambiar metricas, dataset spec ni gates de 0.12. |
| OQ-018 | Plataforma CI exacta para ejecutar regression suite. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Resolver durante Fase 1 segun infraestructura real. |
| OQ-019 | Revisores humanos finales para market readiness. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Nombrar antes de market readiness; 0.12 ya exige revision humana. |

## Preguntas Blocking

No hay preguntas `Blocking` registradas al estado actual de Fase 0.
