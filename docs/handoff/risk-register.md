# Risk Register

**Estado documental:** Draft
**Fecha:** 2026-08-30
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Registro vivo de riesgos de Fase 0

Nota: el estado `Draft` es intencional porque este registro es vivo. Los checklists pueden aceptar que está creado y actualizado sin convertirlo en documento cerrado.

## Revision de Subfase 0.14

- `R-001` se cierra para Fase 0: existe handoff ejecutable, priorizado y aceptado.
- `R-002` permanece en mitigacion: los contratos son implementables documentalmente y Fase 1 debe probar round-trip y policy invariants.
- `R-003` se cierra para Fase 0: la revision cruzada final no deja contradicciones P1/P2/P3 abiertas.
- Los riesgos runtime, beta y market permanecen abiertos o en mitigacion y conservan sus gates.
- Se agregan `R-030` para controlar drift posterior al freeze, `R-031` para impedir que dev auth llegue a beta, `R-032` para bloquear SSRF/DNS rebinding antes de fetch real y `R-033` para impedir payloads JSON o binarios contractualmente ilimitados.

## Escala

| Campo | Valores |
|---|---|
| Probabilidad | Low, Medium, High |
| Impacto | Low, Medium, High, Critical |
| Estado | Open, Mitigating, Accepted, Closed |

## Riesgos iniciales

| ID | Riesgo | Probabilidad | Impacto | Estado | Mitigacion | Responsable |
|---|---|---|---|---|---|---|
| R-001 | Sobrediseno documental sin avance ejecutable hacia Fase 1. | Medium | High | Closed | Handoff 0.13, backlog P0/P1/P2 y plan operativo aceptados; Fase 1 queda habilitada por 0.14. | Codex / JusNova Chief Backend Architect |
| R-002 | Contratos demasiado abstractos para implementacion. | Medium | High | Mitigating | JSON Schemas, ejemplos, invariants y criterios de aceptacion estan cerrados; Fase 1 debe demostrar round-trip y tests policy-valid. | Codex / JusNova Chief Backend Architect |
| R-003 | Contradicciones entre ADRs, policies y contracts. | Medium | High | Closed | Revision cruzada 0.14 y reauditoria adversarial corrigieron los hallazgos residuales antes del merge; schemas, policies, gates y handoff quedan alineados sin P1/P2/P3 abiertos. | Codex / JusNova Chief Backend Architect |
| R-004 | Dependencia accidental de proveedor externo en documentos de Fase 0. | Medium | Critical | Mitigating | Provider interfaces obligatorias, provider registry, provider policy y `ProviderCallAudit` aceptados en 0.10. | Codex / JusNova Chief Backend Architect |
| R-005 | Politica de vigencia insuficiente o ambigua. | Medium | Critical | Mitigating | Estados cerrados de vigencia, frases prohibidas, abstention policy y `validity_awareness` blocker; enforcement queda para runtime/evals. | Codex / JusNova Chief Backend Architect |
| R-006 | Seguridad tratada tarde. | Medium | Critical | Mitigating | Subfase 0.9 crea seguridad documental minima; Subfase 0.10 acepta privacy/security policy, data classification, raw access, provider registry y prompt injection controls. Enforcement runtime queda para Fase 1. | Codex / JusNova Chief Backend Architect |
| R-007 | Trazabilidad insuficiente para auditar respuestas juridicas. | Medium | Critical | Mitigating | Subfase 0.7 acepta TraceObject, model/tool calls, CitationAudit, CostReport y AnswerVersion; implementacion queda para fases posteriores. | Codex / JusNova Chief Backend Architect |
| R-008 | Cost Governor no refleja plan base de 400 Bs. | Medium | High | Mitigating | Subfase 0.8 acepta `budgets.yaml`, `cost-budget.schema.json`, `usage-event.schema.json`, `commercial-plans-v0.md` y `cost-governor-policy.md`; runtime queda para Fase 1. | Codex / JusNova Chief Backend Architect |
| R-009 | Fuente oficial clave caida durante busqueda viva. | High | High | Open | `source-policy.md`, `legal-search-policy.md`, warnings de fallback y Provider Reliability Layer en fases posteriores. | Codex / JusNova Chief Backend Architect |
| R-010 | Vigencia no confirmada presentada como vigente. | Medium | Critical | Mitigating | `validity-policy.md`, estados cerrados de vigencia y frases prohibidas. | Codex / JusNova Chief Backend Architect |
| R-011 | Conflicto irresoluble ocultado en respuesta categorica. | Medium | Critical | Mitigating | `conflict-policy.md`, `uncertainty-policy.md` y abstencion ante conflicto critico. | Codex / JusNova Chief Backend Architect |
| R-012 | Uso excesivo de fuente secundaria por accesibilidad. | Medium | High | Mitigating | Source tiers, advertencias obligatorias y prohibicion de TIER3 como soporte normativo critico unico. | Codex / JusNova Chief Backend Architect |
| R-013 | Trazas filtran prompts, documentos, mensajes o salidas completas del modelo. | Medium | Critical | Mitigating | Subfase 0.7 usa hashes, referencias, objetos cerrados y trace visibility; Subfase 0.8 endurece refs de actor/budget; Subfase 0.9 agrega refs de mensaje sin contenido; Subfase 0.10 agrega raw access, provider audit, data classification y prompt injection risk refs sin contenido crudo. | Codex / JusNova Chief Backend Architect |
| R-014 | Usage Ledger no concilia creditos, budget efectivo y trazas publicadas, o un retry duplica consumo. | Medium | High | Mitigating | Subfase 0.8 exige igualdad exacta de tenant/plan/complejidad/budget entre decision efectiva, `TraceObject` y `UsageEvent`; Fase 1 confirma respuesta, usage, debito y cierre del run en una transaccion terminal con lock de saldo, indices idempotentes, `Idempotency-Key` hasheada tenant/actor-scoped y tests de retry/concurrencia. | Codex / JusNova Chief Backend Architect |
| R-015 | Memoria de caso se confunde con verdad juridica o evidencia probatoria. | Medium | Critical | Mitigating | Subfase 0.9 acepta `case-memory.schema.json` y `memory-policy.md`: memoria separa hechos de usuario, hechos documentales, riesgos y contradicciones; no sostiene vigencia ni claims criticos. | Codex / JusNova Chief Backend Architect |
| R-016 | OCR de baja confianza sostiene claims criticos. | Medium | Critical | Mitigating | Subfase 0.9 acepta `document-evidence.schema.json` y `ocr-policy.md`: baja confianza exige warnings, no es citable como evidencia decisiva sin escalacion o revision. | Codex / JusNova Chief Backend Architect |
| R-017 | Provider externo recibe clases de datos no permitidas o demasiado sensibles. | Medium | Critical | Mitigating | Subfase 0.10 acepta `data-classification.yaml`, `provider-registry.yaml`, `provider-policy.md` y `ProviderCallAudit`; runtime debe hacer subset enforcement en Fase 1. | Codex / JusNova Chief Backend Architect |
| R-018 | Acceso raw/elevado se confunde con soporte normal. | Medium | Critical | Mitigating | `RawAccessEvent` no admite `support_operator` ni `SUPPORT_VIEW`; soporte normal queda limitado a vista redacted. | Codex / JusNova Chief Backend Architect |
| R-019 | Prompt injection documental o HTML contamina instrucciones o claims criticos. | Medium | Critical | Mitigating | `prompt-injection-risk.schema.json` y `prompt-injection-policy.md` delimitan evidencia como datos, registran riesgos y bloquean claims criticos con evidencia `blocking`. | Codex / JusNova Chief Backend Architect |
| R-020 | Fase 1 inventa tablas o relaciones incompatibles con contratos aceptados. | Medium | High | Mitigating | Subfase 0.11 acepta `domain-model.md` y `entity-ownership-matrix.md` con entidades, IDs, refs locales, cardinalidades y ownership por entidad. | Codex / JusNova Chief Backend Architect |
| R-021 | API inicial expone memoria, documentos, OCR, trazas o errores con datos sensibles. | Medium | Critical | Mitigating | Subfase 0.11 acepta `api-draft-v0.md` y `error-envelope.schema.json`: endpoints devuelven vistas seguras y errores usan codigos cerrados sin contenido crudo. | Codex / JusNova Chief Backend Architect |
| R-022 | Eval dataset demasiado facil o no representativo. | Medium | Critical | Mitigating | Subfase 0.12 exige buckets, tags adversariales, documentos, vigencia, conflictos, abstencion y revision humana antes de beta/mercado. | Codex / JusNova Chief Backend Architect |
| R-023 | Metrica blocker no medible por dataset insuficiente. | Medium | Critical | Mitigating | `evaluation-plan-v0.md` bloquea beta si una metrica blocker queda `not_measurable`. | Codex / JusNova Chief Backend Architect |
| R-024 | Eval drift entre fixtures, contratos y taxonomias aceptadas. | Medium | High | Mitigating | `initial-golden-dataset-spec.md` exige `legal_intents[]` canonicos, vigencia aceptada, refs locales scopeados y enums cerrados. | Codex / JusNova Chief Backend Architect |
| R-025 | Revision legal humana ausente o tardia. | Medium | Critical | Open | Market readiness exige signed review artifact y proceso humano para casos criticos. | Codex / JusNova Chief Backend Architect |
| R-026 | Prompt injection no cubierto por suficientes casos adversariales. | Medium | Critical | Mitigating | Dataset objetivo exige 30 casos adversariales y beta gates bloquean si prompt injection resistance no cumple target. | Codex / JusNova Chief Backend Architect |
| R-027 | Document grounding insuficiente en PDFs o OCR. | Medium | Critical | Mitigating | Dataset objetivo exige 50 consultas mixtas, 30 PDFs escaneados y metrica blocker `document_grounding`. | Codex / JusNova Chief Backend Architect |
| R-028 | Regression suite se implementa tarde y no bloquea releases. | Medium | High | Mitigating | Beta y market gates exigen eval report y regression suite antes de avanzar. | Codex / JusNova Chief Backend Architect |
| R-029 | Handoff de Fase 1 introduce rutas, enums o tablas paralelas a contratos aceptados. | Medium | High | Mitigating | Subfase 0.13 alinea brief/backlog/plan con `docs/contracts/`, `docs/schemas/budgets.yaml`, `domain-model.md`, `api-draft-v0.md` y `error-envelope.schema.json`; Fase 1 debe tratar operational runs/events como detalle tecnico. | Codex / JusNova Chief Backend Architect |
| R-030 | Cambios posteriores al freeze alteran decisiones criticas sin trazabilidad. | Medium | Critical | Mitigating | Fase 0 congelada: todo cambio critico requiere ADR nuevo o documento `Superseded`, con impacto en contracts, policies, evals y handoff. | Codex / JusNova Chief Backend Architect |
| R-031 | `DevAuthProvider` o headers de desarrollo llegan a beta/produccion sin autenticacion productiva ni membership activa. | Medium | Critical | Open | P1-09 y gate pre-beta exigen adapter productivo provider-neutral, tokens firmados con issuer/audience/expiry, resolucion actor-membership-tenant y negativos de seguridad; `DevAuthProvider` solo local/test. | Backend Lead / Security Reviewer |
| R-032 | Fetch o headless alcanza loopback, redes privadas, metadata cloud o un destino cambiado por DNS rebinding. | Medium | Critical | Mitigating | `FetchPolicy` y `legal-search-policy.md` permiten solo HTTPS/443 a DNS globalmente routable, deshabilitan proxies de ambiente y redirects automaticos, revalidan cada salto, fijan la direccion validada y limitan tiempo, MIME y bytes; Fase 4 no habilita red real sin la suite SSRF obligatoria. | Backend Lead / Security Reviewer |
| R-033 | Un ID, texto, array, body JSON o stream binario sin cota agota memoria, CPU, ancho de banda, storage o capacidad de indices antes de que CostGovernor intervenga. | Medium | Critical | Mitigating | Los 29 JSON Schemas declaran `minLength: 1`/`maxLength` para strings variables y `maxItems` para arrays; `api-draft-v0.md` limita JSON a 131072 bytes y Message a 20000 code points/20 attachments. `POST /v1/documents` permanece deshabilitado hasta que `OQ-021` fije limite/allowlist versionados y proxy/aplicacion aborten el stream antes de persistir. `make validate-schemas` y boundary tests bloquean regresiones. | Backend Lead / Security Reviewer |
