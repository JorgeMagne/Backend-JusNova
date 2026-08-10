# Beta Readiness Gates

**Estado documental:** Accepted
**Fecha:** 2026-05-25
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** ADR-012 - Evaluation And Quality Gates

## Proposito

Este documento define gates minimos para entrar a beta. Fase 0 solo define gates; Fase 1 y fases posteriores implementan pruebas, harness, reportes y evidencia runtime.

Beta no puede ocurrir si falta eval report, si cualquier metrica blocker no cumple target, si una metrica blocker queda `not_measurable` o si hay hallazgos P1/P2 abiertos en cualquier gate blocker o metrica blocker.

Provider Reliability Layer (PRL) maneja caidas basicas de proveedores: timeout, rate limit, provider unavailable, retry seguro, fallback solo mediante `ProviderRoute` activo y error envelope controlado; sin ruta, falla cerrado.

## Gates

| Gate | Condicion | Evidencia requerida | Owner | Blocker | Fase implementacion | Contrato/policy respaldo |
|---|---|---|---|---|---|---|
| Claim Completeness y Citation Auditor bloquean omisiones/citas inexistentes | Assertion juridica critica visible sin `Claim`, cita inexistente, pasaje inexistente o source mismatch no se publica. | Eval run, tests de `ClaimCompletenessValidator` y tests del auditor. | Backend Lead / QA / Legal Reviewer | true | Fase 2 | `answer-contract.schema.json`, `citation.schema.json`, `citation-audit.schema.json`, `citation-policy.md` |
| Claims criticos sin soporte o semanticamente divergentes quedan bloqueados | Claim critico sin soporte valido, omitido del grafo o distinto del oracle aprobado produce abstencion o bloqueo. | Eval de claims criticos, oracle semantico y audit trail. | Backend Lead / Legal Reviewer | true | Fase 2 | `claim.schema.json`, `initial-golden-dataset-spec.md`, `abstention-policy.md` |
| Cost Governor funciona | Budget efectivo limita discovery, OCR, tools y output segun complexity. | Budget tests y traces con cost budget refs. | Backend Lead | true | Fase 1 | `cost-budget.schema.json`, `cost-governor-policy.md` |
| Usage Ledger funciona | Consumo, creditos y plan se registran y concilian. | Usage events y reportes de periodo. | Backend Lead / Product Owner | true | Fase 1 | `usage-event.schema.json`, `commercial-plans-v0.md` |
| Provider Reliability Layer maneja caidas basicas | Timeout, rate limit, provider unavailable y retry seguro devuelven errores controlados; fallback solo usa `ProviderRoute` activo y la ausencia de ruta falla cerrada. | Provider failure tests, fixture local de ruta, registry productivo sin rutas y ErrorEnvelope. | Backend Lead | true | Fase 1 | `provider-policy.md`, `provider-registry.yaml`, `provider-interfaces.md`, `error-envelope.schema.json` |
| TraceObject existe para respuestas finalizadas | Toda respuesta final, abstencion o bloqueo tiene trace minimizado. | TraceObject por answer version final. | Backend Lead | true | Fase 3 | `trace-object.schema.json`, `trace-visibility-policy.md` |
| Documentos se citan por pagina/pasaje | Claims documentales usan `D#:P#` o DocumentEvidence verificable. | Document eval y OCR/page grounding report. | Backend Lead / QA | true | Fase 9 | `document-evidence.schema.json`, `ocr-policy.md` |
| Seguridad tenant basica esta probada | Acceso cross-tenant a entidades privadas falla. | Tenant isolation tests. | Backend Lead / Security Reviewer | true | Fase 1 | `entity-ownership-matrix.md`, `privacy-security-policy.md` |
| Autenticacion productiva reemplaza DevAuth | Requests beta validan token firmado, issuer, audience y expiracion; actor y membership activa resuelven un unico tenant. `DevAuthProvider` no puede habilitarse fuera de local/test. | Auth adapter tests, configuracion de deployment y negativos de token/membership/tenant. | Backend Lead / Security Reviewer | true | Fase 1 pre-beta | `api-draft-v0.md`, `entity-ownership-matrix.md`, `security-checklist-phase-1.md` |
| Eval suite inicial corre | Las metricas canonicas producen reporte versionado. | Eval report con dataset version. | QA / Legal Reviewer | true | Fase 1 | `evaluation-plan-v0.md`, `initial-golden-dataset-spec.md` |
| ProviderCallAudit cubre llamadas externas | Cada intento externo model/tool/retrieval tiene contexto y audit ref propios; la pareja `(logical_call_id, attempt_number)` es unica y el ref singular model/tool apunta al intento terminal. | Provider audit tests, fixtures multi-intento y conciliacion sin usage/cargos duplicados. | Backend Lead | true | Fase 1 | `provider-call-audit.schema.json`, `provider-policy.md`, `model-call.schema.json`, `tool-call.schema.json` |
| RawAccessEvent cubre accesos raw/elevados | Todo acceso raw/elevado genera evento auditable. | Raw access workflow tests. | Security Reviewer | true | Fase 1 | `raw-access-event.schema.json`, `trace-visibility-policy.md` |
| Prompt injection blockers estan activos | Riesgos blocking no sostienen claims criticos y se bloquean o excluyen. | Adversarial evals y PromptInjectionRisk refs. | Security Reviewer / QA | true | Fase 1 | `prompt-injection-risk.schema.json`, `prompt-injection-policy.md` |
| ErrorEnvelope se usa en errores publicos | Todo no-2xx publico usa envelope cerrado y seguro. | API tests. | Backend Lead | true | Fase 1 | `error-envelope.schema.json`, `api-draft-v0.md` |
| Logs/traces no guardan payloads raw sensibles | Prompts, documentos, OCR completo y provider payloads no aparecen en logs/traces. | Static scans y trace fixtures. | Security Reviewer | true | Fase 1 | `privacy-security-policy.md`, `trace-object.schema.json` |

## Politica de fallo

- No hay beta si falta evidencia requerida para cualquier gate blocker.
- No hay beta si `citation_validity_rate < 1.00`.
- No hay beta si `unsupported_critical_claims > 0`.
- No hay beta si `source_tier_correctness < 0.90`.
- No hay beta si `validity_awareness < 0.85`.
- No hay beta si `document_grounding < 0.85`.
- No hay beta si `prompt_injection_resistance < 0.90`.
- No hay beta si una metrica blocker queda `not_measurable`.
- Un waiver no puede cubrir citas inexistentes, claims criticos sin soporte, `source_tier_correctness`, `validity_awareness`, `document_grounding`, autenticacion productiva, tenant isolation, raw data leakage ni prompt injection blocking no mitigado.
