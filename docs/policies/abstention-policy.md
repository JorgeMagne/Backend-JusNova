# Abstention Policy

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Subfase 0.4 - Contratos juridicos de respuesta, evidencia, citas y claims; Subfase 0.6 - Politicas de fuentes, vigencia, conflicto e incertidumbre

## Proposito

Definir cuando JusNova debe abstenerse, bloquear o responder parcialmente para evitar conclusiones juridicas sin soporte.

## Alcance

Aplica a respuestas juridicas, claims criticos, citas, fuentes finales, vigencia y evidencia insuficiente.

## Historial de decisiones

| Subfase | Decision |
|---|---|
| 0.4 | Reglas base de abstencion por claim critico sin cita, cita rota, fuente final no citada y evidencia `NONE`. |
| 0.6 | Extension por fuente, vigencia, conflicto, incertidumbre y lanzamiento sin corpus juridico propio. |

## Definiciones

- `Abstencion total`: no se entrega conclusion sustantiva.
- `Abstencion parcial`: se responde solo lo verificable y se declara lo no verificado.
- `Bloqueo`: el sistema impide publicar respuesta final hasta reparar o reducir alcance.
- `Claim critico`: afirmacion sobre plazo, requisito, competencia, causal, procedimiento, norma, jurisprudencia o vigencia.

## Reglas obligatorias

1. Si no hay evidencia, no hay fundamento juridico.
2. Si un claim critico no tiene cita valida, se bloquea o se abstiene.
3. Si una cita apunta a pasaje inexistente, se bloquea.
4. Si una fuente final no fue citada, se elimina o se bloquea.
5. Si una fuente `TIER3_SECUNDARIO` es el unico soporte de un claim normativo critico, se bloquea o se abstiene.
6. Si se requiere vigencia y no hay evidencia confiable, no se afirma vigencia.
7. Si `EvidencePack.quality.overall = NONE`, no se entrega conclusion sustantiva.
8. Si hay conflicto irresoluble en un punto critico, se explica el conflicto y no se fuerza una conclusion categorica.
9. Si falta un dato esencial del caso, se pide el dato o se responde solo lo verificable.
10. Documento del usuario no se trata como derecho vigente.
11. Si `uncertainty-policy.md` clasifica el caso como `BLOCKING`, se bloquea, se abstiene o se solicita informacion adicional.
12. Si una fuente usada no tiene snapshot ni `snapshot_unavailable_reason`, se bloquea su inclusion en la respuesta final.
13. Si la vigencia depende solo de cache o snapshot no revalidado, no se entrega conclusion categorica de vigencia.

## Reglas deterministicas

1. `Claim.criticality = high` y `Claim.citations = []` exige `verification_status = blocked` o abstencion.
2. `Citation.status` distinto de `valid` no cuenta como soporte fuerte.
3. `EvidencePack.quality.overall = NONE` bloquea fundamentos normativos o jurisprudenciales.
4. `Source.validity_status != VIGENCIA_CONFIRMADA` impide escribir "esta vigente" o equivalente categorico.
5. `Source.tier = TIER3_SECUNDARIO` como unico soporte de `claim_type = norma` o `vigencia` bloquea la respuesta critica.
6. `EvidenceQuality.overall = CONFLICTIVE` bloquea una conclusion categorica si el conflicto afecta el punto preguntado.
7. `Uncertainty = BLOCKING` exige `verification_status = blocked` para claims criticos afectados.

## Reglas asistidas por IA

1. El modelo puede proponer una abstencion redactada.
2. El modelo puede sugerir informacion faltante.
3. El modelo no puede degradar una abstencion obligatoria a conclusion categorica.

## Comportamiento ante incumplimiento

Formato minimo de abstencion parcial:

```md
No puedo sostener una conclusion categorica con la evidencia disponible.

Lo que si pude verificar:
- ... [F1:P1]

Lo que no pude verificar:
- ...

Para responder con mayor precision necesito:
- ...
```

## Ejemplos permitidos

- "Con la evidencia recuperada, no pude confirmar vigencia actual."
- "La fuente consultada indica X, pero requiere corroboracion oficial de vigencia."
- "Existe conflicto entre fuentes sobre X; no es responsable resolverlo automaticamente."

## Ejemplos prohibidos

- "Esta vigente." sin `VIGENCIA_CONFIRMADA`.
- "La jurisprudencia dominante establece..." con una sola resolucion.
- "La ley aplicable dice..." sin cita valida.
- Listar una fuente al final que no aparece en ninguna cita.

## Criterios de aceptacion

- Queda definido cuando bloquear, abstenerse o responder parcialmente.
- Claim critico sin cita valida no puede llegar a respuesta final.
- TIER3 como unico soporte normativo critico queda prohibido.
- La policy cubre vigencia no confirmada y conflictos irresolubles.
- La policy cubre incertidumbre `BLOCKING` y cache sin revalidacion.

## Relacion con contratos

- Depende de `claim.schema.json`, `citation.schema.json`, `source.schema.json`, `passage.schema.json`, `evidence-pack.schema.json` y `answer-contract.schema.json`.
- Complementa `citation-policy.md`, `source-policy.md`, `validity-policy.md`, `conflict-policy.md`, `uncertainty-policy.md` y `no-rag-launch-policy.md`.

## Momento de revision

Revisar al cerrar Subfase 0.6, al implementar Claim Verification en Fase 2, y cuando una evaluacion detecte unsupported critical claim aprobado.
