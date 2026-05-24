# Entity Ownership Matrix

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.11 - Modelo de datos conceptual y APIs draft

## Proposito

Esta matriz fija ownership, clasificacion y direccionamiento de acceso raw para las entidades conceptuales de Fase 1. No amplia `raw-access-event.schema.json`; cualquier entidad no mapeada explicitamente usa `No aplica`.

## Reglas

1. Toda entidad privada, confidencial o restringida exige tenant.
2. Las entidades tenant-conditional tienen condicion cerrada.
3. `RawAccess resource_type` solo usa enum aceptado en 0.10 o `No aplica`.
4. Los refs locales `F#`, `D#`, `F#:P#`, `D#:P#` y `C#` requieren contexto padre.

| Entity | ID / identity | Tenant required | Owner field | Classification | RawAccess resource_type | Deletion behavior | Notes |
|---|---|---:|---|---|---|---|---|
| Organization | `org_*` | No, tenant root | Self `org_id` | `SECURITY_AUDIT_RESTRICTED` | No aplica | Tombstone administrativo | Raiz del tenant; no requiere FK recursiva a `organization_id`. |
| User | `usr_*` interno | No | Identity provider interno | `SECURITY_AUDIT_RESTRICTED` | No aplica | Desactivacion o anonimizado | Superficies auditables usan `actor_ref`, no `user_id` crudo. |
| Membership | `mbr_*` | Si | `organization_id` | `SECURITY_AUDIT_RESTRICTED` | No aplica | Revocar o tombstone | Une usuario interno con organizacion. |
| Conversation | `conv_*` | Si | `organization_id` | `USER_MESSAGE_CONFIDENTIAL` | No aplica | Tombstone y retencion de refs auditables | No es raw-access-addressable en 0.11. |
| Message | `msg_*` | Si | `organization_id` | `USER_MESSAGE_CONFIDENTIAL` | `message` | Eliminar/tombstone segun politica de conversacion | Raw/elevated access exige `RawAccessEvent`. |
| Case | `case_*` | Si | `organization_id` | `CASE_MEMORY_CONFIDENTIAL` | No aplica | Tombstone de caso y derivados segun policy | No es raw-access-addressable en 0.11. |
| CaseMemory | `case_id` | Si | `organization_id` | `CASE_MEMORY_CONFIDENTIAL` | `case_memory` | Versionar/tombstone; preservar auditoria | No crea `case_memory_id`. |
| Document | `doc_*` | Si | `organization_id` | `USER_DOCUMENT_CONFIDENTIAL` | `document` | Borrar/tombstone objeto y derivados | Privado por defecto. |
| DocumentVersion | `docv_*` | Si | `organization_id` | `USER_DOCUMENT_CONFIDENTIAL` | No aplica | Inmutable; tombstone por borrado documental | Hash de version requerido en evidencia. |
| DocumentPage | `dpg_*` | Si | `organization_id` | `USER_DOCUMENT_CONFIDENTIAL` | No aplica | Borrar/tombstone con documento | Artefacto de procesamiento. |
| DocumentChunk | `chk_*` | Si | `organization_id` | `USER_DOCUMENT_CONFIDENTIAL` | No aplica | Borrar/tombstone con documento | Artefacto no minimizado; no citable por si solo. |
| DocumentEvidence | `de_*` | Si | `organization_id` | `DOCUMENT_EVIDENCE_CONFIDENTIAL` | `document_evidence` | Borrar/tombstone con version documental | Requiere `prompt_injection_risks[]`. |
| StorageObject | `sto_*` | Si | `organization_id` | `USER_DOCUMENT_CONFIDENTIAL` | `storage_object` | Borrar/tombstone con documento, OCR, indices y snapshots privados derivados | Objeto privado no adivinable; requiere `document_id` y puede tener `document_version_id` cuando existe version. |
| OcrArtifact | `ocr_*` | Si | `organization_id` | `USER_DOCUMENT_CONFIDENTIAL` | `ocr_artifact` | Borrar/tombstone con version documental o pagina origen | OCR completo no se copia a trazas, logs ni provider audit. |
| Embedding | `emb_*` | Si | `organization_id` | `DOCUMENT_EVIDENCE_CONFIDENTIAL` o `CASE_MEMORY_CONFIDENTIAL` | `embedding` | Borrar/tombstone con evidencia, documento o memoria origen | Vector derivado; exige exactamente uno de `document_evidence_id` o `case_id`/memoria. |
| SourceRegistryEntry | `src_*` | Condicional | Global o `organization_id` | `PUBLIC_LEGAL_SOURCE` o heredada | No aplica | Global: versionar; tenant: tombstone | Global solo si publico sin enriquecimiento de usuario. |
| SourceSnapshot | `snap_*` | Condicional | Global o `organization_id` | `PUBLIC_LEGAL_SOURCE` o heredada | No aplica | Retener segun fuente; privados se borran/tombstonean | Tenant si contiene datos user/case/document. |
| RetrievalRun | `rr_*` | Si | `organization_id` | `INTERNAL_TRACE_RESTRICTED` | `retrieval_run` | Retener segun trazabilidad | Puede contener refs sensibles, no raw discovery payload. |
| EvidencePack | `ep_*` | Si | `organization_id` | Hereda mayor sensibilidad de sources/passages; minimo `INTERNAL_TRACE_RESTRICTED` | No aplica | Retener con respuesta/traza | No raw-access-addressable en 0.11. |
| EvidenceSource | `(evidence_pack_id, source_ref)` | Si | `organization_id` del pack | Heredada de fuente | No aplica | Retener con EvidencePack | `source_ref` es local, no global. |
| EvidencePassage | `(evidence_pack_id, passage_ref)` | Si | `organization_id` del pack | Heredada de fuente/pasaje | No aplica | Retener con EvidencePack | `passage_ref` es local, no global. |
| Answer | `ans_*` | Si | `organization_id` | Hereda mayor sensibilidad de mensajes, memoria y evidencia usadas; minimo `USER_MESSAGE_CONFIDENTIAL` | No aplica | Versionar/tombstone visible | Respuesta visible no es raw access; redaccion no baja clasificacion sin regla explicita. |
| AnswerVersion | `av_*` | Si | `organization_id` | Hereda mayor sensibilidad de `Answer`, `EvidencePack`, `CaseMemory` y traza; minimo `INTERNAL_TRACE_RESTRICTED` | No aplica | Inmutable salvo superseded metadata | No raw-access-addressable en 0.11. |
| Claim | `cl_*` | Si | `organization_id` | Hereda mayor sensibilidad de evidencia, memoria o fuente que soporta el claim; minimo `INTERNAL_TRACE_RESTRICTED` | No aplica | Retener con answer version | Claims criticos requieren soporte por policy. |
| Citation | `(answer_version_ref, citation_ref)` | Si | `organization_id` | Hereda mayor sensibilidad de claim y pasaje citado; minimo `INTERNAL_TRACE_RESTRICTED` | No aplica | Retener con answer version | `C#` es local a version. |
| CitationAudit | `citation_audit_id` embebido | Si | Heredado de `TraceObject.organization_id` | `INTERNAL_TRACE_RESTRICTED` | No aplica | Retener embebido con TraceObject | 0.11 lo trata como value object; tabla standalone futura requiere `trace_id` y tenant explicitos. |
| TraceObject | `tr_*` | Si | `organization_id` | `INTERNAL_TRACE_RESTRICTED` | `trace` | Retener por auditoria; no raw content | Solo existe al finalizar respuesta/abstencion/bloqueo. |
| ModelCall | `mc_*` | Si | `organization_id` | `INTERNAL_TRACE_RESTRICTED` | `model_call` | Retener hash/metadata | Externo requiere `ProviderCallAudit`. |
| ToolCall | `tc_*` | Si | `organization_id` | `INTERNAL_TRACE_RESTRICTED` | `tool_call` | Retener hash/metadata | Externo requiere `ProviderCallAudit`. |
| ProviderCallAudit | `pca_*` | Si | `organization_id` | `PROVIDER_OPERATIONAL_METADATA` o `SECURITY_AUDIT_RESTRICTED` | `provider_call` | Retener por auditoria proveedor | No contiene payload completo. |
| RawAccessEvent | `rae_*` | Si | `organization_id` | `SECURITY_AUDIT_RESTRICTED` | No aplica | Retencion de auditoria elevada | Es el log de acceso raw, no recurso raw. |
| PromptInjectionRisk | Ubicacion + contenido de riesgo | Condicional | Global o `organization_id` del documento/retrieval/trace/message | Heredada por `detected_in_ref` | No aplica | Retener con recurso donde se detecto | Tenant si el riesgo aparece en documento, mensaje, retrieval o traza tenant-scoped; global solo para fuente publica sin datos de cliente. |
| UsageEvent | `ue_*` | Si | `organization_id` | `BILLING_USAGE` | `usage_event` | Retener por billing/auditoria | No es evidencia juridica. |
| Plan | `plan_*` | No | Catalogo comercial | `BILLING_USAGE` | No aplica | Versionar catalogo | Plan global neutral; pricing policy separada. |
| Subscription | `sub_*` | Si | `organization_id` | `BILLING_USAGE` | No aplica | Retener/tombstone comercial | Puede existir sin uso. |
| ResearchCredit | `rc_*` | Si | `organization_id` | `BILLING_USAGE` | No aplica | Retener movimientos | Puede existir sin consumo. |
| CostBudget | `cb_*` | No | Policy/catalogo | `BILLING_USAGE` | No aplica | Versionar policy | Budget efectivo se referencia desde traza/usage. |
| CostReport | `cr_*` embebido | Si | Heredado de `TraceObject.organization_id` | `BILLING_USAGE` | No aplica | Retener embebido con TraceObject | No crear tabla aislada sin `trace_id` y tenant; no es presupuesto comercial. |
| EvaluationCase | `eval_case_*` | Condicional | Global o `organization_id` | Hereda de dataset/input | No aplica | Global: versionar; tenant: tombstone/anonymize | Tenant si deriva de usuario/caso/documento. |
| EvaluationRun | `eval_run_*` | Condicional | Global o `organization_id` | Hereda de casos evaluados | No aplica | Retener metricas; tenant: sanitizar | Tenant si cualquier input es tenant-scoped. |
