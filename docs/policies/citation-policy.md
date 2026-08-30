# Citation Policy

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Subfase 0.4 - Contratos juridicos de respuesta, evidencia, citas y claims; Subfase 0.7 - Trazabilidad, auditoria y versionado

## Proposito

Definir reglas obligatorias para que las citas de JusNova sean evidencia real y no bibliografia decorativa.

## Alcance

Aplica a `EvidencePack`, `EvidenceSource`, `EvidencePassage`, `Claim`, `Citation`, `AnswerContract`, `CitationAudit` y `TraceObject`.

## Definiciones

- `Claim`: afirmacion verificable generada por JusNova.
- `Citation`: enlace auditado entre un claim, un pasaje y una fuente.
- `Passage`: fragmento localizable extraido de una fuente.
- `Source`: fuente externa o documento del usuario.
- `sources_used`: lista final de fuentes efectivamente citadas.
- `CitationAudit`: resultado estructurado de auditoria de citas, claims, pasajes y fuentes.
- `TraceObject`: traza terminal que une respuesta, claims publicados, fuentes usadas y auditoria.

## Reglas obligatorias

1. Una cita inline debe existir como `Citation`.
2. Toda `Citation.passage_ref` debe existir en `EvidencePack.passages`.
3. Toda `Citation.source_ref` debe existir en `EvidencePack.sources`.
4. `Citation.passage_ref` debe apuntar a un `EvidencePassage.source_ref` igual a `Citation.source_ref`.
5. El identificador antes de `:P#` en `EvidencePassage.passage_ref` debe ser exactamente igual a `EvidencePassage.source_ref`.
6. Todo `Claim.citations[]` debe apuntar a una `Citation.citation_ref` existente.
7. Toda `Citation` incluida en `AnswerContract.citations` debe aparecer en al menos un `Claim.citations[]`.
8. Toda `Citation` incluida en `AnswerContract.citations` debe tener al menos un `supports_claim_ids[]`; `status` expresa el resultado de auditoria, no permite una cita contractual huerfana.
9. Todo `supports_claim_ids[]` debe apuntar a un `Claim.claim_id` existente.
10. Todo claim critico debe tener al menos una cita `valid`. El predicado critico es `criticality = high` o `claim_type` en `plazo`, `requisito`, `competencia`, `causal`, `procedimiento`, `norma`, `jurisprudencia`, `vigencia`.
11. Una fuente final en `sources_used` debe aparecer en al menos una `Citation.source_ref` que soporte un claim existente.
12. Las fuentes externas se citan como `[F#:P#]`.
13. Los documentos del usuario se citan como `[D#:P#]`.
14. Una fuente `TIER2_CONFIABLE` o `TIER3_SECUNDARIO` usada en respuesta visible debe generar advertencia.
15. Una fuente `TIER3_SECUNDARIO` no puede ser unico soporte de un claim normativo critico.
16. No se permite listar fuentes no citadas como bibliografia.
17. Todo `TraceObject` con `citation_audit.overall_status = passed` debe tener cada `Claim.citations[]` representado en `CitationAudit.results[]` con el mismo `claim_id` y `citation_ref`.
18. Todo `TraceObject.sources_used[]` debe aparecer en al menos un `CitationAudit.results[].source_ref` con `status = valid` y asociado a un claim existente en `TraceObject.claims[]`.
19. Un `TraceObject` no puede declarar auditoria `passed` si una cita publicada, claim publicado o fuente usada no fue cubierta por `CitationAudit.results[]`.
20. Toda afirmacion juridica critica visible en `AnswerContract.sections.*.content` debe corresponder por equivalencia semantica a un `Claim` del mismo `AnswerContract`; omitirla de `claims[]` es fallo bloqueante aunque el JSON Schema local valide.
21. Todo `Claim.text` publicado debe representar la misma proposicion juridica que el texto visible del que fue extraido; reutilizar un `claim_id` para otra proposicion es fallo bloqueante.
22. `ClaimCompletenessValidator` se ejecuta antes de aceptar `CitationAudit.overall_status=passed`. Una afirmacion critica visible sin `Claim` produce `blocking_failures[].failure_code=critical_assertion_unmapped`, con `citation_ref=null` y `claim_id=null`.
23. La relacion `Claim <-> Citation` es `n:m` dentro de una misma `AnswerVersion`: `Claim.citations[]` y `Citation.supports_claim_ids[]` son conjuntos sin duplicados y deben ser consistentes en ambas direcciones.
24. Dentro de cada `AnswerContract`, `claims[].claim_id`, `citations[].citation_ref` y `sources_used[]` deben ser unicos; una identidad repetida no crea evidencia ni soporte adicional.
25. Cada `CitationAudit.results[]` debe resolver una unica `Citation` de la misma `AnswerVersion`: `passage_ref` y `source_ref` coinciden exactamente, `claim_id` pertenece a `Citation.supports_claim_ids[]` y la relacion inversa existe en `Claim.citations[]`. El par `(claim_id, citation_ref)` es unico; no se puede reutilizar para otra ruta de evidencia.
26. Todo `Claim` incluido en `AnswerContract.claims[]` es un claim publicado y debe tener `verification_status=passed`. Claims `failed`, `needs_review` o `blocked` pueden conservarse en traza/oracle para explicar retencion o bloqueo, pero no se serializan dentro del contrato final visible.
27. Todo claim con `verification_status=passed` debe usar `support_level=direct|inferential`. `weak` representa soporte insuficiente para aprobar/publicar y debe permanecer `needs_review`, `failed` o `blocked` fuera de `AnswerContract`.

## Reglas deterministicas

Estas reglas deben implementarse en validadores, tests o auditoria de citas; no deben depender solo del prompt del modelo.

1. Construir un mapa `passage_ref -> passage`.
2. Construir un mapa `source_ref -> source`.
3. Construir un mapa `citation_ref -> citation`.
4. Rechazar cita cuyo `passage_ref` no exista.
5. Rechazar cita cuyo `source_ref` no exista.
6. Rechazar cita cuyo source no coincida con el source del pasaje.
7. Rechazar pasaje cuyo identificador de fuente en `passage_ref` no coincida exactamente con `source_ref`.
8. Rechazar `Citation` no referenciada por ningun claim.
9. Rechazar cualquier `Citation` contractual con `supports_claim_ids = []`, cualquiera sea su `status`.
10. Rechazar `supports_claim_ids[]` que no apunte a un claim existente.
11. Rechazar claim critico sin cita valida.
12. Rechazar `sources_used` que contenga source no citado por una cita que soporte claim existente.
13. Rechazar fuente TIER3 como unico soporte normativo critico.
14. Para `TraceObject`, construir el conjunto de pares publicados `(claim_id, citation_ref)` desde `claims[].citations[]`.
15. Para `TraceObject`, construir el conjunto de pares auditados `(claim_id, citation_ref)` desde `citation_audit.results[]`.
16. Rechazar `citation_audit.overall_status = passed` si algun par publicado no aparece como par auditado con `status = valid`.
17. Rechazar `citation_audit.overall_status = passed` si algun `sources_used[]` no aparece como `source_ref` en un resultado auditado valido.
18. Ejecutar `ClaimCompletenessValidator` sobre todas las secciones visibles y construir cobertura bidireccional `assertion critica visible <-> Claim`.
19. Rechazar cualquier afirmacion critica visible sin `Claim` semanticamente equivalente y registrar `critical_assertion_unmapped`.
20. Rechazar cualquier `Claim` cuyo texto no sea semanticamente equivalente a la afirmacion visible que representa o cuyo `claim_type` juridico no use `criticality=high`.
21. Rechazar referencias duplicadas en `Claim.citations[]` o `Citation.supports_claim_ids[]`.
22. Rechazar cualquier par claim/cita que no pertenezca a la misma `AnswerVersion` o que no aparezca en ambos sentidos de la relacion.
23. Rechazar `claim_id` duplicados en `claims[]`, `citation_ref` duplicados en `citations[]` o valores duplicados en `sources_used[]` dentro de un mismo `AnswerContract`. Varias citas distintas pueden compartir `source_ref` cuando apuntan a pasajes validos de la misma fuente.
24. Rechazar cualquier resultado de auditoria cuyo `(claim_id, citation_ref)` este duplicado, no resuelva una `Citation` de la misma version o difiera de ella en `passage_ref`, `source_ref` o pertenencia al claim.
25. Rechazar todo `AnswerContract` que incluya un claim con `verification_status` distinto de `passed`; la abstencion parcial se representa reteniendo ese claim fuera del contrato visible, no publicandolo con estado fallido.
26. Rechazar todo claim `verification_status=passed` cuyo `support_level` sea `weak|unsupported`.

## Reglas asistidas por IA

1. El modelo puede proponer claims y mapear pasajes candidatos.
2. El modelo no decide por si solo que una cita es valida.
3. Si el modelo no puede justificar soporte, el claim debe marcarse `unsupported`, `needs_review` o `blocked`.
4. El modelo puede proponer la extraccion de assertions, pero la completitud se valida contra el oracle aprobado o revision humana registrada; la propia lista `claims[]` del modelo no es oracle suficiente.

## Comportamiento ante incumplimiento

- Cita rota: bloquear respuesta final o reparar sin agregar claims nuevos.
- Cita valida sin claim soportado: bloquear respuesta final por cita decorativa.
- Claim critico sin cita valida: bloquear o abstener parcialmente.
- Afirmacion juridica critica visible sin `Claim` equivalente: bloquear con `critical_assertion_unmapped` o reparar antes de publicar.
- Fuente final no citada: eliminar de `sources_used` o bloquear hasta reparar.
- Fuente `TIER3_SECUNDARIO` como unico soporte de claim normativo o de vigencia critico: no publicar conclusion critica. Si existe presupuesto o credito, ofrecer Modo Investigacion antes de cualquier respuesta sustantiva; si no existe o no se activa, bloquear o emitir abstencion parcial o total.

La advertencia por fuente secundaria nunca es salida suficiente cuando la fuente `TIER3_SECUNDARIO` es el unico soporte de un claim normativo o de vigencia critico.

## Ejemplos permitidos

Los bloques de esta seccion son proyecciones relacionales para mostrar enlaces, no payloads runtime completos ni ejemplos schema-valid por separado. Los fixtures ejecutables deben completar todos los campos requeridos y validar cada objeto contra `claim.schema.json`, `citation.schema.json`, `passage.schema.json` y `source.schema.json`.

Cadena relacional valida:

```json
{
  "claim": {
    "claim_id": "cl_001",
    "citations": ["C1"]
  },
  "citation": {
    "citation_ref": "C1",
    "passage_ref": "F1:P1",
    "source_ref": "F1",
    "status": "valid",
    "supports_claim_ids": ["cl_001"]
  },
  "passage": {
    "passage_ref": "F1:P1",
    "source_ref": "F1"
  },
  "source": {
    "source_ref": "F1",
    "tier": "TIER1_CANONICO"
  }
}
```

## Ejemplos prohibidos

La siguiente proyeccion muestra el fallo de policy; tambien omite campos contractuales no relevantes para el ejemplo y no debe copiarse como fixture runtime.

```json
{
  "claim": {
    "claim_id": "cl_002",
    "criticality": "high",
    "citations": []
  },
  "sources_used": ["F9"]
}
```

## Criterios de aceptacion

- El equipo puede explicar `claim -> citation -> passage -> source`.
- Las proyecciones de 0.4 muestran la cadena relacional completa; los fixtures ejecutables completan y validan cada contrato runtime.
- Las validaciones negativas cubren cita a pasaje inexistente, cita valida sin claim soportado, claim critico sin cita, fuente final no citada y TIER3 como unico soporte normativo critico.
- Las validaciones de 0.7 cubren `TraceObject.claims[].citations[]` y `TraceObject.sources_used[]` contra `CitationAudit.results[]`.
- Las validaciones de completitud cubren afirmacion critica visible omitida de `claims[]`, divergencia entre `Claim.text` y texto visible, y tipo juridico degradado a `criticality!=high`.

## Relacion con contratos

- `claim.schema.json` define la forma del claim.
- `citation.schema.json` define la forma de la cita.
- `passage.schema.json` define la unidad citable.
- `source.schema.json` define la fuente.
- `answer-contract.schema.json` declara que `sources_used` deriva de citas reales.
- `citation-audit.schema.json` registra resultados auditados por cita, claim, pasaje y fuente.
- `trace-object.schema.json` une claims publicados, fuentes usadas y resultado de auditoria.

## Momento de revision

Revisar al cerrar Subfase 0.7, al implementar Citation Auditor en Fase 2, y ante cualquier cita inexistente aprobada.
