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
| OQ-002 | Temporal desde Fase 1 o cola durable inicial si operacion local bloquea avance. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Resolver en handoff de Fase 1 manteniendo Temporal como meta profesional segun ADR-002. |
| OQ-003 | Proveedor secundario de web discovery si adaptadores oficiales y proveedor inicial no alcanzan recall. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Resolver despues de benchmarks iniciales de retrieval. |
| OQ-004 | Limites exactos de OCR por plan despues de medir costos reales. | Non-blocking | Open | Codex / JusNova Chief Backend Architect | Ajustar tras pruebas de OCR y Cost Governor. |
| OQ-005 | Lista canonica ADR-001 a ADR-012 de Fase 0 detallada. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto por `docs/adr/decision-matrix.md`. |
| OQ-006 | Si Evidence Cache puede tratarse como corpus juridico completo. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por ADR-004. |
| OQ-007 | Si una pregunta de Fase 0.3 queda bloqueando aceptacion. | Resolved | Closed | Codex / JusNova Chief Backend Architect | No hay preguntas `Blocking` al cierre de Subfase 0.3. |
| OQ-008 | Donde vive la razon de no snapshot para una fuente usada. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto por `source.schema.json` con `snapshot_unavailable_reason` en Subfase 0.6. |
| OQ-009 | Si cache o snapshot confirman vigencia. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `validity-policy.md` y `no-rag-launch-policy.md`. |
| OQ-010 | Si `Conversation.title` puede usarse como memoria probatoria o fuente juridica. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto negativamente por `conversation.schema.json` y `memory-policy.md` en Subfase 0.9. |
| OQ-011 | Si seguridad documental completa queda cerrada en 0.9. | Resolved | Closed | Codex / JusNova Chief Backend Architect | Resuelto: 0.9 solo define minimo documental; Subfase 0.10 conserva privacy/security, permisos, retencion y prompt injection completos. |

## Preguntas Blocking

No hay preguntas `Blocking` registradas al estado actual de Fase 0.
