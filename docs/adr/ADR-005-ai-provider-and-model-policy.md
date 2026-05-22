# ADR-005 - AI Provider And Model Policy

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Proveedor IA inicial y encapsulamiento de modelos

## Contexto

JusNova necesita IA para clasificar intent, planificar retrieval, estructurar outputs, extraer hechos, redactar respuestas, reparar citas y verificar claims. Pero la IA no puede ser fuente de verdad juridica.

## Problema

Acoplar el core a un SDK concreto o permitir que tareas criticas dependan solo de generacion incrementa riesgo de proveedor, costo, privacidad y errores juridicos.

## Restricciones

- El modelo no decide vigencia sin evidencia.
- Los prompts deben versionarse.
- Datos sensibles deben minimizarse.
- Los outputs criticos deben ser estructurados y auditables.
- Debe existir ruta futura para cambiar proveedor.

## Opciones consideradas

1. Acoplar directamente el core al SDK del proveedor inicial.
2. Usar varios proveedores desde el primer sprint.
3. Usar OpenAI como proveedor inicial detras de `ModelProvider`.

## Decision

OpenAI sera el proveedor IA principal inicial, encapsulado detras de `ModelProvider`.

## Justificacion

OpenAI cubre razonamiento, structured outputs, tool use, vision selectiva y web discovery controlado. Encapsularlo permite evaluar proveedores futuros sin romper contratos juridicos ni dominio central.

## Tareas que pueden usar IA

- Intent classification.
- Retrieval planning.
- Query expansion asistida.
- Fact extraction.
- Structured answer drafting.
- Claim extraction.
- Citation repair controlado.
- Verification assistida.
- Memory update.

## Tareas que no dependen solo de IA

- Citation auditing.
- Budget enforcement.
- Tenant isolation.
- Validity status final.
- Source tier final.
- File ownership.
- Trace persistence.
- Provider logging.

## Dependencias posteriores

- Subfase 0.10 debe completar provider policy.
- Fase 1 debe crear `ModelProvider` stub y prompt version registry.
- Fases posteriores deben medir modelos con evals.

## No afirma todavia

- No afirma modelo concreto final.
- No afirma costos finales.
- No afirma que documentos completos puedan enviarse al proveedor.
- No afirma que web search del proveedor sea evidencia final.

## Riesgos

- Costo variable.
- Filtracion de datos si no hay minimizacion.
- Cambios de API/modelos.
- Outputs estructurados mal validados.

## Mitigaciones

- Provider interface.
- Prompt versions.
- Data minimization.
- Trace de model calls.
- Evals antes de cambiar modelo.
- Structured validation antes de persistir resultados criticos.

## Criterios de aceptacion

- Existe interfaz `ModelProvider` en Fase 1.
- Ningun modulo core se acopla al SDK concreto.
- Hay politica de datos para proveedores.
- Hay criterios de evaluacion antes de cambiar modelo.

## Momento de revision

Revisar al implementar Fase 1, antes de beta, y cuando costo por respuesta, latencia, calidad de evals o politica contractual del proveedor cambien materialmente.

## Consecuencias

El proveedor IA es reemplazable por diseno. Las reglas juridicas criticas viven en contratos, validadores y auditoria, no solo en prompts.

