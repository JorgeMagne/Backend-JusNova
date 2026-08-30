# Security Checklist Phase 1

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.10 - Seguridad, privacidad y proveedores externos

## Proposito

Checklist minimo para que Fase 1 implemente datos, logs, storage, providers y workers sin romper los contratos de seguridad y privacidad de Fase 0.

## Checklist

Los items de upload, storage, OCR o provider productivo son obligatorios cuando esa superficie se habilita. Un stub deshabilitado puede cerrar su trabajo de Fase 1 solo si falla cerrado y no procesa datos reales; no habilita beta/mercado para la superficie productiva hasta cumplir todos sus items aplicables.

- [ ] Toda tabla privada incluye `organization_id` o owner tecnico equivalente.
- [ ] Tests de aislamiento multi-tenant cubren lectura, escritura y listado en toda superficie tenant-scoped implementada; cubren borrado lógico solo donde exista una operación aceptada/implementada, sin inventar rutas `DELETE`.
- [ ] Antes de beta, `AuthProvider` productivo valida firma, issuer, audience, expiracion y membership activa; `DevAuthProvider` y headers dev quedan bloqueados fuera de local/test.
- [ ] Documentos se guardan en storage privado con rutas no adivinables.
- [ ] URLs firmadas son de corta vida y no exponen nombres humanos de archivo/caso.
- [ ] Uploads validan tipo, tamano, MIME, extension y contenido.
- [ ] Uploads rechazados usan estados/codigos cerrados.
- [ ] Cuando se habilita upload/storage/document processing, el borrado documental cubre original, OCR, fragmentos, embeddings, indices, snapshots privados y derivados.
- [ ] Logs no contienen documentos completos, OCR completo, HTML bruto, prompts completos, mensajes completos ni salidas completas.
- [ ] Snapshots, embeddings, trazas, indices, queries reformuladas y otros derivados con contenido heredan la clase con mayor `sensitivity_rank` de sus entradas; solo `CitationAudit`, `ModelCall`, `ToolCall` y summaries de retrieval cerrados/metadata-only usan la excepcion `INTERNAL_TRACE_RESTRICTED`, sin degradar su contenedor o recurso referenciado.
- [ ] `USER_SUMMARY`, `SUPPORT_VIEW`, `INTERNAL_AUDIT` y `RAW_INCIDENT_ACCESS` estan separados.
- [ ] `SUPPORT_VIEW` no puede ver raw prompts, documentos, mensajes ni model outputs.
- [ ] Todo raw/elevated access crea `RawAccessEvent`.
- [ ] `TraceObject` y `RetrievalRun` persisten su clasificacion efectiva heredada con minimo `INTERNAL_TRACE_RESTRICTED`; `RawAccessEvent.classification` coincide con el recurso resuelto y nunca la degrada.
- [ ] Todo provider externo esta en `provider-registry.yaml`.
- [ ] `external_call_mode` se resuelve a booleano antes de habilitar cada provider y toda resolucion externa activa `ProviderCallAudit`.
- [ ] Todo provider tiene feature flag, kill switch, timeout, retry policy y error mapping.
- [ ] Antes de habilitar fetch o headless real, el guard de egreso valida URL inicial y redirects antes de red, permite solo HTTPS/443 y DNS globalmente routable, bloquea IPv4/IPv6 internas o reservadas, fija la direccion validada contra DNS rebinding, desactiva proxies/redirects automaticos y aplica limites de redirects, timeout, MIME y bytes descomprimidos.
- [ ] El core legal no importa SDKs directos de proveedores.
- [ ] Toda llamada externa crea `ProviderCallAudit`.
- [ ] Provider calls validan `data_sent_classes` y `data_returned_classes` contra registry.
- [ ] `training_use_allowed` permanece `false`.
- [ ] Prompts delimitan evidencia como datos no instrucciones.
- [ ] Tools ignoran URLs, comandos e instrucciones dentro de evidencia.
- [ ] `PromptInjectionRisk` se registra para documentos, HTML, OCR o snippets sospechosos.
- [ ] Claims criticos no dependen de evidencia con prompt injection bloqueante.
- [ ] Secrets viven en secret manager o variables de entorno seguras.
- [ ] `make scan-secrets` bloquea commits/PRs con claves y usa solo allowlist puntual para placeholders o hashes sintéticos.
- [ ] `make scan-sensitive-surfaces` rechaza `raw_prompt|raw_output|document_text|full_document|ocr_full_text|html_raw|user_message` fuera de `x-invalid-*` en schemas aceptados y rechaza objetos sanitizados abiertos que podrían admitir esas claves; fixtures generados en test incluyen `Source.metadata` no permitida e inyectan sentinels sintéticos únicos en prompt, mensaje, documento, OCR y payload de proveedor, probando que ninguno sobrevive en logs/traces serializados aunque cambie la clave.

## Criterio de salida

Fase 1 no puede marcar un modulo como listo si incumple cualquier item aplicable de este checklist.
