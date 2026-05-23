# Answer Versioning Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.7 - Trazabilidad, auditoria y versionado

## Proposito

Definir cuando una respuesta juridica cambia de version y prohibir sobrescrituras silenciosas de contenido legal, evidencia, claims o citas.

## Alcance

Aplica a respuestas juridicas finales, respuestas parcialmente abstentas, reparaciones de cita, actualizaciones de evidencia, revalidaciones de vigencia, correcciones por feedback, exportaciones formales y cambios de claims.

No implementa persistencia, UI de historial, permisos finales ni retencion completa; esos puntos se cierran en fases posteriores y en Subfase 0.10.

## Definiciones

- `AnswerVersion`: version estable e inmutable de una respuesta legal.
- `answer_hash`: hash del contenido versionado o representacion canonica aprobada.
- `answer_contract_ref`: referencia al contrato de respuesta persistido o renderizado, sin embebido sensible dentro de `answer-version.schema.json`.
- `rendered_answer_snapshot_id`: referencia opcional a snapshot renderizado futuro.
- `previous_answer_version`: version previa directa de la misma respuesta.

## Reglas deterministicas

1. No se permite sobrescribir una respuesta juridica sin preservar la version anterior.
2. La version inicial usa `answer_version = 1`, `previous_answer_version = null` y `version_reason = initial_answer`.
3. Toda version posterior usa `answer_version >= 2` y `previous_answer_version` entero.
4. En implementacion, `previous_answer_version` debe ser menor que `answer_version`.
5. Toda version requiere `answer_hash`, `trace_id`, `answer_contract_ref`, `created_at` y `created_by_ref`.
6. `answer-version.schema.json` no almacena respuesta completa, prompt, documento completo, mensaje completo ni salida completa del modelo.
7. Si se repara una cita, se crea nueva version con `version_reason = citation_repaired`.
8. Si se regenera respuesta, se crea nueva version con `version_reason = answer_regenerated`.
9. Si se actualiza evidencia, se crea nueva version con `version_reason = evidence_updated`.
10. Si se revalida vigencia y cambia el estado o la advertencia juridica, se crea nueva version con `version_reason = validity_revalidated`.
11. Si se corrige por feedback, se crea nueva version con `version_reason = feedback_correction`.
12. Si se exporta a formato formal con transformacion juridicamente relevante, se crea nueva version con `version_reason = formal_export`.
13. Si se agrega o remueve una fuente, se crea nueva version con `version_reason = source_added` o `source_removed`.
14. Si se corrige un claim, se crea nueva version con `version_reason = claim_corrected`.
15. Una version no puede apuntar a un `TraceObject` inexistente.
16. Una version no puede declararse final si su `TraceObject.citation_audit.overall_status` contradice el resultado publicado.
17. `TraceObject.answer_version_ref` debe ser igual a `AnswerVersion.answer_version_id`.
18. `TraceObject.trace_id`, `TraceObject.answer_id` y `TraceObject.answer_version` deben ser iguales a `AnswerVersion.trace_id`, `AnswerVersion.answer_id` y `AnswerVersion.answer_version`.
19. `AnswerContract.trace_id`, `AnswerContract.answer_id` y `AnswerContract.answer_version` deben ser iguales a los campos equivalentes de `TraceObject` y `AnswerVersion`.
20. `AnswerVersion.answer_contract_ref` debe resolver al `AnswerContract` correspondiente; no puede apuntar a otro `answer_id`, otra version o otra traza.
21. `AnswerContract.evidence_pack_id` debe existir dentro de `TraceObject.evidence_pack_ids[]`.
22. Los claims publicados en `AnswerContract.claims[]` deben corresponder exactamente por `claim_id` a los claims publicados en `TraceObject.claims[]`.
23. `AnswerContract.sources_used[]` debe ser el mismo conjunto que `TraceObject.sources_used[]`.
24. Cada `AnswerContract.citations[]` debe estar cubierta por `TraceObject.citation_audit.results[]` con el mismo `citation_ref`, `passage_ref`, `source_ref` y un `claim_id` soportado.
25. Una respuesta publicada no puede considerarse reconstruible si `AnswerContract` usa evidence pack, claims, citas o fuentes que no aparecen en el `TraceObject` correspondiente.
26. La regla 4 y las reglas 17 a 25 requieren validador custom o constraint de persistencia; no pueden verificarse con cada JSON Schema aislado.

## Reglas asistidas por IA

1. El modelo puede sugerir si un cambio requiere nueva version, pero la decision final debe aplicarse por reglas deterministicas.
2. El modelo no puede decidir sobrescribir una version existente.
3. El modelo no puede eliminar historial de versiones.
4. El modelo puede resumir diferencias entre versiones usando hashes, referencias y metadata aprobada.

## Criterios de aceptacion

- Toda respuesta futura tiene una version inicial inmutable.
- Toda reparacion juridicamente relevante genera nueva version.
- `answer-version.schema.json` rechaza una version 1 con `previous_answer_version` no nulo.
- `answer-version.schema.json` rechaza una version 2 o superior sin `previous_answer_version`.
- La politica declara y ejemplifica que `previous_answer_version < answer_version` es regla obligatoria de implementacion futura.
- La politica impide guardar respuesta completa sensible dentro del contrato de version.
- La politica exige identidad y contenido consistente entre `TraceObject`, `AnswerVersion` y `AnswerContract`.

## Relacion con contratos

- Implementa `answer-version.schema.json`.
- Depende de `trace-object.schema.json` para reconstruccion.
- Complementa `answer-contract.schema.json`, `citation-audit.schema.json`, `citation-policy.md`, `abstention-policy.md` y `trace-visibility-policy.md`.
- Define reglas de identidad y contenido cruzado que deben validarse al persistir o publicar una respuesta final.

## Momento de revision

Revisar al cerrar Subfase 0.7, al implementar persistencia de respuestas en Fase 3, al agregar exportaciones formales y ante cualquier incidente donde una respuesta haya cambiado sin historial suficiente.
