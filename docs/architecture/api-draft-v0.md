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

Devuelve metadata conversacional y pagina de mensajes seguros. No devuelve trace internals ni provider audit.

### POST /v1/conversations/{conversation_id}/messages

Request:

| Campo | Regla |
|---|---|
| `content` | Texto del mensaje que se persiste como `Message`, no se copia a trazas. |
| `attachments` | Solo documentos procesados con `attachment_id=att_*`, `document_id=doc_*` y `document_version_id=docv_*`; el cliente no envia `source_ref`. |

`source_ref` (`D#`) no es input del cliente en este endpoint. Si el `Message` persistido necesita `source_ref` por el contrato 0.9, el backend lo asigna internamente como ref local del adjunto procesado; los refs `D#` de Evidence Packs se asignan despues dentro del `evidence_pack_id` correspondiente y no son IDs globales.

Response:

| Campo | Regla |
|---|---|
| `message_id` | `msg_*`. |
| `run_id` | `tr_*` reservado si inicia ejecucion; no es todavia `TraceObject` valido. |
| `status` | `accepted` o `queued`. |

### GET /v1/conversations/{conversation_id}/runs/{run_id}/stream

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

`GET /v1/cases/{case_id}/memory` devuelve `CaseMemorySafeSummary`, no `case-memory.schema.json` completo. La vista segura puede incluir resumen de hechos, riesgos y preguntas, pero no debe usarse como evidencia juridica ni exponer material raw. Acceso raw/elevado a memoria requiere `RawAccessEvent`.

## Document endpoints

`POST /v1/documents` crea un shell de documento y un intent de carga/procesamiento. No requiere `DocumentVersion` inmediata.

`GET /v1/documents/{document_id}` devuelve solo metadata/status:

- `document_id`;
- estado de procesamiento;
- conteos;
- warnings cerrados;
- timestamps;
- refs internas seguras.

No devuelve binario, signed URL, OCR completo, texto completo, storage key ni documento completo.

`POST /v1/documents/{document_id}/process` inicia procesamiento. `GET /processing-status` devuelve progreso, conteos y warnings, no OCR completo.

## Answer endpoints

`GET /v1/answers/{answer_id}` devuelve respuesta visible. La respuesta debe identificar la version visible mediante `answer_version_ref`.

`GET /v1/answers/{answer_id}/sources` devuelve solo fuentes realmente citadas para la version visible. Cualquier `source_ref` (`F#`/`D#`) o `passage_ref` (`F#:P#`/`D#:P#`) queda estrictamente scoped al `answer_version_ref` y al `evidence_pack_id` devueltos por el endpoint; esos refs no son IDs globales.

`GET /v1/answers/{answer_id}/trace-summary` devuelve resumen seguro, nunca `TraceObject` completo.

## Usage endpoints

`GET /v1/usage/current` devuelve agregados del periodo y limites visibles. `GET /v1/research-credits` devuelve balance y movimientos resumidos. El ledger raw de `UsageEvent` queda interno/auditado.

## Errores

Todo endpoint debe declarar `ErrorEnvelope` para:

- validacion;
- auth/forbidden/tenant mismatch;
- budget/research credit;
- provider/storage/timeout;
- policy/prompt injection;
- evidencia insuficiente;
- documento pendiente o fallido.
