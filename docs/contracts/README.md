# Contract Governance

**Estado documental:** Draft
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Gobierno de contratos internos y JSON Schemas

## Proposito

Los contratos definen las estructuras que Fase 1 implementara en codigo. Un modulo critico no debe implementarse sin contrato aprobado.

## Reglas

- Todo contrato debe tener responsable.
- Todo contrato debe tener estado documental.
- Todo JSON Schema debe usar draft `2020-12`.
- Todo contrato critico debe incluir ejemplos validos e invalidos antes de cerrar Fase 0.
- Todo contrato que referencie otro schema debe usar rutas relativas estables.
- Los nombres de enums deben venir de `docs/schemas/` cuando exista una taxonomia aprobada.

## Contratos aceptados en Subfase 0.4

Estos contratos quedan aceptados como base documental del flujo `EvidencePack -> AnswerDraft -> ClaimExtractor -> CitationAuditor -> ClaimVerifier -> FinalAnswer`. Esta aceptacion no implementa backend funcional ni `citation-audit.schema.json`.

| Contrato | Estado | Archivo |
|---|---|---|
| Answer Contract | Accepted | `answer-contract.schema.json` |
| Evidence Pack | Accepted | `evidence-pack.schema.json` |
| Source | Accepted | `source.schema.json` |
| Passage | Accepted | `passage.schema.json` |
| Citation | Accepted | `citation.schema.json` |
| Claim | Accepted | `claim.schema.json` |

## Fuera de alcance de Subfase 0.4

`citation-audit.schema.json` quedo delegado desde Subfase 0.4 a Subfase 0.7 junto con trazabilidad, auditoria y versionado. Subfase 0.4 define la forma de `Citation` y las policies de citacion/abstencion, pero no el resultado completo del auditor.

## Contratos aceptados en Subfase 0.5

Estos contratos quedan aceptados como base documental del JusNova Live Legal Search Engine. Esta aceptacion no implementa backend funcional, adaptadores oficiales ni providers externos.

| Contrato | Estado | Archivo |
|---|---|---|
| Legal Entity | Accepted | `legal-entity.schema.json` |
| Legal Search Query | Accepted | `legal-search-query.schema.json` |
| Legal Search Result | Accepted | `legal-search-result.schema.json` |
| Retrieval Plan | Accepted | `retrieval-plan.schema.json` |
| Retrieval Run | Accepted | `retrieval-run.schema.json` |
| Evidence Quality | Accepted | `evidence-quality.schema.json` |
| Provider Interfaces | Accepted | `provider-interfaces.md` |

## Reglas especificas de Subfase 0.5

- `LegalSearchQuery.budget` usa un `search_budget` interno tecnico.
- El presupuesto comercial del Cost Governor queda para Subfase 0.8.
- Todo resultado citable requiere pasaje extraido.
- El snapshot es obligatorio salvo imposibilidad documentada.
- `tier` es el nombre canonico alineado con `source.schema.json`.

## Hardening contractual de Subfase 0.6

- `source.schema.json` representa snapshot de fuente usada mediante `snapshot_id` o `snapshot_unavailable_reason`.
- `snapshot_id` no es obligatorio cuando existe `snapshot_unavailable_reason` cerrada; no se exige emitir un `null` sintetico.
- `snapshot_unavailable_reason` usa enum compartido con `legal-search-result.schema.json`.
- `user_private_document_not_snapshotted` solo aplica a `USER_DOCUMENT` o `documento_usuario`.
- Una fuente usada sin snapshot ni razon cerrada no puede entrar al Evidence Pack final.
- `source_type = norma` no puede usar `validity_status = NO_APLICA`.
- `VIGENCIA_CONFIRMADA` y `DEROGADA_CONFIRMADA` requieren fuente `TIER1_CANONICO`, `TIER1_OFICIAL` o `TIER1_STRUCTURED`.
- En `legal-search-result.schema.json`, `VIGENCIA_CONFIRMADA` y `DEROGADA_CONFIRMADA` requieren extraccion con `passage_refs`.
- `TIER2_CONFIABLE` y `TIER3_SECUNDARIO` requieren advertencia en contratos de fuente y resultado.

## Contratos aceptados en Subfase 0.7

Estos contratos quedan aceptados como base documental de trazabilidad, auditoria y versionado. Esta aceptacion no implementa persistencia, endpoints, UI de soporte, permisos finales ni retencion completa.

| Contrato | Estado | Archivo |
|---|---|---|
| Trace Object | Accepted | `trace-object.schema.json` |
| Model Call | Accepted | `model-call.schema.json` |
| Tool Call | Accepted | `tool-call.schema.json` |
| Citation Audit | Accepted | `citation-audit.schema.json` |
| Answer Version | Accepted | `answer-version.schema.json` |
| Abstention Render | Accepted | `abstention-render.schema.json` |
| Cost Report | Accepted | `cost-report.schema.json` |

## Reglas especificas de Subfase 0.7

- `TraceObject` exige organizacion, actor pseudonimizado, version de respuesta, auditoria de citas, costo observado y latencias cerradas.
- `TraceObject.retrieval_runs[]` usa resumen sanitizado; no embebe `RetrievalRun` operativo completo, URLs crudas de discovery, fuentes abiertas o fuentes rechazadas, mensajes libres de error, warning codes libres ni warnings libres.
- `ModelCall` y `ToolCall` guardan hashes y codigos de error; no guardan prompts, salidas, documentos ni mensajes completos.
- `CitationAudit` contiene resultados por cita y fallas bloqueantes; `passed` no admite fallas bloqueantes.
- `AnswerVersion` referencia `AnswerContract` para respuestas sustantivas y `AbstentionRender` para `total_abstention`/`blocked`; no crea EvidencePack sintetico ni embebe respuesta completa sensible.
- `AbstentionRender` guarda hashes, refs internas y codigos cerrados para reconstruir abstenciones/bloqueos sin texto crudo sensible.
- `CostReport` registra consumo observado y no presupuestos comerciales.

## Contratos aceptados en Subfase 0.8

Estos contratos quedan aceptados como base documental del Cost Governor, budgets por complejidad y Usage Ledger. Esta aceptacion no implementa runtime, billing, Stripe, DB ni conciliacion productiva.

| Contrato | Estado | Archivo |
|---|---|---|
| Cost Budget | Accepted | `cost-budget.schema.json` |
| Usage Event | Accepted | `usage-event.schema.json` |
| Trace Object Budget Amendment | Accepted | `trace-object.schema.json` |
| Answer Version Reference Amendment | Accepted | `answer-version.schema.json` |
| Abstention Render Reference Amendment | Accepted | `abstention-render.schema.json` |

## Reglas especificas de Subfase 0.8

- `CostBudget` define limites exactos por `SIMPLE`, `MEDIO`, `COMPLEJO` e `INVESTIGACION`.
- `CostBudget.cost_budget_id` y las referencias `cost_budget_ref` usan forma canonica `cb_<plan>_<complexity>_vNNN` con plan y complejidad canonicos en minusculas.
- Los budgets internos por complejidad no cambian por plan comercial en v0.
- `TraceObject` registra `cost_budget_ref`, `cost_budget_version`, `plan_code` y `complexity` como escalares cerrados.
- `TraceObject` no embebe `CostBudget` ni referencia `cost-budget.schema.json` por `$ref`.
- `UsageEvent` registra consumo y creditos sin `user_id` crudo ni `metadata` libre.
- `TraceObject.actor_ref`, `UsageEvent.actor_ref`, `AnswerVersion.created_by_ref` y `AbstentionRender.created_by_ref` usan identificadores tecnicos cerrados, no PII directa ni nombres libres.
- `LegalSearchQuery.budget` sigue siendo `search_budget` tecnico interno de busqueda viva.

## Contratos aceptados en Subfase 0.9

Estos contratos quedan aceptados como base documental de conversacion, mensajes, memoria de caso, evidencia documental y OCR. Esta aceptacion no implementa DB, storage, OCR worker, busqueda documental ni memoria persistente.

| Contrato | Estado | Archivo |
|---|---|---|
| Conversation | Accepted | `conversation.schema.json` |
| Message | Accepted | `message.schema.json` |
| Case Memory | Accepted | `case-memory.schema.json` |
| Document Evidence | Accepted | `document-evidence.schema.json` |
| Trace Object Message Reference Amendment | Accepted | `trace-object.schema.json` |

## Reglas especificas de Subfase 0.9

- `Conversation` usa `owner_actor_ref` y `owner_actor_type`, no `user_id` crudo.
- `Conversation` requiere validador custom para `updated_at >= created_at` y, si existe `deleted_at`, `deleted_at >= created_at` y `deleted_at >= updated_at`.
- `Message` puede guardar contenido conversacional como registro primario, pero `TraceObject` solo guarda `input_message_ids` y `output_message_id`.
- `Message.attachments[]` en 0.9 solo representa documentos ya procesados.
- `Message.attachments[].attachment_id` requiere unicidad por mensaje mediante validador custom.
- `CaseMemory` separa hechos afirmados por usuario, hechos soportados por documentos, preguntas abiertas, riesgos y contradicciones.
- `CaseMemory` no es verdad juridica, fuente normativa ni confirmacion de vigencia.
- `CaseMemory` requiere IDs internos unicos para hechos, preguntas, riesgos, partes, fechas y contradicciones mediante validador custom cuando JSON Schema no puede expresarlo por propiedad.
- `DocumentEvidence` representa fragmentos `D#:P#`, no documentos completos.
- `DocumentEvidence` exige tenant, version documental, hashes, locator, metodo de extraccion, confianza y warnings.
- `DocumentEvidence.locator.page` debe coincidir con `DocumentEvidence.page` mediante validador custom.
- `Passage.locator.coordinates` y `DocumentEvidence.locator.coordinates` usan bbox cerrado `x`, `y`, `width`, `height`; no aceptan objetos libres.
- Los hashes de `Message` y `DocumentEvidence` se calculan sobre bytes UTF-8 exactos persistidos.
- Subfase 0.10 define contratos documentales para seguridad, privacidad, provider boundaries, raw access y prompt injection; enforcement/runtime de retencion y permisos productivos queda para Fase 1 o fases posteriores.

## Contratos aceptados en Subfase 0.10

Estos contratos quedan aceptados como base documental de seguridad, privacidad, limites de proveedores, auditoria de llamadas externas, auditoria de acceso raw y defensa contra prompt injection. Esta aceptacion no implementa auth, DB, storage, SDKs de proveedores, SIEM ni permisos runtime.

| Contrato | Estado | Archivo |
|---|---|---|
| Provider Call Audit | Accepted | `provider-call-audit.schema.json` |
| Raw Access Event | Accepted | `raw-access-event.schema.json` |
| Prompt Injection Risk | Accepted | `prompt-injection-risk.schema.json` |
| Document Evidence Security Amendment | Accepted | `document-evidence.schema.json` |
| Legal Search Query Classification Amendment | Accepted | `legal-search-query.schema.json` |
| Model Call Provider Audit Amendment | Accepted | `model-call.schema.json` |
| Tool Call Provider Audit Amendment | Accepted | `tool-call.schema.json` |
| Retrieval Run Prompt Injection Amendment | Accepted | `retrieval-run.schema.json` |
| Trace Object Security Amendment | Accepted | `trace-object.schema.json` |

## Reglas especificas de Subfase 0.10

- `ProviderCallAudit` registra llamadas externas, clases de datos intentadas/enviadas/devueltas, decision de policy, hashes, provider family y provider name.
- `ProviderCallAudit.provider_name` debe resolver contra `provider-registry.yaml` y coincidir en familia.
- `ModelCall` y `ToolCall` declaran `external_provider_call` y `provider_call_audit_id`; una llamada externa no puede quedar sin auditoria.
- `RawAccessEvent` registra acceso raw o elevado por recurso, clasificacion, rol de acceso, aprobador y motivo cerrado.
- `support_operator` no es rol valido en `RawAccessEvent`; soporte normal opera solo con `SUPPORT_VIEW` redacted.
- `DocumentEvidence` exige `document_evidence_id` para auditoria resoluble de fragmentos.
- `DocumentEvidence.prompt_injection_risks[]` es requerido; `[]` significa evaluado sin riesgos detectados.
- `LegalSearchQuery.query_classification` distingue busqueda legal publica abstracta de query derivada de cliente.
- `PromptInjectionRisk` es el contrato compartido para riesgos detectados en documentos, retrieval y trazas.
- `TraceObject.prompt_injection_risks[]` agrega riesgos de evidencia/retrieval usados por la respuesta.
- Ningun contrato 0.10 guarda documentos completos, OCR completo, prompts completos, mensajes completos, HTML bruto ni salidas completas.

## Contratos aceptados en Subfase 0.11

Estos contratos quedan aceptados como base documental del modelo conceptual y del API draft minimo. Esta aceptacion no implementa runtime, routers, OpenAPI formal, migraciones, DB ni servicios.

| Contrato | Estado | Archivo |
|---|---|---|
| Error Envelope | Accepted | `error-envelope.schema.json` |

## Reglas especificas de Subfase 0.11

- `ErrorEnvelope` es obligatorio para respuestas HTTP no 2xx en el API draft.
- `ErrorEnvelope.error_code` y `safe_message_code` usan enums cerrados y mapping deterministico.
- `ErrorEnvelope.message` es texto seguro, acotado y no debe interpolar contenido de usuario, documento, provider o stack trace.
- `ErrorEnvelope.metadata` es objeto cerrado; refs locales `F#`, `D#`, `F#:P#` y `D#:P#` solo pueden aparecer con contexto padre resoluble, y `parent_resource_type`/`parent_resource_ref` no pueden aparecer sin `resource_type`/`resource_ref`.
- `ErrorEnvelope.metadata.allowed_values[]` acepta codigos seguros sin `/` y MIME types permitidos por catalogo cerrado como `application/pdf`; no acepta URLs, paths, espacios, query strings, comillas ni texto libre.
- `ErrorEnvelope.created_at` requiere `format: date-time`, patron RFC3339 basico y validacion CI con `ajv-formats`/format assertions para rechazar fechas calendario imposibles.
- Un `run_id` en progreso usa forma `tr_*`, pero no es `TraceObject` schema-valid hasta que exista `answer_id` y `answer_version_ref`.
- `Conversation`, `Message`, `AnswerContract`, `AnswerVersion`, `EvidencePack`, `RetrievalRun`, `Claim`, `TraceObject` y `UsageEvent` producen refs canonicas compatibles con `api-draft-v0.md` y `ErrorEnvelope`: `conv_*`, `msg_*`, `ans_*`, `av_*`, `ep_*`, `rr_*`, `cl_*`, `tr_*` y `ue_*`.
- Todo contrato privado que requiere `organization_id` exige forma canonica `org_*`, alineada con `domain-model.md` y `entity-ownership-matrix.md`.
- `domain-model.md`, `entity-ownership-matrix.md` y `api-draft-v0.md` gobiernan relaciones conceptuales, ownership y visibilidad de endpoints para Fase 1.

## Schemas minimos esperados

```txt
answer-contract.schema.json
abstention-render.schema.json
claim.schema.json
citation.schema.json
citation-audit.schema.json
cost-budget.schema.json
provider-call-audit.schema.json
document-evidence.schema.json
evidence-pack.schema.json
evidence-quality.schema.json
error-envelope.schema.json
legal-search-query.schema.json
legal-search-result.schema.json
message.schema.json
model-call.schema.json
passage.schema.json
prompt-injection-risk.schema.json
raw-access-event.schema.json
retrieval-plan.schema.json
retrieval-run.schema.json
source.schema.json
source-registry-entry.schema.json
source-snapshot.schema.json
trace-object.schema.json
tool-call.schema.json
usage-event.schema.json
case-memory.schema.json
conversation.schema.json
```
