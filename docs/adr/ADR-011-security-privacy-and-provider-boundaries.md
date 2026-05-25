# ADR-011 - Security, Privacy And Provider Boundaries

**Estado:** Accepted
**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Seguridad, privacidad y limites de proveedores

## Contexto

JusNova manejara documentos legales, hechos de casos, datos de clientes, prompts y trazas. Debe asumirse secreto profesional y multi-tenant desde el inicio.

## Problema

Agregar seguridad despues obligaria a rehacer datos, storage, logs y flujos de proveedores. Documentos y HTML externos tambien pueden contener instrucciones maliciosas.

## Restricciones

- Toda entidad privada requiere ownership.
- Logs no guardan documentos completos.
- Proveedores reciben solo datos necesarios.
- Secrets no se escriben en codigo.
- Contenido externo/documental es dato no confiable.

## Opciones consideradas

1. Seguridad despues de beta.
2. Seguridad solo por convencion de equipo.
3. Seguridad, privacidad y boundaries de proveedores desde Fase 0/Fase 1.

## Decision

El diseno asumira documentos sensibles, secreto profesional y separacion por organizacion desde el inicio. Subfase 0.10 acepta clasificacion de datos, registry de providers, auditoria de llamadas externas, auditoria de acceso raw, contrato de riesgos de prompt injection y checklist minimo de seguridad para Fase 1.

## Justificacion

La seguridad es estructural en datos, storage, logs, providers y prompts. No puede agregarse como capa cosmetica al final.

## Reglas aprobadas

- Tenant isolation por organizacion/usuario.
- Roles minimos.
- Validacion de archivos.
- Object storage con rutas no adivinables y permisos.
- Logs minimizados.
- Provider minimization.
- Registro de salidas a proveedores.
- Prompt injection policy para documentos y fuentes externas.
- Retencion y eliminacion documentadas.
- Data classification con ranks de sensibilidad y reglas por familia de provider.
- Herencia obligatoria de clasificacion: todo payload o artefacto derivado conserva la clase con mayor `sensitivity_rank` de sus entradas salvo regla contractual explicita.
- Provider registry con feature flags, kill switches, allowlists de clases y `training_use_allowed = false`.
- `ProviderCallAudit` para toda llamada externa.
- `RawAccessEvent` para acceso raw/elevado; `support_operator` queda fuera de ese contrato.
- `PromptInjectionRisk` como contrato compartido entre documentos, retrieval y trazas.

## Dependencias posteriores

- Subfase 0.9 crea una `document-security-policy.md` minima para documentos, mensajes y evidencia documental sin cerrar permisos, retencion ni controles completos.
- Subfase 0.10 acepta privacy/security policy, provider policy, prompt injection policy, data classification, provider registry, `ProviderCallAudit`, `RawAccessEvent`, `PromptInjectionRisk` y security checklist.
- Subfase 0.11 acepta matriz de ownership y API draft seguro sin ampliar `RawAccessEvent.resource_type`.
- Fase 1 debe crear estructura de ownership en modelos base.
- Fase 1 debe implementar enforcement runtime, storage privado, secrets/config validation, provider adapters y raw access workflows.
- Fase 11 implementara hardening completo si el roadmap mantiene esa etapa.

## No afirma todavia

- No afirma autenticacion completa en Fase 1.
- No afirma despliegue enterprise.
- No afirma que proveedores externos reciban documentos completos.

## Riesgos

- Fuga entre tenants.
- Logs con datos sensibles.
- Prompt injection indirecta.
- Proveedor externo con datos excesivos.

## Mitigaciones

- Ownership obligatorio.
- Log redaction.
- Data classification.
- Provider registry.
- Tool boundaries.
- Security tests desde Fase 1.

## Criterios de aceptacion

- Subfase 0.9 modela `organization_id`, hashes y refs internas para conversacion, mensajes, memoria y evidencia documental.
- Subfase 0.10 crea checklist de seguridad de Fase 1.
- `data-classification.yaml` y `provider-registry.yaml` quedan aceptados.
- `provider-call-audit.schema.json`, `raw-access-event.schema.json` y `prompt-injection-risk.schema.json` quedan aceptados.
- `entity-ownership-matrix.md` cubre toda entidad conceptual con tenant, clasificacion, raw access aplicable y borrado.
- `api-draft-v0.md` devuelve vistas seguras y no expone memoria, documentos, OCR, trazas ni provider payloads crudos.
- No se permite logging de documentos completos.
- Toda salida a proveedor externo se registra.
- Documentos y HTML se tratan como datos no confiables.

## Momento de revision

Revisar al completar Fase 1, antes de beta y ante cualquier cambio de proveedor, storage, logging, raw access, clasificacion de datos o modelo de tenancy.

## Consecuencias

Fase 1 debe modelar ownership, settings, secrets, logs y provider boundaries desde el primer commit funcional.
