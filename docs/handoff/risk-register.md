# Risk Register

**Estado documental:** Draft  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Registro vivo de riesgos de Fase 0

## Escala

| Campo | Valores |
|---|---|
| Probabilidad | Low, Medium, High |
| Impacto | Low, Medium, High, Critical |
| Estado | Open, Mitigating, Accepted, Closed |

## Riesgos iniciales

| ID | Riesgo | Probabilidad | Impacto | Estado | Mitigacion | Responsable |
|---|---|---|---|---|---|---|
| R-001 | Sobrediseno documental sin avance ejecutable hacia Fase 1. | Medium | High | Open | Timebox por subfase, checklist de cierre y handoff concreto a Sprint 1. | Codex / JusNova Chief Backend Architect |
| R-002 | Contratos demasiado abstractos para implementacion. | Medium | High | Open | Exigir JSON Schemas, ejemplos validos/invalidos y criterios de aceptacion. | Codex / JusNova Chief Backend Architect |
| R-003 | Contradicciones entre ADRs, policies y contracts. | Medium | High | Open | Revision cruzada en Subfase 0.14 y decision pack final. | Codex / JusNova Chief Backend Architect |
| R-004 | Dependencia accidental de proveedor externo en documentos de Fase 0. | Medium | Critical | Open | Provider interfaces obligatorias y ADR-011 con boundaries. | Codex / JusNova Chief Backend Architect |
| R-005 | Politica de vigencia insuficiente o ambigua. | Medium | Critical | Open | Estados cerrados de vigencia, frases prohibidas y abstention policy. | Codex / JusNova Chief Backend Architect |
| R-006 | Seguridad tratada tarde. | Medium | Critical | Mitigating | Subfase 0.9 crea seguridad documental minima; security/privacy policy, data classification y ownership matrix completa quedan para Subfase 0.10. | Codex / JusNova Chief Backend Architect |
| R-007 | Trazabilidad insuficiente para auditar respuestas juridicas. | Medium | Critical | Mitigating | Subfase 0.7 acepta TraceObject, model/tool calls, CitationAudit, CostReport y AnswerVersion; implementacion queda para fases posteriores. | Codex / JusNova Chief Backend Architect |
| R-008 | Cost Governor no refleja plan base de 400 Bs. | Medium | High | Mitigating | Subfase 0.8 acepta `budgets.yaml`, `cost-budget.schema.json`, `usage-event.schema.json`, `commercial-plans-v0.md` y `cost-governor-policy.md`; runtime queda para Fase 1. | Codex / JusNova Chief Backend Architect |
| R-009 | Fuente oficial clave caida durante busqueda viva. | High | High | Open | `source-policy.md`, `legal-search-policy.md`, warnings de fallback y PRL en fases posteriores. | Codex / JusNova Chief Backend Architect |
| R-010 | Vigencia no confirmada presentada como vigente. | Medium | Critical | Mitigating | `validity-policy.md`, estados cerrados de vigencia y frases prohibidas. | Codex / JusNova Chief Backend Architect |
| R-011 | Conflicto irresoluble ocultado en respuesta categorica. | Medium | Critical | Mitigating | `conflict-policy.md`, `uncertainty-policy.md` y abstencion ante conflicto critico. | Codex / JusNova Chief Backend Architect |
| R-012 | Uso excesivo de fuente secundaria por accesibilidad. | Medium | High | Mitigating | Source tiers, advertencias obligatorias y prohibicion de TIER3 como soporte normativo critico unico. | Codex / JusNova Chief Backend Architect |
| R-013 | Trazas filtran prompts, documentos, mensajes o salidas completas del modelo. | Medium | Critical | Mitigating | Subfase 0.7 usa hashes, referencias, objetos cerrados y trace visibility; Subfase 0.8 endurece refs de actor/budget; Subfase 0.9 agrega refs de mensaje sin contenido; controles finales quedan para Subfase 0.10. | Codex / JusNova Chief Backend Architect |
| R-014 | Usage Ledger no concilia creditos, budget efectivo y trazas publicadas. | Medium | High | Mitigating | Subfase 0.8 exige `cost_budget_ref`, `cost_budget_version`, `plan_code`, `complexity` en `TraceObject` y `UsageEvent research_credit_used` para `COMPLEJO`/`INVESTIGACION`; conciliacion runtime queda para Fase 1. | Codex / JusNova Chief Backend Architect |
| R-015 | Memoria de caso se confunde con verdad juridica o evidencia probatoria. | Medium | Critical | Mitigating | Subfase 0.9 acepta `case-memory.schema.json` y `memory-policy.md`: memoria separa hechos de usuario, hechos documentales, riesgos y contradicciones; no sostiene vigencia ni claims criticos. | Codex / JusNova Chief Backend Architect |
| R-016 | OCR de baja confianza sostiene claims criticos. | Medium | Critical | Mitigating | Subfase 0.9 acepta `document-evidence.schema.json` y `ocr-policy.md`: baja confianza exige warnings, no es citable como evidencia decisiva sin escalacion o revision. | Codex / JusNova Chief Backend Architect |
