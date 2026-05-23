# Uncertainty Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.6 - Politicas de fuentes, vigencia, conflicto e incertidumbre

## Proposito

Definir como JusNova clasifica y comunica incertidumbre juridica, documental y operativa.

## Alcance

Aplica a respuestas juridicas, Evidence Packs, claims, vigencia, conflictos, documentos de usuario, busqueda viva y presupuestos agotados.

## Definiciones

- `Incertidumbre`: falta de certeza suficiente para afirmar un punto juridico sin advertencia.
- `Incertidumbre bloqueante`: incertidumbre que impide responder responsablemente sin mas datos o investigacion.
- `Cobertura`: grado en que la evidencia recuperada cubre la pregunta.

## Niveles de incertidumbre

| Nivel | Significado |
|---|---|
| LOW | Evidencia oficial suficiente y sin conflicto detectado. |
| MEDIUM | Evidencia suficiente pero vigencia, cobertura o fuente requiere cautela. |
| HIGH | Evidencia parcial, conflicto, fuente secundaria o falta de datos clave. |
| BLOCKING | No es responsable responder sin mas datos o investigacion. |

## Reglas obligatorias

1. La incertidumbre debe declararse cuando afecta una conclusion juridica critica.
2. `BLOCKING` exige abstencion o solicitud de datos.
3. `HIGH` exige advertencia visible y recomendacion de investigacion o revision humana.
4. `MEDIUM` exige cautela redactada.
5. `LOW` no elimina la obligacion de citar fuentes.

## Reglas deterministicas

1. `EvidenceQuality.overall = NONE` produce `BLOCKING`.
2. `EvidenceQuality.overall = CONFLICTIVE` produce como minimo `HIGH`.
3. `validity_status = VIGENCIA_NO_CONFIRMADA` produce como minimo `MEDIUM` en claims de vigencia.
4. `validity_status = CONFLICTIVA` produce `HIGH` o `BLOCKING` si el usuario exige conclusion categorica.
5. Unica fuente `TIER3_SECUNDARIO` para claim critico produce `BLOCKING`.
6. Falta de dato esencial del caso produce `BLOCKING`.
7. Budget agotado antes de evidencia suficiente produce `HIGH` o `BLOCKING` segun criticalidad.

## Reglas asistidas por IA

1. El modelo puede redactar advertencias de incertidumbre.
2. El modelo puede proponer datos faltantes o Modo Investigacion.
3. El modelo no puede rebajar `BLOCKING` a `HIGH`, `MEDIUM` o `LOW`.
4. El modelo no puede ocultar incertidumbre para sonar mas seguro.

## Comportamiento ante incumplimiento

- Incertidumbre critica no declarada: bloquear respuesta final.
- Incertidumbre bloqueante con conclusion categorica: aplicar abstencion.
- Incertidumbre por budget: informar limite y ofrecer investigacion o reduccion de alcance.

## Ejemplos permitidos

- "La evidencia es suficiente para explicar el punto, pero no para confirmar vigencia actual."
- "La respuesta requiere Modo Investigacion porque las fuentes recuperadas son contradictorias."
- "Falta el numero de resolucion para responder con precision."

## Ejemplos prohibidos

- "Es claro que..." cuando `EvidenceQuality.overall = LOW`.
- "Sin duda esta vigente." con vigencia no confirmada.
- "No hay problema." ante conflicto no resuelto.

## Criterios de aceptacion

- Los cuatro niveles quedan definidos.
- La politica mapea calidad, vigencia, tier, conflicto, datos faltantes y budget a incertidumbre.
- `BLOCKING` queda conectado a `abstention-policy.md`.
- El modelo no puede degradar incertidumbre obligatoria.

## Relacion con contratos

- Depende de `evidence-quality.schema.json`, `claim.schema.json`, `source.schema.json`, `retrieval-run.schema.json` y `answer-contract.schema.json`.
- Complementa `source-policy.md`, `validity-policy.md`, `conflict-policy.md` y `abstention-policy.md`.

## Momento de revision

Revisar al implementar Evidence Quality Evaluator, al crear evals de abstencion, y cuando `abstention_accuracy` baje de 0.80.
