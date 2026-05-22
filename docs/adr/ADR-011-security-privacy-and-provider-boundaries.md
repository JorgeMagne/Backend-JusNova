# ADR-011 - Security, Privacy And Provider Boundaries

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
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

El diseno asumira documentos sensibles, secreto profesional y separacion por organizacion desde el inicio.

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

## Dependencias posteriores

- Subfase 0.10 debe crear privacy/security policy, provider policy, prompt injection policy, data classification y security checklist.
- Fase 1 debe crear estructura de ownership en modelos base.
- Fase 11 implementara hardening completo.

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

- Subfase 0.10 crea checklist de seguridad de Fase 1.
- No se permite logging de documentos completos.
- Toda salida a proveedor externo se registra.
- Documentos y HTML se tratan como datos no confiables.

## Momento de revision

Revisar al cerrar Subfase 0.10, al completar Fase 1, antes de beta y ante cualquier cambio de proveedor, storage, logging o modelo de tenancy.

## Consecuencias

Fase 1 debe modelar ownership, settings, secrets, logs y provider boundaries desde el primer commit funcional.

