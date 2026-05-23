# No RAG Launch Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.6 - Politicas de fuentes, vigencia, conflicto e incertidumbre

## Proposito

Definir como JusNova sale al mercado sin depender de un corpus juridico boliviano propio, evitando vender cache o snapshots como cobertura exhaustiva.

## Alcance

Aplica a lenguaje de producto, arquitectura de retrieval, Evidence Cache, snapshots, documentos del usuario y respuestas juridicas del lanzamiento inicial.

## Definiciones

- `Corpus juridico propio`: ingestion sistematica, versionada y evaluada de fuentes juridicas bolivianas.
- `Evidence Cache`: registro de evidencia consultada y snapshots usados; no es corpus exhaustivo.
- `Busqueda viva`: recuperacion, fetch, extraccion, snapshot, ranking y evaluacion en tiempo de consulta.

## Reglas obligatorias

1. JusNova no depende de un corpus juridico boliviano propio para lanzamiento.
2. La base de fundamentacion inicial es busqueda juridica viva, snapshots, Evidence Cache y documentos del usuario.
3. Evidence Cache no se vende ni describe como corpus exhaustivo.
4. Cache nunca confirma vigencia por si solo.
5. Documento de usuario no se presenta como fuente de derecho vigente.
6. Fuente secundaria no reemplaza fuente oficial sin advertencia.

## Reglas deterministicas

1. Toda respuesta desde cache debe mostrar fecha de recuperacion o ultima verificacion.
2. Toda respuesta desde cache con fuente normativa sin revalidacion debe declarar vigencia no confirmada.
3. El sistema debe buscar evidencia viva antes de responder consultas normativas, jurisprudenciales, procedimentales o de vigencia, salvo abstencion por falta de datos o presupuesto.
4. Si solo existe cache y la pregunta exige actualidad, aplicar `validity-policy.md` y `uncertainty-policy.md`.
5. Si no hay evidencia suficiente, aplicar `abstention-policy.md`.

## Reglas asistidas por IA

1. El modelo puede explicar que JusNova trabaja con evidencia recuperada.
2. El modelo no puede afirmar que JusNova tiene cobertura exhaustiva si no existe corpus evaluado.
3. El modelo no puede convertir cache en fuente de verdad actual.

## Comportamiento ante incumplimiento

- Mensaje que presenta cache como corpus completo: bloquear copy o respuesta.
- Claim de vigencia basado solo en cache: bloquear o advertir.
- Respuesta juridica sin busqueda ni evidencia para intent critico: abstencion o Modo Investigacion.

## Ejemplos permitidos

- "La respuesta se basa en fuentes recuperadas y citadas en esta consulta."
- "Esta fuente fue consultada en fecha X; la vigencia actual no fue confirmada."
- "No cuento con evidencia suficiente para sostener una conclusion categorica."

## Ejemplos prohibidos

- "JusNova tiene todo el derecho boliviano indexado."
- "La cache confirma que esta vigente."
- "Nuestro corpus completo garantiza cobertura total."

## Criterios de aceptacion

- ADR-004 queda cerrado en cuanto a politica de lanzamiento sin corpus propio.
- Evidence Cache queda definido como apoyo trazable, no corpus exhaustivo.
- Cache no confirma vigencia.
- La politica conecta live search, source policy, validity policy y abstention policy.

## Relacion con contratos

- Depende de `legal-search-query.schema.json`, `legal-search-result.schema.json`, `retrieval-run.schema.json`, `evidence-pack.schema.json` y `source.schema.json`.
- Complementa ADR-004, `legal-search-policy.md`, `source-policy.md`, `validity-policy.md` y `abstention-policy.md`.

## Momento de revision

Revisar al implementar Evidence Cache en Fase 8, al preparar copy comercial de lanzamiento, y cuando metricas de demanda justifiquen evaluar un corpus juridico propio post-mercado.
