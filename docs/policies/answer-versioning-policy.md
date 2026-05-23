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
- `response_outcome`: resultado trazado de la respuesta versionada: `answered`, `partial_abstention`, `total_abstention` o `blocked`.
- `answer_contract_ref`: referencia al contrato de respuesta persistido o renderizado para respuestas sustantivas; debe ser `null` para abstenciones totales o bloqueos sin AnswerContract.
- `AbstentionRender`: metadata versionada del render de abstencion total o bloqueo, definida en `abstention-render.schema.json`.
- `abstention_render_ref`: referencia estable a `AbstentionRender`, sin embebido sensible dentro de `answer-version.schema.json`.
- `render_hash`: hash del cuerpo canonico renderizado de una abstencion total o bloqueo; no es el texto renderizado.
- `rendered_answer_snapshot_id`: referencia opcional a snapshot renderizado futuro.
- `previous_answer_version`: version previa directa de la misma respuesta.

## Reglas deterministicas

1. No se permite sobrescribir una respuesta juridica sin preservar la version anterior.
2. La version inicial usa `answer_version = 1`, `previous_answer_version = null` y `version_reason = initial_answer`.
3. Toda version posterior usa `answer_version >= 2` y `previous_answer_version` entero.
4. En implementacion, `previous_answer_version` debe ser menor que `answer_version`.
5. Toda version requiere `answer_hash`, `trace_id`, `response_outcome`, `created_at` y `created_by_ref`.
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
18. `TraceObject.trace_id`, `TraceObject.answer_id`, `TraceObject.answer_version` y `TraceObject.response_outcome` deben ser iguales a `AnswerVersion.trace_id`, `AnswerVersion.answer_id`, `AnswerVersion.answer_version` y `AnswerVersion.response_outcome`.
19. Si `response_outcome` es `answered` o `partial_abstention`, `AnswerVersion.answer_contract_ref` debe resolver al `AnswerContract` correspondiente; no puede apuntar a otro `answer_id`, otra version o otra traza.
20. Si `response_outcome` es `answered` o `partial_abstention`, `AnswerContract.trace_id`, `AnswerContract.answer_id` y `AnswerContract.answer_version` deben ser iguales a los campos equivalentes de `TraceObject` y `AnswerVersion`.
21. Si `response_outcome` es `answered` o `partial_abstention`, `AnswerContract.evidence_pack_id` debe existir dentro de `TraceObject.evidence_pack_ids[]`.
22. Si `response_outcome` es `answered` o `partial_abstention`, los claims publicados en `AnswerContract.claims[]` deben corresponder por equivalencia canonica completa a los claims publicados en `TraceObject.claims[]`; no basta coincidir en `claim_id`.
23. Si `response_outcome` es `answered` o `partial_abstention`, `AnswerContract.sources_used[]` debe ser el mismo conjunto que `TraceObject.sources_used[]`.
24. Para cada claim publicado, la equivalencia canonica incluye `claim_id`, `text`, `claim_type`, `criticality`, `support_level`, conjunto de `citations[]`, `verification_status` y `requires_human_review`.
25. Si `response_outcome` es `answered` o `partial_abstention`, cada `AnswerContract.citations[]` debe estar cubierta por `TraceObject.citation_audit.results[]` con el mismo `citation_ref`, `passage_ref`, `source_ref`, `status = valid` y claims soportados equivalentes.
26. Para cada cita publicada, `AnswerContract.citations[].status` debe ser `valid` y el conjunto `supports_claim_ids[]` debe coincidir con los `claim_id` auditados en `TraceObject.citation_audit.results[]` para la misma cita/pasaje/fuente.
27. Si `response_outcome` es `total_abstention` o `blocked`, `AnswerVersion.answer_contract_ref` debe ser `null`, `abstention_render_ref` debe existir y no se debe crear un EvidencePack sintetico para satisfacer el versionado.
28. Si `response_outcome` es `total_abstention` o `blocked`, `abstention_render_ref` debe resolver a un `AbstentionRender` con el mismo `trace_id`, `answer_id`, `answer_version` y `response_outcome`.
29. Para abstenciones y bloqueos, `AnswerVersion.answer_hash` debe coincidir con `AbstentionRender.render_hash`; ese hash cubre el cuerpo canonico renderizado almacenado bajo `AbstentionRender.render_storage_ref`.
30. `AbstentionRender` no puede almacenar prompts, salidas completas del modelo, documentos completos, mensajes completos, HTML crudo, OCR completo, URLs crudas ni texto libre del usuario; solo hashes, referencias internas, codigos cerrados y refs de traza.
31. Una respuesta publicada no puede considerarse reconstruible si `AnswerContract` usa evidence pack, claims, citas o fuentes que no aparecen en el `TraceObject` correspondiente, si reutiliza IDs con contenido juridico distinto, o si una abstencion/bloqueo no tiene `AbstentionRender` versionado y consistente.
32. La regla 4 y las reglas 17 a 31 requieren validador custom o constraint de persistencia; no pueden verificarse con cada JSON Schema aislado.

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
- La politica define ruta versionable para `total_abstention` y `blocked` mediante `abstention-render.schema.json`, sin EvidencePack sintetico.
- La politica exige identidad y contenido consistente entre `TraceObject`, `AnswerVersion` y `AnswerContract`.

## Relacion con contratos

- Implementa `answer-version.schema.json`.
- Implementa `abstention-render.schema.json` para renders de abstencion total y bloqueo.
- Depende de `trace-object.schema.json` para reconstruccion.
- Complementa `answer-contract.schema.json`, `citation-audit.schema.json`, `citation-policy.md`, `abstention-policy.md` y `trace-visibility-policy.md`.
- Define reglas de identidad y contenido cruzado que deben validarse al persistir o publicar una respuesta final.

## Momento de revision

Revisar al cerrar Subfase 0.7, al implementar persistencia de respuestas en Fase 3, al agregar exportaciones formales y ante cualquier incidente donde una respuesta haya cambiado sin historial suficiente.
