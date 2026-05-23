# ADR-009 - Source Registry And Validity Policy

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Fuentes, tiers, vigencia, conflictos y snapshots

## Contexto

JusNova debe distinguir fuentes oficiales, secundarias, documentos del usuario, snapshots y evidencia cacheada. La vigencia normativa no se presume.

## Problema

Sin Source Registry y politica de vigencia, el sistema puede tratar mirrors, blogs, cache o documentos privados como fuente juridica primaria.

## Restricciones

- Fuente usada requiere tier, source type y retrieved_at.
- Fuente normativa requiere validity status.
- Fuente secundaria requiere advertencia.
- Cache no confirma vigencia.
- Documento del usuario no es derecho vigente.

## Opciones consideradas

1. Lista informal de URLs.
2. Clasificacion libre por modelo.
3. Source Registry con schema, tiers, snapshots, validity y conflict policies.

## Decision

Toda fuente debe pasar por Source Registry, clasificacion de tier, metadata y politica de vigencia.

## Justificacion

El Source Registry permite priorizar autoridad, controlar fuentes debiles, auditar snapshots, detectar conflicto y evitar mezclar documentos privados con derecho aplicable.

## Fuentes iniciales prioritarias

- Gaceta Oficial de Bolivia.
- Tribunal Constitucional Plurinacional.
- Tribunal Supremo de Justicia / Genesis.
- Sistemas oficiales estructurados validados.
- Organo Judicial y entidades oficiales sectoriales.
- Documentos del usuario como evidencia privada, no como norma.

## Dependencias posteriores

- Subfase 0.6 acepta `source-policy.md`, `validity-policy.md`, `conflict-policy.md` y `uncertainty-policy.md`.
- Subfase 0.5 debe definir source routing.
- Subfase 0.9 debe separar document evidence.
- Fase 4 y Fase 5 implementaran registry/adapters iniciales.

## No afirma todavia

- No afirma que source registry schema ya exista.
- No afirma que vigencia pueda confirmarse siempre.
- No afirma que TIER2/TIER3 sostengan claims criticos por si solos.

## Riesgos

- Fuente oficial puede estar caida.
- Fuente secundaria puede parecer mas accesible que fuente canonica.
- Versiones normativas pueden entrar en conflicto.

## Mitigaciones

- Tiers y warnings.
- Snapshots con estado.
- Validity status conservador.
- Conflict reports.
- Abstencion o investigacion ante evidencia insuficiente.

## Criterios de aceptacion

- Subfase posterior crea `source-registry-entry.schema.json`.
- `source-policy.md`, `validity-policy.md`, `conflict-policy.md` y `uncertainty-policy.md` quedan aceptadas como politicas operativas de Subfase 0.6.
- TIER2/TIER3 quedan reguladas.
- Cache no se trata como confirmacion de vigencia.

## Momento de revision

Revisar al cerrar Subfase 0.6, al implementar Fase 5, y cuando `source_tier_correctness`, `validity_awareness` o `conflict_detection_rate` no cumplan metas.

## Consecuencias

Las fuentes son objetos gobernados, no simples URLs.
