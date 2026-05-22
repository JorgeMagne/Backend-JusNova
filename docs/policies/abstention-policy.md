# Abstention Policy

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Subfase 0.4 - Contratos juridicos de respuesta, evidencia, citas y claims

## Proposito

Definir cuando JusNova debe abstenerse, bloquear o responder parcialmente para evitar conclusiones juridicas sin soporte.

## Alcance

Aplica a respuestas juridicas, claims criticos, citas, fuentes finales, vigencia y evidencia insuficiente.

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

## Reglas deterministicas

1. `Claim.criticality = high` y `Claim.citations = []` exige `verification_status = blocked` o abstencion.
2. `Citation.status` distinto de `valid` no cuenta como soporte fuerte.
3. `EvidencePack.quality.overall = NONE` bloquea fundamentos normativos o jurisprudenciales.
4. `Source.validity_status != VIGENCIA_CONFIRMADA` impide escribir "esta vigente" o equivalente categorico.
5. `Source.tier = TIER3_SECUNDARIO` como unico soporte de `claim_type = norma` o `vigencia` bloquea la respuesta critica.

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

## Relacion con contratos

- Depende de `claim.schema.json`, `citation.schema.json`, `source.schema.json`, `passage.schema.json`, `evidence-pack.schema.json` y `answer-contract.schema.json`.
- Complementa `citation-policy.md`.

## Momento de revision

Revisar al cerrar Subfase 0.6, al implementar Claim Verification en Fase 2, y cuando una evaluacion detecte unsupported critical claim aprobado.
