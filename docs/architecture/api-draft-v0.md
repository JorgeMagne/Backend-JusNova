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
8. Todo body JSON tiene limite de `131072` bytes una vez aplicada la decodificacion HTTP y antes de parsear JSON. Se rechaza `Content-Encoding` no soportado; ningun proxy puede medir solo bytes comprimidos. Los endpoints que permiten `payload_too_large` devuelven ese codigo al superar el limite; los demas rechazan antes del handler con `validation_error` seguro hasta que su fila de errores sea enmendada. Multipart/upload usa su limite binario separado y visible.
9. Identificadores y refs de transporte tienen `1..160` caracteres. El pattern especifico sigue siendo obligatorio y no se recorta ni normaliza antes de validar.
10. Todo campo API que proyecta un contrato aceptado hereda su enum, pattern, formato, `minLength`, `maxLength`, `minItems`, `maxItems`, unicidad y reglas condicionales; una tabla resumida no relaja el JSON Schema. Para campos exclusivos del DTO publico: titulo visible `1..500`, label/texto seguro de una linea `1..240`, summary redacted `1..2000`, MIME declarado `1..255`, cursor opaco `1..160` y ninguna cadena admite controles C0/C1 ni `U+2028/U+2029`. Una cota especifica menor prevalece.
11. Ninguna coleccion publica es ilimitada. Warnings/notas admiten maximo `32` valores unicos; summaries/steps no paginados maximo `100`; toda coleccion declarada como pagina usa `limit` default `50`, rango `1..100`, `cursor` opaco opcional con pattern `^[A-Za-z0-9_-]{1,160}$`, `next_cursor` nullable y `has_more` boolean. El cursor se valida por valor completo, es tenant/resource-scoped y no contiene datos raw.
12. Los objetos auxiliares API-only son cerrados y tipados; no aceptan propiedades libres. Mapas como `usage_totals` y `limits` solo usan categorias canonicas versionadas y numeros no negativos. Serializers rechazan limite + 1 y propiedades extra; nunca truncan silenciosamente un request.

## Identificadores de transporte y path parameters

| Parametro | Patron |
|---|---|
| `conversation_id` | `^conv_[A-Za-z0-9_-]+$` |
| `run_id` | `^tr_[A-Za-z0-9_-]+$` |
| `case_id` | `^case_[A-Za-z0-9_-]+$` |
| `document_id` | `^doc_[A-Za-z0-9_-]+$` |
| `answer_id` | `^ans_[A-Za-z0-9_-]+$` |
| `answer_version_ref` | `^av_[A-Za-z0-9_-]+$` |
| `request_id` | `^rq_[A-Za-z0-9_-]+$` |
| Header `Idempotency-Key` | `^[A-Za-z0-9._:-]{1,128}$`; obligatorio en `POST /v1/conversations/{conversation_id}/messages`. |

Los patrones de esta tabla usan `^...$` solo como notacion legible. La validacion implementada debe ser de valor completo (`fullmatch` o equivalente al cierre real de `docs/contracts/README.md`), sin trim/coercion previa, y debe rechazar `CR`, `LF`, `U+2028` y `U+2029` terminales; no se compila `$` como unica garantia de fin.

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
| `title` | Opcional `string/null`; si existe, `1..180` caracteres y texto seguro de una linea, igual que `Conversation.title`; no es evidencia. |

Response:

| Campo | Regla |
|---|---|
| `conversation_id` | `conv_*`. |
| `case_id` | `case_*` o `null`. |
| `status` | `active`. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

No crea `Message` obligatorio.

Perfil de implementación de Fase 1: mientras no exista tabla/servicio `Case` tenant-scoped, `POST /v1/conversations` solo acepta `case_id=null` u omitido. Un `case_id=case_*` no puede persistirse como ref libre: devuelve `ErrorEnvelope.error_code=validation_error`. Habilitarlo exige resolver existencia y `organization_id` del `Case` dentro de la misma transacción; esta restricción temporal no crea una ruta ni un contrato paralelo.

### GET /v1/conversations/{conversation_id}

Request: sin body. Query opcional de la pagina de mensajes: `cursor` y `limit` conforme a la regla global 11.

Response:

| Campo | Regla |
|---|---|
| `conversation_id` | `conv_*`. |
| `case_id` | `case_*` o `null`. |
| `title` | `string/null`; si existe conserva `1..180` y el pattern seguro de `Conversation.title`; no es evidencia. |
| `status` | `active`, `archived` o `deleted`. |
| `message_summaries[]` | Pagina de maximo `100` refs/summaries seguros; cada ref usa el perfil `1..160` y cada summary `1..240`; no incluye trace internals ni provider audit. |
| `next_cursor` | Cursor opaco `1..160` o `null`. |
| `has_more` | Boolean coherente con `next_cursor`. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

Cada item de `message_summaries[]` es un objeto cerrado con `message_id`, `role`, `message_kind`, `safe_summary` y `created_at`. `role`/`message_kind` usan los enums de `Message`; `safe_summary` es texto sintetico/redacted de `1..240`, no `Message.content`, y ninguna propiedad adicional se admite.

### POST /v1/conversations/{conversation_id}/messages

Header obligatorio: `Idempotency-Key`, entre `1` y `128` caracteres ASCII del conjunto `[A-Za-z0-9._:-]`. El backend persiste solo `sha256:<hex>` de la key y un fingerprint JCS/SHA-256 del request tenant/actor-scoped; nunca guarda ni registra la key raw. Repetir la misma key con el mismo fingerprint reproduce el resultado ya confirmado y reutiliza los mismos `message_id`/`run_id` cuando existan, sin repetir pipeline, usage ni cargos. Reutilizarla con otro fingerprint devuelve `ErrorEnvelope.error_code=conflict`. El scope de unicidad es `(organization_id, actor_ref, conversation_id, idempotency_key_hash)` y `actor_ref` se deriva de auth, nunca del body.

Request:

| Campo | Regla |
|---|---|
| `content` | Texto de `1..20000` code points Unicode que se persiste como `Message`; no se copia a trazas. |
| `attachments` | Maximo `20`; solo documentos procesados con `document_id=doc_*` y `document_version_id=docv_*`; el cliente no envia `attachment_id` ni `source_ref`. |

`attachment_id` y `source_ref` (`D#`) no son input del cliente en este endpoint. Si el `Message` persistido necesita esos campos por el contrato 0.9, el backend asigna `attachment_id=att_*` y `source_ref` internamente como refs locales del adjunto procesado; los refs `D#` de Evidence Packs se asignan despues dentro del `evidence_pack_id` correspondiente y no son IDs globales.

Perfil de implementación de Fase 1: mientras no exista `Document`/`DocumentVersion` tenant-scoped ni pipeline documental, `attachments` debe ser `[]` u omitirse. Un array no vacío devuelve `ErrorEnvelope.error_code=document_processing_required` antes de persistir mensaje o run; habilitar adjuntos exige resolver documento, versión, estado `processed` y `organization_id` en la misma transacción. Esta restricción temporal no cambia el contrato completo del endpoint.

El body completo conserva el limite global de `131072` bytes. Superar ese limite, `content.maxLength` o `attachments.maxItems` devuelve `payload_too_large` antes de persistir `Message`, reservar IDs o crear un run. Un JSON mal formado o una violacion de shape dentro de los limites devuelve `validation_error`.

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
| `title` | Opcional, `string/null`; si existe, `1..500` caracteres y texto seguro de una linea; no es evidencia. |
| `procedural_stage_code` | Opcional; si existe usa enum de `CaseMemory.procedural_stage_code`. |
| `procedural_stage_label` | Opcional, `string/null`; si existe, `1..120` caracteres y texto seguro de una linea, igual que `CaseMemory.procedural_stage_label`. |
| `conversation_id` | Opcional `conv_*` para vincular una conversacion inicial. |

Response:

| Campo | Regla |
|---|---|
| `case_id` | `case_*`. |
| `title` | `string/null`; si existe, `1..500` y texto seguro de una linea. |
| `status` | `active`. |
| `procedural_stage_code` | Enum cerrado o `unknown`. |
| `conversation_ids[]` | `conv_*`; puede ser `[]` y en esta respuesta de creacion contiene maximo el unico `conversation_id` solicitado. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

### GET /v1/cases/{case_id}

Request: sin body. Query opcional independiente para refs: `conversation_cursor`, `document_cursor`, `conversation_limit` y `document_limit`; cada cursor/limit conserva los perfiles de la regla global 11.

Response:

| Campo | Regla |
|---|---|
| `case_id` | `case_*`. |
| `title` | `string/null`; si existe, `1..500` y texto seguro de una linea; no es fuente juridica. |
| `status` | `active`, `archived` o `deleted`. |
| `procedural_stage_code` | Enum cerrado de memoria o `unknown`. |
| `procedural_stage_label` | `string/null`; si existe, `1..120` y texto seguro de una linea. |
| `conversation_ids[]` | Pagina de maximo `100` refs `conv_*` unicas, no mensajes completos. |
| `conversation_next_cursor` | Cursor opaco `1..160` o `null`. |
| `conversation_has_more` | Boolean coherente con `conversation_next_cursor`. |
| `document_ids[]` | Pagina de maximo `100` refs `doc_*` unicas, no documentos ni OCR. |
| `document_next_cursor` | Cursor opaco `1..160` o `null`. |
| `document_has_more` | Boolean coherente con `document_next_cursor`. |
| `created_at` | date-time. |
| `updated_at` | date-time. |

### PATCH /v1/cases/{case_id}

Request:

| Campo | Regla |
|---|---|
| `title` | Opcional, `string/null`; si existe, `1..500` y texto seguro de una linea; no es evidencia. |
| `status` | Opcional: `active`, `archived` o `deleted`. |
| `procedural_stage_code` | Opcional; enum cerrado de memoria. |
| `procedural_stage_label` | Opcional, `string/null`; si existe, `1..120` y texto seguro de una linea. |

Response:

| Campo | Regla |
|---|---|
| `case_id` | `case_*`. |
| `status` | Estado persistido. |
| `procedural_stage_code` | Valor persistido. |
| `procedural_stage_label` | `string/null`; si existe, `1..120` y texto seguro de una linea. |
| `updated_at` | date-time. |

### GET /v1/cases/{case_id}/memory

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `case_id` | `case_*`. |
| `memory_version` | Entero `>= 1`. |
| `safe_summary` | Texto `CaseMemorySafeSummary` redacted de `1..2000`; no `case-memory.schema.json` completo. |
| `facts_summary[]` | Maximo `100` textos minimizados/redacted de `1..240`; no prueba juridica por si solo. |
| `risk_flags[]` | Maximo `100` objetos cerrados de codigos/severidad, sin raw document/message content. |
| `open_legal_questions[]` | Maximo `100` preguntas seguras de `1..240`, no evidence pack. |
| `summary_truncated` | Boolean; `true` si alguna coleccion interna supera la proyeccion segura y no fue incluida completa. |
| `updated_at` | date-time. |

La vista segura puede incluir resumen de hechos, riesgos y preguntas, pero no debe usarse como evidencia juridica ni exponer material raw. Acceso raw/elevado a memoria requiere `RawAccessEvent`.

Cada item de `risk_flags[]` es un objeto cerrado con `risk_type`, `severity` y `safe_label` opcional de `1..240`; los dos enums provienen de `CaseMemory.risk_flags[]`. El DTO completo solo admite los campos de la tabla y nunca serializa parties, source history, contradictions, facts raw ni refs privadas no incluidas explicitamente.

## Document endpoints

### POST /v1/documents

Request: `multipart/form-data`. El binario viaja en la parte `file`; no existe upload out-of-band, storage key aportada por cliente ni body JSON alternativo en v1.

| Campo | Regla |
|---|---|
| `file` | Requerido; stream binario. El backend calcula tamano y tipo efectivo mediante sniffing antes de aceptar, rechazar o poner en cuarentena. |
| `case_id` | Opcional `case_*` o `null`. |
| `filename_hint` | Opcional `1..240`, texto seguro de una linea y solo display redacted; no storage key, path ni evidencia. |
| `content_type` | Hint opcional ASCII `1..255` con shape MIME `type/subtype`, sin parametros ni controles. No sustituye el MIME detectado por servidor ni decide por si solo la allowlist. |
| `size_bytes` | Hint opcional entero positivo. El limite se aplica sobre bytes realmente recibidos y el valor divergente produce `validation_error`. |

Gate de habilitacion fail-closed:

1. `POST /v1/documents` permanece deshabilitado y no puede exponerse en ningun ambiente hasta que una decision versionada resuelva `OQ-021`, antes de implementar la superficie documental productiva, y fije un `max_size_bytes` entero positivo y la allowlist exacta de MIME/extensiones. Esta delegacion no autoriza un default ilimitado ni un valor elegido ad hoc por implementacion.
2. Proxy, aplicacion y storage gateway aplican el mismo limite o uno mas estricto. `Content-Length` y `size_bytes` son solo hints: la aplicacion cuenta los bytes decodificados realmente recibidos durante el stream y aborta al superar el limite.
3. El exceso devuelve `payload_too_large` y una violacion de tipo devuelve `unsupported_file_type`, siempre antes de crear `Document`, reservar `document_id`, escribir un objeto privado o generar derivados. Un stream parcial se descarta y no deja shell, cuarentena ni storage ref huerfanos.
4. La respuesta publica `max_size_bytes` coincide exactamente con el limite efectivo de la configuracion versionada. Las pruebas cubren limite exacto, limite + 1, `Content-Length` ausente/falso, stream fragmentado y tipo declarado distinto del detectado.

Response:

| Campo | Regla |
|---|---|
| `document_id` | `doc_*`. |
| `case_id` | `case_*` o `null`. |
| `upload_status` | `accepted`, `rejected` o `quarantined`. |
| `processing_status` | En respuesta de creacion solo `queued` o `blocked`. |
| `storage_object_ref` | `sto_*` o `null`; si `upload_status=rejected`, debe ser `null`; si `quarantined`, apunta a objeto privado no usable hasta validacion. |
| `accepted_content_types[]` | Maximo `32` MIME/codigos seguros unicos de la allowlist versionada. |
| `max_size_bytes` | Limite visible. |
| `created_at` | date-time. |

Crea un shell de documento y un objeto privado inicial cuando el upload es aceptado o puesto en cuarentena. No requiere `DocumentVersion` inmediata.

Compatibilidad de estados documentales:

| `upload_status` | `processing_status` permitido |
|---|---|
| `accepted` | `queued`, `processing`, `processed`, `failed`, `blocked` |
| `rejected` | `blocked`, `failed` |
| `quarantined` | `queued`, `blocked` |

En `POST /v1/documents`, el estado inicial solo puede ser `queued` o `blocked`: `accepted -> queued|blocked`, `quarantined -> queued|blocked`, `rejected -> blocked`.

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
| `warnings[]` | Maximo `32` codigos cerrados unicos, no texto OCR. |
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
| `warnings[]` | Maximo `32` codigos cerrados unicos. |

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
| `warnings[]` | Maximo `32` codigos cerrados unicos. |
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
| `sections` | Requerido para `answered` o `partial_abstention`; secciones visibles/redacted del `AnswerContract`, nunca payload de modelo. `null` u omitido para `total_abstention` o `blocked`. |
| `abstention` | Requerido para `total_abstention` o `blocked`; vista publica segura con `reason_code`, `rendered_body` y `warning_codes[]`. `null` u omitido para `answered` o `partial_abstention`. No expone `render_storage_ref`. |
| `warnings[]` | Maximo `32` codigos/mensajes seguros unicos, cada texto de `1..240`. |
| `created_at` | date-time. |

Devuelve una union discriminada por `response_outcome`. Una respuesta o abstencion ya materializada se devuelve con `200`; `policy_blocked`, `prompt_injection_blocked`, `unsupported_critical_claim` y `evidence_insufficient` se usan como `ErrorEnvelope` solo cuando no existe un artefacto publico seguro que devolver. La respuesta siempre identifica la version visible mediante `answer_version_ref`.

### GET /v1/answers/{answer_id}/sources

Request: sin body. Query opcional `cursor`/`limit` conforme a la regla global 11; cada pagina conserva cierre del grafo cita-fuente-pasaje.

Response:

| Campo | Regla |
|---|---|
| `answer_id` | `ans_*`. |
| `answer_version_ref` | `av_*`. |
| `evidence_pack_id` | `ep_*` para respuesta con evidencia; `null` para `total_abstention` o `blocked` sin pack. |
| `sources[]` | Maximo `100` fuentes unicas realmente citadas por la pagina; `source_ref` local al pack. Debe ser `[]` si `evidence_pack_id=null`. |
| `passages[]` | Maximo `100` pasajes unicos citados por la pagina; cada `passage_ref` resuelve una fuente incluida en la misma pagina. Debe ser `[]` si `evidence_pack_id=null`. |
| `citations[]` | Maximo `100` citas unicas; `citation_ref=C#` local a `answer_version_ref` y cada cita resuelve su fuente/pasaje en la misma pagina. Debe ser `[]` si `evidence_pack_id=null`. |
| `next_cursor` | Cursor opaco `1..160` o `null`. |
| `has_more` | Boolean coherente con `next_cursor`. |

Devuelve solo fuentes realmente citadas para la version visible. Cualquier `source_ref` (`F#`/`D#`) o `passage_ref` (`F#:P#`/`D#:P#`) queda estrictamente scoped al `answer_version_ref` y al `evidence_pack_id` devueltos por el endpoint; esos refs no son IDs globales. Una abstencion o bloqueo sin evidencia no fabrica `ep_*`, fuentes, pasajes ni citas.

### GET /v1/answers/{answer_id}/trace-summary

Request: sin body.

Response:

| Campo | Regla |
|---|---|
| `answer_id` | `ans_*`. |
| `answer_version_ref` | `av_*`. |
| `trace_id` | `tr_*`. |
| `summary_visibility` | `USER_SUMMARY`. |
| `steps[]` | Maximo `100` codigos seguros unicos de alto nivel, sin prompts ni payloads. |
| `warnings[]` | Maximo `32` codigos seguros unicos. |
| `cost_summary` | Objeto cerrado con `currency`, `is_estimated`, `model_input_tokens`, `model_output_tokens`, `tool_calls` y `estimated_total_cost`, usando enums/tipos no negativos de `CostReport`; no contiene costos por provider ni `CostReport` raw. |

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
| `usage_totals` | Objeto cerrado de cantidades no negativas. Solo permite las keys de `UsageEvent.event_type`: `standard_query`, `complex_query`, `research_credit_used`, `discovery_call`, `fetch`, `ocr_page`, `document_processed`, `model_input_tokens`, `model_output_tokens`, `storage_mb_day`; key omitida equivale a cero. No expone ledger raw. |
| `limits` | Objeto cerrado del `CostBudget` efectivo con exactamente `discovery_calls_max`, `fetch_urls_max`, `ocr_pages_inline_max`, `reformulations_max`, `tool_rounds_max`, `time_budget_ms`, `output_tokens_cap`, `research_credit_cost` y `async_allowed`. |
| `research_credits_balance` | Numero visible o `null`. |

Devuelve agregados del periodo y limites visibles.

### GET /v1/research-credits

Request: sin body. Query opcional `cursor`/`limit` para movimientos conforme a la regla global 11.

Response:

| Campo | Regla |
|---|---|
| `balance` | Numero de creditos disponibles. |
| `currency` | `NONE` o codigo futuro si aplica. |
| `movements[]` | Pagina de maximo `100` movimientos resumidos, objetos cerrados con fecha, tipo y delta; no `UsageEvent` raw. |
| `next_cursor` | Cursor opaco `1..160` o `null`. |
| `has_more` | Boolean coherente con `next_cursor`. |
| `updated_at` | date-time. |

Devuelve balance y movimientos resumidos. Una suscripcion activa sin saldo devuelve `200` con `balance=0`; no usa `research_credit_required`. Si no existe suscripcion activa para la organizacion, devuelve `ErrorEnvelope.error_code=not_found`. El ledger raw de `UsageEvent` queda interno/auditado.

Cada item de `movements[]` es un objeto cerrado con `date` date-time, `type=grant|debit|expiry`, `delta` numerico y `balance_after >= 0`. `grant` exige `delta>0`, `debit` exige `delta<0` y `expiry` exige `delta<=0`/`balance_after=0`; no se exponen `actor_ref`, `usage_event_id`, `trace_id`, `answer_id` ni motivos internos. El orden estable es `(date, movement_id)` ascendente aunque `movement_id` no se publique.

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
| `GET /v1/research-credits` | `validation_error`, `auth_required`, `forbidden`, `tenant_mismatch`, `not_found`, `rate_limited`, `internal_error` |
