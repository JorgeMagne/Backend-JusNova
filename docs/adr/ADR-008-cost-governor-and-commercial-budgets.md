# ADR-008 - Cost Governor And Commercial Budgets

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Costos, planes y profundidad de consulta

## Contexto

La busqueda viva, IA, fetches, OCR, snapshots y evaluacion consumen recursos variables. El plan de entrada no puede ser inferior a 400 Bs/mes.

## Problema

Sin Cost Governor, el sistema puede exceder margen, latencia o proveedores. Reducir citas o advertencias para ahorrar costo dañaria veracidad y confianza.

## Restricciones

- Plan base minimo: 400 Bs/mes.
- Profundidad se controla por budgets y creditos.
- Veracidad, citas y advertencias no se degradan por costo.
- Usage Ledger debe registrar consumo.

## Opciones consideradas

1. Sin limites por consulta.
2. Limitar tokens solamente.
3. Cost Governor por complejidad, tools, fetches, OCR, tiempo, output y creditos.

## Decision

JusNova usara Cost Governor con budgets por `SIMPLE`, `MEDIO`, `COMPLEJO` e `INVESTIGACION`, mas creditos de investigacion y Usage Ledger. El plan base sera 400 Bs/mes.

## Justificacion

Controlar profundidad permite alinear costo y experiencia sin permitir respuestas engañosas. El usuario debe ver limitaciones cuando el presupuesto no alcance.

## Presupuestos conceptuales

- SIMPLE: pocas fuentes, sin OCR inline salvo evidencia ya procesada.
- MEDIO: mas fetches y hasta pocas paginas OCR.
- COMPLEJO: mas rondas, fuentes y OCR; consume credito.
- INVESTIGACION: mayor profundidad, async permitido y mayor costo.

## Dependencias posteriores

- Subfase 0.8 debe crear `budgets.yaml`, `cost-budget.schema.json`, `usage-event.schema.json`, commercial plans y cost-governor policy.
- Fase 1 debe implementar CostGovernor stub y UsageLedger basico.

## No afirma todavia

- No afirma margenes finales.
- No afirma cuotas definitivas hasta medir costos reales.
- No afirma que budget agotado permita ocultar baja calidad de evidencia.

## Riesgos

- Cuotas iniciales pueden no reflejar costo real.
- OCR puede ser mas caro de lo esperado.
- Usuarios pueden no entender creditos.

## Mitigaciones

- Registrar usage events.
- Medir costo por evidence pack.
- Mensajes claros ante limite.
- Ajustar limites comerciales con datos.

## Criterios de aceptacion

- Budgets quedan en YAML en Subfase 0.8.
- Plan de 400 Bs queda reflejado.
- No hay silencio ante limitacion.
- No se reduce citacion para ahorrar costo.

## Momento de revision

Revisar al cerrar Subfase 0.8, al completar Fase 1, y mensualmente durante beta con metricas de costo por consulta, OCR pages y discovery/fetch usage.

## Consecuencias

El sistema debe registrar costos y decidir continuar, abstenerse, pedir datos o proponer investigacion sin sacrificar soporte juridico.

