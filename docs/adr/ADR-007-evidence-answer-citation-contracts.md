# ADR-007 - Evidence, Answer, Citation And Claim Contracts

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Contratos de respuesta juridica

## Contexto

JusNova debe responder con evidencia y trazabilidad. Las fuentes no pueden ser decoracion y los claims criticos no pueden quedar sin soporte.

## Problema

Generar respuestas directamente desde resultados web, memoria o documentos sin contratos permite citas rotas, claims no soportados, vigencia inventada y bibliografia decorativa.

## Restricciones

- Claim critico requiere cita valida o bloqueo.
- Citas externas usan `[F#:P#]`.
- Documentos del usuario usan `[D#:P#]`.
- Fuentes finales se derivan de citas reales.
- Repair no agrega nuevos claims sustantivos.

## Opciones consideradas

1. Respuesta libre del modelo con fuentes al final.
2. Respuesta con URLs sin auditoria de pasajes.
3. Pipeline EvidencePack -> AnswerDraft -> Claims -> CitationAuditor -> FinalAnswer.

## Decision

Toda respuesta juridica critica debe pasar por Evidence, Answer, Citation y Claim contracts.

## Justificacion

El pipeline contractual permite auditar claim -> cita -> pasaje -> fuente -> snapshot y bloquear respuestas sin soporte suficiente.

## Estructura visible aprobada

- Conclusion operativa.
- Hechos considerados.
- Fundamento normativo.
- Jurisprudencia relevante.
- Analisis aplicado.
- Incertidumbres, vigencia y conflictos.
- Recomendacion profesional.
- Fuentes utilizadas.

## Dependencias posteriores

- Subfase 0.4 debe crear JSON Schemas de evidence pack, source, passage, citation, claim y answer contract.
- Subfase 0.6 debe crear citation, abstention, source y validity policies.
- Fase 2 implementara Citation Auditor y bloqueo de claims criticos.

## No afirma todavia

- No afirma que los schemas ya existan.
- No afirma que el auditor ya este implementado.
- No afirma que toda respuesta simple use formato largo.

## Riesgos

- Schemas demasiado laxos pueden permitir fuentes decorativas.
- Auditor demasiado generativo puede ser inconsistente.
- Formato largo puede ser pesado para consultas simples.

## Mitigaciones

- Validaciones deterministicas para existencia de citas.
- Claims criticos con soporte obligatorio.
- Formato reducido permitido solo sin eliminar citas necesarias.
- Evals de citation validity y unsupported critical claims.

## Criterios de aceptacion

- Subfase 0.4 crea schemas JSON requeridos.
- Se puede auditar claim -> cita -> pasaje.
- Se define cuando una respuesta se bloquea.
- Citas decorativas se consideran fallo critico.

## Momento de revision

Revisar al cerrar Subfase 0.4, al completar Fase 2, y si `citation_validity_rate` baja de 1.0 o aparece cualquier unsupported critical claim aprobado.

## Consecuencias

La generacion final queda subordinada a evidencia normalizada y auditoria.

