# Domain Model

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.11 - Modelo de datos conceptual y APIs draft

## Proposito

Este documento define el modelo conceptual minimo que Fase 1 debe respetar al crear migraciones, repositorios y servicios. No es un modelo fisico final de base de datos, pero fija entidades, identidad, cardinalidades, ownership y limites de verdad juridica.

## Alcance

0.11 no implementa runtime, tablas, routers, jobs, workers, OpenAPI formal ni migraciones. Las entidades aqui descritas guian Fase 1 y deben mantenerse coherentes con contratos aceptados de 0.4 a 0.10.

## Reglas rectoras

1. Toda entidad privada o derivada de cliente requiere `organization_id` o owner tecnico equivalente.
2. `CaseMemory` es contexto operacional, no evidencia juridica.
3. `Conversation.title`, nombre de archivo, contenido de mensaje y memoria de caso no son fuentes probatorias ni normativas.
4. `DocumentChunk` es artefacto de procesamiento; solo `DocumentEvidence` o `EvidencePassage` pueden ser unidad citable.
5. `TraceObject` guarda refs/hashes, no mensajes completos, documentos completos, OCR completo ni payloads completos de proveedor.
6. `RawAccessEvent.resource_type` no se amplia en 0.11.
7. Un `run_id` en progreso usa patron `tr_*`, pero no es un `TraceObject` valido hasta que exista `answer_id` y `answer_version_ref`.
8. Refs locales como `F#`, `D#`, `F#:P#`, `D#:P#` y `C#` no son IDs globales.
9. `StorageObject`, `OcrArtifact` y `Embedding` son entidades conceptuales porque 0.10 ya permite auditarlas por `RawAccessEvent`; esto no amplia el enum de raw access.

## Entidades conceptuales

| Grupo | Entidades |
|---|---|
| Identity/Tenancy | `Organization`, `User`, `Membership` |
| Conversation/Case | `Conversation`, `Message`, `Case`, `CaseMemory` |
| Documents | `Document`, `DocumentVersion`, `DocumentPage`, `DocumentChunk`, `DocumentEvidence` |
| Storage/Derived Artifacts | `StorageObject`, `OcrArtifact`, `Embedding` |
| Sources/Retrieval | `SourceRegistryEntry`, `SourceSnapshot`, `RetrievalRun` |
| Evidence/Answer | `EvidencePack`, `EvidenceSource`, `EvidencePassage`, `Answer`, `AnswerVersion`, `Claim`, `Citation`, `CitationAudit` |
| Audit/Security | `TraceObject`, `ModelCall`, `ToolCall`, `ProviderCallAudit`, `RawAccessEvent`, `PromptInjectionRisk` |
| Commercial | `UsageEvent`, `Plan`, `Subscription`, `ResearchCredit`, `CostBudget`, `CostReport` |
| Evaluation | `EvaluationCase`, `EvaluationRun` |

## Identidad global

| Entidad | ID global conceptual | Contrato o fuente |
|---|---|---|
| Organization | `^org_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| User | `^usr_[A-Za-z0-9_-]+$` interno; superficies auditables usan `actor_ref` | 0.11 conceptual, 0.7-0.10 actor refs |
| Membership | `^mbr_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| Conversation | `^conv_[A-Za-z0-9_-]+$` | `conversation.schema.json` |
| Message | `^msg_[A-Za-z0-9_-]+$` | `message.schema.json` |
| Case | `^case_[A-Za-z0-9_-]+$` | 0.11 conceptual, `case-memory.schema.json` usa `case_id` |
| CaseMemory | Identidad por `case_id`, no crea `case_memory_id` | `case-memory.schema.json` |
| Document | `^doc_[A-Za-z0-9_-]+$` | `document-evidence.schema.json` |
| DocumentVersion | `^docv_[A-Za-z0-9_-]+$` | `document-evidence.schema.json` |
| DocumentPage | `^dpg_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| DocumentChunk | `^chk_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| DocumentEvidence | `^de_[A-Za-z0-9_-]+$` | `document-evidence.schema.json` |
| StorageObject | `^sto_[A-Za-z0-9_-]+$` | `raw-access-event.schema.json` resource ref |
| OcrArtifact | `^ocr_[A-Za-z0-9_-]+$` | `raw-access-event.schema.json` resource ref |
| Embedding | `^emb_[A-Za-z0-9_-]+$` | `raw-access-event.schema.json` resource ref |
| SourceRegistryEntry | `^src_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| SourceSnapshot | `^snap_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| RetrievalRun | `^rr_[A-Za-z0-9_-]+$` | `retrieval-run.schema.json` |
| EvidencePack | `^ep_[A-Za-z0-9_-]+$` | `evidence-pack.schema.json` |
| Answer | `^ans_[A-Za-z0-9_-]+$` | `answer-contract.schema.json` |
| AnswerVersion | `^av_[A-Za-z0-9_-]+$` | `answer-version.schema.json` |
| Claim | `^cl_[A-Za-z0-9_-]+$` | `claim.schema.json` |
| CitationAudit | `citation_audit_id` de value object embebido; ejemplos usan `ca_*` | `citation-audit.schema.json` embebido en `TraceObject` |
| TraceObject | `^tr_[A-Za-z0-9_-]+$` | `trace-object.schema.json` |
| ModelCall | `^mc_[A-Za-z0-9_-]+$` | `model-call.schema.json` |
| ToolCall | `^tc_[A-Za-z0-9_-]+$` | `tool-call.schema.json` |
| ProviderCallAudit | `^pca_[A-Za-z0-9_-]+$` | `provider-call-audit.schema.json` |
| RawAccessEvent | `^rae_[A-Za-z0-9_-]+$` | `raw-access-event.schema.json` |
| PromptInjectionRisk | Sin ID global en v0.11; identidad por ubicacion y contenido del riesgo | `prompt-injection-risk.schema.json` |
| UsageEvent | `^ue_[A-Za-z0-9_-]+$` | `usage-event.schema.json` |
| Plan | `^plan_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| Subscription | `^sub_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| ResearchCredit | `^rc_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| CostBudget | `^cb_[A-Za-z0-9_-]+$` | `cost-budget.schema.json` |
| CostReport | `^cr_[A-Za-z0-9_-]+$` como value object embebido | `cost-report.schema.json` embebido en `TraceObject` |
| EvaluationCase | `^eval_case_[A-Za-z0-9_-]+$` | 0.11 conceptual |
| EvaluationRun | `^eval_run_[A-Za-z0-9_-]+$` | 0.11 conceptual |

## Refs locales

| Entidad local | Identidad conceptual | Patron | Regla |
|---|---|---|---|
| EvidenceSource | `(evidence_pack_id, source_ref)` | `^F[0-9]+$` o `^D[0-9]+$` | `source_ref` no es clave global. |
| EvidencePassage | `(evidence_pack_id, passage_ref)` | `^F[0-9]+:P[0-9]+$` o `^D[0-9]+:P[0-9]+$` | `passage_ref` no es clave global. |
| Citation | `(answer_version_ref, citation_ref)` | `^C[0-9]+$` | `citation_ref` no es clave global. |

## Cardinalidades requeridas

| Relacion | Cardinalidad | Nota |
|---|---|---|
| Organization -> Membership | `0..n` | Una organizacion puede crearse antes de invitar miembros adicionales. |
| User -> Membership | `0..n` | Un usuario tecnico puede existir antes de membresia activa. |
| Membership -> Organization | `n..1` | Toda membresia pertenece a una organizacion. |
| Membership -> User | `n..1` | Toda membresia pertenece a un usuario interno. |
| Organization -> Conversation | `0..n` | Permite tenant sin conversaciones. |
| Organization -> Case | `0..n` | Permite tenant sin casos. |
| Conversation -> Message | `0..n` | `POST /v1/conversations` crea shell sin mensaje inicial obligatorio. |
| Message -> Conversation | `n..1` | Todo mensaje pertenece a una conversacion. |
| Case -> Conversation | `0..n` | Un caso puede agrupar varias conversaciones. |
| Conversation -> Case | `0..1` | Una conversacion puede no estar asociada a caso. |
| Case -> Document | `0..n` | Un caso puede empezar sin documentos. |
| Document -> Case | `0..1` | Un documento puede existir antes de asociarse a caso. |
| Case -> CaseMemory | `0..1` | La memoria puede crearse tras el caso. |
| CaseMemory -> Case | `1..1` | No existe memoria sin caso. |
| Document -> DocumentVersion | `0..n` | `POST /v1/documents` puede crear shell/upload intent sin version. |
| DocumentVersion -> Document | `n..1` | Toda version pertenece a un documento. |
| DocumentVersion -> DocumentPage | `0..n` | Antes de procesar no hay paginas extraidas. |
| DocumentPage -> DocumentVersion | `n..1` | Toda pagina pertenece a una version. |
| DocumentPage -> DocumentChunk | `0..n` | Una pagina puede no tener chunks. |
| DocumentChunk -> DocumentPage | `n..1` | Todo chunk pertenece a una pagina. |
| DocumentChunk -> DocumentEvidence | `0..n` | Solo algunos chunks generan evidencia citable. |
| DocumentEvidence -> DocumentChunk | `0..1` | Puede derivarse de chunk o de extraccion directa. |
| DocumentEvidence -> PromptInjectionRisk | `0..n` | `[]` significa evaluado sin riesgos. |
| DocumentVersion -> StorageObject | `0..n` | Version documental puede tener objeto original y artefactos privados derivados. |
| StorageObject -> DocumentVersion | `0..1` | Storage publico de fuentes se modela por `SourceSnapshot`, no por este recurso raw privado. |
| DocumentVersion -> OcrArtifact | `0..n` | Version puede no estar procesada o tener OCR por pagina/fragmento. |
| DocumentPage -> OcrArtifact | `0..n` | OCR puede estar asociado a pagina especifica. |
| OcrArtifact -> DocumentVersion | `n..1` | Todo OCR privado deriva de una version documental tenant-scoped. |
| OcrArtifact -> DocumentPage | `0..1` | OCR multipagina o fragmentario puede no ser pagina unica. |
| DocumentEvidence -> Embedding | `0..n` | Embeddings derivados de evidencia documental heredan su clasificacion. |
| CaseMemory -> Embedding | `0..n` | Embeddings derivados de memoria heredan `CASE_MEMORY_CONFIDENTIAL`. |
| Embedding -> DocumentEvidence | `0..1` | Embedding documental apunta a un fragmento citable cuando aplique. |
| Embedding -> CaseMemory | `0..1` | Embedding de memoria apunta al caso/memoria cuando aplique. |
| SourceRegistryEntry -> SourceSnapshot | `0..n` | Una fuente puede registrarse antes de snapshots. |
| SourceSnapshot -> SourceRegistryEntry | `0..1` | Snapshots privados pueden existir fuera del registry publico global. |
| RetrievalRun -> EvidencePack | `0..n` | Runs fallidos pueden no producir pack. |
| EvidencePack -> RetrievalRun | `0..1` | Packs manuales o documentales pueden no venir de retrieval vivo. |
| RetrievalRun -> PromptInjectionRisk | `0..n` | Riesgos detectados durante discovery/fetch/extraction. |
| EvidencePack -> EvidenceSource | `1..n` cuando se usa para respuesta; draft packs pueden ser `0..n` | No emitir respuesta sustantiva con pack vacio. |
| EvidenceSource -> EvidencePassage | `1..n` cuando es citable; source shells pueden ser `0..n` | Una fuente citada requiere pasajes. |
| Answer -> AnswerVersion | `0..n` | Permite answer shell antes de version publicada. |
| AnswerVersion -> Answer | `n..1` | Toda version pertenece a una respuesta. |
| AnswerVersion -> TraceObject | `1..1` al finalizar | Mientras el run esta en progreso no hay `TraceObject` valido. |
| TraceObject -> AnswerVersion | `1..1` | Trace final requiere `answer_version_ref`. |
| AnswerVersion -> Claim | `0..n` | Abstenciones/bloqueos pueden no publicar claims. |
| Claim -> Citation | `0..n` | Claims no soportados o no criticos pueden no tener citas; claims criticos soportados requieren cita valida por policy. |
| Citation -> Claim | `n..1` cuando soporta claim | Una cita de soporte apunta a claim. |
| TraceObject -> CitationAudit | `1..1` embebido | `citation_audit` es value object dentro de `TraceObject`; tabla standalone futura requiere `organization_id` y `trace_id`. |
| CitationAudit -> AnswerVersion | Por el `answer_version_ref` del `TraceObject` contenedor | 0.11 no lo persiste como tabla standalone. |
| TraceObject -> ModelCall | `0..n` | Algunas respuestas pueden no requerir llamada de modelo. |
| TraceObject -> ToolCall | `0..n` | Algunas respuestas pueden no usar tools. |
| TraceObject -> RetrievalRun | `0..n` | Abstenciones o respuestas documentales pueden no usar retrieval vivo. |
| TraceObject -> ProviderCallAudit | `0..n` mediante `model_call_id`, `tool_call_id` o `retrieval_run_id` | Auditoria externa se enlaza por refs propios de `ProviderCallAudit`. |
| ProviderCallAudit -> ModelCall | `0..1` | Auditoria de llamada de modelo externa si aplica. |
| ProviderCallAudit -> ToolCall | `0..1` | Auditoria de tool/provider externo si aplica. |
| ProviderCallAudit -> RetrievalRun | `0..1` | Auditoria de discovery/fetch/extraction/OCR/ranking externo si aplica. |
| ProviderCallAudit -> UsageEvent | `0..1` | Conciliacion posterior de uso/costo cuando aplique. |
| ProviderCallAudit -> CostReport | `0..1` | Conciliacion posterior con costo observado cuando aplique. |
| Plan -> Subscription | `0..n` | Plan puede existir antes de suscripciones. |
| Subscription -> Plan | `n..1` | Toda suscripcion activa referencia plan. |
| Organization -> Subscription | `0..n` | Tenant puede existir antes de suscripcion comercial. |
| Subscription -> UsageEvent | `0..n` | Suscripcion puede no tener uso aun. |
| UsageEvent -> Organization | `n..1` | Todo evento de uso pertenece a organizacion. |
| UsageEvent -> Subscription | `0..1` | Eventos prebilling/migracion pueden no estar conciliados aun. |
| ResearchCredit -> UsageEvent | `0..n` | Credito puede existir sin consumo. |
| UsageEvent -> ResearchCredit | `0..1` | Solo eventos de credito consumido referencian credito. |
| CostBudget -> TraceObject | `0..n` | Budget puede existir antes de trazas. |
| TraceObject -> CostReport | `1..1` embebido al finalizar | `cost` es value object dentro de `TraceObject`. |
| CostReport -> TraceObject | Embebido en exactamente un `TraceObject` final | Tabla standalone futura requiere `organization_id` y `trace_id`; no inferir tabla aislada desde 0.11. |
| EvaluationCase -> EvaluationRun | `0..n` | Caso eval puede no ejecutarse aun. |
| EvaluationRun -> EvaluationCase | `n..1` para single-case o `n..m` para suite documentada | Suite runs deben registrar lista de casos. |

## Run lifecycle

El API puede reservar un `run_id` con forma `tr_*` al iniciar una ejecucion. Ese valor no debe persistirse como `TraceObject` hasta que exista respuesta final, abstencion o bloqueo con `answer_id` y `answer_version_ref`.

Si la ejecucion falla antes de crear respuesta/version, el `run_id` queda como referencia operacional de streaming y diagnostico futuro; no satisface `trace-object.schema.json`.

## Condiciones tenant-conditional

| Entidad | Condicion global | Condicion tenant-scoped |
|---|---|---|
| SourceRegistryEntry | Fuente juridica publica sin enriquecimiento de usuario. | Fuente privada, registry de organizacion, documento de usuario, contexto de caso o anotacion derivada de usuario. |
| SourceSnapshot | Snapshot de fuente publica sin query/caso/documento de usuario. | Snapshot privado, user-derived, case-specific o restricted. |
| PromptInjectionRisk | Riesgo detectado en fuente publica sin datos de cliente. | Riesgo detectado en documento, mensaje, retrieval, traza o fuente derivada de usuario/caso/documento. |
| EvaluationCase | Sintetico, publico o anonimizado no reversible. | Derivado de mensaje, caso, documento, traza, respuesta, provider output o incidente de soporte. |
| EvaluationRun | Ejecuta solo casos globales sinteticos/publicos. | Cualquier input es tenant-scoped o user-derived. |

Todo artefacto derivado hereda la mayor sensibilidad (`sensitivity_rank`) de sus entradas.

## Artefactos raw addressable

- `StorageObject` representa objeto privado addressable por `RawAccessEvent.storage_object`; objetos publicos de fuentes se modelan por `SourceSnapshot`.
- `OcrArtifact` representa OCR privado por version, pagina o fragmento; el OCR completo no se copia a trazas, logs, errores ni provider audit.
- `Embedding` debe derivar de una fuente contractual unica en 0.11: `DocumentEvidence` o `CaseMemory`; si necesita ambas, se crean artefactos separados.
- `CitationAudit` y `CostReport` permanecen como value objects embebidos en `TraceObject` salvo que Fase 1 los materialice con `organization_id`, `trace_id` y constraint de unicidad contra el `TraceObject` owner.

## Reglas de verdad juridica

- Memoria de caso orienta contexto; no sostiene `norma`, `vigencia`, `plazo`, `competencia`, `jurisprudencia` ni claims criticos.
- Fuentes finales salen de Evidence Packs y pasajes citables.
- Las citas decorativas siguen prohibidas.
- Uso, billing, costos, provider audit y raw access no son evidencia juridica.
