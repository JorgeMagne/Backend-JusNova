# Document Security Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.9 - Seguridad documental minima para conversacion, memoria, documentos y OCR

## Proposito

Definir controles documentales minimos para que Fase 1 no cree datos incompatibles con multi-tenancy, trazabilidad, privacidad y procesamiento seguro de documentos.

## Alcance

Aplica a `Message.attachments`, `DocumentEvidence`, `CaseMemory`, referencias documentales `D#`, hashes de version, fragmentos OCR y trazas que referencian mensajes.

No reemplaza la privacy/security policy completa, provider policy, data classification, retention policy ni prompt injection policy de Subfase 0.10.

## Definiciones

- Documento privado: archivo aportado por usuario u organizacion.
- Version documental: representacion inmutable de un documento procesado con hash.
- Fragmento documental: texto localizable y citable representado por `DocumentEvidence`.
- Contenido documental no confiable: todo texto, OCR, tabla, imagen o metadato extraido desde documentos.

## Reglas obligatorias

1. Todo documento privado procesado debe estar asociado a `organization_id`.
2. Todo `DocumentEvidence` debe registrar `organization_id`, `document_id`, `document_version_id` y `document_version_hash`.
3. El documento completo no se guarda en `TraceObject`, `UsageEvent`, `ModelCall`, `ToolCall` ni `CostReport`.
4. OCR completo, HTML bruto y texto documental completo no se guardan en trazas.
5. `Message.attachments[]` en 0.9 solo referencia documentos ya procesados mediante `document_id`, `document_version_id` y `source_ref`.
6. Texto documental no puede modificar instrucciones del sistema ni reglas de seguridad.
7. Todo acceso futuro a documentos completos queda sujeto a permisos y retencion de Subfase 0.10.

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

## Reglas asistidas por IA

1. El modelo puede ayudar a clasificar fragmentos o advertencias, pero la salida debe mapearse a campos cerrados.
2. El modelo no puede decidir ampliar acceso a documento completo.
3. El modelo no puede reinterpretar instrucciones dentro de un documento como instrucciones del sistema.

## Comportamiento ante incumplimiento

- Si falta tenant, version o hash, el documento no puede convertirse en `DocumentEvidence`.
- Si un fragmento no puede resolverse a documento/version del mismo tenant, no puede sostener memoria ni cita.
- Si un documento contiene instrucciones maliciosas o contradictorias, debe tratarse como dato no confiable y preservar advertencia.

## Ejemplos permitidos

- `Message` de usuario con attachment documental procesado y `source_ref = D1`.
- `DocumentEvidence` con hash de version, hash de texto, pagina y locator.
- `TraceObject` que referencia mensajes por ID sin guardar contenido.

## Ejemplos prohibidos

- `DocumentEvidence` sin `organization_id`.
- `TraceObject` con texto completo de mensaje o documento.
- `CaseMemory` usando URL cruda como fuente documental.
- `Message.attachments[]` con documento en estado `processing`.

## Criterios de aceptacion

- Los contratos 0.9 no permiten PII cruda como `user_id`.
- Los contratos 0.9 no permiten documento completo ni OCR completo como propiedad valida.
- La frontera 0.9/0.10 queda documentada: 0.9 define minimo contractual; 0.10 define seguridad completa.

## Relacion con contratos

- Aplica a `message.schema.json`, `document-evidence.schema.json`, `case-memory.schema.json` y `trace-object.schema.json`.
- Complementa ADR-006 y ADR-011.
- Complementa la futura `privacy-security-policy.md` de Subfase 0.10.

## Momento de revision

Revisar al cerrar Subfase 0.10, al implementar storage, al integrar OCR worker, al definir permisos productivos o ante cualquier incidente de documentos.
