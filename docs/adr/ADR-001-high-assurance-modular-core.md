# ADR-001 - High-Assurance Modular Core + Distributed Execution Layer

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Arquitectura base de JusNova Backend

## Contexto

JusNova debe producir respuestas juridicas bolivianas auditables, con evidencia, citas, trazabilidad, busqueda viva, documentos, OCR, seguridad y control de costos. Es un sistema con alta cohesion juridica y tareas externas fallables.

## Problema

Separar demasiado pronto en microservicios aumenta latencia, coordinacion, fallos distribuidos y dificultad para reconstruir respuestas. Un monolito plano, en cambio, mezclaria conversacion, evidencia, costos, busqueda, OCR y seguridad sin limites claros.

## Restricciones

- La respuesta juridica debe reconstruirse de punta a punta.
- La busqueda viva y OCR pueden ser lentos o fallar.
- Los documentos del usuario requieren tenancy y privacidad.
- El core no debe depender directamente de proveedores externos.
- Fase 1 debe poder iniciar con una estructura implementable.

## Opciones consideradas

1. Microservices mesh desde el inicio.
2. Monolito plano.
3. Nucleo modular de alta garantia con workers/workflows separados.

## Decision

JusNova usara un nucleo backend modular de alta garantia y una capa distribuida de ejecucion para trabajos pesados, lentos, fallables o reintentables.

## Justificacion

El core mantiene consistencia transaccional, contratos juridicos, trazabilidad, seguridad y costos en un mismo limite logico. Los workers separan cargas externas o intensivas sin partir prematuramente el dominio.

## Limites aprobados

- La vista de arquitectura queda en `docs/architecture/architecture-overview.md`.
- Los limites modulares, workers y criterios de extraccion futura quedan en `docs/architecture/module-boundaries.md`.
- El core incluye conversacion, memoria, orquestacion, busqueda viva, source registry, evidencia, answer contract, citation auditor, claim verification, documentos, costos, usage, seguridad y trazabilidad.
- Los workers incluyen retrieval, adaptadores oficiales, fetch/snapshot, extraction, OCR, indexing y evaluation.

## Dependencias posteriores

- Fase 1 debe crear estructura de paquetes coherente con estos limites.
- Fase 1 debe crear health checks, settings, logging y trazabilidad minima.
- Fases posteriores deben implementar workers segun madurez operacional.

## No afirma todavia

- No afirma que Temporal ya este desplegado.
- No afirma que existan servicios separados.
- No afirma que todos los workers esten implementados en Fase 1.

## Riesgos

- El core puede crecer demasiado si los limites no se respetan.
- Workers prematuros pueden agregar complejidad operativa.
- Falta de boundaries en codigo puede degradar hacia monolito plano.

## Mitigaciones

- Mantener interfaces internas por modulo.
- Crear tests de boundaries cuando exista codigo.
- Extraer servicios solo con metrica operativa y ADR posterior.
- Mantener workers para tareas con latencia, retries o CPU altos.

## Criterios de aceptacion

- Existe diagrama de arquitectura en `architecture-overview.md`.
- Existen limites de modulos en `module-boundaries.md`.
- Existe lista de modulos core.
- Existe lista de workers/workflows.
- Existen criterios de extraccion futura.

## Momento de revision

Revisar al cerrar Fase 1, antes de implementar Fase 4 Live Legal Search Core, y cuando un modulo supere limites de latencia, escala u ownership definidos en `module-boundaries.md`.

## Consecuencias

Fase 1 debe construir un backend modular, no una malla de microservicios ni un monolito sin limites.

