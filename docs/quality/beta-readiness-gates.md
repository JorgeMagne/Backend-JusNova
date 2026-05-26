# Beta Readiness Gates

**Estado documental:** Accepted
**Fecha:** 2026-05-25
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** ADR-012 - Evaluation And Quality Gates

## Proposito

Este documento define gates minimos para entrar a beta. Fase 0 solo define gates; Fase 1 y fases posteriores implementan pruebas, harness, reportes y evidencia runtime.

Beta no puede ocurrir si falta eval report, si cualquier metrica blocker no cumple target, si una metrica blocker queda `not_measurable` o si hay hallazgos P1/P2 abiertos en cualquier gate blocker o metrica blocker.

Provider Reliability Layer (PRL) maneja caidas basicas de proveedores: timeout, rate limit, provider unavailable, retry seguro, fallback permitido y error envelope controlado.

## Gates

| Gate | Condicion | Evidencia requerida | Owner | Blocker | Fase implementacion | Contrato/policy respaldo |
|---|---|---|---|---|---|---|
| Citation Auditor bloquea citas inexistentes | Cita inexistente, pasaje inexistente o source mismatch no se publica. | Eval run y tests del auditor. | Backend Lead / QA | true | Fase 2 | `citation.schema.json`, `citation-audit.schema.json`, `citation-policy.md` |
| Claims criticos sin soporte quedan bloqueados | Claim critico sin soporte valido produce abstencion o bloqueo. | Eval de claims criticos y audit trail. | Backend Lead / Legal Reviewer | true | Fase 2 | `claim.schema.json`, `abstention-policy.md` |
| Cost Governor funciona | Budget efectivo limita discovery, OCR, tools y output segun complexity. | Budget tests y traces con cost budget refs. | Backend Lead | true | Fase 1 | `cost-budget.schema.json`, `cost-governor-policy.md` |
| Usage Ledger funciona | Consumo, creditos y plan se registran y concilian. | Usage events y reportes de periodo. | Backend Lead / Product Owner | true | Fase 1 | `usage-event.schema.json`, `commercial-plans-v0.md` |
| Provider Reliability Layer maneja caidas basicas | Timeout, rate limit, provider unavailable y retry seguro devuelven errores controlados. | Provider failure tests y ErrorEnvelope. | Backend Lead | true | Fase 1 | `provider-policy.md`, `error-envelope.schema.json` |
| TraceObject existe para respuestas finalizadas | Toda respuesta final, abstencion o bloqueo tiene trace minimizado. | TraceObject por answer version final. | Backend Lead | true | Fase 3 | `trace-object.schema.json`, `trace-visibility-policy.md` |
| Documentos se citan por pagina/pasaje | Claims documentales usan `D#:P#` o DocumentEvidence verificable. | Document eval y OCR/page grounding report. | Backend Lead / QA | true | Fase 9 | `document-evidence.schema.json`, `ocr-policy.md` |
| Seguridad tenant basica esta probada | Acceso cross-tenant a entidades privadas falla. | Tenant isolation tests. | Backend Lead / Security Reviewer | true | Fase 1 | `entity-ownership-matrix.md`, `privacy-security-policy.md` |
| Eval suite inicial corre | Las metricas canonicas producen reporte versionado. | Eval report con dataset version. | QA / Legal Reviewer | true | Fase 1 | `evaluation-plan-v0.md`, `initial-golden-dataset-spec.md` |
| ProviderCallAudit cubre llamadas externas | Toda llamada externa model/tool/retrieval tiene audit ref. | Provider audit tests y fixtures. | Backend Lead | true | Fase 1 | `provider-call-audit.schema.json`, `model-call.schema.json`, `tool-call.schema.json` |
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
- Un waiver no puede cubrir citas inexistentes, claims criticos sin soporte, `source_tier_correctness`, `validity_awareness`, `document_grounding`, tenant isolation, raw data leakage ni prompt injection blocking no mitigado.
