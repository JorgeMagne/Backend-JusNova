# ADR-004 - Launch Without Own Legal RAG

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Politica de lanzamiento sin corpus juridico propio

## Contexto

Construir un corpus juridico boliviano propio, completo, versionado y evaluado requiere permisos, ingestion sistematica, reconciliacion normativa y mantenimiento. Eso no debe bloquear la salida inicial si el sistema puede recuperar evidencia viva.

## Problema

Prometer un corpus propio antes de tener cobertura y evaluacion crea riesgo juridico, comercial y tecnico. Tambien puede confundir Evidence Cache con una base de verdad exhaustiva.

## Restricciones

- La fuente de verdad debe ser evidencia recuperada y trazable.
- Evidence Cache solo registra fuentes consultadas.
- La vigencia no se presume por cache.
- Documentos del usuario no son derecho vigente.

## Opciones consideradas

1. Esperar a tener corpus juridico propio completo.
2. Lanzar con corpus parcial presentado como cobertura juridica.
3. Lanzar con Live Legal Search, snapshots, Evidence Cache y Document Evidence Search.

## Decision

JusNova saldra al mercado sin depender de un corpus juridico boliviano propio indexado masivamente.

## Justificacion

La busqueda viva, adaptadores oficiales, snapshots y documentos del usuario permiten construir respuestas auditables sin sobredimensionar ingestion antes de conocer demanda real y restricciones de fuentes.

## Dentro de lanzamiento

- Live Legal Search.
- Official Source Adapters.
- Source Snapshots.
- Evidence Cache con fecha y estado.
- Document Evidence Search.
- OCR para documentos publicos y privados.

## Fuera de lanzamiento

- Corpus juridico boliviano exhaustivo.
- Grafo normativo completo.
- Ingestion masiva programada de todas las fuentes.
- Reconciliacion automatica total de vigencia.

## Dependencias posteriores

- Subfase 0.6 debe crear `no-rag-launch-policy.md` y source/cache warnings.
- Fase 8 implementara Evidence Cache y Source Snapshot Registry.
- Fase post-mercado podra reconsiderar corpus propio con ADR nuevo.

## No afirma todavia

- No afirma que cache confirme vigencia.
- No afirma cobertura exhaustiva.
- No afirma que RAG avanzado quede prohibido para siempre.

## Riesgos

- El usuario puede interpretar cache como base completa.
- Portales externos pueden afectar disponibilidad.
- Consultas complejas pueden requerir Modo Investigacion.

## Mitigaciones

- Etiquetas y warnings obligatorios.
- Fecha de recuperacion y revalidacion.
- Abstencion cuando no hay evidencia suficiente.
- Revisar corpus futuro solo con datos de uso y permisos claros.

## Criterios de aceptacion

- El documento no usa terminologia ambigua de RAG parcial para lanzamiento.
- El equipo entiende que la informacion viene de recuperacion viva y documentos.
- Respuestas desde cache muestran fecha y advertencia cuando corresponda.

## Momento de revision

Revisar despues de tres meses de beta o cuando el 60% de consultas repetidas provengan de las mismas fuentes y exista permiso operativo para ingestion sistematica.

## Consecuencias

El diferencial inicial se construye en busqueda, extraccion, ranking, citacion, trazabilidad y evaluacion, no en prometer corpus propio.

