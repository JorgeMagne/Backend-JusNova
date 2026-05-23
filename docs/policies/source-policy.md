# Source Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.6 - Politicas de fuentes, vigencia, conflicto e incertidumbre

## Proposito

Definir cuando una fuente puede usarse, citarse, advertirse o descartarse dentro de JusNova.

## Alcance

Aplica a fuentes externas, fuentes oficiales, fuentes secundarias, fuentes cacheadas, snapshots y documentos aportados por usuarios dentro de `EvidencePack`, `LegalSearchResult` y respuestas finales.

## Definiciones

- `Fuente usada`: fuente incluida en un Evidence Pack o respuesta final.
- `Fuente citable`: fuente que tiene al menos un pasaje extraido y localizable.
- `Fuente oficial canonica`: publicacion primaria o tribunal competente.
- `Documento de usuario`: documento privado aportado por usuario; sostiene hechos o texto documental, no derecho vigente.
- `Snapshot`: captura o registro reproducible de la fuente consultada.

## Reglas obligatorias

1. Fuente oficial canonica tiene prioridad sobre fuente secundaria.
2. Fuente secundaria no puede sostener por si sola una afirmacion critica de vigencia.
3. Fuente institucional oficial puede sostener contenido si se identifica entidad emisora y fecha.
4. Documento del usuario sostiene hechos o texto documental, no derecho vigente.
5. Blog o articulo sostiene contexto doctrinal, no fundamento principal.
6. Una fuente sin pasaje extraido no puede ser citada.
7. Una fuente sin snapshot debe explicar por que no pudo snapshotearse mediante `snapshot_unavailable_reason`.
8. Toda fuente usada debe tener `retrieved_at`.
9. Toda fuente usada debe tener `tier`.
10. Toda fuente usada debe tener `source_type`.

## Reglas deterministicas

1. El orden de autoridad por defecto es `TIER1_CANONICO`, `TIER1_OFICIAL`, `TIER1_STRUCTURED`, `TIER2_CONFIABLE`, `TIER3_SECUNDARIO`, `USER_DOCUMENT`.
2. `source_type = documento_usuario` exige `tier = USER_DOCUMENT` y `validity_status = NO_APLICA`.
3. `tier = USER_DOCUMENT` exige `source_type = documento_usuario` y `validity_status = NO_APLICA`.
4. `snapshot_id` y `snapshot_unavailable_reason` no pueden coexistir como estados positivos.
5. Si `snapshot_id = null`, `snapshot_unavailable_reason` debe ser una razon cerrada.
6. `user_private_document_not_snapshotted` solo aplica a `USER_DOCUMENT`.
7. `TIER3_SECUNDARIO` no puede ser soporte unico de `claim_type = norma` o `claim_type = vigencia`.
8. `TIER2_CONFIABLE` usado por caida de fuente oficial exige warning visible.
9. Una fuente listada en `sources_used` debe tener una cita real asociada.
10. Discovery web no es evidencia hasta fetch, extraccion, snapshot o razon documentada, normalizacion y passage.

## Reglas asistidas por IA

1. El modelo puede sugerir clasificacion de tier, pero un clasificador deterministico o regla de registry debe confirmarla antes de usarla.
2. El modelo puede resumir debilidad de fuente, pero no puede eliminar warnings obligatorios.
3. El modelo puede proponer fuentes alternativas cuando una fuente oficial cae, pero el fallback debe quedar etiquetado por tier.

## Comportamiento ante incumplimiento

- Fuente sin `retrieved_at`, `tier` o `source_type`: bloquear uso en respuesta final.
- Fuente sin pasaje extraido: permitir discovery o contexto interno, pero impedir cita.
- Fuente secundaria como unico soporte critico: abstencion o Modo Investigacion.
- Documento de usuario tratado como norma vigente: bloquear respuesta.
- Snapshot ausente sin razon: bloquear inclusion en Evidence Pack final.

## Ejemplos permitidos

- "Fuente oficial canonica consultada en fecha X, vigencia no confirmada."
- "Se usa TIER2 por caida de fuente oficial, con advertencia visible."
- "Documento del usuario citado para el texto contractual, no para derecho vigente."

## Ejemplos prohibidos

- "El documento subido demuestra que la ley esta vigente."
- "Blog juridico como unico fundamento de vigencia."
- "Fuente incluida al final sin cita inline."
- "Fuente sin pasaje extraido usada como cita."

## Criterios de aceptacion

- El equipo sabe cuando usar fuente oficial, secundaria y documento de usuario.
- La politica bloquea fuente sin pasaje como cita.
- La politica exige snapshot o razon cerrada de imposibilidad.
- La politica impide tratar `USER_DOCUMENT` como derecho vigente.
- La politica alinea `source.schema.json`, `legal-search-result.schema.json`, `citation-policy.md` y `abstention-policy.md`.

## Relacion con contratos

- Depende de `source.schema.json`, `passage.schema.json`, `citation.schema.json`, `evidence-pack.schema.json`, `legal-search-result.schema.json` y `answer-contract.schema.json`.
- Usa `source-tiers.yaml` como taxonomia canonica.
- Complementa `citation-policy.md`, `legal-search-policy.md`, `validity-policy.md`, `conflict-policy.md` y `uncertainty-policy.md`.

## Momento de revision

Revisar al implementar Source Registry en Fase 4, al integrar adaptadores oficiales en Fase 5, y cuando `source_tier_correctness` baje de 0.90 en evaluaciones.
