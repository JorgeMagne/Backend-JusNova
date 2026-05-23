# ADR-003 - JusNova Live Legal Search Engine

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Motor de busqueda juridica viva

## Contexto

JusNova saldra al mercado sin corpus juridico propio masivo. La fundamentacion debe venir de fuentes publicas, documentos del usuario y evidencia recuperada, extraida, clasificada y auditada.

## Problema

Un buscador generico o una respuesta web directa no garantizan jurisdiccion boliviana, autoridad, vigencia, citas reales ni trazabilidad. Un resultado recuperado no es evidencia hasta abrirse, extraerse y normalizarse.

## Restricciones

- Bolivia es jurisdiccion base.
- El modelo no es fuente de verdad.
- Portales publicos pueden ser lentos o inestables.
- Las fuentes secundarias requieren advertencia.
- No debe existir proveedor unico de discovery.

## Opciones consideradas

1. Responder desde memoria del modelo.
2. Usar un buscador web generico como fuente final.
3. Construir Live Legal Search Engine con adaptadores, discovery, fetch, extract, snapshots, ranking y Evidence Packs.

## Decision

El JusNova Live Legal Search Engine sera el corazon de fundamentacion del producto de lanzamiento.

## Justificacion

La busqueda viva permite operar sin corpus propio y, a la vez, sostener respuestas con evidencia localizable. La arquitectura separa discovery, fetch, extraction, source authority, ranking, validity, conflict checks y Evidence Pack.

## Componentes aprobados

- Legal Query Understanding.
- Source Routing.
- Official Source Adapters.
- Search Discovery Providers.
- Fetchers.
- Extractors.
- Source Snapshot Registry.
- Evidence Cache.
- Legal Ranking Engine.
- Evidence Quality Evaluator.
- Evidence Pack Builder.

## Dependencias posteriores

- Subfase 0.5 creo contratos de legal search, provider interfaces, retrieval plan/run y source routing matrix.
- Subfase 0.6 debe completar source, validity, conflict y uncertainty policies.
- Fase 4 implementara core del motor.
- Fases 5 a 8 implementaran adaptadores, resiliencia, vigencia, conflicto y cache.

## No afirma todavia

- No afirma que adaptadores oficiales ya funcionen.
- No afirma que Evidence Cache sea corpus juridico.
- No afirma que discovery web sea fuente final.
- No afirma que un proveedor comercial de discovery sea obligatorio.

## Riesgos

- Los portales oficiales pueden fallar.
- Discovery web puede traer ruido extranjero o secundario.
- Ranking juridico puede requerir calibracion.
- La extraccion de PDFs escaneados puede ser costosa.

## Mitigaciones

- Source Registry y tiers.
- Portal Resilience Layer.
- Snapshots y stale-with-warning.
- Evals de recall/precision.
- OCR progresivo y budget controlado.

## Criterios de aceptacion

- El contrato Live Legal Search queda definido en Subfase 0.5 mediante `legal-search-query.schema.json`, `legal-search-result.schema.json`, `legal-entity.schema.json`, `retrieval-plan.schema.json`, `retrieval-run.schema.json`, `evidence-quality.schema.json`, `provider-interfaces.md`, `legal-search-policy.md` y `source-routing-matrix.md`.
- El motor distingue fuente oficial, secundaria y documento de usuario.
- El motor produce trazas.
- El motor no devuelve texto crudo sin normalizacion.
- No hay dependencia de proveedor unico de discovery.

## Momento de revision

Revisar al cerrar Subfase 0.5, al completar Fase 4, y cuando `official_source_recall@10`, `precision@5` o `cost_per_successful_evidence_pack` no alcancen los umbrales de evaluacion.

## Consecuencias

Toda consulta juridica critica debera pasar por plan de retrieval, Evidence Pack o abstencion estructurada antes de respuesta final.
