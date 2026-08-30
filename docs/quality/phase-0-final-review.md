# Fase 0 - Final Review

**Estado documental:** Accepted
**Fecha de revision:** 2026-08-30
**Responsable:** Codex / JusNova Chief Backend Architect
**Aprobacion de cierre:** Usuario / Owner del producto
**Decision relacionada:** Subfase 0.14 - Revision final, congelamiento y aprobacion

## Proposito

Este documento registra la auditoria final de Fase 0 antes de iniciar Fase 1. La revision confirma que las decisiones aceptadas son coherentes, implementables y suficientes para el alcance aprobado, sin convertir dependencias posteriores en blockers artificiales de Fase 0.

## Veredicto

**Resultado:** `PASS`

- No quedan hallazgos accionables P1, P2 o P3 abiertos dentro del alcance documental canónico de Fase 0.
- No quedan preguntas `Blocking` en `docs/handoff/open-questions.md`.
- Los riesgos vivos permanecen registrados y no invalidan el inicio de Fase 1.
- El handoff 0.13 es implementable y queda habilitado por el cierre formal de 0.14.
- Fase 0 queda congelada. Cambios posteriores a decisiones criticas requieren ADR nuevo o documento `Superseded`.

## Alcance y metodo

La revision cruzo:

- ADR-001 a ADR-012;
- arquitectura, limites modulares, modelo de dominio, ownership y API draft;
- contratos JSON Schema y taxonomias YAML;
- policies juridicas, de costos, seguridad, privacidad y proveedores;
- evaluation plan, golden dataset spec y readiness gates;
- implementation brief, Sprint 1 backlog y development plan;
- open questions, risk register y responsabilidades de revision.

Ademas de la lectura cruzada, se ejecutaron checks de parseo/compilacion de schemas, parseo YAML, referencias Markdown, anchors, tablas y scope documental.

## Lente tecnico

| Pregunta | Resultado | Evidencia | Conclusion |
|---|---|---|---|
| La arquitectura es coherente | PASS | `docs/adr/ADR-001-high-assurance-modular-core.md`, `docs/architecture/architecture-overview.md`, `docs/architecture/module-boundaries.md` | El modular core, los boundaries y la extraccion futura usan el mismo modelo. |
| Los contratos son implementables | PASS | `docs/contracts/`, `docs/architecture/domain-model.md`, `docs/handoff/phase-1-implementation-brief.md` | Los schemas compilan y el handoff define orden, persistencia, invariants y tests. |
| El stack es suficiente y no sobredimensionado | PASS | `docs/adr/ADR-002-stack-backend-and-infrastructure.md`, `docs/handoff/phase-1-implementation-brief.md` | PostgreSQL es fuente transaccional; Redis/OpenSearch/storage/workflows quedan detras de flags, perfiles o interfaces. |
| Los proveedores estan encapsulados | PASS | `docs/contracts/provider-interfaces.md`, `docs/policies/provider-policy.md`, `docs/schemas/provider-registry.yaml` | Ningun proveedor externo es dependencia directa obligatoria del core. |
| Los budgets son hipotesis iniciales operables | PASS | `docs/schemas/budgets.yaml`, `docs/contracts/cost-budget.schema.json`, `docs/policies/cost-governor-policy.md` | Los limites son cerrados, versionados y revisables con metricas reales. |
| Las interfaces permiten cambiar proveedores | PASS | `docs/contracts/provider-interfaces.md`, `docs/adr/ADR-005-ai-provider-and-model-policy.md` | Los gateways y auditorias evitan acoplamiento contractual a SDKs concretos. |

## Lente juridico

| Pregunta | Resultado | Evidencia | Conclusion |
|---|---|---|---|
| Las respuestas requieren evidencia | PASS | `docs/contracts/answer-contract.schema.json`, `docs/contracts/claim.schema.json`, `docs/policies/citation-policy.md` | Claims criticos publicados requieren soporte y citas resolubles. |
| La vigencia se maneja de forma conservadora | PASS | `docs/policies/validity-policy.md`, `docs/schemas/validity-statuses.yaml` | No se presume vigencia; estados inciertos exigen warning, bloqueo o abstencion segun el caso. |
| Las fuentes secundarias quedan etiquetadas | PASS | `docs/schemas/source-tiers.yaml`, `docs/policies/source-policy.md` | TIER2/TIER3 tienen usos y warnings cerrados y no sustituyen soporte primario critico. |
| Jurisprudencia, norma e inferencia estan separadas | PASS | `docs/contracts/legal-entity.schema.json`, `docs/contracts/claim.schema.json`, `docs/quality/initial-golden-dataset-spec.md` | Entidades y claim types mantienen separacion semantica y evaluable. |
| La abstencion esta definida | PASS | `docs/policies/abstention-policy.md`, `docs/contracts/abstention-render.schema.json`, `docs/policies/answer-versioning-policy.md` | Outcomes, razones y reconstruccion de respuestas bloqueadas estan cerrados. |
| Bolivia es jurisdiccion por defecto | PASS | `docs/policies/non-negotiable-principles.md`, `docs/adr/ADR-003-live-legal-search-engine.md` | El canon de Fase 0 es Bolivia-first y el golden dataset usa `BO`. |

## Lente producto y operacion

| Pregunta | Resultado | Evidencia | Conclusion |
|---|---|---|---|
| El plan base de 400 Bs esta reflejado | PASS | `docs/policies/commercial-plans-v0.md`, `docs/adr/ADR-008-cost-governor-and-commercial-budgets.md` | `PROFESIONAL` es el plan base y no se introduce un plan inferior. |
| El sistema puede explicar limites sin parecer roto | PASS | `docs/policies/cost-governor-policy.md`, `docs/contracts/error-envelope.schema.json`, `docs/architecture/api-draft-v0.md` | Limites, creditos, degradacion y errores usan estados/codigos seguros y trazables. |
| La trazabilidad sera util para soporte | PASS | `docs/contracts/trace-object.schema.json`, `docs/policies/trace-visibility-policy.md` | Hay vistas seguras, refs reconstruibles y separacion entre soporte normal y acceso elevado. |
| Los criterios de beta y mercado son verificables | PASS | `docs/quality/evaluation-plan-v0.md`, `docs/quality/beta-readiness-gates.md`, `docs/quality/market-readiness-gates.md` | Cada gate tiene evidencia, owner, consecuencia y blockers no renunciables. |
| Fase 1 esta suficientemente guiada | PASS | `docs/handoff/phase-1-implementation-brief.md`, `docs/handoff/sprint-1-backlog.md`, `docs/phases/phase-1-development-plan.md` | Stack, modulos, migraciones, contratos, tests, orden y cutlines estan definidos. |

## Delegaciones preservadas

Estas dependencias no bloquean Fase 0 y no deben reinterpretarse como entregables faltantes:

- `source-registry-entry.schema.json` y `source-snapshot.schema.json` completos quedan para subfases posteriores; Fase 1 usa seed/DTO interno sin crear contratos paralelos.
- `validity-status.schema.json` historico queda reemplazado por `validity-statuses.yaml` y campos `validity_status`; `conflict-report.schema.json` historico queda reemplazado por `EvidenceQuality`, status/warnings y traza. No se crean contratos paralelos.
- `UNKNOWN` no es `Source.tier`; solo se usa donde una taxonomia aceptada lo declara o como estado interno no publicable.
- Runtime de API y migraciones queda para Fase 1. Auth productiva minima es obligatoria en P1-09 antes de beta; storage real/provider SDKs se habilitan solo con policy, auditoria y checklist aplicables.
- Citation Auditor productivo queda en Fase 2.
- Versionado/trazabilidad juridica productiva continua en Fase 3.
- Live Legal Search, adapters, validity resolver y Source Registry productivo quedan en Fases 4/5.
- Evidence Cache y Source Snapshot Registry quedan en Fase 8.
- OCR productivo queda en Fase 9.
- Revisores juridicos, seguridad y comerciales nominados son requisito pre-market, no blocker de Fase 0.

## Preguntas no bloqueantes al freeze

Permanecen abiertas o diferidas las preguntas `OQ-001`, `OQ-002`, `OQ-003`, `OQ-004`, `OQ-017`, `OQ-018`, `OQ-019`, `OQ-020` y `OQ-021`. Todas tienen owner y momento de resolucion documentado; `OQ-020` no bloquea Fase 0 pero bloquea beta, y `OQ-021` bloquea especificamente la habilitacion de `POST /v1/documents` hasta fijar limite/allowlist versionados.

## Riesgos vivos al freeze

El registro canonico permanece en `docs/handoff/risk-register.md`. Los riesgos abiertos o en mitigacion pasan a Fase 1 y a los gates de beta/mercado con sus owners y mitigaciones; no existe un riesgo sin tratamiento que invalide el cierre documental.

## Evidencia mecanica de revision

| Check | Resultado |
|---|---|
| ADR-001 a ADR-012 con estado de decision y documental `Accepted` | PASS |
| 29 JSON Schemas contractuales + plantilla Draft 2020-12 compilables | PASS |
| 10 taxonomias/configuraciones YAML parseables | PASS |
| Fenced JSON/YAML de documentos cambiados parseables | PASS |
| Referencias locales y anchors Markdown resolubles | PASS |
| Tablas Markdown de archivos cambiados consistentes | PASS |
| Checklist de aceptacion | 243 items: 242 `Accepted`; 1 `Pending` no bloqueante para nominacion humana pre-market |
| Open questions `Blocking` abiertas | 0 |
| Cambios fuera de `docs/` | 0 |

### Doble reauditoria final independiente

Despues de aplicar la ultima correccion se ejecutaron dos revisiones profundas desde cero y con lentes distintos. Ninguna reutilizo la ausencia de hallazgos de la otra como criterio de aprobacion:

| Revision | Cobertura independiente | Evidencia reproducible | Hallazgos abiertos |
|---|---|---|---:|
| A - Contratos y seguridad | Compilacion y estructura de los 29 JSON Schemas; clasificacion de ejemplos validos, invalidos y policy-invalid; refs; limites de strings/arrays; patterns de full-match; YAML estricto; providers/budgets; API, errores y boundaries de `Message` | 55 ejemplos validos, 282 invalidos y 49 policy-invalid correctamente clasificados; 61 nodos object, 352 nodos string (337 variables y 15 finitos), 316 patterns positivos, 2 denylist y 74 arrays auditados; 10 YAML, 11 providers, 16 budgets materializados, 17 rutas API y 21 mappings de error verificados | 0 |
| B - Gobernanza y ejecutabilidad | Indices, estados, links, anchors, tablas, checklist, ADRs, policies, contratos, taxonomias, open questions, riesgos, metricas, grafo P0/P1/P2, estrategia de PR, subfases de Fase 1, settings, migraciones, scope Git y freeze | 63 Markdown, 674 refs literales exactas y 136 tablas; 15 subfases de Fase 0, 12 ADRs, 20 policies, 29 contratos, 243 items de checklist, 21 open questions y 33 riesgos; 28 items de backlog, 54 dependencias aciclicas, 10 PRs, 20 subfases de Fase 1, 3 bloques de settings identicos y 4 migraciones canonicas | 0 |

El unico item `Pending` del checklist es la nominacion humana pre-market ya declarada no bloqueante. El upload multipart permanece deliberadamente deshabilitado por `OQ-021`; este estado fail-closed es una decision controlada y no un defecto abierto de Fase 0.

## Correcciones de la reauditoria de freeze

Antes del merge final se ejecutaron lecturas adversariales sucesivas sobre todas las subfases. Se corrigieron, sin cambiar el producto ni reabrir decisiones fundacionales:

- trazabilidad de los artefactos historicos `validity-status.schema.json`/`conflict-report.schema.json` y exclusion de `UNKNOWN` como `Source.tier`;
- bypass de criticalidad y omision de assertions visibles mediante `ClaimCompletenessValidator` y oracle semantico independiente;
- cardinalidad reciproca opcional `RetrievalRun <-> EvidencePack 0..1`, con query/tenant compartidos, reutilizacion unica de `EvidenceQuality` y enum de rechazo identico entre `RetrievalRun` y `TraceObject`;
- transporte binario de upload y variantes publicas de respuesta bloqueada/abstenida;
- gate de auth productiva pre-beta, retry/fallback determinista y resolucion de llamadas externas en provider registry;
- inclusion explicita de Subfase 0.1 y de la matriz de cobertura ADR en el Final Decision Pack;
- fallback productivo fail-closed sin rutas activas, fixture local cerrado para la prueba positiva y contexto auditable unico por intento de provider;
- ownership de `ValidityResolver` base en Fase 4 y ampliacion por adapters en Fase 5;
- separacion entre dependencias dirigidas P0 y la integracion atomica P0-12+P0-13, sin ciclo documental;
- alcance fail-closed de storage/providers de ADR-011;
- pipes sin escapar dentro de regex en tablas de `domain-model.md` y `entity-ownership-matrix.md`;
- propagacion ejecutable del `PromptVersionRegistry` requerido por ADR-005, sin crear contrato publico ni migracion paralela;
- ciclo de vida cerrado de prompts: solo `active` admite uso nuevo y `deprecated` queda limitado a reconstruccion historica sin provider;
- formula atomica unica de `validity_awareness`, con denominador fijado por las fuentes que el oracle exige materializar, omisiones puntuadas como fallo y `NO_APLICA` como guard obligatorio no diluyente sin crear fuentes runtime ficticias;
- igualdad tenant y resolución materializada de `ProviderCallAudit.trace_id -> TraceObject.organization_id`, sin aceptar un `operational_runs.run_id` reservado como traza contractual, incluida en schema metadata, modelo, policy, handoff y ejemplo negativo cross-contract;
- subset inicial de `ErrorEnvelope` de Fase 1 completado con `research_credit_required` para bloquear de forma contractual ejecuciones sin saldo suficiente;
- P-NN-004 acotado a los intents que exigen búsqueda viva, preservando `EvidencePack.retrieval_run_id=null` para packs manuales/documentales permitidos;
- canonicalizacion de `AnswerContract` y `AbstentionRender.render_hash` cerrada byte a byte con un unico helper JCS (RFC 8785), dependencia fijada, UTF-8 sin BOM, orden de arrays preservado, sin normalizacion Unicode implicita y con rechazo temprano de claves duplicadas;
- perfil de `Answer` de Fase 1 aclarado: no se persisten shells con cero versiones y el puntero inicial no nulo se crea atomicamente con la version 1;
- lectura y consumo de research credits separados: saldo cero se consulta con `200`, falta de suscripcion usa `not_found` y una ejecucion sin saldo usa `research_credit_required` sin publicacion, debito ficticio ni downgrade de complejidad;
- ownership tenant de research credits cerrado de extremo a extremo entre suscripcion, saldo, movimiento, `UsageEvent` y anchors de respuesta/traza, con rechazo cross-tenant;
- ownership de `EvidencePack.query_id`/`retrieval_run_id` cerrado sin ampliar alcance: Fase 1 resuelve manifests de fixtures same-tenant y Fase 4 conserva la persistencia/FK productiva;
- dependencia de P1-01 corregida para ejecutar `0003` despues de P0-13/`0002` y resolver `answer_versions` antes de crear citas versionadas;
- gate de cierre de Fase 1 endurecido: el trabajo P1/P2 obligatorio debe estar cerrado y no puede aprobarse como meramente planificado;
- cutline de Sprint 1 separado del cierre global: diferir P1/P2 fuera del primer sprint no autoriza dejarlos pendientes al cerrar Fase 1;
- persistencia de llamada externa y `ProviderCallAudit` terminal cerrada como transaccion atomica o FKs diferibles equivalentes, sin estados confirmados huerfanos o sin audit;
- fallo de debito de research credits normalizado en todas las capas: devuelve `research_credit_required` sin publicacion, debito ficticio ni downgrade de complejidad; `degrade` solo recorta salida no esencial dentro de la complejidad solicitada;
- perfil de adjuntos de Fase 1 cerrado fail-closed: solo `attachments=[]`/omitido hasta existir pipeline documental tenant-scoped y cualquier adjunto no vacio devuelve `document_processing_required` antes de persistir mensaje o run;
- prohibicion de providers productivos acotada a modelo, busqueda, extraccion, OCR, storage y workflow, sin desactivar el `AuthProvider` productivo obligatorio de P1-09;
- casos API de `prompt_injection_resistance` obligados a capturar conjuntamente `ErrorEnvelope` y `PromptInjectionRisk` runtime schema-valid; el error aislado no satisface el blocker;
- dependencia espuria P0-05 -> P0-06 eliminada: logging define el formatter primero y PR-3 prueba la propagacion end-to-end despues de integrar el middleware, sin invertir el orden P0;
- `AnswerContract.citations[]` endurecido para aceptar solo citas publicadas con `status=valid`; estados debiles, rotos o pendientes permanecen disponibles en `Citation` para auditoria, pero no cruzan al contrato final;
- reconstruccion de `answer_blocked` ligada al output visible: render ref DB-local unico, body cerrado `{content}`, igualdad byte a byte con el mensaje `assistant_final` y hashes independientes pero verificables de render y mensaje;
- `AuthProvider` explicitamente separado de las 11 familias de processing providers: no crea un enum paralelo en `ProviderCallAudit` ni habilita llamadas de autenticacion externas por request sin decision/auditoria posterior;
- alcance de borrado/tombstone alineado: Fase 1 prueba solo superficies implementadas y el borrado documental completo se activa con su pipeline, sin rutas `DELETE` inventadas;
- ownership de Redis alineado: Fase 1 prepara compose/readiness y cache/locks se implementan solo con una funcionalidad aceptada que los necesite;
- `case_id` cerrado como `null`/omitido en Fase 1 hasta existir un agregado `Case` tenant-scoped, sin aceptar referencias controladas por cliente;
- gates ejecutables de secret scanning y deteccion de superficies raw sensibles incorporados a CI, con fixtures negativos y allowlists puntuales;
- indices de gobierno ADR/contratos/policies promovidos a `Accepted` como parte del freeze;
- ownership de interfaces provider repartido por fase sin atribuir modelos, storage, workflows u OCR a Fase 4;
- modos operativos alineados con `response-modes.yaml` y regression suite contractual inicial de ADR-012 obligatoria al cierre de Fase 1, separada del reporte beta completo;
- ejemplos abreviados de `citation-policy.md` marcados como proyecciones relacionales, sin presentarlos como payloads runtime schema-valid;
- semantica de fixtures relacionales cerrada: `x-cross-contract-policy-invalid-examples` contiene contratos completos validables por separado, mientras `x-integrity-examples` usa claves `*_projection` deliberadamente parciales que deben expandirse antes de una prueba ejecutable;
- identidad local de `EvidencePack` cerrada como conjuntos de areas, fuentes y pasajes, con resolucion exacta `EvidencePassage -> EvidenceSource` dentro del mismo pack;
- puntero `Answer.latest_answer_version_ref` cerrado al mismo answer/tenant, a la version vigente y con `response_outcome` consistente;
- relacion `Claim <-> Citation` endurecida como n:m bidireccional same-version, con `claim_id`, `citation_ref` y `sources_used[]` unicos, citas distintas autorizadas a compartir fuente y ninguna cita contractual huerfana; `Source.metadata` queda cerrado a metadata publica minimizada;
- resultados de `CitationAudit` ligados a la `Citation` resuelta de la misma version, con par `(claim_id, citation_ref)` unico y sin pasajes/fuentes alternativos bajo una misma ref;
- indice contractual completado con los 29 JSON Schemas aceptados;
- indice y gobernanza de las 20 policies completados, con alcance y fronteras explicitas entre controles deterministas y asistencia IA en busqueda, routing, privacidad, prompt injection y providers;
- metadatos de gobierno normalizados en los tres documentos rectores de 0.13 y en las taxonomias de seguridad/proveedores; `non-negotiable-principles.md` incorporado al indice de policies aceptadas y nominacion humana aclarada como requisito del gate aplicable, no del cierre de Fase 0;
- cadena Alembic P1/P2 cerrada de forma lineal: `P2-01/0004` depende de `P1-01/0003` y no abre un segundo head ni omite la migracion de evidencia;
- secuencia post-esqueleto unificada entre brief, backlog y plan: P1-07/P1-08/P1-09 preceden al trabajo P2, y P2-06 conserva ownership separado dentro del cierre P2;
- identidades de busqueda cerradas para scoring determinista: `LegalEntity.entity_id` y `LegalSearchResult.result_id` son unicos por contenedor, y fuentes rechazadas se consolidan por URL canonica antes de persistir o puntuar;
- `source-routing-matrix.md` conserva sus rutas aceptadas y explicita sus validaciones como criterios de aceptacion verificables;
- condicion de snapshot de `LegalSearchResult` expresada sin ambiguedad: pasaje extraido con `snapshot_id` ausente o `null` exige razon cerrada, la pareja `null` + razon es valida y solo un `snap_*` materializado excluye esa razon, sin depender de verdad vacia;
- integridad de `CitationAudit.results[]` propagada a los fixtures y criterios ejecutables de Fase 1, sin adelantar el Citation Auditor productivo asignado a Fase 2;
- fixtures cross-contract de `AnswerVersion` actualizados para conservar validez individual tras cerrar `AnswerContract.citations[].status=valid` y para que los hashes JCS de los casos de identidad/evidencia/claim cubran su `AnswerContract`; cada caso negativo queda aislado a la divergencia declarada;
- momento de resolucion de `OQ-001` ligado al gate real de storage productivo, sin fingir que ADR-002 ya resolvio la seleccion comercial ni habilitar persistencia privada desde el stub P1-08;
- criterio P1-04 acotado: solo el cierre tecnico `answer_blocked` materializa `TraceObject`; un fallo temprano conserva trazabilidad operativa sin fabricar un contrato final invalido;
- frontera comercial del `CostGovernor` cerrada: `BudgetRequest.plan_code` se deriva de la suscripción current del mismo tenant y nunca de input controlado por cliente;
- orden documental de `expected_error_code` sincronizado con el enum canónico de `ErrorEnvelope`, además de conservar igualdad exacta de conjunto;
- round-trip de `ProviderCallAudit` cerrado: refs y campos opcionales no-nullable se omiten cuando la columna DB es nula, en vez de serializar un `null` rechazado por el contrato.
- `PromptInjectionRisk.detected_in_ref` cerrado a refs resolubles y `url_hash` canonico; se retiro `source_hash` mientras no existan bytes canonicos ni mapping contractual hacia una fuente o pasaje.
- `AnswerContract.claims[]` cerrado a claims publicados con `verification_status=passed`; estados fallidos, pendientes o bloqueados quedan fuera del contrato visible y solo pueden conservarse en traza/oracle.
- ciclo de vida de `OperationalRun` cerrado para fallos tempranos desde `queued`, sin fabricar `started_at` ni `run_execution_started`, con timestamps consistentes y unicidad de eventos singulares/terminales por run;
- consumo de research credits acotado a grants activos no expirados, con debito atomico, movimiento conciliable y saldo nunca negativo;
- boundary de fetch endurecido contra SSRF y DNS rebinding: HTTPS/443, resolucion global, IP fijada, proxies de ambiente y redirects automaticos deshabilitados, revalidacion por salto y URL cruda omitida cuando `reason=access_not_allowed`;
- YAML canonicos cerrados con un loader UTF-8 compartido y estricto, modelos tipados sin extras y rechazo previo de claves duplicadas, tags, anchors, aliases, merge keys y documentos multiples;
- taxonomia de errores de provider separada: el registry produce codigos internos de `ProviderCallAudit` y la PRL aplica una traduccion cerrada al `ErrorEnvelope` publico, sin copiar enums incompatibles.
- fallback positivo de PRL acotado a un registry fixture completo, cerrado y exclusivo de tests con dos providers stub resolubles y una ruta compatible; el registry runtime canonico permanece sin rutas y falla con `provider_unavailable`.
- fixture negativo cross-contract de `ProviderCallAudit.trace_id` completado con un `TraceObject` individualmente schema-valid, de modo que el unico fallo intencional sea la divergencia tenant declarada.
- secuencia transaccional del endpoint de mensajes cerrada: prevalidacion sin persistencia, reserva de IDs, decision de budget en memoria y persistencia atomica de `Message` antes de `OperationalRun`/`BudgetDecision`, respetando FKs y evitando runs o decisiones huerfanos.
- dependencias de P0-13 completadas con `Message` y `ErrorEnvelope`, y procedencia de eventos internos aclarada para no emitir `error_envelope_returned` sin un `ErrorEnvelope` realmente devuelto.
- orden ejecutivo de servicios alineado con backlog, plan y secuencia vinculante: `ModelProvider`, `CostGovernor`, commercial/`UsageLedger` y cierre funcional de P0-13.
- bifurcacion de enforcement de budget cerrada: solo `allow|degrade` puede iniciar el run; `block` termina desde `queued` con error contractual y sin provider call, usage ni artefactos finales fabricados.
- settlement terminal cerrado como una sola transaccion: respuesta/traza, evento de consulta, debito de credito cuando aplica y `run_completed` se confirman o revierten juntos; SSE terminal solo se emite despues del commit.
- identidad de budget y consumo cerrada entre `BudgetDecision`, `TraceObject` y `UsageEvent`, con mapping unico `standard_query|complex_query`, anchors `trace_id`/`answer_id`, indices de idempotencia y prueba concurrente sobre el ultimo credito.
- frontera de mensajes protegida con `Idempotency-Key` obligatoria, hash/fingerprint tenant/actor-scoped, replay del resultado original y rechazo `conflict` ante reutilizacion con payload distinto; la key raw queda fuera de persistencia, logs y trazas.
- frontera de trazas/retrieval aclarada y minimizada: `TraceObject` hereda sensibilidad porque contiene claims derivados, el run operativo hereda query/resultados, y `RawAccessEvent`/`PromptInjectionRisk` conservan la clase efectiva aplicable; la traza solo recibe summaries de retrieval metadata-only, los snippets de discovery se normalizan y acotan, y warnings/notas sanitizados quedan limitados a una linea, sin controles y sin duplicados en todas las superficies contractuales aplicables.
- cierre exacto de los 316 patterns positivos contractuales endurecido con una asercion de fin real; los 2 patterns restantes son denylist bajo `not` para email y telefono en `UsageEvent.actor_ref`: IDs, refs, hashes, codigos, fechas y textos seguros de una linea ya no aceptan controles o terminadores no declarados por la semantica permisiva de `$` en motores compatibles; las regex abreviadas del API, modelo, provider interfaces y golden spec declaran la misma semantica de full-match.
- proyecciones de persistencia verificadas contra los campos `required` de `Message`, `ModelCall`, `ToolCall`, `AnswerVersion`, `AbstentionRender`, `UsageEvent`, `ProviderCallAudit` y `RawAccessEvent`; las tablas JSONB de evidencia y traza conservan identidad, tenant y reglas de igualdad con sus payloads cerrados. El vocabulario `created_by_ref` del handoff coincide con el contrato, incluido `support_hash_*`.
- limites de recursos cerrados en los 29 contratos: todo string variable declara `minLength: 1`/`maxLength` y no depende de `format` para rechazar vacio; todo array declara `maxItems`. Los perfiles quedan documentados, `Message` limita contenido/adjuntos/refs, los DTOs API-only son cerrados y sus colecciones quedan acotadas/paginadas, y el API rechaza bodies JSON mayores a 131072 bytes antes de parsear, reservar IDs o persistir; el upload multipart queda deshabilitado hasta que `OQ-021` fije un limite binario/allowlist versionados y su stream fail-closed.
- limites contractuales propagados a persistencia: columnas escalares, arrays JSONB y payloads cerrados conservan las mismas cotas o unas mas estrictas, rechazan `limite + 1`, no truncan y verifican igualdad columna-payload.
- `Idempotency-Key` unificada en todo el canon como ASCII de 1 a 128 caracteres; desaparece la variante residual de 16 caracteres minimos.
- estados vacios ambiguos eliminados en issuer, localizadores, entidades, consultas normalizadas y secciones; titulos, issuer y `ErrorEnvelope.message` rechazan controles de linea, y `Citation.auditor_notes[]` respeta el perfil maximo de 32 notas.
- semantica de ejecucion cerrada: solo un `Message` iniciador confirmado crea exactamente un `OperationalRun`; mensajes assistant/system/tool de salida o estado nunca disparan otro run.
- fixture cross-contract de `Conversation` actualizado tras endurecer `AnswerContract`: todas sus secciones son individualmente schema-valid y el unico fallo intencional permanece en usar el titulo de conversacion como evidencia.

Cada correccion queda en contrato, policy, gate o handoff verificable. No quedan hallazgos accionables P1/P2/P3 abiertos tras esta reauditoria.

## Aprobacion

Con los tres lentes en `PASS`, Fase 0 cumple su Definition of Done. La Subfase 0.14 y Fase 0 pasaron a `Accepted` el 2026-08-10; la reauditoria final del 2026-08-30 ratifica ese cierre sobre el paquete que se congela en Git.

Fase 1 puede iniciar usando el handoff 0.13 como orden vinculante. Esto no habilita beta ni mercado: esos avances continúan sujetos a sus readiness gates.
