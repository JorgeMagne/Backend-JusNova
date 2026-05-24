# Trace Visibility Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.7 - Trazabilidad, auditoria y versionado
**Enmiendas:** Subfase 0.8 - Cost Governor, planes y presupuestos; Subfase 0.9 - Conversacion, memoria, documentos y OCR; Subfase 0.10 - Security, Privacy and Provider Boundaries

## Proposito

Definir que partes de una traza puede ver cada actor y como se protegen datos sensibles dentro de `TraceObject`, model calls, tool calls, citation audits y reportes de costo.

## Alcance

Aplica a visualizacion de trazas, soporte, auditoria interna, reconstruccion de respuestas, revision de fallos de citacion y diagnostico tecnico.

No define retencion final, permisos productivos completos ni proceso de incidente con acceso a material crudo; esos controles se cierran en Subfase 0.10.

## Niveles de visibilidad

| Nivel | Visible para | Incluye | Excluye |
|---|---|---|---|
| `USER_SUMMARY` | Usuario final | Fuentes usadas, advertencias, fecha de consulta, nivel de evidencia, resultado de vigencia expresado para usuario | Prompts, salidas completas de modelo, documentos completos, mensajes completos, IDs internos de mensajes, hashes internos innecesarios, plan interno, budget ref, budget version |
| `SUPPORT_VIEW` | Soporte autorizado | Errores, latencia, proveedores, consumo tecnico estimado, estado de auditoria, IDs de trazabilidad, IDs de conversacion/mensaje, plan neutral, complejidad y budget ref | Prompts, documentos completos, mensajes completos, OCR completo, salida completa del modelo, precio mensual |
| `INTERNAL_AUDIT` | Equipo tecnico autorizado | Hashes, prompt versions, model calls, tool calls, provider audit refs, raw access refs, fuentes rechazadas, citation audit, costos observados, refs completas de mensajes, relacion completa con Usage Ledger, `cost_budget_ref`, `cost_budget_version`, `plan_code`, `complexity` | Acceso libre a material crudo; por defecto se usan hashes y referencias |
| `RAW_INCIDENT_ACCESS` | Seguridad autorizada bajo evento raw | Acceso puntual a recurso crudo aprobado y auditado por `RawAccessEvent` | Soporte normal, `SUPPORT_VIEW`, browsing libre de documentos o mensajes |

## Reglas deterministicas

1. Toda respuesta juridica final requiere `TraceObject`.
2. `TraceObject.actor_ref` no debe ser PII directa; debe ser hash, pseudonimo o identificador tecnico controlado.
3. `TraceObject.actor_type` debe distinguir `user`, `system`, `support` o `service`.
4. `USER_SUMMARY` nunca muestra prompt completo, salida completa del modelo, documento completo ni mensaje completo.
5. `SUPPORT_VIEW` nunca muestra prompt completo, documento completo, OCR completo ni salida completa del modelo.
6. `INTERNAL_AUDIT` no significa acceso libre: usa hashes y referencias por defecto.
7. Material crudo solo puede consultarse mediante `RawAccessEvent` definido en Subfase 0.10.
8. Todo acceso raw o elevado registra `raw_access_event_id`, `organization_id`, `resource_type`, `resource_ref`, `classification`, `actor_ref`, `actor_type`, `access_role`, `reason`, `access_case_ref`, `approved_by_ref`, `accessed_at` y `visibility_level`.
9. `sources_rejected` y `retrieval_runs[].sources_rejected` deben guardar razon cerrada, `warning_codes` cerrados y hashes, no contenido crudo.
10. `TraceObject.retrieval_runs[]` debe ser resumen sanitizado; no puede embeber `RetrievalRun` operativo completo, `LegalSearchResult.url`, `sources_opened[]` como URL cruda, `sources_rejected[].url`, mensajes libres de error ni warnings libres.
11. `latency_ms`, `cost`, `token_usage` y `cost_units` deben ser objetos cerrados sin metadata libre.
12. Model calls y tool calls registran `input_hash`; `output_hash` solo es obligatorio en `success`.
13. Todo `ModelCall` registra `token_usage`; si una llamada falla antes de producir salida, `output_tokens` puede ser `0`, pero el contador debe existir para reconciliar costos.
14. Estados fallidos registran `error_code` y no guardan salida completa.
15. `citation_audit` registra evaluacion estructurada de cita y claim; no guarda pasajes crudos fuera de las referencias aprobadas.
16. Si una traza se marca `blocked` o `total_abstention`, debe existir `abstention_reason`; `warnings[]` puede complementar la explicacion visible, pero no sustituye la razon reconstruible.
17. Si una traza se marca `answered` o `partial_abstention`, su `citation_audit.overall_status` debe ser `passed`.
18. Si una traza se marca `partial_abstention`, debe existir `abstention_reason` o una advertencia visible sobre la parte no respondida.
19. Si `abstention_reason = policy_blocked`, `citation_audit.blocking_failures[]` debe contener `failure_code = policy_blocked`; no se debe representar ese bloqueo como `unsupported_claim` u otra falla de cita.
20. Si una traza se marca `answered`, todos sus claims deben tener `verification_status = passed`.
21. Si una traza se marca `partial_abstention`, debe existir al menos un claim con `verification_status = passed`.
22. Si una traza se marca `answered` o `partial_abstention`, debe existir al menos un `ModelCall` con `purpose = answer_generation` y `status = success`.
23. Si `citation_audit.overall_status = passed`, cada `claims[].citations[]` publicado debe aparecer en `citation_audit.results[]` con el mismo `claim_id`, `citation_ref` y `status = valid`.
24. Si `citation_audit.overall_status = passed`, cada `sources_used[]` debe aparecer como `source_ref` en al menos un resultado auditado valido.
25. Un `TraceObject` no puede presentarse en `USER_SUMMARY`, `SUPPORT_VIEW` ni `INTERNAL_AUDIT` como auditoria aprobada si tiene claims, citas o fuentes publicadas fuera de `CitationAudit.results[]`.
26. Si una traza se marca `answered`, no debe existir `abstention_reason`.
27. Si una traza se marca `total_abstention`, no debe contener claims publicados ni fuentes usadas.
28. Si una traza se marca `blocked`, puede conservar claims intentados, pero ninguno puede tener `verification_status = passed`.
29. Ningun nivel de visibilidad puede convertir una fuente decorativa en fuente usada.
30. Cada `ModelCall.token_usage.total_tokens` debe ser igual a `input_tokens + output_tokens`.
31. `TraceObject.cost.model_input_tokens` debe ser igual a la suma de `model_calls[].token_usage.input_tokens`.
32. `TraceObject.cost.model_output_tokens` debe ser igual a la suma de `model_calls[].token_usage.output_tokens`.
33. `TraceObject.cost.tool_calls` debe ser igual a la cantidad de elementos en `tool_calls[]`; si en 0.8 se requiere distinguir llamadas facturables, se agregara un campo separado.
34. `CostReport.estimated_total_cost` debe coincidir con la suma de `provider_estimated_costs[].estimated_cost`, `tool_calls[].cost_units.estimated_cost` y `retrieval_runs[].estimated_cost` bajo la politica de redondeo definida por implementacion.
35. Cada `ModelCall`, `ToolCall` y `RetrievalRun` debe cumplir `completed_at >= started_at`.
36. `latency_ms.total` representa duracion wall-clock de la traza y no puede ser menor que ningun componente: `model_total`, `tool_total`, `retrieval_total`, `citation_audit`, `persistence` o `queue`.
37. Un componente de latencia positivo con `latency_ms.total = 0` es inconsistente y debe rechazarse.
38. `latency_ms.model_total` no puede ser menor que la mayor duracion derivada de `model_calls[]`.
39. `latency_ms.tool_total` no puede ser menor que la mayor duracion derivada de `tool_calls[]`.
40. `latency_ms.retrieval_total` no puede ser menor que la mayor duracion derivada de `retrieval_runs[]`.
41. `latency_ms.total` no puede ser menor que la mayor duracion derivada de cualquier `ModelCall`, `ToolCall` o `RetrievalRun`.
42. Dentro de un `TraceObject`, `model_calls[].model_call_id`, `tool_calls[].tool_call_id`, `retrieval_runs[].retrieval_run_id` y `claims[].claim_id` deben ser unicos.
43. Dentro de `citation_audit.results[]`, la clave `(claim_id, citation_ref, passage_ref, source_ref)` debe ser unica.
44. Si `CostReport.currency = NONE`, entonces `provider_estimated_costs[]`, `tool_calls[].cost_units.estimated_cost`, `retrieval_runs[].estimated_cost` y `estimated_total_cost` deben ser `0`.
45. Todo campo `input_hash`, `output_hash`, `answer_hash`, `render_hash` o `url_hash` debe usar formato `sha256:` seguido de 64 caracteres hexadecimales; ningun hash puede contener texto crudo, URL cruda, prompt, documento, mensaje o salida completa.
46. Todo `TraceObject` debe registrar `cost_budget_ref`, `cost_budget_version`, `plan_code` y `complexity` como escalares cerrados.
47. `TraceObject` no puede embeber `CostBudget` completo ni usar `$ref` a `cost-budget.schema.json`.
48. `USER_SUMMARY` no muestra `plan_code`, `cost_budget_ref`, `cost_budget_version` ni limites de budget.
49. `SUPPORT_VIEW` puede mostrar `plan_code` neutral, `complexity` y `cost_budget_ref`, pero no precio mensual ni condiciones comerciales completas.
50. `INTERNAL_AUDIT` puede ver `cost_budget_ref`, `cost_budget_version`, `plan_code`, `complexity` y relacion con `UsageEvent`.
51. La relacion entre `TraceObject` y Usage Ledger debe poder conciliar ejecucion, creditos y budget efectivo sin guardar PII directa.
52. `TraceObject.input_message_ids[]` y `TraceObject.output_message_id` guardan solo referencias a `Message`, nunca contenido.
53. Cada `input_message_ids[]` debe resolver a un `Message` existente con el mismo `conversation_id` y `organization_id`.
54. `output_message_id` debe resolver a un `Message` con `role = assistant`, `message_kind = assistant_final`, mismo `conversation_id`, mismo `organization_id`, mismo `trace_id`, mismo `answer_id` y mismo `answer_version_ref`.
55. `output_message_id` no puede aparecer dentro de `input_message_ids[]`.
56. `USER_SUMMARY` no muestra IDs internos de mensajes; `SUPPORT_VIEW` puede mostrar IDs de conversacion/mensaje; `INTERNAL_AUDIT` puede ver refs completas.
57. `ModelCall` y `ToolCall` deben declarar `external_provider_call` y `provider_call_audit_id`; las llamadas externas no pueden quedar sin auditoria de provider.
58. `ProviderCallAudit` puede verse en `INTERNAL_AUDIT`; `SUPPORT_VIEW` solo puede ver diagnostico redacted, sin payload crudo ni clases sensibles no necesarias.
59. `TraceObject.prompt_injection_risks[]` agrega riesgos detectados en retrieval y evidencia usados por la respuesta; el resumen de usuario solo muestra advertencias comprensibles, no detalles explotables.
60. `RawAccessEvent.visibility_level` no admite `SUPPORT_VIEW`; soporte normal no es acceso raw.
61. `support_operator` queda fuera de `RawAccessEvent`; un actor de soporte solo puede aparecer como `incident_responder` o `security_auditor` bajo `RAW_INCIDENT_ACCESS`.
62. Las reglas 30 a 61 requieren validador custom o tests de contrato; no deben inferirse del prompt ni de la UI de soporte.

## Reglas asistidas por IA

1. El modelo puede resumir una traza para `USER_SUMMARY` usando solo campos permitidos.
2. El modelo puede proponer diagnostico para `SUPPORT_VIEW`, pero no puede solicitar material crudo sin motivo de incidente.
3. El modelo puede ayudar a agrupar warnings, pero no puede ocultar fallos bloqueantes.
4. El modelo no puede ampliar su propio nivel de visibilidad.

## Redaccion y hashing

| Campo o material | Regla |
|---|---|
| Prompt completo | No se guarda en contratos 0.7; usar `prompt_version` e `input_hash`. |
| Salida completa del modelo | No se guarda en contratos 0.7; usar `output_hash` cuando exista. |
| Documento completo | No se guarda en trazas; usar referencias de documento, pagina, pasaje o hash. |
| Mensaje completo del usuario | No se guarda en trazas; usar `input_message_ids`, `output_message_id`, hash o referencia conversacional. |
| URL sensible | Guardar hash cuando se registre fuente abierta o rechazada dentro de `TraceObject`; URLs operativas crudas quedan fuera de la traza 0.7. |
| Error tecnico | Guardar `error_code` y `safe_message_code`; el texto humano se genera desde catalogo cerrado en la vista. |
| Costos y latencias | Guardar valores numericos observados. En Subfase 0.8, `TraceObject` puede guardar plan neutral, complejidad y budget ref; no guarda precio mensual ni `CostBudget` embebido. |

## Registro de acceso elevado

Todo acceso a `SUPPORT_VIEW` o `INTERNAL_AUDIT` debe registrar evento de vista interna redacted. Todo acceso a material crudo debe registrar `RawAccessEvent`:

```txt
raw_access_event_id
organization_id
resource_type
resource_ref
classification
actor_ref
actor_type
access_role
reason
access_case_ref
approved_by_ref
accessed_at
visibility_level
```

El motivo debe ser concreto y cerrado. `support_operator` no puede abrir `RawAccessEvent`; soporte general opera solo con `SUPPORT_VIEW` redacted.

## Criterios de aceptacion

- `trace-object.schema.json` no permite objetos libres para fuentes rechazadas ni latencias.
- `trace-object.schema.json` no embebe retrieval runs operativos completos con URLs crudas; usa resumen sanitizado con hashes.
- `model-call.schema.json` y `tool-call.schema.json` no almacenan material crudo.
- `citation-audit.schema.json` audita cita, claim, pasaje y fuente por referencia.
- `citation_audit.overall_status = passed` cubre todos los claims citados y todas las fuentes usadas de la traza.
- `cost-report.schema.json` debe cuadrar con `model_calls[]`, `tool_calls[]`, `retrieval_runs[]` y estimaciones de proveedor de la misma traza.
- `model-call.schema.json` debe cuadrar `token_usage.total_tokens` con sus tokens de entrada y salida.
- `latency_ms` y timestamps de llamadas o retrieval runs no pueden expresar duraciones negativas o totales imposibles.
- Los IDs internos de `TraceObject` y las claves de `citation_audit.results[]` no pueden duplicarse.
- Los niveles de visibilidad quedan definidos y no permiten acceso libre en `INTERNAL_AUDIT`.
- Todo acceso raw o elevado tiene contrato `RawAccessEvent` y no se confunde con `SUPPORT_VIEW`.
- Riesgos de prompt injection se conservan como referencias estructuradas, no como texto malicioso reutilizable.

## Relacion con contratos

- Implementa visibilidad para `trace-object.schema.json`, `model-call.schema.json`, `tool-call.schema.json`, `citation-audit.schema.json`, `answer-version.schema.json`, `cost-report.schema.json`, `cost-budget.schema.json` y `usage-event.schema.json`.
- Complementa `privacy-security-policy.md`, `provider-policy.md` y `prompt-injection-policy.md` de Subfase 0.10.
- Complementa `answer-versioning-policy.md` y `citation-policy.md`.

## Momento de revision

Revisar al cerrar Subfase 0.7, ante enmiendas de budget/plan como Subfase 0.8, ante enmiendas conversacionales como Subfase 0.9, al definir permisos de Subfase 0.10, al crear vistas de soporte, y ante cualquier incidente de privacidad, soporte o auditoria.
