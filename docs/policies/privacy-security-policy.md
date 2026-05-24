# Privacy And Security Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.10 - Seguridad, privacidad y proveedores externos

## Proposito

Definir los limites minimos de privacidad, seguridad, acceso raw, logs, storage, documentos y visibilidad que condicionan Fase 1.

## Alcance

Aplica a conversaciones, mensajes, documentos, OCR, evidencia documental, memoria de caso, trazas, llamadas de modelo, tools, provider audit, usage ledger, storage y workflows.

## Reglas obligatorias

1. Toda entidad privada requiere `organization_id` o owner tecnico equivalente.
2. Todo documento privado tiene tenant, owner tecnico, version y hash de version.
3. Logs, traces, usage, provider audit, model calls y tool calls no guardan documentos completos, OCR completo, HTML bruto, prompts completos, mensajes completos ni salidas completas.
4. Secrets no se hardcodean en codigo, docs ejecutables, configs versionadas ni ejemplos reales.
5. Object storage usa buckets privados, rutas no adivinables, URLs firmadas de corta vida y claves sin nombres humanos de archivo, cliente o caso.
6. Todo archivo subido requiere allowlist de tipo/tamano, MIME sniffing, control de extension, validacion de contenido y estado cerrado de rechazo.
7. Documentos, HTML, snippets, OCR y evidencia externa son datos no confiables.
8. Eliminar documento privado implica borrar o tombstonear objeto original, OCR, fragmentos, embeddings, indices, snapshots privados y derivados.
9. `TraceObject` y `AnswerVersion` pueden conservar hashes, refs y marcas de eliminacion para auditoria; no conservan contenido raw.
10. Todo acceso raw o elevado se registra con `RawAccessEvent`.

## Matriz de visibilidad

| Actor | Puede ver | No puede ver por defecto | Acceso raw |
|---|---|---|---|
| `end_user` | Respuestas propias, fuentes visibles, advertencias, documentos propios, resumen de trazas | Provider audit interno, prompts completos, mensajes de otros usuarios, documentos de otros usuarios | No |
| `organization_admin` | Uso agregado, plan, miembros, configuracion del tenant | Documentos o mensajes de otros usuarios sin permiso futuro explicito | No |
| `support` | `SUPPORT_VIEW` redacted: errores, latencia, refs, estado, plan neutral | Raw prompts, documentos, mensajes, OCR completo, model outputs | Solo como `incident_responder` o `security_auditor` mediante `RawAccessEvent` y `RAW_INCIDENT_ACCESS` |
| `security_auditor` | Auditoria interna con hashes, refs, provider audit y eventos de acceso | Contenido raw sin evento aprobado | Si, mediante `RawAccessEvent` |
| `service_worker` | Minimo necesario por job y allowlist de clasificacion | Navegacion libre por tenant o documentos no relacionados | Solo con `access_role=system_job` cuando el job lo exige |
| `provider` | Payload minimo permitido por `provider_family_rules` | Clases fuera de allowlist, documentos completos salvo `StorageProvider` autorizado | No aplica; recibe payload filtrado |

## Reglas deterministicas

1. `RawAccessEvent.actor_type=support` no puede usar `access_role=support_operator`.
2. `SUPPORT_VIEW` no es raw access y no se registra como `RawAccessEvent`.
3. `RawAccessEvent.approved_by_ref` no puede ser igual a `actor_ref`.
4. `RawAccessEvent.expires_at`, si existe, debe ser posterior a `accessed_at`.
5. `RawAccessEvent.resource_type=document_evidence` debe resolver a `DocumentEvidence` con mismo `organization_id`, `document_evidence_id`, `document_id`, `document_version_id` y `passage_ref`.
6. Si `resource_type != document_evidence`, `document_id`, `document_version_id` y `passage_ref` deben estar ausentes o ser `null`.
7. Cualquier log que contenga `raw_prompt`, `raw_output`, `document_text`, `full_document`, `ocr_full_text`, `html_raw`, `user_message` o mensaje completo falla revision de seguridad salvo que aparezca como ejemplo invalido.
8. `provider_registry.training_use_allowed` y `ProviderCallAudit.training_use_allowed` deben ser `false` en v0.10.

## Comportamiento ante incumplimiento

- Si una accion requiere exponer raw material y no existe `RawAccessEvent` valido, se bloquea.
- Si un proveedor no esta en registry o no permite la clase de datos, la llamada queda `policy_blocked`.
- Si un documento no cumple validacion minima, no entra a OCR, EvidencePack ni memoria.
- Si un borrado documental no puede borrar un derivado, debe registrarse tombstone y razon cerrada.

## Relacion con contratos

- `raw-access-event.schema.json` define auditoria raw/elevada.
- `provider-call-audit.schema.json` define auditoria de proveedores.
- `data-classification.yaml` define clases y reglas por provider family.
- `document-evidence.schema.json`, `message.schema.json`, `trace-object.schema.json` y `usage-event.schema.json` no deben almacenar material raw fuera de su responsabilidad primaria.

## Criterios de aceptacion

- Se define que ve cada actor.
- Se define como se audita acceso raw.
- Se prohiben raw prompts, documentos completos, mensajes completos y OCR completo en logs/traces/provider audit.
- Se define borrado/tombstone de documentos y derivados.
- Fase 1 puede crear tablas y workers con ownership, logging y storage compatibles.

## Momento de revision

Antes de implementar auth, soporte, storage, OCR worker, provider SDKs o retencion productiva.
