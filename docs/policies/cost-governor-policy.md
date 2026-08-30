# Cost Governor Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.8 - Cost Governor, planes y presupuestos

## Proposito

Definir como JusNova controla costo, profundidad y creditos sin degradar veracidad juridica, soporte probatorio, vigencia, citas ni trazabilidad.

## Alcance

Aplica a budgets por complejidad, creditos de investigacion, Usage Ledger, comportamiento ante budget agotado y limites permitidos por plan.

No implementa CostGovernor runtime, billing, Stripe, base de datos, workers ni UI de upgrade.

## Principio rector

El Cost Governor puede limitar profundidad. No puede debilitar la verdad documentada.

## Reglas deterministicas

1. Toda ejecucion usa un `CostBudget` efectivo identificado por `cost_budget_ref` y `cost_budget_version`.
2. `TraceObject` guarda solo escalares cerrados de budget: `cost_budget_ref`, `cost_budget_version`, `plan_code` y `complexity`.
3. `TraceObject` no embebe `CostBudget` completo ni referencia el schema por `$ref`.
4. El `CostBudget` resuelto por `TraceObject.cost_budget_ref` debe existir y coincidir exactamente con `TraceObject.cost_budget_version`, `TraceObject.plan_code` y `TraceObject.complexity`; `cost_budget_ref` y `CostBudget.cost_budget_id` usan forma canonica `cb_<plan>_<complexity>_vNNN`, y sus tokens de plan/complejidad deben coincidir con `plan_code` y `complexity`.
5. Cuando `UsageEvent.cost_budget_ref` no sea `null`, el `CostBudget` resuelto debe existir y coincidir exactamente con `UsageEvent.cost_budget_version`, `UsageEvent.plan_code` y, cuando `event_scope = execution`, `UsageEvent.complexity`.
6. La igualdad de `cost_budget_ref` contra un `CostBudget` real requiere validador custom; JSON Schema solo valida forma local.
7. `CostReport` registra consumo observado; `CostBudget` registra limites aplicados.
8. `UsageEvent` registra consumo conciliable sin `user_id` crudo ni `metadata` libre.
9. `LegalSearchQuery.budget` sigue siendo `search_budget` tecnico de busqueda viva; no es el presupuesto comercial 0.8.
10. Si se agota el budget, el sistema no inventa evidencia.
11. Si falta evidencia por limite de plan o budget, se informa de forma visible.
12. Si se requiere mas profundidad, se ofrece Modo Investigacion cuando corresponda.
13. Si no hay credito suficiente, se ofrece respuesta parcial con advertencia o upgrade; nunca una conclusion critica sostenida por evidencia incompleta.
14. Nunca se elimina citacion para ahorrar tokens.
15. Nunca se oculta baja calidad de evidencia para ahorrar costo.
16. Nunca se usa fuente inferior sin advertencia para ahorrar costo.
17. Nunca se reduce `EvidenceQuality`, `CitationAudit`, `TraceObject`, warnings de vigencia ni trazabilidad por costo.
18. Puede limitar discovery, fetches, OCR, reformulaciones, tool rounds, async y output no esencial.
19. Puede diferir trabajo profundo a modo asincrono solo si el budget y el plan lo permiten.
20. Toda ejecucion publicada con `complexity = COMPLEJO` debe tener un `UsageEvent` `research_credit_used` con `quantity = 1`.
21. Toda ejecucion publicada con `complexity = INVESTIGACION` debe tener un `UsageEvent` `research_credit_used` con `quantity = 2`.
22. El evento de credito de una respuesta publicada debe compartir `trace_id` y `answer_id` con esa ejecucion; no basta una referencia conversacional o un identificador reservado sin artefacto final.
23. El evento de credito debe apuntar al mismo `cost_budget_ref`, `cost_budget_version`, `plan_code` y `complexity` que la ejecucion publicada.
24. La cantidad del evento `research_credit_used` debe coincidir con `CostBudget.research_credit_cost` del budget resuelto.
25. En 0.8 no existen exenciones de creditos.
26. Cualquier exencion futura requiere enum cerrado y nueva policy o ADR.
27. Si `event_scope = execution`, `UsageEvent` debe registrar `complexity`, `cost_budget_ref` y `cost_budget_version`.
28. Si `event_scope = organization_period`, `UsageEvent` no puede aparentar pertenecer a una consulta: `complexity`, `cost_budget_ref`, `cost_budget_version` y refs de ejecucion deben estar ausentes o ser `null`.
29. Si `event_scope = execution`, `UsageEvent` debe tener al menos una referencia cerrada de ejecucion: `conversation_id`, `trace_id`, `answer_id`, `retrieval_run_id`, `cost_report_id`, `model_call_id` o `tool_call_id`.
30. Los eventos `model_input_tokens` y `model_output_tokens` deben registrar `model_call_id`.
31. `standard_query` y `complex_query` representan eventos por ejecucion publicada; `quantity` debe ser `1`.
32. `storage_mb_day` solo se registra como `organization_period`.
33. `document_processed` puede registrarse como `execution` o `organization_period`.
34. `research_credit_used` no se agrega en v0; la cantidad debe ser exactamente el costo de credito de la complejidad.
35. Si una afirmacion critica queda sin evidencia suficiente por budget agotado, la salida debe ser abstencion parcial, abstencion total, bloqueo o Modo Investigacion.
36. En v0, `billing_period=YYYY-MM` se deriva de `UsageEvent.created_at` en UTC y representa el intervalo semiabierto entre el primer dia de ese mes a `00:00:00Z` y el primer dia del mes siguiente a `00:00:00Z`; eventos fuera de ese intervalo no pueden contabilizarse en el periodo.
37. Toda ejecucion publicada registra exactamente un evento de consulta con `quantity = 1`: `standard_query` para `SIMPLE|MEDIO` y `complex_query` para `COMPLEJO|INVESTIGACION`. Ambos eventos deben incluir el `trace_id` y el `answer_id` materializados.
38. Para una misma ejecucion publicada, la decision efectiva de budget, `TraceObject` y todos los `UsageEvent` de scope `execution` deben coincidir exactamente en `organization_id`, `plan_code`, `complexity`, `cost_budget_ref` y `cost_budget_version`; ninguna validacion local sustituye esta igualdad cross-artifact.
39. La publicacion terminal es atomica: mensaje de salida, `Answer`, `AnswerVersion`, `AbstentionRender` o `AnswerContract`, `TraceObject`, evento de consulta, debito de credito cuando corresponda y cierre exitoso del run se confirman juntos o se revierten juntos.
40. En `COMPLEJO|INVESTIGACION`, la transaccion terminal vuelve a bloquear y comprobar el saldo efectivo antes del debito. Si otra ejecucion consumio el saldo desde la decision de admision, se revierte todo artefacto final y la ejecucion falla cerrada con `research_credit_required`.
41. Los eventos de consulta, credito y tokens son idempotentes por su ancla de ejecucion o llamada; un retry no puede duplicar consumo ni producir simultaneamente `standard_query` y `complex_query` para la misma traza.
42. Ningun evento terminal publico se emite antes de confirmar la transaccion terminal; un rollback nunca deja `answer_final` o `answer_blocked` observable.
43. El endpoint que inicia una ejecucion facturable exige idempotencia tenant/actor-scoped: una misma key y fingerprint resuelven el mismo mensaje/run, y una key reutilizada por ese actor con otro payload falla con `conflict`; la key raw no se persiste ni aparece en logs o trazas.

## Reglas asistidas por IA

1. El modelo puede explicar al usuario que la respuesta esta limitada por profundidad o creditos.
2. El modelo puede proponer Modo Investigacion cuando la evidencia recuperada no alcanza para una conclusion critica.
3. El modelo no puede decidir ocultar una cita, warning, conflicto o incertidumbre para ahorrar tokens.
4. El modelo no puede presentar una respuesta como completa si el Cost Governor detuvo recuperacion critica.

## Criterios de aceptacion

- `docs/schemas/budgets.yaml` define `SIMPLE`, `MEDIO`, `COMPLEJO` e `INVESTIGACION`.
- `docs/contracts/cost-budget.schema.json` valida valores exactos por complejidad.
- `docs/contracts/usage-event.schema.json` valida scope, unidades, creditos, periodo, plan y actor pseudonimizado.
- `docs/contracts/trace-object.schema.json` exige budget ref, budget version, plan code y complexity.
- El validador custom 0.8 rechaza divergencias entre `TraceObject`, `UsageEvent` y el `CostBudget` resuelto por `cost_budget_ref`.
- El validador custom 0.8 rechaza ejecuciones publicadas sin exactamente un evento de consulta del tipo correspondiente a su complejidad, conciliado por `trace_id` y `answer_id`.
- El validador custom 0.8 rechaza ejecuciones publicadas `COMPLEJO` o `INVESTIGACION` sin evento `research_credit_used` conciliado por los mismos `trace_id` y `answer_id`.
- Tests transaccionales demuestran que respuesta/traza, usage, debito y cierre del run se confirman juntos, que dos ejecuciones concurrentes no consumen el mismo ultimo credito y que retries no duplican consumo.
- Tests de frontera demuestran que retries HTTP secuenciales y concurrentes con la misma `Idempotency-Key` no crean otro mensaje, run, usage o debito, y que un payload distinto con la misma key falla cerrado.
- `docs/policies/trace-visibility-policy.md` define visibilidad de plan y budget por nivel.
- Ningun contrato 0.8 introduce contadores ligados a un proveedor especifico.
- Ninguna policy 0.8 permite degradar evidencia, cita, vigencia o trazabilidad por costo.

## Relacion con contratos

- `cost-budget.schema.json`: limites aplicados por complejidad.
- `usage-event.schema.json`: ledger de consumo, creditos, unidades y scope.
- `trace-object.schema.json`: relacion auditable entre respuesta, complejidad, plan y budget efectivo.
- `cost-report.schema.json`: consumo observado que debe reconciliar con llamadas y runs de la traza.
- `legal-search-query.schema.json`: conserva `search_budget` tecnico interno, separado del presupuesto comercial.

## Momento de revision

Revisar al cerrar Subfase 0.8, al implementar CostGovernor en Fase 1, al medir costos reales en beta, al cambiar precios o cuotas, y ante cualquier incidente donde costo haya limitado evidencia critica.
