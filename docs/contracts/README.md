# Contract Governance

**Estado documental:** Accepted
**Fecha:** 2026-08-10
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

## Semantica de fixtures embebidos

- `examples` contiene payloads raiz completos que deben validar contra el schema que los declara.
- `x-invalid-examples` contiene payloads raiz completos que deben fallar validacion contra ese schema.
- `x-policy-invalid-examples` contiene payloads raiz completos schema-valid cuya combinacion incumple una policy o una regla custom expresamente citada.
- `x-cross-contract-policy-invalid-examples` contiene wrappers relacionales de prueba, no payloads raiz del schema que los aloja. Cada objeto interno que represente un contrato debe validar por separado contra su schema propio. El runner debe rechazar despues la relacion cross-contract indicada; el wrapper completo no se valida como instancia del schema raiz.
- `x-integrity-examples` contiene proyecciones relacionales deliberadamente parciales y no runtime para documentar integridad entre contratos. Sus claves deben usar sufijo `_projection`; no representan payloads completos, no se validan individualmente contra el schema runtime, no se tratan como `examples` y no se serializan. Un test ejecutable debe expandir cada proyeccion a un payload completo schema-valid antes de comprobar la relacion.
- Todo validador de contratos debe distinguir estas categorias: en `x-cross-contract-policy-invalid-examples` valida por separado cada objeto contractual completo y despues la relacion; en `x-integrity-examples` valida solo la shape/proyeccion documentada y nunca aplica el schema runtime a un fragmento parcial. Ningun wrapper relacional se valida como instancia del schema raiz que lo aloja.

## Contratos aceptados en Subfase 0.4

Estos contratos quedaron aceptados en Subfase 0.4 como base documental del flujo `EvidencePack -> AnswerDraft -> ClaimExtractor -> CitationAuditor -> ClaimVerifier -> FinalAnswer`. Esa subfase no implemento backend funcional y delego `citation-audit.schema.json` a Subfase 0.7; el schema ya forma parte del catalogo aceptado actual descrito mas abajo.

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
- En `LegalSearchResult`, un pasaje extraido con `snapshot_id` ausente o `null` exige `snapshot_unavailable_reason`; el condicional enumera ambos casos de forma explicita y no depende de la evaluacion vacia de `properties` en JSON Schema. La exclusion mutua prohíbe razon solo cuando `snapshot_id` contiene un `snap_*` materializado, no cuando su valor contractual es `null`.
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
- `TraceObject` no expone un campo `classification` porque el payload es cerrado; persistencia conserva una columna DB-only con la mayor sensibilidad de mensajes, claims, evidencia, retrieval y riesgos, minimo `INTERNAL_TRACE_RESTRICTED`, y cualquier `RawAccessEvent` debe usar exactamente esa clase.
- `TraceObject.retrieval_runs[]` usa resumen sanitizado; no embebe `RetrievalRun` operativo completo, URLs crudas de discovery, fuentes abiertas o fuentes rechazadas, mensajes libres de error, warning codes libres ni warnings libres.
- El `RetrievalRun` operativo es tenant-scoped y puede conservar resultados minimizados autorizados, pero sus codigos de error son internos y sus mensajes/warnings son sanitizados, de una linea y acotados; nunca copian excepciones, queries, URLs, snippets, documentos ni payloads raw.
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
- `PromptInjectionRisk` es el contrato compartido para riesgos detectados en documentos, retrieval y trazas. `detected_in_ref` solo acepta refs resolubles o `url_hash` canonico; `source_hash` queda fuera hasta definir bytes y mapping contractuales.
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
- `Citation`, `CitationAudit` y `AbstentionRender` usan los mismos refs locales/canonicos para claims, citas, evidence packs y answers: `cl_*`, `C#`, `ep_*`, `ca_*` y `ans_*`.
- `Message.attachments[].attachment_id` usa `att_*`; `Source.snapshot_id` y `LegalSearchResult.snapshot_id` usan `snap_*`; `AnswerVersion.answer_contract_ref` usa `ans_*:vN`.
- `AnswerVersion.abstention_render_ref` usa `abstention_render_*` y `rendered_answer_snapshot_id` usa `render_snap_*` cuando existen.
- `LegalSearchQuery.query_id`, `RetrievalPlan.retrieval_plan_id`, `LegalSearchResult.result_id` y `LegalEntity.entity_id` usan prefijos canonicos `q_*`, `rp_*`, `lsr_*` y `ent_*`.
- `LegalSearchQuery.entities[].entity_id` y `RetrievalRun.discovery_results[].result_id` son identidades unicas dentro de su contenedor. `RetrievalRun.sources_rejected[]` conserva una sola entrada por `url_hash` canonico y comparte exactamente el enum de razones con `TraceObject.sources_rejected[]`, incluido `unsupported_for_critical_claim`; `access_not_allowed` omite la URL cruda y las demas razones solo pueden conservar una URL publica ya autorizada por `FetchPolicy`. Estas invariantes requieren validacion relacional ademas de `uniqueItems` para detectar objetos distintos con la misma identidad.
- `RetrievalPlan.query_id` resuelve la `LegalSearchQuery` del mismo tenant; `RetrievalRun` conserva ese `query_id` y tenant. Un run produce como maximo un `EvidencePack`, y cuando existe ambas refs (`RetrievalRun.evidence_pack_id` y `EvidencePack.retrieval_run_id`) deben ser reciprocas.
- `RetrievalPlan` exige `organization_id=org_*`; los planes de busqueda son tenant-scoped aunque consulten fuentes publicas.
- `AnswerContract`, `AnswerVersion`, `AbstentionRender`, `EvidencePack`, `Claim` y `Citation` exigen `organization_id=org_*`.
- Todo subobjeto o contrato referenciado dentro de un agregado tenant-scoped debe resolver al mismo `organization_id` que el contenedor. Esto se valida por policy/custom para `AnswerContract -> Claim/Citation`, `AnswerVersion -> TraceObject/AnswerContract/AbstentionRender`, `TraceObject -> ModelCall/ToolCall/RetrievalRun/EvidencePack` y `ProviderCallAudit -> TraceObject/ModelCall/ToolCall/RetrievalRun/UsageEvent/CostReport`.
- `Answer.latest_answer_version_ref` resuelve a la version vigente del mismo answer/tenant y su `response_outcome` coincide con esa version; esta integridad pertenece a persistencia/policy porque `Answer` no tiene JSON Schema standalone en Fase 0.
- `EvidencePack.legal_area[]` es un conjunto; `sources[].source_ref` y `passages[].passage_ref` son identidades locales unicas, y cada pasaje resuelve exactamente una fuente dentro del mismo pack. Las colisiones por ref requieren validador relacional aunque los objetos individuales sean schema-valid. Toda fuente del pack con `validity_status=VIGENCIA_CONFIRMADA|DEROGADA_CONFIRMADA` debe tener al menos un pasaje del mismo pack cuyo `source_ref` coincida; un `Source` aislado schema-valid no basta para confirmar vigencia o derogacion.
- `Claim <-> Citation` es una relacion n:m bidireccional dentro de la misma `AnswerVersion`; todo claim publicado en `AnswerContract` usa `verification_status=passed`, toda cita publicada usa `status=valid`, se asocia al menos a un claim, e IDs/refs y arrays se tratan como conjuntos sin duplicados.
- Cada `CitationAudit.results[]` resuelve una `Citation` de la misma `AnswerVersion`; `(claim_id, citation_ref)` es unico y sus `passage_ref`/`source_ref` deben ser exactamente los de esa cita. Esta integridad es cross-contract y no se sustituye por schema-validity local.
- `Source.metadata` es un objeto publico cerrado y minimizado; no admite claves raw, prompts, OCR, HTML, mensajes ni payloads de provider.
- Todo contrato privado que requiere `organization_id` exige forma canonica `org_*`, alineada con `domain-model.md` y `entity-ownership-matrix.md`.
- Todo `pattern` que expresa un valor completo usa, ademas de sus anchors legibles, la asercion de fin real `(?![\s\S])`. El `$` aislado no se considera cierre suficiente porque algunos motores lo hacen coincidir antes de un terminador de linea; IDs, refs, hashes, fechas y codigos con `CR`, `LF`, `U+2028` o `U+2029` terminal deben fallar. Esta regla equivale a `full-match` sobre el string recibido: no se aplica `trim`, normalizacion ni coercion antes de validar.
- Los textos contractuales declarados como sanitizados y de una linea rechazan controles C0/C1 (`U+0000..U+001F`, `U+007F..U+009F`) y separadores `U+2028`/`U+2029`; no se admite limpiar esos caracteres despues de validar.
- Ningun schema aceptado deja un `string` variable sin `minLength: 1` y `maxLength`, ni un `array` sin `maxItems`. `format` es una anotacion opcional en JSON Schema y nunca sustituye el rechazo estructural de `""`; los enums y `const` son conjuntos finitos por definicion. Cualquier excepcion futura requiere ADR y tests de semantica/consumo de recursos.
- `maxLength` cuenta code points Unicode conforme a JSON Schema, no bytes UTF-8. Los limites de transporte en bytes se aplican adicionalmente antes de parsear o persistir y nunca amplian el limite contractual del campo.
- Perfiles cerrados de longitud: IDs, refs, hashes y valores tecnicos con pattern usan como envolvente `160`; fechas `10`; date-time `40`; dominios `253`; URI `2048`; titulos `500`; labels, warnings, errores y texto seguro de una linea `240`; descripciones estructuradas seguras de memoria `1200`; query/reformulacion `2000`; `Claim.text`, `Passage.text` y fragmentos probatorios `4000`; `Message.content` y contenido de seccion de respuesta `20000`; valores de entidad `1000`. Un pattern o `maxLength` menor sigue gobernando dentro de esa envolvente.
- Perfiles cerrados de coleccion: warnings/notas `32`; taxonomias/data classes/intents `16`; attachments `20`; model/tool calls `200`; retrieval runs `50`; input messages `100`; evidence passages `2000`; las demas colecciones contractuales `500`. Un schema puede declarar un limite menor, nunca omitirlo.
- `make validate-schemas` debe recorrer recursivamente cada schema aceptado y fallar si encuentra un nodo `type: string` variable sin `minLength: 1`/`maxLength` o `type: array` sin `maxItems`. Los tests generados rechazan `""`, `limite + 1` y formatos invalidos, y aceptan el mayor valor que tambien satisfaga pattern/format/item constraints; no se mantienen fixtures gigantes copiados a mano.
- `domain-model.md`, `entity-ownership-matrix.md` y `api-draft-v0.md` gobiernan relaciones conceptuales, ownership y visibilidad de endpoints para Fase 1.

## Schemas minimos esperados

```txt
answer-contract.schema.json
abstention-render.schema.json
answer-version.schema.json
claim.schema.json
citation.schema.json
citation-audit.schema.json
cost-budget.schema.json
cost-report.schema.json
provider-call-audit.schema.json
document-evidence.schema.json
evidence-pack.schema.json
evidence-quality.schema.json
error-envelope.schema.json
legal-entity.schema.json
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
trace-object.schema.json
tool-call.schema.json
usage-event.schema.json
case-memory.schema.json
conversation.schema.json
```

`SourceRegistryEntry` y `SourceSnapshot` permanecen como entidades conceptuales/delegadas en Fase 0; no tienen schema JSON minimo aceptado en este directorio.
