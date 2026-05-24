# Document Security Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.9 - Seguridad documental minima para conversacion, memoria, documentos y OCR; Subfase 0.10 - Security, Privacy and Provider Boundaries
**Enmiendas:** Subfase 0.10 - document evidence identity, data classification, raw access and prompt injection controls

## Proposito

Definir controles documentales minimos para que Fase 1 no cree datos incompatibles con multi-tenancy, trazabilidad, privacidad y procesamiento seguro de documentos.

## Alcance

Aplica a `Message.attachments`, `DocumentEvidence`, `CaseMemory`, referencias documentales `D#`, hashes de version, fragmentos OCR y trazas que referencian mensajes.

Se complementa con `privacy-security-policy.md`, `provider-policy.md`, `data-classification.yaml`, `raw-access-event.schema.json`, `provider-call-audit.schema.json` y `prompt-injection-policy.md` de Subfase 0.10.

## Definiciones

- Documento privado: archivo aportado por usuario u organizacion.
- Version documental: representacion inmutable de un documento procesado con hash.
- Fragmento documental: texto localizable y citable representado por `DocumentEvidence`.
- Contenido documental no confiable: todo texto, OCR, tabla, imagen o metadato extraido desde documentos.

## Reglas obligatorias

1. Todo documento privado procesado debe estar asociado a `organization_id`.
2. Todo `DocumentEvidence` debe registrar `organization_id`, `document_id`, `document_version_id` y `document_version_hash`.
3. Todo `DocumentEvidence` debe registrar `document_evidence_id` resoluble con patron `de_*`.
4. El documento completo no se guarda en `TraceObject`, `UsageEvent`, `ModelCall`, `ToolCall`, `ProviderCallAudit` ni `CostReport`.
5. OCR completo, HTML bruto y texto documental completo no se guardan en trazas, logs ni auditorias de provider.
6. `Message.attachments[]` en 0.9 solo referencia documentos ya procesados mediante `document_id`, `document_version_id` y `source_ref`.
7. Texto documental no puede modificar instrucciones del sistema ni reglas de seguridad.
8. Todo acceso a documentos completos o fragmentos crudos queda sujeto a `RawAccessEvent`.
9. Documento, OCR, HTML y snippets son datos no confiables y deben tratarse bajo `prompt-injection-policy.md`.

## Reglas deterministicas

1. `DocumentEvidence.source_ref` debe usar `D#`.
2. `DocumentEvidence.passage_ref` debe usar `D#:P#`.
3. `DocumentEvidence.passage_ref` debe pertenecer al mismo `source_ref`.
4. `CaseMemory.facts_supported_by_documents[].passage_refs[]` debe resolver a `DocumentEvidence` del mismo `organization_id`, `document_id` y `document_version_id`.
5. `TraceObject.input_message_ids[]` y `TraceObject.output_message_id` solo guardan IDs de mensaje, no contenido.
6. Ningun schema 0.9 debe permitir propiedades validas llamadas `user_id`, `document_text`, `full_document`, `ocr_full_text`, `html_raw`, `raw_prompt` o `raw_output`.
7. `Message.content_hash`, `document_version_hash` y `text_hash` usan `sha256:<64 hex>`.
8. `Message.content_hash` debe ser el hash `sha256` de los bytes UTF-8 exactos persistidos en `Message.content`.
9. `document_version_hash` debe coincidir con la version documental resuelta.
10. Cada `Message.attachments[]` debe resolver a documento/version/source_ref del mismo `organization_id` del `Message`; no se permite adjuntar por referencia documentos de otro tenant.
11. `Message.attachments[].attachment_id` debe ser unico dentro del mensaje; un identificador no puede apuntar a dos documentos, versiones o `source_ref` distintos.
12. `DocumentEvidence.prompt_injection_risks[]` es obligatorio, usa `prompt-injection-risk.schema.json` y no texto libre; `[]` significa evaluado sin riesgos detectados.
13. Si un fragmento documental se envia a un provider externo, esa llamada debe quedar en `ProviderCallAudit` con clases de datos permitidas por `data-classification.yaml` y `provider-registry.yaml`.

## Reglas asistidas por IA

1. El modelo puede ayudar a clasificar fragmentos o advertencias, pero la salida debe mapearse a campos cerrados.
2. El modelo no puede decidir ampliar acceso a documento completo.
3. El modelo no puede reinterpretar instrucciones dentro de un documento como instrucciones del sistema.

## Comportamiento ante incumplimiento

- Si falta tenant, version o hash, el documento no puede convertirse en `DocumentEvidence`.
- Si un fragmento no puede resolverse a documento/version del mismo tenant, no puede sostener memoria ni cita.
- Si un documento contiene instrucciones maliciosas o contradictorias, debe tratarse como dato no confiable y preservar advertencia.
- Si un documento o fragmento contiene riesgo de prompt injection bloqueante, no puede sostener un claim critico sin exclusion auditada o bloqueo.

## Ejemplos permitidos

- `Message` de usuario con attachment documental procesado y `source_ref = D1`.
- `DocumentEvidence` con `document_evidence_id`, hash de version, hash de texto, pagina y locator.
- `TraceObject` que referencia mensajes por ID sin guardar contenido.

## Ejemplos prohibidos

- `DocumentEvidence` sin `organization_id`.
- `DocumentEvidence` sin `document_evidence_id`.
- `TraceObject` con texto completo de mensaje o documento.
- `CaseMemory` usando URL cruda como fuente documental.
- `Message.attachments[]` con documento en estado `processing`.
- `ProviderCallAudit` guardando documento completo o OCR completo como payload.

## Criterios de aceptacion

- Los contratos 0.9 no permiten PII cruda como `user_id`.
- Los contratos 0.9 no permiten documento completo ni OCR completo como propiedad valida.
- La frontera 0.9/0.10 queda documentada: 0.9 define minimo contractual; 0.10 define clasificacion, provider audit, raw access y defensa prompt injection.

## Relacion con contratos

- Aplica a `message.schema.json`, `document-evidence.schema.json`, `case-memory.schema.json` y `trace-object.schema.json`.
- Complementa ADR-006 y ADR-011.
- Complementa `privacy-security-policy.md`, `prompt-injection-policy.md`, `raw-access-event.schema.json` y `provider-call-audit.schema.json` de Subfase 0.10.

## Momento de revision

Revisar al cerrar Subfase 0.10, al implementar storage, al integrar OCR worker, al definir permisos productivos o ante cualquier incidente de documentos.
