# Memory Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.9 - Conversacion, memoria, documentos y OCR

## Proposito

Definir como JusNova guarda memoria de caso sin convertirla en verdad juridica, evidencia normativa, vigencia confirmada ni sustituto de citas.

## Alcance

Aplica a `CaseMemory`, conversaciones asociadas a casos, hechos afirmados por usuario, hechos soportados por documentos, preguntas abiertas, riesgos, fechas y contradicciones.

No implementa motor de memoria, embeddings, recuperacion contextual ni permisos finales.

## Definiciones

- Hecho afirmado por usuario: dato ingresado por el usuario y no necesariamente probado.
- Hecho soportado por documento: dato vinculado a pasajes `D#:P#` de `DocumentEvidence`.
- Pregunta legal abierta: cuestion que requiere evidencia, busqueda o decision posterior.
- Contradiccion: choque entre hechos registrados o entre hecho de usuario y documento.

## Reglas obligatorias

1. Memoria no es fuente juridica.
2. Memoria no confirma vigencia.
3. Memoria no sostiene claims normativos, de jurisprudencia, competencia, plazo o vigencia sin EvidencePack y citas.
4. Hechos afirmados por usuario deben separarse de hechos soportados por documentos.
5. Hechos soportados por documentos requieren `passage_refs[]` con `D#:P#`.
6. Contradicciones deben preservarse y no resolverse artificialmente.
7. Memoria debe versionarse con `memory_version`.
8. `CaseMemory` solo aplica a jurisdiccion `BO` en Fase 0.

## Reglas deterministicas

1. `CaseMemory.jurisdiction` debe ser `BO`.
2. `facts_asserted_by_user[]` usa `asserted_by_type = user` y `asserted_by_ref` pseudonimo.
3. `facts_supported_by_documents[].passage_refs[]` debe resolver a `DocumentEvidence` del mismo tenant, documento y version.
4. `source_history[]` solo acepta `F#`, `D#`, `F#:P#` o `D#:P#`.
5. `source_history[]` no acepta URLs crudas.
6. `contradictions[].fact_refs[]` debe resolver a hechos existentes en la misma memoria.
7. `parties[].display_ref` debe ser pseudonimo o alias interno, no nombre completo libre.
8. `dates[].source_ref` solo puede existir si `source_type = evidence_ref`.
9. Toda actualizacion de memoria debe incrementar `memory_version`.
10. `fact_id` debe ser unico en la union de `facts_asserted_by_user[]` y `facts_supported_by_documents[]`; un mismo ID no puede representar dos hechos.
11. `question_id`, `risk_flag_id`, `party_id`, `date_id` y `contradiction_id` deben ser unicos dentro de su arreglo.

## Reglas asistidas por IA

1. El modelo puede sugerir facts o contradicciones, pero deben registrarse con estado y procedencia.
2. El modelo puede redactar etiquetas visibles, pero no debe inventar hechos ni ocultar contradicciones.
3. El modelo puede proponer preguntas legales abiertas, pero no puede marcarlas como respondidas sin evidencia.

## Comportamiento ante incumplimiento

- Si un hecho documental no tiene `D#:P#`, queda como hecho afirmado o pregunta abierta, no como hecho soportado.
- Si hay contradiccion no resuelta, la respuesta debe advertirla o abstenerse si afecta un claim critico.
- Si la memoria contiene solo afirmaciones del usuario, la respuesta no debe presentarlas como hechos probados.

## Ejemplos permitidos

- Hecho de usuario con `confidence = 0.6` y estado `active`.
- Hecho documental con `passage_refs = ["D1:P1"]`.
- Contradiccion entre `fact_user_001` y `fact_doc_001`.

## Ejemplos prohibidos

- Guardar una norma vigente dentro de memoria como hecho probado.
- Registrar una fecha procesal con URL cruda.
- Resolver una contradiccion sin fuente.
- Usar `Conversation.title` como evidencia.

## Criterios de aceptacion

- `case-memory.schema.json` diferencia hechos de usuario y hechos documentales.
- `case-memory.schema.json` exige campos estructurales de version, riesgos, contradicciones, partes y fechas.
- La policy declara que memoria no reemplaza evidencia, citas ni vigencia.

## Relacion con contratos

- Aplica a `case-memory.schema.json`.
- Se relaciona con `conversation.schema.json`, `message.schema.json`, `document-evidence.schema.json`, `claim.schema.json` y `evidence-pack.schema.json`.
- Complementa `source-policy.md`, `citation-policy.md`, `validity-policy.md` y `uncertainty-policy.md`.

## Momento de revision

Revisar al implementar memoria persistente, al definir embeddings o recuperacion contextual, al cerrar Subfase 0.10 y cuando se agreguen nuevas jurisdicciones.
