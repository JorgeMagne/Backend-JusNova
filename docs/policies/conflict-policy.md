# Conflict Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.6 - Politicas de fuentes, vigencia, conflicto e incertidumbre

## Proposito

Definir como JusNova detecta, clasifica y comunica conflictos entre fuentes, documentos y evidencia juridica.

## Alcance

Aplica a conflictos normativos, jurisprudenciales, de vigencia, fecha, jurisdiccion, documento del usuario y diferencias entre fuentes oficiales y secundarias.

## Definiciones

- `Conflicto material`: contradiccion que afecta una conclusion juridica, plazo, requisito, vigencia, competencia o estrategia.
- `Conflicto no resoluble automaticamente`: conflicto sin regla objetiva suficiente para elegir una fuente sin revision humana.
- `Fuente prevalente`: fuente con mayor autoridad por tier, fecha, jurisdiccion o competencia institucional.

## Reglas obligatorias

1. Si hay conflicto, mostrar el conflicto.
2. Indicar fuentes involucradas.
3. Priorizar por tier si corresponde.
4. No resolver artificialmente si no hay base.
5. Recomendar verificacion humana si afecta estrategia o plazo.

## Tipos de conflicto

1. Conflicto de texto normativo.
2. Conflicto de vigencia.
3. Conflicto de jurisprudencia.
4. Conflicto de fecha.
5. Conflicto de jurisdiccion.
6. Conflicto entre documento del usuario y fuente publica.
7. Conflicto entre fuentes oficiales y secundarias.

## Reglas deterministicas

1. Fuente oficial canonica prevalece sobre fuente secundaria cuando ambas tratan el mismo punto.
2. Fuente boliviana prevalece sobre fuente extranjera salvo comparacion solicitada.
3. Documento del usuario no prevalece sobre fuente publica para derecho vigente.
4. Conflicto de vigencia usa `validity_status = CONFLICTIVA`.
5. Conflicto material debe elevar `EvidenceQuality.overall = CONFLICTIVE` cuando afecta la respuesta principal.
6. Si el usuario exige conclusion categorica sobre un conflicto no resoluble, aplicar abstencion parcial o total.

## Reglas asistidas por IA

1. El modelo puede agrupar pasajes conflictivos por tema.
2. El modelo puede redactar la explicacion del conflicto.
3. El modelo no puede ocultar una fuente conflictiva usada en el Evidence Pack.
4. El modelo no puede inventar una regla de desempate.

## Comportamiento ante incumplimiento

- Conflicto omitido: bloquear respuesta final o marcar `needs_review`.
- Conflicto no resoluble tratado como resuelto: bloquear claim asociado.
- Conflicto de fuente extranjera mezclado con derecho boliviano: descartar o etiquetar como comparativo.

## Ejemplos permitidos

- "Existe conflicto entre F1 y F2 sobre la vigencia de X; F1 es fuente oficial canonica y F2 es secundaria."
- "No es responsable resolver automaticamente este punto porque afecta plazo procesal."
- "El documento del usuario afirma X, pero la fuente publica recuperada indica Y."

## Ejemplos prohibidos

- "La fuente mas conveniente es la correcta."
- "Ambas fuentes dicen lo mismo." cuando los pasajes difieren.
- "La jurisprudencia dominante establece..." con conflicto no resuelto.

## Criterios de aceptacion

- Los siete tipos de conflicto quedan definidos.
- La politica fija respuesta minima ante conflicto.
- La politica conecta conflicto con `CONFLICTIVA`, `CONFLICTIVE` y abstencion.
- La politica impide resolver artificialmente conflictos criticos.

## Relacion con contratos

- Depende de `source.schema.json`, `passage.schema.json`, `claim.schema.json`, `evidence-quality.schema.json`, `legal-search-result.schema.json` y `answer-contract.schema.json`.
- Complementa `source-policy.md`, `validity-policy.md`, `uncertainty-policy.md` y `abstention-policy.md`.

## Momento de revision

Revisar al implementar Conflict Resolver, al construir evals adversariales, y cuando `conflict_detection_rate` baje de 0.70.
