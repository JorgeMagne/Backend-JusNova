# JusNova — Sprint 1 Backlog

**Ruta objetivo en repo:** `/docs/handoff/sprint-1-backlog.md`
**Versión:** 1.0
**Fecha:** 2026-05-26
**Estado:** Accepted como backlog de Fase 1; ejecucion habilitada por el cierre formal de 0.14/Fase 0
**Fase:** Fase 1 — Fundaciones backend, datos, telemetría y Cost Governor
**Sprint:** Sprint 1 — Foundation Skeleton
**Duración recomendada:** 1 sprint técnico cerrado; la duración real depende del equipo, pero el alcance no debe expandirse.

---

## 1. Objetivo del Sprint 1

Crear el esqueleto backend productivo mínimo de JusNova para que toda ejecución futura tenga:

- estructura modular;
- settings tipados;
- base de datos y migraciones;
- health checks;
- logging estructurado;
- `request_id` y `correlation_id`;
- modelos base;
- ErrorEnvelope;
- wrapper de modelo;
- CostGovernor stub funcional;
- UsageLedger básico;
- trazabilidad mínima.

El Sprint 1 no busca implementar inteligencia jurídica. Busca impedir que el sistema nazca sin medición, sin costos, sin trazas y sin estructura.

---

## 2. Regla de prioridad

### P0

Debe hacerse primero. Si un P0 no está terminado, no se acepta el sprint.

### P1

Debe hacerse después del esqueleto. Puede entrar en Sprint 1 si P0 está sólido. Si no, pasa al Sprint 2 sin romper el handoff.

### P2

Preparación para Fase 2/3/4. No bloquea el cierre del Sprint 1 salvo que el equipo haya terminado P0/P1.

---

## 3. Definition of Done del Sprint 1

Sprint 1 está terminado cuando:

```txt
[ ] Repo backend creado con estructura modular.
[ ] App FastAPI levanta localmente.
[ ] Docker Compose levanta Postgres y Redis; OpenSearch queda definido en profile `search` deshabilitado por defecto.
[ ] Alembic aplica migraciones P0 `0001_core_foundation` y `0002_trace_usage_budget`; si entra trabajo P1/P2, `0003_evidence_contract_stubs` y `0004_source_registry_seed` también aplican desde cero.
[ ] Settings tipados funcionan.
[ ] Logging estructurado JSON funciona.
[ ] request_id/correlation_id se propagan.
[ ] Health checks funcionan.
[ ] Modelos base Organization/User/Conversation/Message existen.
[ ] ErrorEnvelope aplicado a errores de validación y errores internos controlados.
[ ] ModelProvider stub existe y no hace llamadas externas reales.
[ ] `docs/schemas/budgets.yaml` existe.
[ ] CostGovernor carga budgets y devuelve decisiones.
[ ] UsageLedger registra eventos básicos.
[ ] `research_credit_used` se debita para `COMPLEJO`/`INVESTIGACION` o esas ejecuciones se bloquean/degradan.
[ ] Plan/Subscription/ResearchCredit y CommercialService mínimos resuelven suscripción activa y saldo de créditos.
[ ] Trazabilidad mínima persiste run/trace events.
[ ] Tests de aislamiento tenant base cubren lectura, escritura, listado y borrado lógico cross-tenant solo donde exista superficie aceptada; no se crean rutas `DELETE` nuevas.
[ ] `make test` pasa para P0 obligatorio.
[ ] Si entra trabajo P1/P2, sus tests aplicables pasan.
[ ] No existe respuesta jurídica real.
[ ] No existe búsqueda jurídica real hardcodeada.
[ ] No existen secretos reales en repo.
```

---

## 4. P0 — Debe hacerse primero

### P0-01 — Crear estructura de repo backend

**Objetivo:** Crear la base del repositorio con estructura profesional y extensible.

**Entregables:**

- `pyproject.toml`;
- `README.md`;
- `.env.example`;
- `.gitignore`;
- `uv.lock` o lockfile del gestor elegido;
- `docker-compose.yml`;
- `Makefile`;
- carpetas `src/jusnova`, `tests`, `docs`, `alembic`.

**Criterios de aceptación:**

- `make help` muestra comandos disponibles;
- `make test` existe aunque todavía falle si no hay tests;
- `make dev` o equivalente levanta backend;
- estructura coincide con el implementation brief;
- no hay código en raíz salvo archivos de configuración.

**Errores a evitar:**

- app monolítica en un solo archivo;
- carpetas sin criterio;
- mezclar tests, app y scripts;
- incluir secretos reales.

**Dependencias:** ninguna.

---

### P0-02 — Crear app FastAPI modular base

**Objetivo:** Levantar una API base con routers versionados y composición modular.

**Entregables:**

- `src/jusnova/main.py`;
- `src/jusnova/api/router.py`;
- `src/jusnova/api/health.py`;
- router `/v1` preparado.

**Criterios de aceptación:**

- `GET /health/live` responde 200;
- health se monta fuera de `/v1`;
- OpenAPI genera rutas en dev;
- el router principal compone routers por módulo;
- no hay lógica de dominio en `main.py`.

**Errores a evitar:**

- importar módulos internos sin límites;
- crear endpoints sin schemas;
- meter lógica de DB en handlers.

**Dependencias:** P0-01.

---

### P0-03 — Configurar Pydantic settings

**Objetivo:** Centralizar configuración por ambiente.

**Entregables:**

- `src/jusnova/core/config.py`;
- `.env.example`;
- tests de settings.

**Campos mínimos de Settings:**

Esta lista hereda la lista canónica de `phase-1-implementation-brief.md` y debe implementarse completa en P0-03.

```txt
APP_ENV
APP_NAME
APP_VERSION
API_PREFIX
DATABASE_URL
REDIS_URL
OPENSEARCH_URL
S3_ENDPOINT_URL
S3_BUCKET
S3_ACCESS_KEY_ID
S3_SECRET_ACCESS_KEY
LOG_LEVEL
LOG_FORMAT
REQUEST_ID_HEADER
CORRELATION_ID_HEADER
BUDGETS_FILE
ENABLE_OPENSEARCH
ENABLE_REDIS
ENABLE_S3
ENABLE_OPENAI_REAL_CALLS
```

**Criterios de aceptación:**

- settings cargan desde env;
- defaults seguros para dev;
- la lista completa existe como campos tipados, pero URLs/secrets de Redis/OpenSearch/S3 solo son env vars requeridas cuando `ENABLE_REDIS`, `ENABLE_OPENSEARCH` o `ENABLE_S3` están en `true`;
- con `ENABLE_OPENSEARCH=false` y `ENABLE_S3=false`, boot/CI no falla por ausencia de `OPENSEARCH_URL` ni credenciales S3;
- `ENABLE_OPENAI_REAL_CALLS=false` por defecto;
- `ENABLE_OPENAI_REAL_CALLS=true` no es permitido en Fase 1 aunque existan stubs de `ProviderCallAudit`;
- producción falla si falta configuración crítica.

**Errores a evitar:**

- leer `os.environ` disperso;
- hardcodear URLs;
- defaults inseguros en producción.

**Dependencias:** P0-01.

---

### P0-04 — Configurar PostgreSQL y migraciones

**Objetivo:** Crear base transaccional versionada.

**Entregables:**

- SQLAlchemy setup;
- Alembic setup;
- `0001_core_foundation.py`;
- `0002_trace_usage_budget.py` obligatorio para cerrar P0;
- tests de migración.

**Criterios de aceptación:**

- `alembic upgrade head` funciona en DB limpia;
- app usa sesiones controladas;
- no se usa `metadata.create_all()` en runtime;
- migraciones son reproducibles.

**Errores a evitar:**

- crear tablas manualmente;
- modelos sin migración;
- usar sync/async mezclado sin criterio;
- no indexar claves de consulta básicas.

**Dependencias:** P0-03.

---

### P0-05 — Configurar logging estructurado

**Objetivo:** Loggear eventos técnicos en formato auditable.

**Entregables:**

- `src/jusnova/core/logging.py`;
- logs JSON;
- sanitización mínima de secrets;
- tests básicos.

**Campos mínimos:**

```txt
timestamp
level
service
env
request_id
correlation_id
organization_id
actor_ref
event
message
```

`organization_id` y `actor_ref` son obligatorios en logs de requests tenant-scoped despues de resolver auth/membership. En `/health/live`, errores pre-auth o rutas sin tenant resuelto, deben ser `null` u omitirse; no se deben inventar valores.

**Criterios de aceptación:**

- cada request genera log con request_id;
- errores incluyen `error_code` y `request_id`;
- no aparecen secrets en logs;
- formato consistente.

**Errores a evitar:**

- logs planos imposibles de consultar;
- stack traces expuestos en producción;
- loggear bodies completos con datos sensibles.

**Dependencias:** P0-02, P0-06 puede integrarse después.

---

### P0-06 — Configurar request_id/correlation_id

**Objetivo:** Trazar cada request y agrupar operaciones relacionadas.

**Entregables:**

- middleware de request context;
- `X-Request-Id`;
- `X-Correlation-Id`;
- propagación a logs;
- tests de headers.

**Criterios de aceptación:**

- si el cliente no envía ID, el sistema genera uno;
- si el cliente envía correlation ID válido, se conserva;
- ambos IDs aparecen en respuesta;
- ambos IDs aparecen en logs.

**Errores a evitar:**

- regenerar correlation ID en cada capa;
- no propagar a servicios;
- aceptar IDs absurdamente largos o maliciosos.

**Dependencias:** P0-02.

---

### P0-07 — Implementar health checks

**Objetivo:** Distinguir vida del proceso y readiness operacional.

**Entregables:**

- `GET /health/live`;
- `GET /health/ready`;
- `GET /health/version`.

**Criterios de aceptación:**

- live responde aunque DB esté caída;
- ready falla si DB crítica está caída;
- Redis/OpenSearch/S3 se reportan como `{enabled, status}`; `status` usa `docs/schemas/host-statuses.yaml` (`HEALTHY|DEGRADED|DOWN|UNKNOWN`) y una dependencia apagada por flag usa `enabled=false`, `status=UNKNOWN`;
- response incluye versión y ambiente.

**Errores a evitar:**

- health check que oculta DB caída;
- health check que hace operaciones costosas;
- mezclar live y ready.

**Dependencias:** P0-02, P0-03, P0-04.

---

### P0-08 — Implementar modelos base: Organization, User, Conversation, Message

**Objetivo:** Crear la base tenant/conversacional mínima.

**Entregables:**

- ORM models;
- Pydantic schemas;
- repositories;
- services;
- migración;
- seeds dev opcionales.

**Modelos requeridos:**

- `Organization`;
- `User`;
- `Membership`;
- `Conversation`;
- `Message`.

**Criterios de aceptación:**

- conversación pertenece a organización;
- mensaje pertenece a conversación y organización;
- usuario pertenece a organización vía membership;
- índices mínimos existen;
- tests crean entidades sin violar constraints.

**Errores a evitar:**

- conversaciones sin organization_id;
- mensajes sin tenant;
- usar email como ID;
- no controlar timestamps.

**Dependencias:** P0-04.

---

### P0-09 — Implementar schema Pydantic de ErrorEnvelope

**Objetivo:** Normalizar errores desde el inicio.

**Entregables:**

- `ErrorEnvelope` Pydantic;
- exception handlers;
- códigos de error iniciales;
- tests.

**Criterios de aceptación:**

- validation errors devuelven envelope;
- not found devuelve envelope;
- tenant mismatch devuelve envelope con `error_code=tenant_mismatch`;
- internal controlled errors devuelven envelope;
- envelope incluye `request_id`;
- `correlation_id` se propaga en headers/logs, no como campo requerido del envelope;
- no se filtra stack trace.

**Errores a evitar:**

- respuestas de error distintas por endpoint;
- mensajes técnicos al usuario final;
- ocultar retryability;
- codigos fuera de `docs/contracts/error-envelope.schema.json` o fuera del subconjunto permitido por endpoint en `docs/architecture/api-draft-v0.md`.

**Dependencias:** P0-06.

---

### P0-10 — Implementar wrapper `ModelProvider` stub

**Objetivo:** Impedir que el backend llame proveedores de IA de forma dispersa.

**Entregables:**

- interfaz `ModelProvider`;
- `StubModelProvider`;
- schema `ModelRequest`;
- schema `ModelResponse`;
- persistencia opcional de `model_call` si P0-13 ya está listo;
- tests.

**Criterios de aceptación:**

- ningún módulo llama OpenAI directamente;
- provider default es stub;
- stub devuelve respuesta determinista;
- feature flag impide llamada real en Fase 1.

**Errores a evitar:**

- importar SDK de OpenAI en routers;
- meter prompts legales en Fase 1;
- simular consumo sin registrarlo cuando se use.

**Dependencias:** P0-03.

---

### P0-11 — Implementar `CostGovernor` stub con budgets YAML

**Objetivo:** Aplicar límites de plan y complejidad desde el primer sprint.

**Entregables:**

- `docs/schemas/budgets.yaml`;
- loader validado;
- `CostGovernorService`;
- `BudgetDecision`;
- tabla `budget_decisions` en `0002_trace_usage_budget.py`;
- tests.

**Criterios de aceptación:**

- carga `docs/schemas/budgets.yaml`;
- respeta `PROFESIONAL` como plan comercial mínimo desde `docs/policies/commercial-plans-v0.md`;
- devuelve límites para `SIMPLE`, `MEDIO`, `COMPLEJO` e `INVESTIGACION`;
- puede devolver `allow`, `degrade`, `block`;
- para `COMPLEJO`/`INVESTIGACION`, la decisión interna expone `research_credit_cost` y marca que la ejecución requiere débito/reserva antes de publicarse;
- el enforcement runtime contra `UsageLedger`, saldo y anchors `trace_id`/`answer_id` pertenece a P0-12/P0-13 en PR-6, no a P0-11 aislado;
- YAML inválido falla readiness;
- no hay límites hardcodeados en handlers.

**Errores a evitar:**

- usar `serp_calls` como nombre base; debe usarse `discovery_calls`;
- crear plan menor a 400 Bs;
- ocultar degradación.

**Dependencias:** P0-03.

---

### P0-12 — Implementar `UsageLedger` básico y commercial mínimo

**Objetivo:** Registrar consumo por organización desde el inicio y resolver plan/suscripción activa sin fuentes paralelas.

**Entregables:**

- tabla `usage_events`;
- tablas `plans`, `subscriptions`, `research_credits` y `research_credit_movements`;
- seed `PROFESIONAL`, `monthly_research_credits=8` y suscripción/bootstrap grant por organización de desarrollo;
- `CommercialService`/repository para resolver `subscription_id`, `plan_code` y límites visibles;
- service;
- repository;
- endpoint `GET /v1/usage/current` básico;
- endpoint `GET /v1/research-credits` básico;
- tests.

**Eventos mínimos:**

```txt
standard_query
complex_query
research_credit_used
```

**Criterios de aceptación:**

- cada ejecución publicable del usuario registra un usage event `standard_query` o `complex_query` con `quantity=1`;
- cada ejecución publicable `COMPLEJO` registra `research_credit_used` con `quantity=1`;
- cada ejecución publicable `INVESTIGACION` registra `research_credit_used` con `quantity=2`;
- cada `research_credit_used` incluye `trace_id` o `answer_id` y valida contra `docs/contracts/usage-event.schema.json`;
- cada débito `research_credit_used` crea `research_credit_movements.movement_type=debit` enlazado a `usage_event_id`;
- `research_credits.balance_quantity` baja de forma transaccional y nunca queda negativo;
- si el débito `research_credit_used` no puede registrarse, `CostGovernor` bloquea o degrada la ejecución;
- mensajes técnicos `assistant`/`system`/`tool` no generan usage separado;
- budget decision se registra en `budget_decisions` y `run_events`, no como `UsageEvent.event_type` inventado;
- endpoint devuelve consumo mensual básico;
- endpoint devuelve `period_start`, `period_end`, `plan_code`, `subscription_id`, `usage_totals`, `limits` y `research_credits_balance`;
- `GET /v1/research-credits` devuelve `balance`, `currency`, `movements[]` y `updated_at`, sin exponer `UsageEvent` raw;
- `subscription_id` se resuelve desde `subscriptions`, no desde `organizations.plan_code`;
- existe una sola suscripción current por organización;
- usage está asociado a `organization_id`;
- usage usa `actor_ref`/`actor_type` sin PII directa;
- eventos de tokens (`model_input_tokens`, `model_output_tokens`) se habilitan solo después de existir `model_call_id` por P0-13/P1-05.

**Errores a evitar:**

- usage sin tenant;
- calcular todo desde logs;
- no guardar unidad/cantidad.

**Dependencias:** P0-04, P0-08, P0-11.

**Integración:** Los criterios de débito `research_credit_used` que requieren `trace_id` o `answer_id` se validan en la unidad atómica P0-12+P0-13 de PR-6; esta nota no agrega una dependencia de P0-12 sobre sí mismo ni invierte la dependencia P0-13 -> P0-12.

---

### P0-13 — Implementar estructura de trazabilidad mínima

**Objetivo:** Persistir una reconstrucción mínima de cada ejecución.

**Entregables:**

- `operational_runs`;
- `run_events`;
- `model_calls`;
- `tool_calls`;
- `answers`;
- `answer_versions`;
- `abstention_renders`;
- `trace_objects`;
- mensaje tecnico `assistant_final` para `TraceObject.output_message_id`;
- service de tracing;
- endpoint interno/protegido `GET /internal/runs/{run_id}/events` o equivalente;
- tests.

**Criterios de aceptación:**

- cada message pipeline crea run;
- cada run registra al menos eventos internos cerrados de P0: run_created, message_received, message_persisted y budget_checked;
- `usage_event_recorded` se registra solo cuando existe un `UsageEvent` valido y enlazado para una ejecucion publicada; `run_failed` no fuerza usage ni billing por si mismo;
- `run_failed` interno se registra en P0 cuando una ejecución falla después de crear `operational_run`;
- `stream_event_emitted`, `run_completed` y el `run_failed` público SSE permanecen en el enum operativo, pero su emisión obligatoria pertenece a P1-04 al integrar SSE y cierre del pipeline;
- model provider stub registra model_call si se invoca;
- errores quedan asociados a run.
- `operational_runs` y `run_events` son tablas operativas, no entidades canonicas nuevas.
- `run_id` expuesto usa forma `tr_*` y solo se considera `TraceObject` valido cuando existe `trace_objects.trace_id` con `answer_id`, `answer_version_ref`, `response_outcome=blocked`, `abstention_reason=policy_blocked` y `trace_object_payload` validado contra `docs/contracts/trace-object.schema.json`, incluido `citation_audit` minimo para `policy_blocked`;
- los shells de `answer_blocked` se crean en una sola transaccion con FKs circulares deferrable o estrategia equivalente documentada;
- `abstention_renders` incluye `source_trace_refs`;
- para bloqueos/abstenciones, `AnswerVersion.answer_hash` coincide con `AbstentionRender.render_hash`;
- `AbstentionRender.render_storage_ref` apunta a `render_body_canonical` DB-local, reconstruible sin S3/filesystem/storage externo, y `render_hash=sha256(canonicalize(render_body_canonical))`;
- para bloqueos/abstenciones, `AbstentionRender` resuelve al mismo `trace_id`, `answer_id`, `answer_version` y `response_outcome` que `AnswerVersion`/`TraceObject`;
- `AbstentionRender.source_trace_refs` es subconjunto real de `TraceObject.evidence_pack_ids[]`, `TraceObject.claims[].claim_id`, citas en `TraceObject.citation_audit` y fuentes usadas/rechazadas/auditadas, conforme a `answer-versioning-policy.md`;
- eventos internos de `run_events` no se exponen como taxonomia publica SSE.

**Errores a evitar:**

- responder sin run_id;
- logs sin persistencia;
- tool/model calls imposibles de auditar.

**Dependencias:** P0-04, P0-06, P0-10, P0-11, P0-12.

---

## 5. P1 — Después del esqueleto

### P1-01 — Implementar `EvidencePack` models

**Objetivo:** Preparar contratos de evidencia sin construir retrieval real.

**Entregables:**

- Pydantic schemas y modelos ORM/stubs para `evidence_packs`, `evidence_sources`, `evidence_passages`, `claims` y `citations`;
- `0003_evidence_contract_stubs.py` como migración de tablas stubs contractuales;
- columnas mínimas tenant-scoped: `organization_id` obligatorio en `evidence_packs`, `evidence_sources`, `evidence_passages`, `claims` y `citations`;
- refs contractuales: `evidence_pack_id=ep_*`, `query_id=q_*`, `retrieval_run_id=rr_*` opcional/nullable, `claim_id=cl_*`, `citation_ref=C#`, `source_ref=F#|D#` y `passage_ref=F#:P#|D#:P#`;
- payloads JSONB schema-valid para `Source`, `Passage`, `Claim` y `Citation`, sin exponer columnas internas DB-only en el payload contractual;
- fixtures de prueba.

**Criterios de aceptación:**

- schema valida source/passages;
- `0003_evidence_contract_stubs.py` queda asignada a este trabajo y crea las tablas stub sin endpoints;
- `evidence_packs.organization_id` coincide con `LegalSearchQuery` resuelta por `query_id`; si `retrieval_run_id` existe, tambien coincide con el `RetrievalRun` resuelto;
- packs manuales/documentales no inventan `rr_*` falso para satisfacer la tabla stub;
- `retrieval_run_id` se declara siempre y puede ser `null` en packs manuales/documentales; esos packs cuentan como contractuales si validan el schema completo y nunca inventan un `rr_*`;
- `quality` usa exactamente `evidence-quality.schema.json`, sin DTO inline paralelo;
- `evidence_sources` y `evidence_passages` heredan `organization_id` de `evidence_packs`;
- `citations` usa identidad `(answer_version_ref, citation_ref)` y resuelve `source_ref`/`passage_ref` dentro del mismo `evidence_pack_id`;
- `citations.passage_ref` resuelve a un `evidence_passages` cuyo `source_ref` coincide exactamente con `citations.source_ref`, y el prefijo de `passage_ref` antes de `:P#` coincide con ese `source_ref`;
- `Citation.supports_claim_ids[]` solo puede apuntar a claims del mismo `organization_id` y `answer_version_ref`; no puede soportar claims de otra version ni claims aun no enlazados a version;
- `Claim.claim_payload.citations[]` y `Citation.supports_claim_ids[]` son consistentes en ambas direcciones dentro de la misma `answer_version_ref`;
- columnas canonicas y payloads JSONB coinciden para IDs/refs/organization: source, passage, claim y citation;
- no hay filas de evidencia, claims o citas sin `organization_id`;
- no implica RAG;
- no requiere OpenSearch.

**Dependencias:** P0-04.

---

### P1-02 — Implementar `Citation` y `Claim` schemas y validadores

**Objetivo:** Preparar Fase 2 de Citation Auditor y bloqueo de claims críticos.

**Entregables:**

- `Claim` schema;
- `Citation` schema;
- `CitationAudit` fixture/value object embebido en `TraceObject`, sin tabla standalone;
- interface y fixtures de `ClaimCompletenessValidator` para que Fase 2 implemente el validador productivo sin depender del output autoevaluado por el modelo;
- fixtures del oracle semántico de claims con `expected_claim_safe_text`, `expected_claim_text_hash` y `semantic_match_mode` conforme al golden dataset;
- tests de refs contractuales `F1:P1` y `D1:P1`.

**Criterios de aceptación:**

- referencias inválidas fallan validación;
- claims incluyen `criticality`/`support_level`, y los tipos jurídicos críticos cerrados por `claim.schema.json` no pueden degradarse a `criticality=low|medium`;
- citation target queda explícito;
- una afirmación jurídica crítica visible omitida de `claims[]` produce `critical_assertion_unmapped` en el fixture del validador;
- el match semántico usa el oracle esperado y no `Claim.verification_status` ni `CitationAudit.support_assessment` como fuente de verdad;
- el validador productivo y el bloqueo real permanecen asignados a Fase 2 por ADR-007.

**Dependencias:** P1-01.

---

### P1-03 — Implementar validador de schemas

**Objetivo:** Garantizar que los contratos JSON del repo son válidos.

**Entregables:**

- script `scripts/validate_schemas.py`;
- JSON Schemas en `docs/contracts`;
- test o make target.

**Criterios de aceptación:**

- `make validate-schemas` pasa;
- falla si schema está mal formado;
- documentación indica cómo agregar schemas.

**Dependencias:** P0-01.

---

### P1-04 — Implementar endpoint de conversación simple con streaming de estados

**Objetivo:** Probar pipeline de run + budget + trace + usage + SSE.

**Entregables:**

- `POST /v1/conversations/{conversation_id}/messages`;
- `GET /v1/conversations/{conversation_id}/runs/{run_id}/stream`;
- eventos SSE técnicos.

**Eventos SSE publicos minimos:**

```txt
run_queued
run_started
answer_blocked
run_failed
```

`answer_blocked` es el cierre publico esperado cuando el pipeline tecnico termina sin emitir analisis juridico. Debe incluir `answer_id`, `answer_version_ref` y `trace_summary_url`, respaldado por `AnswerVersion.response_outcome=blocked`, `AbstentionRender.reason_code=policy_blocked` y `TraceObject` schema-valid con `abstention_reason=policy_blocked` y `citation_audit` minimo contract-compatible. Eventos como `run_created`, `message_received`, `message_persisted`, `budget_checked`, `model_provider_stubbed`, `usage_event_recorded`, `stream_event_emitted` y `run_completed` son internos de `run_events`; `usage_event_recorded` solo se emite si hay `UsageEvent` valido enlazado.

**Criterios de aceptación:**

- no emite respuesta legal;
- genera run_id;
- genera usage solo si la ejecucion queda publicada con `UsageEvent` valido enlazado;
- genera trace;
- emite terminal publico `answer_blocked` en ejecucion exitosa tecnica;
- registra `stream_event_emitted` cuando emite SSE;
- registra `run_completed` al cerrar con `answer_blocked`, o `run_failed` si falla antes de crear `AnswerVersion`/`TraceObject`;
- no expone analisis juridico ni `answer_final` en Sprint 1;
- puede ser consumido por frontend.

**Dependencias:** P0-08, P0-10, P0-11, P0-12, P0-13.

---

### P1-05 — Completar repos/services/tests de model calls/tool calls

**Objetivo:** Completar la capa de acceso y validación sobre las tablas/DTOs creados por P0-13.

**Entregables:**

- repositories;
- service;
- tests.

**Criterios de aceptación:**

- model_call registra campos requeridos por `docs/contracts/model-call.schema.json`;
- tool_call registra campos requeridos por `docs/contracts/tool-call.schema.json`;
- tool/model calls guardan hashes, contadores y metadata minimizada; no guardan payloads crudos ni resumidos libres.
- eventos de tokens se anclan a `model_call_id` cuando exista llamada de modelo stub;

**Dependencias:** P0-13.

---

### P1-06 — Implementar tests de Cost Governor

**Objetivo:** Bloquear cambios que rompan presupuestos.

**Tests requeridos en PR-5, solo budget puro:**

- plan `PROFESIONAL` existe como política comercial base de 400 Bs/mes;
- `SIMPLE` permite;
- `MEDIO` permite;
- `COMPLEJO` resuelve `research_credit_cost=1`;
- `INVESTIGACION` resuelve `research_credit_cost=2`;
- plan inexistente falla;
- YAML inválido falla;
- límites usan `discovery_calls_max`.

**Tests de débito/conciliación en PR-6:**

- `COMPLEJO` registra `research_credit_used` con quantity `1`;
- `INVESTIGACION` registra `research_credit_used` con quantity `2`;
- el débito comparte `trace_id` o `answer_id` con la ejecución publicada;
- el movimiento de crédito queda enlazado al `usage_event_id`;
- saldo insuficiente bloquea o degrada.

**Criterios de aceptación:**

- tests cubren allow/degrade/block;
- cambios de budget requieren actualizar fixture.

**Dependencias:** P0-11 para budget puro; P0-12 y P0-13 para débito/conciliación.

---

### P1-07 — Implementar foundations de beta blockers de Fase 1

**Objetivo:** Cubrir los blockers de beta asignados a Fase 1 sin activar proveedores reales ni lógica jurídica completa.

**Entregables:**

- `ProviderReliabilityLayerStub`;
- `ProviderCallAuditService` sobre la tabla `provider_call_audits` creada por `0002_trace_usage_budget`;
- `RawAccessAuditService` sobre la tabla `raw_access_events` creada por `0002_trace_usage_budget`;
- `PromptInjectionGuardStub`;
- tests de timeout/rate limit/provider unavailable contra `ErrorEnvelope`;
- fixtures schema-valid y policy-valid de `ProviderCallAudit`, `RawAccessEvent` y `PromptInjectionRisk`.

**Criterios de aceptación:**

- Provider Reliability Layer (PRL) básico devuelve errores controlados sin red real;
- PRL interpreta `max_attempts` como intentos totales, ejecuta como máximo un retry para una operación idempotente y nunca reintenta errores de policy/validación;
- `RetrievalPlan.fallback_allowed=true` es permiso y no selector; PRL usa fallback solo si existe un `ProviderRoute` activo compatible con familia, tenant, allowlist y budget;
- `provider-registry.yaml` mantiene `fallback_routes: []`, por lo que la ejecución productiva sin ruta devuelve `provider_unavailable`; el test positivo usa un fixture local cerrado con dos providers stub distintos de la misma familia;
- el modo `external_call_mode` del registry resuelve de forma determinista `ProviderMetadata.external_call` antes de habilitar cada provider;
- cada intento persiste contexto completo (`logical_call_id`, request/correlation, número/tipo de intento e idempotencia), con `(logical_call_id, attempt_number)` único;
- retries/fallback no duplican `UsageEvent` ni cargos; `ModelCall`/`ToolCall.provider_call_audit_id` apunta al intento terminal y los intentos previos se resuelven por la ref dueña + `logical_call_id`;
- toda llamada externa futura falla si no puede crear `provider_call_audit_id`;
- `ProviderCallAuditService` persiste los campos de `x-reliability-policy.required_per_attempt` y las refs contractuales `trace_id|model_call_id|tool_call_id|retrieval_run_id|usage_event_id|cost_report_id`; fixtures contables no pueden quedar sin al menos una ref resoluble y tenant-compatible;
- `ProviderCallAuditService` rechaza `provider_name` desconocido, `provider_family` divergente, clases enviadas/devueltas fuera de allowlist y estados incompatibles con `provider-policy.md`/`provider-registry.yaml`;
- `ProviderCallAuditService` acepta `attempted_data_classes[]` conocidas fuera de allowlist solo cuando `status=policy_blocked`, `data_sent_classes=[]` y no hubo payload enviado;
- todo acceso raw/elevado falla si no puede crear `raw_access_event_id` policy-valid;
- `RawAccessAuditService` rechaza `approved_by_ref == actor_ref`, `expires_at <= accessed_at` y refs documentales incompletas o presentes fuera de `resource_type=document_evidence`;
- riesgo prompt injection `blocking` se bloquea o excluye en fixtures;
- cuando `PromptInjectionGuardStub` opera en frontera API debe devolver `ErrorEnvelope.error_code=prompt_injection_blocked`;
- no crea migración paralela para `provider_call_audits` ni `raw_access_events`;
- los tests no requieren OpenAI, internet ni documentos reales.

**Dependencias:** P0-09, P0-10, P0-13.

---

### P1-08 — Implementar `StorageProvider` stub y límites privados

**Objetivo:** Cumplir ADR-002/ADR-011 sin subir documentos reales ni acoplar un proveedor único.

**Entregables:**

- `StorageProvider` interface;
- `StorageProviderStub`;
- `StorageService` con validación de límites privados;
- tests fail-closed de escritura sin provider;
- tests de keys/rutas sin tenant, usuario, nombre de archivo original ni texto raw.

**Criterios de aceptación:**

- no escribe documentos reales en Fase 1;
- cualquier escritura no habilitada falla con `ErrorEnvelope.error_code=storage_unavailable`;
- las rutas/keys de storage son no adivinables y no exponen PII ni raw filenames;
- health/readiness reporta object storage con `enabled=false`, `status=UNKNOWN` cuando `ENABLE_S3=false`.

**Dependencias:** P0-03, P0-07, P0-09.

---

### P1-09 — Implementar `AuthProvider` productivo pre-beta

**Objetivo:** Reemplazar dev auth antes de beta sin acoplar el dominio a un proveedor concreto.

**Entregables:**

- `AuthProvider` interface estable y adapter productivo seleccionado por ADR o decision de deployment;
- validacion de firma, `issuer`, `audience`, expiracion y revocacion/estado cuando el proveedor lo soporte;
- resolucion `external_subject -> actor_ref -> membership activa -> organization_id`;
- configuracion fail-closed y `DevAuthProvider` restringido a `APP_ENV=local|test`;
- tests negativos de token invalido/expirado, issuer/audience incorrectos, membership inactiva y acceso cross-tenant.

**Criterios de aceptación:**

- ningun request beta usa `X-Dev-Organization-Id` o `X-Dev-User-Id` como autenticacion;
- un token valido sin membership activa no obtiene tenant context;
- el tenant no se acepta desde un header/body controlado por cliente si contradice la membership;
- el adapter no filtra tipos de SDK externo al dominio ni a contratos publicos;
- OQ-020 queda `Resolved|Closed` y la evidencia requerida por el gate beta queda versionada.

**Dependencias:** P0-03, P0-08, P0-09.

---

## 6. P2 — Preparación de Fase 2/3/4

### P2-01 — Source Registry seed

**Objetivo:** Registrar fuentes iniciales sin fetch real.

**Fuentes mínimas:**

- Gaceta Oficial de Bolivia;
- TCP;
- TSJ/Génesis;
- SILEP;
- dominios oficiales `.gob.bo`.

**Criterios de aceptación:**

- seed aplica por la migración `0004_source_registry_seed.py`;
- `0004_source_registry_seed.py` queda asignada a este trabajo y aplica el seed de forma idempotente;
- cada fuente tiene `source_registry_entry_id` con forma `src_*`, `code`, `name`, `tier`, `source_type` y `live_fetch_enabled=false`;
- `code` es alias operativo/idempotency key, no reemplaza `source_registry_entry_id`;
- `source_registry_health` es tabla separada, no columna alternativa: una fila 1:1 por `source_registry_entry_id`, con `health_status=UNKNOWN`, `last_checked_at=null` y `last_error_code=null` para todo seed de Fase 1;
- `health_status` usa `docs/schemas/host-statuses.yaml`; el estado disabled se expresa con `live_fetch_enabled=false`, no con un enum paralelo;
- la idempotencia cubre `source_registry` y `source_registry_health`, sin duplicar filas al re-ejecutar `0004_source_registry_seed.py`;
- `tier` y `source_type` usan enums de `docs/contracts/source.schema.json`;
- no hay fetch real.

**Dependencias:** P0-04.

---

### P2-02 — Fetcher stub

**Objetivo:** Definir interfaz de recuperación externa.

**Entregables:**

- `Fetcher` protocol/interface;
- `FetcherStub`;
- tests.

**Criterios de aceptación:**

- no hace red real;
- devuelve respuesta fixture;
- registra tool_call si se invoca.

**Dependencias:** P0-13.

---

### P2-03 — Snapshot stub

**Objetivo:** Preparar registro futuro de snapshots.

**Entregables:**

- `SnapshotProvider` interface;
- `SnapshotStub`;
- DTO/shape operativo interno de snapshot stub, sin JSON Schema nuevo ni contrato paralelo.

**Criterios de aceptación:**

- genera `snapshot_id` stub con forma `snap_*`;
- no escribe archivos reales salvo fixture opcional;
- el shape interno incluye hash/content_type/retrieved_at y no sustituye `source.schema.json` ni `legal-search-result.schema.json`.

**Dependencias:** P0-13.

---

### P2-04 — OpenSearch connection stub

**Objetivo:** Preparar búsqueda sin obligarla en Sprint 1.

**Entregables:**

- settings OpenSearch;
- cliente stub;
- readiness usa `enabled=false,status=UNKNOWN` cuando OpenSearch está deshabilitado, o `enabled=true,status=HEALTHY|DEGRADED|DOWN|UNKNOWN` cuando está habilitado;
- tests con disabled.

**Criterios de aceptación:**

- app arranca sin OpenSearch si `ENABLE_OPENSEARCH=false`;
- no hay dependencia dura;
- no indexa corpus.

**Dependencias:** P0-03, P0-07.

---

### P2-05 — Evaluation runner skeleton

**Objetivo:** Preparar evaluación desde la fundación.

**Entregables:**

- `src/jusnova/modules/evals/runner_stub.py`;
- comando `make eval-smoke`;
- fixture vacío para smoke;
- estructura de reporte compatible con el reporte versionado exigido por `docs/quality/beta-readiness-gates.md`.

**Criterios de aceptación:**

- corre sin dataset real;
- produce reporte técnico mínimo;
- no evalúa respuestas jurídicas aún.
- el skeleton de Sprint 1 no satisface beta por sí solo;
- antes de beta, este workstream debe producir reporte versionado con `dataset_version`, versión de golden dataset y métricas/gates de `docs/quality/evaluation-plan-v0.md` y `docs/quality/initial-golden-dataset-spec.md`.

**Dependencias:** P0-01.

---

### P2-06 — WorkflowGateway y LocalWorkflowGateway

**Objetivo:** Implementar el contrato ejecutable requerido para cerrar OQ-002 sin acoplar el core a Temporal.

**Entregables:**

- `src/jusnova/workers/workflow_gateway.py`;
- `src/jusnova/workers/local_workflow_gateway.py`;
- tests unitarios del gateway local;
- nota corta de integración futura con Temporal.

**Criterios de aceptación:**

- el core de conversación depende de la interfaz `WorkflowGateway`, no de Temporal;
- `LocalWorkflowGateway` ejecuta o encola de forma determinista en tests, sin servicio externo;
- Temporal no es requerido para `make test` ni para Sprint 1 local;
- al completar este item, OQ-002 puede pasar a `Resolved|Closed`.

**Dependencias:** P0-03, P0-13.

---

## 7. Sprint 1 cutline recomendado

### Debe cerrarse sí o sí

- P0-01 a P0-13.

### Ideal si el equipo avanza rápido

- P1-04;
- P1-06;
- P1-07;
- P2-01.

### Obligatorio antes de beta aunque no entre en Sprint 1

- P1-07;
- P1-08;
- P1-09;
- P2-05 en modo beta report versionado con `dataset_version` y resultado de gates 0.12.

### Obligatorio antes de cerrar Fase 1 aunque no entre en Sprint 1

- P2-06, para que `WorkflowGateway` no quede solo como decisión documental.

### No forzar si compromete calidad

- P1-01/P1-02 tablas completas;
- P2-04 OpenSearch connection stub/profile opcional;
- P2-05 como reporte beta dentro de Sprint 1; el smoke skeleton puede quedar como preparación, pero beta queda bloqueada sin reporte versionado.

---

## 8. Pull request strategy

### PR-1 — Repo + FastAPI + settings

Incluye:

- P0-01;
- P0-02;
- P0-03;
- health live simple.

### PR-2 — DB + migraciones + modelos base

Incluye:

- P0-04;
- P0-08;
- tests de migración.

### PR-3 — Request context + logging + ErrorEnvelope

Incluye:

- P0-05;
- P0-06;
- P0-09.

### PR-4 — Health ready + dependencies

Incluye:

- P0-07;
- Docker Compose con Postgres/Redis y profile `search` para OpenSearch.

### PR-5 — ModelProvider stub + CostGovernor + budgets

Incluye:

- P0-10;
- P0-11;
- tests P1-06 de budget puro, sin débito ni conciliación.

### PR-6 — UsageLedger + commercial + tracing

Incluye:

- P0-12;
- P0-13;
- persistencia mínima;
- tests P1-06 de débito/conciliación con `trace_id` o `answer_id`.

PR-6 es unidad atómica para research credits: no se mergea el débito `research_credit_used` sin los anchors mínimos de P0-13.

### PR-7 — Conversation message pipeline

Incluye:

- P1-04 si entra.

### PR-8 — Stubs/contracts P1/P2

Incluye:

- P1-01;
- P1-02;
- P1-03;
- P1-05;
- P1-07;
- P1-08;
- P2-01 a P2-06 si el sprint lo permite.

### PR-9 — Auth productiva pre-beta

Incluye:

- P1-09;
- decision versionada del adapter;
- tests de token, membership y tenant isolation;
- eliminacion de cualquier ruta de dev auth en configuracion beta/production.

---

## 9. Quality gates por PR

Cada PR debe pasar:

```txt
[ ] Tests del módulo.
[ ] No secretos.
[ ] No llamadas externas reales no autorizadas.
[ ] ErrorEnvelope respetado si toca API.
[ ] request_id/correlation_id respetado si toca request pipeline.
[ ] Logs no exponen datos sensibles.
[ ] Migración incluida si cambia modelo.
[ ] Documentación breve si agrega contrato.
[ ] No introduce generación jurídica real.
```

---

## 10. Riesgos del Sprint 1

### Riesgo crítico: intentar hacer demasiado

Mitigación:

- cortar en P0;
- no implementar retrieval real;
- no crear prompts legales;
- no implementar OCR.

### Riesgo crítico: CostGovernor decorativo

Mitigación:

- tests obligatorios;
- toda ejecución crea budget decision;
- no hardcodear budget.

### Riesgo crítico: trazabilidad solo en logs

Mitigación:

- persistir `operational_runs` y `run_events`;
- logs son complemento, no fuente única.

### Riesgo crítico: modelos sin tenant

Mitigación:

- organization_id obligatorio en conversations/messages/runs/usage/trace;
- tests de constraints.

### Riesgo crítico: usar OpenAI real en tests

Mitigación:

- stub default;
- feature flag bloqueado para llamadas reales;
- CI sin API key.

---

## 11. Resultado esperado del Sprint 1

Al terminar Sprint 1, el equipo debe poder ejecutar:

```bash
make dev
make migrate
make test
```

y validar:

1. backend arriba;
2. base migrada;
3. health checks sanos;
4. conversación creada;
5. mensaje creado por `MessageService` o endpoint si P1-04 entra;
6. run creado por el message pipeline interno;
7. budget decision creada;
8. usage registrado;
9. trace registrado;
10. eventos internos P0 registrados en `run_events`;
11. sin llamadas externas reales;
12. sin respuesta jurídica real.

Si P1-04 entra en Sprint 1, también debe validarse que los eventos SSE públicos se emiten con la taxonomía del API draft.

---

## 12. Cierre del Sprint 1

Sprint 1 queda aprobado solamente si P0 completo está funcionando y testeado.

Si P1/P2 quedan incompletos, no es problema. Si P0 queda incompleto, Fase 1 no puede avanzar.

La regla final:

> **No se acepta ningún avance visual o conversacional que no deje trazabilidad, presupuesto y persistencia correcta.**
