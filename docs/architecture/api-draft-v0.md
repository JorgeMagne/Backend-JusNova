# API Draft v0

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.11 - Modelo de datos conceptual y APIs draft

## Proposito

Este documento define el API minimo que Fase 1 puede usar para crear routers y contratos de transporte sin inventar estructura. No es OpenAPI formal ni implementacion FastAPI.

## Reglas globales

1. Todo endpoint requiere contexto autenticado de tenant, salvo health checks futuros fuera de 0.11.
2. `organization_id` se resuelve desde auth/membership; no se acepta en body normal de cliente.
3. Toda respuesta no 2xx usa `ErrorEnvelope`.
4. Ningun endpoint devuelve `TraceObject` completo, payload completo de proveedor, prompt completo, mensaje completo fuera del recurso `Message`, OCR completo, documento completo, storage key ni signed URL irrestricta.
5. Las vistas de traza para usuario son `USER_SUMMARY`.
6. Endpoints internos de soporte, admin y auditoria quedan fuera de 0.11 salvo los listados aqui.
7. Runs en progreso usan `run_id` reservado con forma `tr_*`; ese run no es `TraceObject` valido hasta la finalizacion.

## Path parameters

| Parametro | Patron |
|---|---|
| `conversation_id` | `^conv_[A-Za-z0-9_-]+$` |
| `run_id` | `^tr_[A-Za-z0-9_-]+$` |
| `case_id` | `^case_[A-Za-z0-9_-]+$` |
| `document_id` | `^doc_[A-Za-z0-9_-]+$` |
| `answer_id` | `^ans_[A-Za-z0-9_-]+$` |
| `answer_version_ref` | `^av_[A-Za-z0-9_-]+$` |
| `request_id` | `^rq_[A-Za-z0-9_-]+$` |

## Endpoints

| Metodo | Ruta | Proposito | Visibilidad | Error |
|---|---|---|---|---|
| POST | `/v1/conversations` | Crear `Conversation` shell. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/conversations/{conversation_id}` | Leer metadata y pagina de mensajes seguros. | `USER_SUMMARY` | `ErrorEnvelope` |
| POST | `/v1/conversations/{conversation_id}/messages` | Crear `Message` y opcionalmente iniciar run. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/conversations/{conversation_id}/runs/{run_id}/stream` | Stream de progreso seguro. | `USER_SUMMARY` | `ErrorEnvelope` |
| POST | `/v1/cases` | Crear `Case`. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/cases/{case_id}` | Leer metadata de caso. | `USER_SUMMARY` | `ErrorEnvelope` |
| PATCH | `/v1/cases/{case_id}` | Actualizar metadata permitida del caso. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/cases/{case_id}/memory` | Leer `CaseMemorySafeSummary`. | `USER_SUMMARY` redacted | `ErrorEnvelope` |
| POST | `/v1/documents` | Crear shell/upload intent de documento. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/documents/{document_id}` | Leer metadata/status documental. | `USER_SUMMARY` | `ErrorEnvelope` |
| POST | `/v1/documents/{document_id}/process` | Iniciar procesamiento documental. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/documents/{document_id}/processing-status` | Leer progreso de procesamiento. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/answers/{answer_id}` | Leer respuesta visible. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/answers/{answer_id}/sources` | Leer fuentes citadas usadas. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/answers/{answer_id}/trace-summary` | Leer resumen seguro de traza. | `USER_SUMMARY` | `ErrorEnvelope` |
| GET | `/v1/usage/current` | Leer uso agregado del periodo actual. | `USER_SUMMARY` o admin tenant futuro | `ErrorEnvelope` |
| GET | `/v1/research-credits` | Leer balance y movimientos resumidos de creditos. | `USER_SUMMARY` | `ErrorEnvelope` |

## Conversation endpoints

### POST /v1/conversations

Request:

| Campo | Regla |
|---|---|
| `case_id` | Opcional, `case_*` o `null`. |
| `title` | Opcional, no es evidencia. |

Response:

| Campo | Regla |
|---|---|
| `conversation_id` | `conv_*`. |
| `case_id` | `case_*` o `null`. |
| `status` | `active`. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

No crea `Message` obligatorio.

### GET /v1/conversations/{conversation_id}

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `conversation_id` | `conv_*`. |
| `case_id` | `case_*` o `null`. |
| `title` | `string|null`; no es evidencia. |
| `status` | `active`, `archived` o `deleted`. |
| `message_summaries[]` | Pagina de refs/summaries seguros; no incluye trace internals ni provider audit. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

### POST /v1/conversations/{conversation_id}/messages

Request:

| Campo | Regla |
|---|---|
| `content` | Texto del mensaje que se persiste como `Message`, no se copia a trazas. |
| `attachments` | Solo documentos procesados con `document_id=doc_*` y `document_version_id=docv_*`; el cliente no envia `attachment_id` ni `source_ref`. |

`attachment_id` y `source_ref` (`D#`) no son input del cliente en este endpoint. Si el `Message` persistido necesita esos campos por el contrato 0.9, el backend asigna `attachment_id=att_*` y `source_ref` internamente como refs locales del adjunto procesado; los refs `D#` de Evidence Packs se asignan despues dentro del `evidence_pack_id` correspondiente y no son IDs globales.

Response:

| Campo | Regla |
|---|---|
| `message_id` | `msg_*`. |
| `run_id` | `tr_*` reservado si inicia ejecucion; no es todavia `TraceObject` valido. |
| `status` | `accepted` o `queued`. |

### GET /v1/conversations/{conversation_id}/runs/{run_id}/stream

Request: sin body; stream asociado a path params.

Response: stream de eventos seguros.

Eventos in-progress:

| Campo | Regla |
|---|---|
| `run_id` | `tr_*` reservado. |
| `event_type` | `run_queued`, `run_started`, `retrieval_started`, `retrieval_progress`, `document_processing_required`, `evidence_ready` o `run_failed`. |
| `status` | `queued`, `running`, `waiting_for_document_processing` o `failed`. |

Evento final exitoso/bloqueado:

| Campo | Regla |
|---|---|
| `event_type` | `answer_final` o `answer_blocked`. |
| `status` | `finalized` o `blocked`. |
| `run_id` | Ahora resuelve a `TraceObject.trace_id`. |
| `answer_id` | `ans_*`. |
| `answer_version_ref` | `av_*`. |
| `trace_summary_url` | URL relativa a `/v1/answers/{answer_id}/trace-summary`. |

Si la ejecucion falla antes de crear respuesta/version, el error usa `ErrorEnvelope` sin afirmar que existe `TraceObject`.

## Case endpoints

### POST /v1/cases

Request:

| Campo | Regla |
|---|---|
| `title` | Opcional, `string|null`; no es evidencia. |
| `procedural_stage_code` | Opcional; si existe usa enum de `CaseMemory.procedural_stage_code`. |
| `procedural_stage_label` | Opcional, `string|null`. |
| `conversation_id` | Opcional `conv_*` para vincular una conversacion inicial. |

Response:

| Campo | Regla |
|---|---|
| `case_id` | `case_*`. |
| `title` | `string|null`. |
| `status` | `active`. |
| `procedural_stage_code` | Enum cerrado o `unknown`. |
| `conversation_ids[]` | `conv_*`; puede ser `[]`. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

### GET /v1/cases/{case_id}

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `case_id` | `case_*`. |
| `title` | `string|null`; no es fuente juridica. |
| `status` | `active`, `archived` o `deleted`. |
| `procedural_stage_code` | Enum cerrado de memoria o `unknown`. |
| `procedural_stage_label` | `string|null`. |
| `conversation_ids[]` | Refs `conv_*`, no mensajes completos. |
| `document_ids[]` | Refs `doc_*`, no documentos ni OCR. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

### PATCH /v1/cases/{case_id}

Request:

| Campo | Regla |
|---|---|
| `title` | Opcional, `string|null`; no es evidencia. |
| `status` | Opcional: `active`, `archived` o `deleted`. |
| `procedural_stage_code` | Opcional; enum cerrado de memoria. |
| `procedural_stage_label` | Opcional, `string|null`. |

Response:

| Campo | Regla |
|---|---|
| `case_id` | `case_*`. |
| `status` | Estado persistido. |
| `procedural_stage_code` | Valor persistido. |
| `procedural_stage_label` | `string|null`. |
| `updated_at` | date-time. |

### GET /v1/cases/{case_id}/memory

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `case_id` | `case_*`. |
| `memory_version` | Entero `>= 1`. |
| `safe_summary` | `CaseMemorySafeSummary`, no `case-memory.schema.json` completo. |
| `facts_summary[]` | Texto minimizado/redacted; no prueba juridica por si solo. |
| `risk_flags[]` | Codigos/severidad, sin raw document/message content. |
| `open_legal_questions[]` | Preguntas seguras, no evidence pack. |
| `updated_at` | date-time. |

La vista segura puede incluir resumen de hechos, riesgos y preguntas, pero no debe usarse como evidencia juridica ni exponer material raw. Acceso raw/elevado a memoria requiere `RawAccessEvent`.

## Document endpoints

### POST /v1/documents

Request:

| Campo | Regla |
|---|---|
| `case_id` | Opcional `case_*` o `null`. |
| `filename_hint` | Opcional, solo display redacted; no storage key ni evidencia. |
| `content_type` | MIME declarado; se valida por allowlist/sniffing en processing. |
| `size_bytes` | Entero positivo. |

Response:

| Campo | Regla |
|---|---|
| `document_id` | `doc_*`. |
| `case_id` | `case_*` o `null`. |
| `upload_status` | `accepted`, `rejected` o `quarantined`. |
| `processing_status` | `queued`, `processing`, `processed`, `failed` o `blocked`; inicial normalmente `queued` o `blocked`. |
| `storage_object_ref` | `sto_*` o `null`; si `upload_status=rejected`, debe ser `null`; si `quarantined`, apunta a objeto privado no usable hasta validacion. |
| `accepted_content_types[]` | Codigos seguros o MIME allowlist. |
| `max_size_bytes` | Limite visible. |
| `created_at` | date-time. |

Crea un shell de documento y un objeto privado inicial cuando el upload es aceptado o puesto en cuarentena. No requiere `DocumentVersion` inmediata.

### GET /v1/documents/{document_id}

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `document_id` | `doc_*`. |
| `case_id` | `case_*` o `null`. |
| `latest_document_version_id` | `docv_*` o `null`. |
| `upload_status` | `accepted`, `rejected` o `quarantined`. |
| `processing_status` | `queued`, `processing`, `processed`, `failed` o `blocked`. |
| `storage_object_ref` | `sto_*` o `null`; `null` cuando `upload_status=rejected`. |
| `page_count` | Entero `>= 0` o `null`. |
| `chunk_count` | Entero `>= 0` o `null`. |
| `evidence_count` | Entero `>= 0`. |
| `warnings[]` | Codigos cerrados, no texto OCR. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

Devuelve solo metadata/status. No devuelve binario, signed URL, OCR completo, texto completo, storage key ni documento completo.

### POST /v1/documents/{document_id}/process

Request:

| Campo | Regla |
|---|---|
| `document_version_id` | Opcional `docv_*`; si falta, backend usa version pendiente/latest. |
| `force_reprocess` | Opcional boolean; default `false`. |

Response:

| Campo | Regla |
|---|---|
| `document_id` | `doc_*`. |
| `document_version_id` | `docv_*` o `null` si aun no existe version procesable. |
| `processing_status` | `queued`, `processing` o `blocked`. |
| `processing_status_url` | URL relativa a `/v1/documents/{document_id}/processing-status`. |
| `warnings[]` | Codigos cerrados. |

### GET /v1/documents/{document_id}/processing-status

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `document_id` | `doc_*`. |
| `document_version_id` | `docv_*` o `null`. |
| `processing_status` | `queued`, `processing`, `processed`, `failed` o `blocked`. |
| `pages_total` | Entero `>= 0` o `null`. |
| `pages_processed` | Entero `>= 0`. |
| `warnings[]` | Codigos cerrados. |
| `updated_at` | date-time. |

Devuelve progreso, conteos y warnings, no OCR completo.

## Answer endpoints

### GET /v1/answers/{answer_id}

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `answer_id` | `ans_*`. |
| `answer_version_ref` | `av_*` visible. |
| `response_outcome` | Outcome cerrado de respuesta/version. |
| `sections` | Secciones visibles/redacted del `AnswerContract`; no payload de modelo. |
| `warnings[]` | Codigos/mensajes seguros. |
| `created_at` | date-time. |

Devuelve respuesta visible. La respuesta debe identificar la version visible mediante `answer_version_ref`.

### GET /v1/answers/{answer_id}/sources

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `answer_id` | `ans_*`. |
| `answer_version_ref` | `av_*`. |
| `evidence_pack_id` | `ep_*`. |
| `sources[]` | Fuentes realmente citadas; `source_ref` local al pack. |
| `passages[]` | Pasajes citados; `passage_ref` local al pack. |
| `citations[]` | `citation_ref=C#` local a `answer_version_ref`. |

Devuelve solo fuentes realmente citadas para la version visible. Cualquier `source_ref` (`F#`/`D#`) o `passage_ref` (`F#:P#`/`D#:P#`) queda estrictamente scoped al `answer_version_ref` y al `evidence_pack_id` devueltos por el endpoint; esos refs no son IDs globales.

### GET /v1/answers/{answer_id}/trace-summary

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `answer_id` | `ans_*`. |
| `answer_version_ref` | `av_*`. |
| `trace_id` | `tr_*`. |
| `summary_visibility` | `USER_SUMMARY`. |
| `steps[]` | Codigos seguros de alto nivel, sin prompts ni payloads. |
| `warnings[]` | Codigos seguros. |
| `cost_summary` | Totales agregados permitidos; no `CostReport` raw. |

Devuelve resumen seguro, nunca `TraceObject` completo.

## Usage endpoints

### GET /v1/usage/current

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `period_start` | date-time. |
| `period_end` | date-time. |
| `plan_code` | Codigo de plan visible. |
| `subscription_id` | `sub_*` o `null`. |
| `usage_totals` | Agregados por categoria; no ledger raw. |
| `limits` | Limites visibles del plan/presupuesto. |
| `research_credits_balance` | Numero visible o `null`. |

Devuelve agregados del periodo y limites visibles.

### GET /v1/research-credits

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `balance` | Numero de creditos disponibles. |
| `currency` | `NONE` o codigo futuro si aplica. |
| `movements[]` | Movimientos resumidos con fecha, tipo y delta; no `UsageEvent` raw. |
| `updated_at` | date-time. |

Devuelve balance y movimientos resumidos. El ledger raw de `UsageEvent` queda interno/auditado.

## Errores

Todo endpoint debe declarar `ErrorEnvelope` para:

- validacion;
- auth/forbidden/tenant mismatch;
- budget/research credit;
- provider/storage/timeout;
- policy/prompt injection;
- evidencia insuficiente;
- documento pendiente o fallido.

Cada endpoint queda limitado a los siguientes `error_code` de `error-envelope.schema.json`. Fase 1 no debe emitir codigos fuera de la fila aplicable sin enmendar este draft o el contrato de errores.

| Endpoint | `error_code` permitidos |
|---|---|
| `POST /v1/conversations` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `conflict`, `rate_limited`, `internal_error` |
| `GET /v1/conversations/{conversation_id}` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `rate_limited`, `internal_error` |
| `POST /v1/conversations/{conversation_id}/messages` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `conflict`, `rate_limited`, `payload_too_large`, `document_processing_required`, `budget_exhausted`, `research_credit_required`, `policy_blocked`, `prompt_injection_blocked`, `evidence_insufficient`, `provider_unavailable`, `storage_unavailable`, `timeout`, `internal_error` |
| `GET /v1/conversations/{conversation_id}/runs/{run_id}/stream` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `document_processing_required`, `document_processing_failed`, `budget_exhausted`, `research_credit_required`, `policy_blocked`, `prompt_injection_blocked`, `unsupported_critical_claim`, `evidence_insufficient`, `provider_unavailable`, `storage_unavailable`, `timeout`, `internal_error` |
| `POST /v1/cases` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `conflict`, `rate_limited`, `internal_error` |
| `GET /v1/cases/{case_id}` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `rate_limited`, `internal_error` |
| `PATCH /v1/cases/{case_id}` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `conflict`, `rate_limited`, `internal_error` |
| `GET /v1/cases/{case_id}/memory` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `policy_blocked`, `rate_limited`, `internal_error` |
| `POST /v1/documents` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `conflict`, `rate_limited`, `payload_too_large`, `unsupported_file_type`, `storage_unavailable`, `internal_error` |
| `GET /v1/documents/{document_id}` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `rate_limited`, `internal_error` |
| `POST /v1/documents/{document_id}/process` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `conflict`, `rate_limited`, `document_processing_required`, `provider_unavailable`, `storage_unavailable`, `timeout`, `internal_error` |
| `GET /v1/documents/{document_id}/processing-status` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `rate_limited`, `document_processing_failed`, `timeout`, `internal_error` |
| `GET /v1/answers/{answer_id}` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `policy_blocked`, `prompt_injection_blocked`, `unsupported_critical_claim`, `evidence_insufficient`, `rate_limited`, `internal_error` |
| `GET /v1/answers/{answer_id}/sources` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `policy_blocked`, `prompt_injection_blocked`, `evidence_insufficient`, `rate_limited`, `internal_error` |
| `GET /v1/answers/{answer_id}/trace-summary` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `policy_blocked`, `rate_limited`, `internal_error` |
| `GET /v1/usage/current` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `rate_limited`, `internal_error` |
| `GET /v1/research-credits` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `rate_limited`, `research_credit_required`, `internal_error` |
