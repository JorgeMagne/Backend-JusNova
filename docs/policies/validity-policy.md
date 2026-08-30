# Validity Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.6 - Politicas de fuentes, vigencia, conflicto e incertidumbre

## Proposito

Definir cuando JusNova puede afirmar vigencia, cuando debe advertir incertidumbre y cuando debe abstenerse sobre normas o fuentes juridicas.

## Alcance

Aplica a fuentes normativas, jurisprudenciales, institucionales, cacheadas, snapshots y respuestas que incluyan plazos, requisitos, competencia, derogacion, modificacion o aplicabilidad actual.

## Definiciones

- `Vigencia`: estado actual verificable de una norma o disposicion.
- `Evidencia explicita`: pasaje oficial o fuente primaria que sustenta vigencia, modificacion o derogacion.
- `Cache`: evidencia previamente consultada; no confirma vigencia por si sola.
- `Modo Investigacion`: ejecucion de mayor profundidad definida por `no-rag-launch-policy.md` y budgets futuros; no autoriza afirmar vigencia sin evidencia explicita.

## Reglas obligatorias

1. `VIGENCIA_CONFIRMADA` exige evidencia explicita.
2. Por defecto, normativa recuperada se marca `VIGENCIA_NO_CONFIRMADA` si no hay verificacion suficiente.
3. No se puede escribir "vigente" si `validity_status != VIGENCIA_CONFIRMADA`.
4. Si hay senales de modificacion, usar `POSIBLEMENTE_MODIFICADA`.
5. Si hay fuentes contradictorias, usar `CONFLICTIVA`.
6. Si una fuente esta en cache, debe mostrar fecha de ultima verificacion.
7. Si se usa TIER2 por caida de fuente oficial, debe advertirse.

## Reglas deterministicas

1. `source_type = norma` no puede usar `validity_status = NO_APLICA`.
2. `source_type = documento_usuario` debe usar `validity_status = NO_APLICA`.
3. `VIGENCIA_CONFIRMADA` requiere fuente oficial o canonica, pasaje extraido y fecha de recuperacion.
4. `DEROGADA_CONFIRMADA` requiere evidencia explicita de derogacion.
5. `CONFLICTIVA` requiere identificar las fuentes o pasajes en conflicto.
6. Cache o snapshot sin revalidacion produce como maximo `VIGENCIA_NO_CONFIRMADA`.
7. Fuente secundaria no confirma vigencia critica ni derogacion critica sin soporte oficial.
8. Dentro de un `EvidencePack`, toda fuente con `validity_status=VIGENCIA_CONFIRMADA` o `DEROGADA_CONFIRMADA` debe tener al menos un `EvidencePassage` del mismo pack cuyo `source_ref` coincida. La validez aislada de `Source` no sustituye esta comprobacion relacional.

## Reglas asistidas por IA

1. El modelo puede detectar posibles senales de modificacion, pero no puede confirmar vigencia por inferencia.
2. El modelo puede redactar advertencias de vigencia, pero no puede omitirlas.
3. El modelo puede sugerir nuevas busquedas de vigencia cuando la evidencia sea incompleta.

## Comportamiento ante incumplimiento

- Fuente cacheada sin revalidacion: mostrar fecha de consulta y advertencia.
- Fuentes contradictorias: usar `CONFLICTIVA` y aplicar `conflict-policy.md`.
- Usuario pide afirmacion categorica no soportada: aplicar `abstention-policy.md`.

| Caso | Accion obligatoria |
|---|---|
| Usuario pide afirmar vigencia o derogacion categorica y no existe evidencia explicita suficiente | Bloquear la conclusion categorica y emitir abstencion parcial o total. |
| Claim de vigencia o derogacion afecta plazo, estrategia, competencia o requisito critico | Exigir `VIGENCIA_CONFIRMADA` o `DEROGADA_CONFIRMADA`; si no existe, abstencion parcial y recomendacion de verificacion humana o Modo Investigacion. |
| Se explica una fuente normativa sin confirmar vigencia actual | Permitir respuesta solo con advertencia visible y `validity_status = VIGENCIA_NO_CONFIRMADA` o `POSIBLEMENTE_MODIFICADA`. |
| Vigencia depende solo de cache o snapshot no revalidado | Prohibir afirmacion categorica; mostrar fecha de consulta y advertencia de vigencia no confirmada. |
| Fuentes contradictorias sobre vigencia o derogacion | Usar `CONFLICTIVA`, explicar el conflicto y no resolver artificialmente. |

La abstencion es parcial solo cuando existe algun punto verificable con cita valida. Si la pregunta depende por completo de vigencia o derogacion no verificable, la abstencion es total.

## Frases permitidas

- "Con la evidencia recuperada, no pude confirmar vigencia actual."
- "La fuente consultada indica X, pero requiere corroboracion oficial de vigencia."
- "Existe posible conflicto entre fuentes sobre X."
- "Esta conclusion depende de que la norma citada no haya sido modificada posteriormente."

## Frases prohibidas sin soporte

- "Esta vigente." sin `VIGENCIA_CONFIRMADA`.
- "Fue derogada." sin `DEROGADA_CONFIRMADA`.
- "La jurisprudencia dominante establece." sin evidencia suficiente.
- "El plazo actual es." sin fuente vigente confirmada o advertencia.
- "La ley aplicable dice." sin cita valida.

## Criterios de aceptacion

- La politica impide presumir vigencia.
- La politica distingue vigencia confirmada, no confirmada, posible modificacion, derogacion y conflicto.
- Cache y snapshot no se tratan como confirmacion de vigencia.
- La confirmacion o derogacion materializada en un EvidencePack resuelve al menos un pasaje de la misma fuente.
- Las frases permitidas y prohibidas quedan documentadas.

## Relacion con contratos

- Depende de `source.schema.json`, `legal-search-result.schema.json`, `claim.schema.json`, `evidence-quality.schema.json` y `answer-contract.schema.json`.
- Usa `validity-statuses.yaml` como taxonomia canonica.
- Complementa `source-policy.md`, `conflict-policy.md`, `abstention-policy.md` y `no-rag-launch-policy.md`.

## Momento de revision

Revisar al implementar Validity Resolver, al detectar falsa vigencia en evaluaciones, y cuando `validity_awareness` baje de 0.85.
