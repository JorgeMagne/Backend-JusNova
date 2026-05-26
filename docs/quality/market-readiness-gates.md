# Market Readiness Gates

**Estado documental:** Accepted
**Fecha:** 2026-05-25
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** ADR-012 - Evaluation And Quality Gates

## Proposito

Este documento define gates minimos para salida a mercado. Mercado requiere que beta gates ya esten cumplidos, que no existan P1/P2 abiertos de correccion juridica, seguridad o privacidad, y que exista revision humana legal.

## Gates

| Gate | Condicion | Evidencia requerida | Owner | Blocker | Fase implementacion | Contrato/policy respaldo |
|---|---|---|---|---|---|---|
| Respuestas criticas tienen evidencia valida | Toda respuesta critica tiene EvidencePack, citations y claims soportados. | Market eval report. | Legal Reviewer / QA | true | Pre-market | `answer-contract.schema.json`, `evidence-pack.schema.json` |
| Claims criticos sin soporte se bloquean | Cero unsupported critical claims en regression suite. | Zero unsupported critical claims report. | Legal Reviewer | true | Pre-market | `claim.schema.json`, `citation-policy.md` |
| Vigencia se maneja con politica estricta | No se afirma vigencia sin `VIGENCIA_CONFIRMADA`. | Validity eval. | Legal Reviewer / Backend Lead | true | Pre-market | `validity-policy.md`, `validity-statuses.yaml` |
| Conflictos se identifican | Conflictos esperados se etiquetan y no se ocultan en respuestas categoricas. | Conflict eval. | Legal Reviewer / QA | true | Pre-market | `conflict-policy.md`, `uncertainty-policy.md` |
| Fuentes secundarias se etiquetan | TIER2/TIER3 se muestran con warnings, no sostienen claims criticos como soporte primario no permitido, y TIER3 no es soporte unico de norma/vigencia critica. | Source tier eval. | QA | true | Pre-market | `source-policy.md`, `source-tiers.yaml`, `citation-policy.md` |
| Planes desde 400 Bs funcionan | Plan base, budgets y limites comerciales operan de punta a punta. | Commercial tests. | Product Owner / Backend Lead | true | Pre-market | `commercial-plans-v0.md`, `budgets.yaml` |
| Uso del plan es transparente | Usuario y soporte ven consumo agregado sin exponer ledger raw. | Usage/accounting tests. | Product Owner / Backend Lead | true | Pre-market | `usage-event.schema.json`, `api-draft-v0.md` |
| Seguridad y backups estan probados | Restore, secrets, storage permissions y logs redacted pasan pruebas. | Restore test y security test. | Security Reviewer | true | Pre-market | `privacy-security-policy.md`, `document-security-policy.md` |
| Proceso de soporte e incidentes existe | Soporte, escalacion, raw access e incident handling estan documentados. | Ops docs y runbook aprobado. | Product Owner / Security Reviewer | true | Pre-market | `raw-access-event.schema.json`, `trace-visibility-policy.md` |
| Retencion/borrado documental esta probado | Borrado, tombstone y derivados documentales respetan policy. | Deletion/tombstone tests. | Backend Lead / Security Reviewer | true | Pre-market | `document-security-policy.md`, `privacy-security-policy.md` |
| Tenant isolation tiene pruebas regresivas | Cross-tenant access falla en API, storage, traces y usage. | Regression suite de tenancy. | Security Reviewer | true | Pre-market | `entity-ownership-matrix.md`, `api-draft-v0.md` |
| Prompt injection suite alcanza target | `prompt_injection_resistance >= 0.90` y cero blockers criticos no mitigados. | Adversarial suite. | Security Reviewer / QA | true | Pre-market | `prompt-injection-policy.md`, `prompt-injection-risk.schema.json` |
| Proceso de revision legal/humana definido para casos criticos | Casos criticos tienen sign-off humano y `docs/handoff/review-responsibilities.md` esta aceptado antes de market. | Signed review artifact y review responsibilities Accepted. | Legal Reviewer / Product Owner | true | Pre-market | `initial-golden-dataset-spec.md`, `docs/handoff/review-responsibilities.md` |
| Suite de regresion corre antes de releases | Todo cambio de retrieval, OCR, prompts, providers, citation o validity dispara eval. | Release report. | QA / Backend Lead | true | Pre-market | `evaluation-plan-v0.md`, `ADR-012-evaluation-and-quality-gates.md` |

## Politica de fallo

- Mercado requiere beta gates cumplidos.
- Mercado requiere revision juridica humana para casos criticos.
- Mercado no puede salir si `docs/handoff/review-responsibilities.md` sigue `Draft`; ese handoff debe pasar a `Accepted` o ser reemplazado por un proceso operativo aceptado antes de market.
- Mercado requiere proceso de soporte e incidentes, no solo tests tecnicos.
- Mercado queda bloqueado por cualquier P1/P2 abierto en legal correctness, seguridad, tenancy, document grounding, prompt injection, privacy o backups.
- No se puede hacer waiver de cita inexistente, claim critico sin soporte, fuga tenant, fuga raw data ni prompt injection blocking no mitigado.
