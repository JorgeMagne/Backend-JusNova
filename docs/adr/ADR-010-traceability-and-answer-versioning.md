# ADR-010 - Traceability And Answer Versioning

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Trazabilidad, auditoria y versionado

## Contexto

JusNova debe reconstruir cada respuesta: pregunta, contexto, fuentes, busquedas, modelos, tools, claims, citas, costos, latencias y versiones.

## Problema

Sin trazabilidad estructurada, una respuesta juridica no puede auditarse, depurarse, corregirse ni defenderse ante un error.

## Restricciones

- Trazas pueden contener metadata sensible.
- Usuario ve resumen, no necesariamente traza completa.
- Soporte no debe ver documentos completos.
- Respuestas legales no se sobrescriben silenciosamente.

## Opciones consideradas

1. Logs tecnicos generales.
2. Guardar solo respuesta final.
3. TraceObject estructurado y AnswerVersion por cambio juridico relevante.

## Decision

Cada respuesta tendra TraceObject, versionado y auditoria de fuentes, prompts, modelos, tools, costs y claims.

## Justificacion

TraceObject permite reproducibilidad, soporte, evaluacion, seguridad y mejora continua. AnswerVersion preserva historia juridica y evita correcciones invisibles.

## Trazabilidad aprobada

- Model calls con purpose, provider, model, prompt version y hashes.
- Tool calls con purpose, status, errores y costos.
- Retrieval run summaries sanitizados y evidence pack IDs.
- Sources used/rejected.
- Claims.
- Citation audit.
- Cost report.
- Latency report.
- Warnings.

## Dependencias posteriores

- Subfase 0.7 creo `trace-object.schema.json`, `model-call.schema.json`, `tool-call.schema.json`, `citation-audit.schema.json`, `answer-version.schema.json` y `cost-report.schema.json`.
- Subfase 0.7 creo `answer-versioning-policy.md` y `trace-visibility-policy.md`.
- Subfase 0.10 debe alinear privacy/security policy.
- Fase 3 implementara versionado basico de respuestas.

## No afirma todavia

- No afirma que persistencia, endpoints, UI de soporte ni permisos finales ya existan.
- No afirma que prompts completos sean visibles a soporte.
- No afirma que toda traza sea visible al usuario.

## Riesgos

- Trazas pueden filtrar datos sensibles.
- Guardar demasiado aumenta costo.
- Guardar poco impide auditoria.

## Mitigaciones

- Trace visibility levels.
- Hashes y minimizacion.
- Retrieval runs embebidos como resumen sanitizado, sin URLs crudas, resultados discovery completos, mensajes libres de error ni warnings libres.
- Redaccion de logs.
- Ownership/tenant en trazas.
- Politica de retencion.

## Criterios de aceptacion

- Subfase 0.7 aprueba `trace-object.schema.json`.
- Subfase 0.7 aprueba `model-call.schema.json`, `tool-call.schema.json` y `citation-audit.schema.json`.
- Subfase 0.7 aprueba `answer-version.schema.json`.
- Subfase 0.7 aprueba `answer-versioning-policy.md` y `trace-visibility-policy.md`.
- Logs y privacidad quedan alineados a nivel contractual; controles finales quedan para Subfase 0.10.
- Nueva version se crea ante cambio de evidencia, claim, cita o respuesta.

## Momento de revision

Revisar al cerrar Subfase 0.7, al completar Fase 3, y ante cualquier incidente de soporte, privacidad o citation audit failure.

## Consecuencias

Ninguna respuesta juridica critica debe persistirse sin traza estructurada futura.
