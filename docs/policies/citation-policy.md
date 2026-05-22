# Citation Policy

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Subfase 0.4 - Contratos juridicos de respuesta, evidencia, citas y claims

## Proposito

Definir reglas obligatorias para que las citas de JusNova sean evidencia real y no bibliografia decorativa.

## Alcance

Aplica a `EvidencePack`, `EvidenceSource`, `EvidencePassage`, `Claim`, `Citation` y `AnswerContract`.

## Definiciones

- `Claim`: afirmacion verificable generada por JusNova.
- `Citation`: enlace auditado entre un claim, un pasaje y una fuente.
- `Passage`: fragmento localizable extraido de una fuente.
- `Source`: fuente externa o documento del usuario.
- `sources_used`: lista final de fuentes efectivamente citadas.

## Reglas obligatorias

1. Una cita inline debe existir como `Citation`.
2. Toda `Citation.passage_ref` debe existir en `EvidencePack.passages`.
3. Toda `Citation.source_ref` debe existir en `EvidencePack.sources`.
4. `Citation.passage_ref` debe apuntar a un `EvidencePassage.source_ref` igual a `Citation.source_ref`.
5. Todo `Claim.citations[]` debe apuntar a una `Citation.citation_ref` existente.
6. Todo claim critico con `criticality = high` y `claim_type` juridico debe tener al menos una cita `valid`.
7. Una fuente final en `sources_used` debe aparecer en al menos una `Citation.source_ref`.
8. Las fuentes externas se citan como `[F#:P#]`.
9. Los documentos del usuario se citan como `[D#:P#]`.
10. Una fuente `TIER2_CONFIABLE` o `TIER3_SECUNDARIO` usada en respuesta visible debe generar advertencia.
11. Una fuente `TIER3_SECUNDARIO` no puede ser unico soporte de un claim normativo critico.
12. No se permite listar fuentes no citadas como bibliografia.

## Reglas deterministicas

Estas reglas deben implementarse en validadores, tests o auditoria de citas; no deben depender solo del prompt del modelo.

1. Construir un mapa `passage_ref -> passage`.
2. Construir un mapa `source_ref -> source`.
3. Construir un mapa `citation_ref -> citation`.
4. Rechazar cita cuyo `passage_ref` no exista.
5. Rechazar cita cuyo `source_ref` no exista.
6. Rechazar cita cuyo source no coincida con el source del pasaje.
7. Rechazar claim critico sin cita valida.
8. Rechazar `sources_used` que contenga source no citado.
9. Rechazar fuente TIER3 como unico soporte normativo critico.

## Reglas asistidas por IA

1. El modelo puede proponer claims y mapear pasajes candidatos.
2. El modelo no decide por si solo que una cita es valida.
3. Si el modelo no puede justificar soporte, el claim debe marcarse `unsupported`, `needs_review` o `blocked`.

## Comportamiento ante incumplimiento

- Cita rota: bloquear respuesta final o reparar sin agregar claims nuevos.
- Claim critico sin cita valida: bloquear o abstener parcialmente.
- Fuente final no citada: eliminar de `sources_used` o bloquear hasta reparar.
- Fuente secundaria como unico soporte critico: bloquear, advertir o pedir Modo Investigacion.

## Ejemplos permitidos

Cadena valida:

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
    "status": "valid"
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
- Los ejemplos validos de 0.4 muestran la cadena completa.
- Las validaciones negativas cubren cita a pasaje inexistente, claim critico sin cita, fuente final no citada y TIER3 como unico soporte normativo critico.

## Relacion con contratos

- `claim.schema.json` define la forma del claim.
- `citation.schema.json` define la forma de la cita.
- `passage.schema.json` define la unidad citable.
- `source.schema.json` define la fuente.
- `answer-contract.schema.json` declara que `sources_used` deriva de citas reales.

## Momento de revision

Revisar al cerrar Subfase 0.7, al implementar Citation Auditor en Fase 2, y ante cualquier cita inexistente aprobada.
