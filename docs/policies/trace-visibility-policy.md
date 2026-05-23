# Trace Visibility Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.7 - Trazabilidad, auditoria y versionado

## Proposito

Definir que partes de una traza puede ver cada actor y como se protegen datos sensibles dentro de `TraceObject`, model calls, tool calls, citation audits y reportes de costo.

## Alcance

Aplica a visualizacion de trazas, soporte, auditoria interna, reconstruccion de respuestas, revision de fallos de citacion y diagnostico tecnico.

No define retencion final, permisos productivos completos ni proceso de incidente con acceso a material crudo; esos controles se cierran en Subfase 0.10.

## Niveles de visibilidad

| Nivel | Visible para | Incluye | Excluye |
|---|---|---|---|
| `USER_SUMMARY` | Usuario final | Fuentes usadas, advertencias, fecha de consulta, nivel de evidencia, resultado de vigencia expresado para usuario | Prompts, salidas completas de modelo, documentos completos, mensajes completos, hashes internos innecesarios |
| `SUPPORT_VIEW` | Soporte autorizado | Errores, latencia, proveedores, consumo tecnico estimado, estado de auditoria, IDs de trazabilidad | Prompts, documentos completos, mensajes completos, OCR completo, salida completa del modelo |
| `INTERNAL_AUDIT` | Equipo tecnico autorizado | Hashes, prompt versions, model calls, tool calls, fuentes rechazadas, citation audit, costos observados | Acceso libre a material crudo; por defecto se usan hashes y referencias |

## Reglas deterministicas

1. Toda respuesta juridica final requiere `TraceObject`.
2. `TraceObject.actor_ref` no debe ser PII directa; debe ser hash, pseudonimo o identificador tecnico controlado.
3. `TraceObject.actor_type` debe distinguir `user`, `system`, `support` o `service`.
4. `USER_SUMMARY` nunca muestra prompt completo, salida completa del modelo, documento completo ni mensaje completo.
5. `SUPPORT_VIEW` nunca muestra prompt completo, documento completo, OCR completo ni salida completa del modelo.
6. `INTERNAL_AUDIT` no significa acceso libre: usa hashes y referencias por defecto.
7. Material crudo solo puede consultarse mediante proceso de incidente definido en Subfase 0.10.
8. Todo acceso elevado registra `trace_id`, `actor_ref`, `actor_type`, `reason`, `accessed_at` y `visibility_level`.
9. `sources_rejected` debe guardar razon cerrada y hashes, no contenido crudo.
10. `latency_ms`, `cost`, `token_usage` y `cost_units` deben ser objetos cerrados sin metadata libre.
11. Model calls y tool calls registran `input_hash`; `output_hash` solo es obligatorio en `success`.
12. Estados fallidos registran `error_code` y no guardan salida completa.
13. `citation_audit` registra evaluacion estructurada de cita y claim; no guarda pasajes crudos fuera de las referencias aprobadas.
14. Si una traza se marca `blocked` o `total_abstention`, debe existir `abstention_reason` o una advertencia visible.
15. Si una traza se marca `answered` o `partial_abstention`, su `citation_audit.overall_status` debe ser `passed`.
16. Si una traza se marca `partial_abstention`, debe existir `abstention_reason` o una advertencia visible sobre la parte no respondida.
17. Si una traza se marca `answered`, todos sus claims deben tener `verification_status = passed`.
18. Si una traza se marca `partial_abstention`, debe existir al menos un claim con `verification_status = passed`.
19. Si una traza se marca `answered`, no debe existir `abstention_reason`.
20. Si una traza se marca `total_abstention`, no debe contener claims publicados ni fuentes usadas.
21. Si una traza se marca `blocked`, puede conservar claims intentados, pero ninguno puede tener `verification_status = passed`.
22. Ningun nivel de visibilidad puede convertir una fuente decorativa en fuente usada.

## Reglas asistidas por IA

1. El modelo puede resumir una traza para `USER_SUMMARY` usando solo campos permitidos.
2. El modelo puede proponer diagnostico para `SUPPORT_VIEW`, pero no puede solicitar material crudo sin motivo de incidente.
3. El modelo puede ayudar a agrupar warnings, pero no puede ocultar fallos bloqueantes.
4. El modelo no puede ampliar su propio nivel de visibilidad.

## Redaccion y hashing

| Campo o material | Regla |
|---|---|
| Prompt completo | No se guarda en contratos 0.7; usar `prompt_version` e `input_hash`. |
| Salida completa del modelo | No se guarda en contratos 0.7; usar `output_hash` cuando exista. |
| Documento completo | No se guarda en trazas; usar referencias de documento, pagina, pasaje o hash. |
| Mensaje completo del usuario | No se guarda en trazas 0.7; usar hash o referencia conversacional. |
| URL sensible | Guardar hash cuando se registre fuente rechazada. |
| Error tecnico | Guardar `error_code` y mensaje controlado cuando el contrato lo permita. |
| Costos y latencias | Guardar valores numericos observados, sin plan comercial ni precio mensual. |

## Registro de acceso elevado

Todo acceso a `SUPPORT_VIEW` o `INTERNAL_AUDIT` debe registrar:

```txt
trace_id
actor_ref
actor_type
reason
accessed_at
visibility_level
```

El motivo debe ser concreto: soporte de usuario, investigacion de incidente, evaluacion de calidad, revision de seguridad o auditoria tecnica.

## Criterios de aceptacion

- `trace-object.schema.json` no permite objetos libres para fuentes rechazadas ni latencias.
- `model-call.schema.json` y `tool-call.schema.json` no almacenan material crudo.
- `citation-audit.schema.json` audita cita, claim, pasaje y fuente por referencia.
- Los niveles de visibilidad quedan definidos y no permiten acceso libre en `INTERNAL_AUDIT`.
- Todo acceso elevado tiene campos minimos de registro.

## Relacion con contratos

- Implementa visibilidad para `trace-object.schema.json`, `model-call.schema.json`, `tool-call.schema.json`, `citation-audit.schema.json`, `answer-version.schema.json` y `cost-report.schema.json`.
- Complementa `privacy-security-policy.md` futura de Subfase 0.10.
- Complementa `answer-versioning-policy.md` y `citation-policy.md`.

## Momento de revision

Revisar al cerrar Subfase 0.7, al definir permisos de Subfase 0.10, al crear vistas de soporte, y ante cualquier incidente de privacidad, soporte o auditoria.
