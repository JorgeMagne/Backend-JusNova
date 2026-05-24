# Security Checklist Phase 1

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.10 - Seguridad, privacidad y proveedores externos

## Proposito

Checklist minimo para que Fase 1 implemente datos, logs, storage, providers y workers sin romper los contratos de seguridad y privacidad de Fase 0.

## Checklist

- [ ] Toda tabla privada incluye `organization_id` o owner tecnico equivalente.
- [ ] Tests de aislamiento multi-tenant cubren lectura, escritura, borrado y listado.
- [ ] Documentos se guardan en storage privado con rutas no adivinables.
- [ ] URLs firmadas son de corta vida y no exponen nombres humanos de archivo/caso.
- [ ] Uploads validan tipo, tamano, MIME, extension y contenido.
- [ ] Uploads rechazados usan estados/codigos cerrados.
- [ ] Borrado documental cubre original, OCR, fragmentos, embeddings, indices, snapshots privados y derivados.
- [ ] Logs no contienen documentos completos, OCR completo, HTML bruto, prompts completos, mensajes completos ni salidas completas.
- [ ] Summaries, snapshots, embeddings, trazas, indices, queries reformuladas y otros derivados heredan la clase con mayor `sensitivity_rank` de sus entradas.
- [ ] `USER_SUMMARY`, `SUPPORT_VIEW`, `INTERNAL_AUDIT` y `RAW_INCIDENT_ACCESS` estan separados.
- [ ] `SUPPORT_VIEW` no puede ver raw prompts, documentos, mensajes ni model outputs.
- [ ] Todo raw/elevated access crea `RawAccessEvent`.
- [ ] Todo provider externo esta en `provider-registry.yaml`.
- [ ] Todo provider tiene feature flag, kill switch, timeout, retry policy y error mapping.
- [ ] El core legal no importa SDKs directos de proveedores.
- [ ] Toda llamada externa crea `ProviderCallAudit`.
- [ ] Provider calls validan `data_sent_classes` y `data_returned_classes` contra registry.
- [ ] `training_use_allowed` permanece `false`.
- [ ] Prompts delimitan evidencia como datos no instrucciones.
- [ ] Tools ignoran URLs, comandos e instrucciones dentro de evidencia.
- [ ] `PromptInjectionRisk` se registra para documentos, HTML, OCR o snippets sospechosos.
- [ ] Claims criticos no dependen de evidencia con prompt injection bloqueante.
- [ ] Secrets viven en secret manager o variables de entorno seguras.
- [ ] Secret scanning bloquea commits con claves.
- [ ] Validadores revisan que schemas no permitan propiedades raw sensibles fuera de ejemplos invalidos.

## Criterio de salida

Fase 1 no puede marcar un modulo como listo si incumple cualquier item aplicable de este checklist.
