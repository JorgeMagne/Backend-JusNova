# JusNova — Sprint 1 Backlog

**Ruta objetivo en repo:** `/docs/handoff/sprint-1-backlog.md`
**Versión:** 1.0
**Fecha:** 2026-05-26
**Estado documental:** Accepted
**Condicion de ejecucion:** Habilitada por el cierre formal de 0.14/Fase 0
**Responsable:** Codex / JusNova Chief Backend Architect
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
[ ] PromptVersionRegistry interno resuelve toda `ModelCall.prompt_version` contra manifest/path/hash versionados.
[ ] `docs/schemas/budgets.yaml` existe.
[ ] CostGovernor carga budgets y devuelve decisiones.
[ ] UsageLedger registra exactamente un evento de consulta por ejecución publicada: `standard_query` para `SIMPLE|MEDIO` o `complex_query` para `COMPLEJO|INVESTIGACION`, con `trace_id`, `answer_id` y budget conciliados.
[ ] `research_credit_used` se debita para `COMPLEJO`/`INVESTIGACION` en la misma transacción terminal que respuesta/traza/cierre; sin saldo, incluida concurrencia por el último crédito, se devuelve `research_credit_required` sin publicar ni crear debito ficticio, y Fase 1 no reduce la complejidad para evitar el credito.
[ ] Plan/Subscription/ResearchCredit y CommercialService mínimos resuelven suscripción activa y saldo de créditos.
[ ] Trazabilidad mínima persiste run/trace events.
[ ] El pipeline de mensajes exige `Idempotency-Key`; retries secuenciales/concurrentes no duplican mensaje, run, usage o créditos y la key raw no se persiste/loguea.
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
- `.gitleaks.toml` con allowlist puntual, sin exclusiones amplias;
- `uv.lock` o lockfile del gestor elegido;
- dependencia directa `rfc8785==0.1.4` fijada en `pyproject.toml` y lockfile;
- dependencia directa `PyYAML==6.0.3` fijada en `pyproject.toml` y lockfile;
- `docker-compose.yml`;
- `Makefile`;
- carpetas `src/jusnova`, `tests`, `docs`, `alembic`.

**Criterios de aceptación:**

- `make help` muestra comandos disponibles;
- `make test` existe aunque todavía falle si no hay tests;
- `make dev` o equivalente levanta backend;
- `make scan-secrets` ejecuta el scanner versionado y falla ante una clave sintética de prueba;
- `src/jusnova/core/canonical_json.py` expone un único helper JCS y `test_canonical_json.py` cubre vectores RFC 8785, claves duplicadas y números no admitidos;
- `src/jusnova/core/config_documents.py` expone el único loader YAML estricto y `test_config_documents.py` cubre claves duplicadas, aliases/anchors, merge keys, tags explícitos, múltiples documentos, metadata, campos desconocidos y referencias cruzadas;
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
- migraciones son reproducibles;
- columnas escalares y arrays JSONB que materializan contratos aplican `minLength: 1`/`maxLength`/`maxItems` iguales o mas estrictos que `docs/contracts/*.schema.json`; nullable no admite `""` como sustituto de `null`;
- repositorios validan el payload completo y la igualdad columna-payload antes de persistir, sin truncado silencioso;
- tests de migracion/repositorio rechazan string vacio, aceptan el limite, rechazan `limite + 1` y rechazan al menos una divergencia columna-payload por familia de tabla contractual.

**Errores a evitar:**

- crear tablas manualmente;
- modelos sin migración;
- usar sync/async mezclado sin criterio;
- no indexar claves de consulta básicas;
- confiar solo en validacion HTTP y dejar columnas/JSONB sin defensa de limites.

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

- el formatter estructurado incluye `request_id`/`correlation_id` cuando existen en el contexto;
- el gate integrado de PR-3, despues de P0-06, prueba que cada request y cada error controlado incluyen `request_id`;
- no aparecen secrets en logs;
- formato consistente.

**Errores a evitar:**

- logs planos imposibles de consultar;
- stack traces expuestos en producción;
- loggear bodies completos con datos sensibles.

**Dependencias:** P0-02.

P0-06 integra el middleware de contexto en el mismo PR-3; no es una dependencia previa de P0-05 ni altera el orden P0.

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
- `ConversationCreate.case_id` solo acepta `null`/omitido en Fase 1 y cualquier `case_*` devuelve `validation_error` antes de persistir;
- no existen refs `case_*` huérfanas ni asociaciones de caso sin validación de tenant;
- `MessageService` recibe la idempotency key como dato de frontera, persiste solo `idempotency_key_hash`/`request_fingerprint` DB-only y no los serializa dentro de `Message`;
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
- `PromptVersionRegistry` y loader fail-closed;
- manifest `src/jusnova/modules/model_provider/prompts/registry.yaml`;
- seed técnico `phase-1-pipeline-check-v1.txt` con versión/hash inmutables;
- schema `ModelRequest`;
- schema `ModelResponse`;
- persistencia opcional de `model_call` si P0-13 ya está listo;
- tests.

**Criterios de aceptación:**

- ningún módulo llama OpenAI directamente;
- provider default es stub;
- stub devuelve respuesta determinista;
- cada `ModelCall.prompt_version` resuelve antes de invocar el provider o persistir la llamada;
- el registry rechaza versión desconocida/duplicada, path inexistente y hash divergente;
- cambiar template, contenido o purpose exige crear un `prompt_version` nuevo;
- solo `status=active` puede usarse en llamadas/`ModelCall` nuevos; `deprecated` solo resuelve auditoría o replay controlado histórico sin invocar provider;
- la transición de estado permitida es `active -> deprecated`; una versión usada no se elimina ni se reactiva;
- feature flag impide llamada real en Fase 1.

**Errores a evitar:**

- importar SDK de OpenAI en routers;
- meter prompts legales en Fase 1;
- aceptar `prompt_version` libre, mutable o no resoluble;
- seleccionar un prompt `deprecated` para una llamada nueva;
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
- cada `BudgetDecision` persistido identifica `budget_decision_id`, `run_id`, `organization_id`, `cost_budget_ref`, `cost_budget_version`, `budget_snapshot` y `created_at`; `run_id`/`organization_id` deben resolver al mismo tenant;
- existe exactamente una `budget_decision` confirmada por `operational_run`; `run_id` no es nulo y mode/complejidad/tenant coinciden con el run;
- `budget_snapshot` coincide exactamente con los limites del `CostBudget` resuelto y sus escalares coinciden en plan/complejidad/version; `decision_reason_code` usa `within_budget|nonessential_output_capped|budget_exhausted|research_credit_required|policy_blocked`, sin texto libre;
- Fase 1 no degrada complejidad: `degrade` solo recorta salida no esencial dentro de la misma complejidad, y falta de research credits siempre produce `block/research_credit_required`;
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

- cada ejecución publicable del usuario registra exactamente un usage event con `quantity=1`: `standard_query` para `SIMPLE|MEDIO` o `complex_query` para `COMPLEJO|INVESTIGACION`; una traza no admite ambos ni duplicados por retry;
- cada ejecución publicable `COMPLEJO` registra `research_credit_used` con `quantity=1`;
- cada ejecución publicable `INVESTIGACION` registra `research_credit_used` con `quantity=2`;
- el evento de consulta y cada `research_credit_used` incluyen los mismos `trace_id` y `answer_id`, validan contra `docs/contracts/usage-event.schema.json` y coinciden con `BudgetDecision`/`TraceObject` en tenant, plan, complejidad, `cost_budget_ref` y `cost_budget_version`;
- cada débito `research_credit_used` crea `research_credit_movements.movement_type=debit` enlazado a `usage_event_id`;
- `research_credits.balance_quantity` baja de forma transaccional y nunca queda negativo;
- respuesta/traza, evento de consulta, débito/movimiento cuando aplique y `run_completed` se confirman en una única transacción terminal; si falla cualquier parte se revierte el conjunto y una transacción separada registra `run_failed`;
- cada débito conserva `abs(delta) == UsageEvent.quantity`, incrementa `used_quantity` por esa cantidad y deja `research_credit_movements.balance_after == research_credits.balance_quantity`; la cadena de movimientos cumple `previous.balance_after + delta == balance_after`;
- la cadena inicia con `grant`, `granted_quantity` equivale a grants, `used_quantity` a débitos y `balance_quantity` al ultimo `balance_after`/suma de deltas; Fase 1 solo ejecuta `grant|debit|expiry` y rechaza `adjustment` hasta definir actor, aprobación y motivo cerrados;
- cada credito/periodo tiene exactamente un grant inicial cuyo monto coincide con `plans.monthly_research_credits`; retries no duplican grants y, al cerrar el periodo, se crea exactamente un `expiry` terminal;
- `research_credits.status` es derivado y consistente: `active` usa saldo positivo, `exhausted` saldo cero en periodo vigente y `expired` saldo cero en periodo cerrado tras su `expiry` terminal; ese movimiento usa `delta=-remanente` o `delta=0` si el saldo ya estaba agotado, y no admite movimientos posteriores;
- un periodo `exhausted` no se reactiva en Fase 1: el siguiente grant crea el registro del mes siguiente; `adjustment` y cualquier reactivacion permanecen fail-closed hasta contar con contrato de actor, aprobacion, motivo y conciliacion;
- `billing_period=YYYY-MM` usa el mes calendario UTC del credito/grant, con intervalo semiabierto `[primer dia 00:00:00Z, primer dia del mes siguiente 00:00:00Z)`;
- `grant` y `debit` ocurren dentro de ese intervalo; el `expiry` terminal ocurre en o despues de su limite superior. Los timestamps no retroceden, `UsageEvent.billing_period` de cada debito coincide con el credito y `research_credits.updated_at` coincide con el ultimo movimiento confirmado;
- `research_credits`, `research_credit_movements`, `subscriptions`, `UsageEvent` y los anchors `trace_id|answer_id` de cada debito resuelven al mismo `organization_id`; un mismatch cross-tenant se rechaza;
- si el debito `research_credit_used` no puede registrarse, `CostGovernor` bloquea con `ErrorEnvelope.error_code=research_credit_required`, no publica respuesta, no registra un debito ficticio y no reduce la complejidad para eludir el credito;
- mensajes técnicos `assistant`/`system`/`tool` no generan usage separado;
- budget decision se registra en `budget_decisions` y `run_events`, no como `UsageEvent.event_type` inventado;
- endpoint devuelve consumo mensual básico;
- endpoint devuelve `period_start`, `period_end`, `plan_code`, `subscription_id`, `usage_totals`, `limits` y `research_credits_balance`; `usage_totals` y `limits` son objetos cerrados con las keys canonicas de `api-draft-v0.md`, no mapas libres;
- `GET /v1/research-credits` devuelve `balance`, `currency`, pagina `movements[]` maximo 100, `next_cursor`, `has_more` y `updated_at`, con item cerrado `date|type|delta|balance_after` y sin exponer `UsageEvent` raw; una suscripcion activa agotada devuelve `balance=0` y una organizacion sin suscripcion activa recibe `not_found`;
- `subscription_id` se resuelve desde `subscriptions`, no desde `organizations.plan_code`;
- `BudgetRequest.plan_code` se resuelve internamente desde esa suscripción current y no puede ser elegido ni sobreescrito por el cliente;
- existe una sola suscripción current por organización;
- `active|trialing|past_due` exige `ended_at=null`; `cancelled` exige `ended_at>=started_at` y nunca cuenta como current;
- la suscripción current debe resolver un plan `active` y una `cost_budget_version` existente en `budgets.yaml`; plan inactivo o versión desconocida falla antes de budget/usage;
- usage está asociado a `organization_id`;
- usage usa `actor_ref`/`actor_type` sin PII directa;
- eventos de tokens (`model_input_tokens`, `model_output_tokens`) se habilitan solo después de existir `model_call_id` por P0-13/P1-05.

**Errores a evitar:**

- usage sin tenant;
- calcular todo desde logs;
- no guardar unidad/cantidad.

**Dependencias:** P0-04, P0-08, P0-11.

**Integración:** Los criterios de débito `research_credit_used` que requieren `trace_id` y `answer_id` se validan en la unidad atómica P0-12+P0-13 de PR-6; esta nota no agrega una dependencia de P0-12 sobre sí mismo ni invierte la dependencia P0-13 -> P0-12.

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

- cada message pipeline valido que supera prevalidacion y persiste el Message crea run; errores anteriores no inventan un run;
- cada run confirmado registra `run_created` y `budget_checked` junto con su unico `BudgetDecision` en una transaccion atomica; `message_received` y `message_persisted` son logs estructurados anteriores al run, no `run_events`, porque `input_message_id` ya debe existir al crear el run;
- cada `RunEvent.organization_id` coincide con `operational_runs.organization_id` y `(run_id, sequence)` es único;
- la secuencia por run comienza en `1` con un unico `run_created`, es contigua/creciente y conserva `created_at` no decreciente mediante asignacion transaccional;
- `operational_runs` aplica la maquina cerrada `queued -> running|failed|cancelled` y `running -> completed|failed|cancelled`; `queued -> failed` conserva `started_at=null`, exige `completed_at >= created_at` y solo cubre fallos posteriores al commit del run pero anteriores al inicio; cancelación y estados iniciados conservan timestamps coherentes y ningún terminal se reabre;
- cada run P0 tiene `conversation_id`, `input_message_id`, `actor_ref` y `actor_type` no nulos; el mensaje iniciador es unico, pertenece al mismo tenant/conversation y comparte actor, sin copiar `user_id` crudo a la superficie operativa;
- si existe `TraceObject`, incluye ese `input_message_id` y coincide con run/mensaje en tenant, conversation y actor;
- `usage_event_recorded` se registra solo cuando existe un `UsageEvent` valido y enlazado para una ejecucion publicada; `run_failed` no fuerza usage ni billing por si mismo;
- `run_failed` interno se registra en P0 cuando una ejecución falla después de crear `operational_run`;
- `run_created`, `budget_checked` y `run_execution_started` tienen unicidad por `(run_id,event_name)`; `run_execution_started` se registra exactamente una vez al confirmar `queued -> running`, y una cancelacion o fallo desde `queued` no lo inventa;
- todo run terminal registra exactamente uno de `run_completed|run_failed|run_cancelled`, coherente con su estado; `budget_checked` siempre resuelve la decision creada atomicamente con el run, y `model_provider_stubbed`, `usage_event_recorded`, `stream_event_emitted` y `error_envelope_returned` solo existen cuando resuelven el artefacto o accion real de esa etapa; el ultimo conserva solo `error_code`/`safe_message_code` sanitizados y nunca payload crudo;
- `run_completed|run_failed|run_cancelled` pertenecen al lifecycle interno P0 y se emiten cuando el run alcanza ese estado; `stream_event_emitted` solo se vuelve obligatorio en P1-04 al integrar SSE. El `run_failed` publico es un `StreamEvent` separado que solo se emite al ejecutar P1-04;
- model provider stub registra model_call si se invoca;
- los errores posteriores a crear `operational_run` quedan asociados a ese run; errores de validación anteriores no fuerzan crear un run ficticio.
- la creación por `MessageService` reclama de forma atómica `(organization_id, actor_ref, conversation_id, idempotency_key_hash)`: mismo fingerprint reutiliza mensaje/run/resultado y fingerprint distinto devuelve `conflict`, sin duplicar filas, usage o créditos;
- `operational_runs` y `run_events` son tablas operativas, no entidades canonicas nuevas.
- `run_id` expuesto usa forma `tr_*` y solo se considera `TraceObject` valido cuando existe `trace_objects.trace_id` con `answer_id`, `answer_version_ref`, `response_outcome=blocked`, `abstention_reason=policy_blocked` y `trace_object_payload` validado contra `docs/contracts/trace-object.schema.json`, incluido `citation_audit` minimo para `policy_blocked`;
- las columnas de `trace_objects` coinciden exactamente con `trace_object_payload.trace_id|organization_id|answer_id|answer_version_ref|conversation_id|output_message_id|response_outcome|abstention_reason`, y `trace_object_payload.answer_version` coincide con el `AnswerVersion` resuelto; una proyeccion divergente falla aunque cada artefacto valide por separado;
- `trace_objects.data_classification` es DB-only, usa el enum canonico y se calcula como la mayor sensibilidad de mensajes/claims/evidencia/retrieval/riesgos con minimo `INTERNAL_TRACE_RESTRICTED`; no se filtra al payload cerrado y cualquier `RawAccessEvent` de la traza usa exactamente esa clase;
- los shells de `answer_blocked`, el evento de consulta, el débito/movimiento requerido y `run_completed` se crean en una sola transaccion terminal con FKs circulares deferrable o estrategia equivalente documentada; el SSE de éxito solo se emite después del commit;
- un fallo de settlement, incluida una carrera por el último crédito, revierte todos los artefactos finales/usage y una transacción separada cierra `running -> failed`; no queda `Answer` o `TraceObject` publicable;
- `AnswerVersion <-> AbstentionRender` es 1:1: `answer_versions.abstention_render_ref` es unico cuando no es null y `abstention_renders(answer_id, answer_version)` es unico;
- `answers.latest_answer_version_ref` resuelve por FK compuesta deferrable o constraint equivalente a un `AnswerVersion` del mismo `answer_id` y `organization_id`; apunta a la version vigente de mayor `answer_version` y `answers.response_outcome` coincide con esa version;
- `abstention_renders` incluye `source_trace_refs`;
- para bloqueos/abstenciones, `AnswerVersion.answer_hash` coincide con `AbstentionRender.render_hash`;
- `AbstentionRender.render_storage_ref == abstention_render_id`, es unico y apunta al `render_body_canonical` DB-local de esa fila, con shape exacta `{ "content": <string> }`; ese `content` coincide byte a byte con el mensaje `assistant_final` resuelto por `TraceObject.output_message_id`, y `render_hash=sha256(canonicalize(render_body_canonical))` usa JCS (RFC 8785), UTF-8 sin BOM, sin normalizacion Unicode y con rechazo de nombres de propiedad duplicados;
- para bloqueos/abstenciones, `AbstentionRender` resuelve al mismo `trace_id`, `answer_id`, `answer_version` y `response_outcome` que `AnswerVersion`/`TraceObject`;
- `AbstentionRender.source_trace_refs` es subconjunto real de `TraceObject.evidence_pack_ids[]`, `TraceObject.claims[].claim_id`, citas en `TraceObject.citation_audit` y fuentes usadas/rechazadas/auditadas, conforme a `answer-versioning-policy.md`;
- eventos internos de `run_events` no se exponen como taxonomia publica SSE.

**Errores a evitar:**

- responder sin run_id;
- logs sin persistencia;
- tool/model calls imposibles de auditar.
- fijar toda traza a `INTERNAL_TRACE_RESTRICTED` aunque sus entradas sean mas sensibles.

**Dependencias:** P0-04, P0-06, P0-08, P0-09, P0-10, P0-11, P0-12.

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
- `evidence_packs.query_id` resuelve contra un fixture/manifest versionado de `LegalSearchQuery` schema-valid del mismo `organization_id`; si `retrieval_run_id` existe, tambien resuelve un `RetrievalRun` y su `RetrievalPlan` schema-valid del mismo tenant, con el mismo `query_id` y refs reciprocas run/pack;
- `0003` no crea tablas/FKs productivas de `LegalSearchQuery` o `RetrievalRun`: sus owners persisten en Fase 4, y una ref ausente del manifest invalida el fixture P1;
- packs manuales/documentales no inventan `rr_*` falso para satisfacer la tabla stub;
- `retrieval_run_id` se declara siempre y puede ser `null` en packs manuales/documentales; esos packs cuentan como contractuales si validan el schema completo y nunca inventan un `rr_*`;
- `quality` usa exactamente `evidence-quality.schema.json`, sin DTO inline paralelo;
- `EvidencePack.legal_area[]` no contiene duplicados; `sources[].source_ref` y `passages[].passage_ref` son unicos dentro del pack incluso ante payloads distintos con la misma ref;
- cada `EvidencePassage.source_ref` resuelve exactamente una fuente del mismo `EvidencePack`; pasajes huerfanos y colisiones de identidad local fallan validacion;
- cada fuente con `validity_status=VIGENCIA_CONFIRMADA|DEROGADA_CONFIRMADA` resuelve al menos un `EvidencePassage` del mismo pack y mismo `source_ref`; un `Source` aislado schema-valid no confirma vigencia ni derogacion;
- `evidence_sources` y `evidence_passages` heredan `organization_id` de `evidence_packs`;
- `citations` usa identidad `(answer_version_ref, citation_ref)` y resuelve `source_ref`/`passage_ref` dentro del mismo `evidence_pack_id`;
- `citations.passage_ref` resuelve a un `evidence_passages` cuyo `source_ref` coincide exactamente con `citations.source_ref`, y el prefijo de `passage_ref` antes de `:P#` coincide con ese `source_ref`;
- `Citation.supports_claim_ids[]` solo puede apuntar a claims del mismo `organization_id` y `answer_version_ref`; no puede soportar claims de otra version ni claims aun no enlazados a version;
- toda `Citation` contractual tiene `supports_claim_ids[]` no vacio y aparece en al menos un `Claim.claim_payload.citations[]` de la misma version, incluso cuando su `status` no sea `valid`;
- `Claim.claim_payload.citations[]` y `Citation.supports_claim_ids[]` son consistentes en ambas direcciones dentro de la misma `answer_version_ref`;
- la relacion `Claim <-> Citation` es `n:m`, no usa un `claim_id` escalar en `citations`, y ambos arrays son conjuntos sin referencias duplicadas;
- `claim_id` y `citation_ref` son unicos por `answer_version_ref`, y `AnswerContract.sources_used[]` no admite duplicados;
- columnas canonicas y payloads JSONB coinciden para IDs/refs/organization: source, passage, claim y citation;
- no hay filas de evidencia, claims o citas sin `organization_id`;
- no implica RAG;
- no requiere OpenSearch.

**Dependencias:** P0-04, P0-13. `0003_evidence_contract_stubs.py` sigue a `0002_trace_usage_budget.py`; la segunda dependencia aporta los `answer_versions` requeridos por citas y claims versionados.

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
- todo claim incluido en un fixture `AnswerContract.claims[]` usa `verification_status=passed` y `support_level=direct|inferential`; un claim `weak|unsupported` o retenido `failed|needs_review|blocked` no se publica dentro del contrato final;
- citation target queda explícito;
- cada `CitationAudit.results[]` del fixture resuelve una `Citation` de la misma `AnswerVersion`; `(claim_id, citation_ref)` es único, `passage_ref`/`source_ref` coinciden con esa cita y el par existe en la relación bidireccional `Claim.citations[]`/`Citation.supports_claim_ids[]`;
- una afirmación jurídica crítica visible omitida de `claims[]` produce `critical_assertion_unmapped` en el fixture del validador;
- el match semántico usa el oracle esperado y no `Claim.verification_status` ni `CitationAudit.support_assessment` como fuente de verdad;
- el validador productivo y el bloqueo real permanecen asignados a Fase 2 por ADR-007.

**Dependencias:** P1-01.

---

### P1-03 — Implementar validador de schemas

**Objetivo:** Garantizar que los contratos JSON y documentos YAML canónicos del repo son válidos y no ambiguos.

**Entregables:**

- script `scripts/validate_schemas.py`;
- JSON Schemas en `docs/contracts`;
- YAML canónicos en `docs/schemas`, validados mediante `core.config_documents`;
- test o make target.

**Criterios de aceptación:**

- `make validate-schemas` pasa;
- falla si schema está mal formado;
- el parser rechaza nombres de propiedad duplicados antes de JSON Schema/Pydantic y reutiliza `core.canonical_json.loads_unique_json`, sin un segundo parser permisivo;
- el loader YAML compartido rechaza UTF-8 inválido, claves duplicadas en cualquier nivel, aliases/anchors, merge keys, tags explícitos, múltiples documentos, raíz no mapping, metadata faltante, campos desconocidos y referencias cruzadas inválidas;
- los modelos YAML son cerrados; `budgets.yaml` cubre exactamente los planes/complejidades canónicos y `provider-registry.yaml` valida `error_mapping_target=provider_call_audit_error_code` más la traducción cerrada a `ErrorEnvelope` definida por policy;
- una prueba parametrizada recorre los `pattern` positivos que expresan un valor completo y demuestra que cualquier valor valido deja de serlo al agregar `\r`, `\n`, `\r\n`, `U+2028` o `U+2029`; los detectores anidados bajo `not` se prueban por separado como denylists, y el validador no normaliza ni recorta antes de aplicar el schema;
- los campos sanitizados de una linea rechazan controles C0/C1, incluidos NUL, TAB, ESC y NEL, ademas de todos los terminadores de linea; la prueba cubre insercion interna y sufijo;
- el recorrido recursivo falla si cualquier `type: string` variable no declara `minLength: 1`/`maxLength` o cualquier `type: array` no declara `maxItems`; enums y `const` son las unicas formas finitas que no requieren esas keywords;
- los boundary tests generados rechazan string vacio, formato invalido y `limite + 1`, y aceptan el mayor valor que tambien satisfaga pattern/format/item constraints, incluidas propiedades nullable y ramas condicionales;
- documentación indica cómo agregar schemas.

**Dependencias:** P0-01.

---

### P1-04 — Implementar endpoint de conversación simple con streaming de estados

**Objetivo:** Probar pipeline de run + budget + trace + usage + SSE.

**Entregables:**

- `POST /v1/conversations/{conversation_id}/messages`;
- `GET /v1/conversations/{conversation_id}/runs/{run_id}/stream`;
- `GET /v1/answers/{answer_id}/trace-summary` como vista segura que resuelve la URL terminal;
- eventos SSE técnicos.

**Eventos SSE publicos minimos:**

```txt
run_queued
run_started
answer_blocked
run_failed
```

`answer_blocked` es el cierre publico esperado cuando el pipeline tecnico termina sin emitir analisis juridico. Debe incluir `answer_id`, `answer_version_ref` y `trace_summary_url`, respaldado por `AnswerVersion.response_outcome=blocked`, `AbstentionRender.reason_code=policy_blocked` y `TraceObject` schema-valid con `abstention_reason=policy_blocked` y `citation_audit` minimo contract-compatible. Eventos como `run_created`, `budget_checked`, `run_execution_started`, `model_provider_stubbed`, `usage_event_recorded`, `stream_event_emitted`, `run_completed`, `run_failed` y `run_cancelled` son internos de `run_events`; `message_received`/`message_persisted` permanecen en logging estructurado y `usage_event_recorded` solo se emite si hay `UsageEvent` valido enlazado.

**Criterios de aceptación:**

- no emite respuesta legal;
- exige `Idempotency-Key`; retries secuenciales o concurrentes con el mismo fingerprint reutilizan el resultado y una key con payload diferente devuelve `conflict`;
- exige `Idempotency-Key` ASCII de `1..128` y rechaza con `payload_too_large`, antes de persistir/reservar IDs, body JSON decodificado mayor a `131072` bytes, `content` mayor a `20000` code points o mas de `20` attachments;
- acepta `attachments=[]` u omitido y devuelve `document_processing_required` sin persistir mensaje/run ante cualquier adjunto mientras no exista `Document`/`DocumentVersion` tenant-scoped;
- genera run_id;
- genera usage solo si la ejecucion queda publicada con `UsageEvent` valido enlazado;
- publica exactamente un evento de consulta según complejidad; usage, respuesta/traza y `run_completed` se confirman juntos y conservan el mismo budget efectivo;
- una ejecucion tecnica exitosa que termina en `answer_blocked` genera `TraceObject` schema-valid; un `run_failed` temprano conserva `run_id` y trazabilidad operativa sin inventar `TraceObject`;
- una `BudgetDecision.decision=block` nunca inicia ejecucion ni invoca `ModelProvider`: el run confirmado termina `queued -> failed`, devuelve el `ErrorEnvelope` correspondiente y no crea `run_execution_started`, `ModelCall`, usage ni artefactos finales de respuesta/traza;
- emite terminal publico `answer_blocked` en ejecucion exitosa tecnica;
- `answer_blocked.trace_summary_url` apunta a `/v1/answers/{answer_id}/trace-summary`, responde para el mismo `answer_id`/`answer_version_ref`/`trace_id` y nunca expone `TraceObject` crudo;
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
- cada payload completo valida contra su schema, pertenece al mismo tenant que el run, cumple las reglas condicionales de `status`/`error_code`/`provider_call_audit_id` y conserva `completed_at >= started_at`;
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
- el débito y el evento de consulta comparten `trace_id` y `answer_id` con la ejecución publicada y coinciden con `BudgetDecision`/`TraceObject` en tenant y budget efectivo;
- el movimiento de crédito queda enlazado al `usage_event_id`;
- crédito, suscripción, movimiento, `UsageEvent` y anchors resuelven al mismo tenant; el test negativo cross-tenant falla;
- saldo insuficiente devuelve `research_credit_required`, sin respuesta publicada, debito ficticio ni downgrade de complejidad; leer una suscripcion activa agotada devuelve `balance=0`.
- dos ejecuciones concurrentes sobre el último crédito producen como máximo una publicación/débito; la otra revierte todos sus artefactos finales y falla con `research_credit_required`;
- retries no duplican `standard_query|complex_query`, `research_credit_used` ni eventos de tokens para el mismo `model_call_id`.

**Criterios de aceptación:**

- tests cubren allow/degrade/block;
- cambios de budget requieren actualizar fixture.

**Dependencias:** P0-11 para budget puro; P0-12 y P0-13 para débito/conciliación.

---

### P1-07 — Implementar foundations de beta blockers de Fase 1

**Objetivo:** Cubrir los blockers de beta asignados a Fase 1 sin activar providers productivos de modelo, búsqueda, extracción, OCR, storage o workflow ni lógica jurídica completa; el `AuthProvider` productivo se implementa por separado en P1-09.

**Entregables:**

- `ProviderReliabilityLayerStub`;
- `ProviderCallAuditService` sobre la tabla `provider_call_audits` creada por `0002_trace_usage_budget`;
- `RawAccessAuditService` sobre la tabla `raw_access_events` creada por `0002_trace_usage_budget`;
- `PromptInjectionGuardStub`;
- tests de timeout/rate limit/provider unavailable contra `ErrorEnvelope`;
- fixtures schema-valid y policy-valid de `ProviderCallAudit`, `RawAccessEvent` y `PromptInjectionRisk`;
- script `scripts/scan_sensitive_surfaces.py` y `make scan-sensitive-surfaces`;
- fixtures negativos de secretos y de la denylist cerrada de `privacy-security-policy.md` en schemas, logs y trazas.

**Criterios de aceptación:**

- Provider Reliability Layer (PRL) básico devuelve errores controlados sin red real;
- PRL interpreta `max_attempts` como intentos totales, ejecuta como máximo un retry para una operación idempotente y nunca reintenta errores de policy/validación;
- `RetrievalPlan.fallback_allowed=true` es permiso y no selector; PRL usa fallback solo si existe un `ProviderRoute` activo compatible con familia, tenant, allowlist y budget;
- `provider-registry.yaml` mantiene `fallback_routes: []`, por lo que la ejecución productiva sin ruta devuelve `provider_unavailable`; el test positivo usa un registry fixture completo, cerrado y exclusivo de tests con dos providers stub distintos de la misma familia y una ruta entre ellos, cuyos nombres resuelven en ese fixture sin modificar el registry runtime;
- el modo `external_call_mode` del registry resuelve de forma determinista `ProviderMetadata.external_call` antes de habilitar cada provider;
- cada intento persiste contexto completo (`logical_call_id`, request/correlation, número/tipo de intento e idempotencia), con `(logical_call_id, attempt_number)` único;
- retries/fallback no duplican `UsageEvent` ni cargos; `ModelCall`/`ToolCall.provider_call_audit_id` apunta al intento terminal y los intentos previos se resuelven por la ref dueña + `logical_call_id`;
- llamada externa, audits por intento y asignación del `provider_call_audit_id` terminal se confirman en una sola transacción o mediante FKs diferibles equivalentes; el valor nulo solo puede existir de forma transitoria antes del commit y nunca como estado persistido de una llamada externa;
- toda llamada externa futura falla si no puede crear `provider_call_audit_id`;
- `ProviderCallAuditService` persiste los campos de `x-reliability-policy.required_per_attempt` y las refs contractuales `trace_id|model_call_id|tool_call_id|retrieval_run_id|usage_event_id|cost_report_id`; fixtures contables no pueden quedar sin al menos una ref resoluble y tenant-compatible;
- al serializar `ProviderCallAudit`, columnas DB nulas se omiten para refs y campos opcionales que el schema declara omitibles pero no nullable; solo se emite `null` en propiedades cuyo schema lo permite expresamente;
- `ProviderCallAudit.trace_id`, cuando no es nulo, resuelve a un `TraceObject` schema-valid del mismo tenant; un `operational_runs.run_id=tr_*` reservado no cuenta como trace contractual y los intentos previos a la traza final se anclan por otra ref resoluble;
- `ProviderCallAuditService` rechaza `provider_name` desconocido, `provider_family` divergente, clases enviadas/devueltas fuera de allowlist y estados incompatibles con `provider-policy.md`/`provider-registry.yaml`;
- `ProviderCallAuditService` valida que los valores de `provider-registry.error_mapping` sean códigos internos de `ProviderCallAudit` y aplica la traducción pública cerrada de `provider-policy.md`; nunca expone `provider_error` ni `cancelled_by_system` como `ErrorEnvelope.error_code`;
- `ProviderCallAuditService` acepta `attempted_data_classes[]` conocidas fuera de allowlist solo cuando `status=policy_blocked`, `data_sent_classes=[]` y no hubo payload enviado;
- cada audit enlazado a `ModelCall|ToolCall` comparte el `input_hash` del owner; el audit terminal comparte tambien `output_hash`, incluido `null`; el stub calcula el hash una sola vez sobre el DTO JSON minimizado con JCS (RFC 8785) y reutiliza el valor;
- `ProviderCallAudit.completed_at >= started_at`; orden temporal inverso falla como policy-invalid;
- todo acceso raw/elevado falla si no puede crear `raw_access_event_id` policy-valid;
- `RawAccessAuditService` rechaza `approved_by_ref == actor_ref`, `expires_at <= accessed_at` y refs documentales incompletas o presentes fuera de `resource_type=document_evidence`;
- riesgo prompt injection `blocking` se bloquea o excluye en fixtures;
- riesgos `D#:P#|F#:P#`, `url_hash:sha256:*` y `rr_*` propagan contaminacion respectivamente al pasaje, a la fuente resoluble con sus pasajes y a todas las fuentes/pasajes del run salvo delimitacion mas especifica persistida; citas/claims publicados sobre evidencia `blocking|excluded_from_evidence|blocked` fallan;
- riesgo `msg_*` se resuelve en scope de request: una severidad `blocking` bloquea la publicacion con `prompt_injection_blocked`, mientras evidencia independiente no queda contaminada solo por compartir el run;
- cuando `PromptInjectionGuardStub` opera en frontera API debe devolver `ErrorEnvelope.error_code=prompt_injection_blocked`;
- `make scan-secrets` bloquea claves en archivos trackeados e historial del rango del PR y solo permite placeholders/hash sintéticos mediante allowlist puntual;
- `make scan-sensitive-surfaces` falla si schemas aceptados permiten `raw_prompt|raw_output|document_text|full_document|ocr_full_text|html_raw|user_message` fuera de `x-invalid-*` o dejan objetos sanitizados abiertos capaces de admitirlas; incluye un fixture negativo de `Source.metadata`, e inyecta sentinels sintéticos únicos en prompt, mensaje, documento, OCR y payload de proveedor para comprobar que ninguno sobrevive en logs/traces serializados aunque cambie la clave;
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
- baseline de Fase 1 con verificacion local usando material configurado/cached, sin persistir/loguear token raw; introspeccion o llamadas de red de auth por request requieren decision versionada y auditoria especifica antes de habilitarse;
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

**Dependencias:** P0-04, P1-01. La revision `0004_source_registry_seed.py` continua linealmente despues de `0003_evidence_contract_stubs.py`; no crea un segundo head de Alembic ni salta la migracion P1.

---

### P2-02 — Fetcher stub

**Objetivo:** Definir interfaz de recuperación externa.

**Entregables:**

- `Fetcher` protocol/interface;
- `FetcherStub`;
- DTO interno cerrado `FetchPolicy`, compartido con el guard de egreso futuro;
- tests.

**Criterios de aceptación:**

- no hace red real;
- devuelve respuesta fixture;
- registra tool_call si se invoca;
- el parser/guard local rechaza sin red esquemas distintos de HTTPS, userinfo, literales IP, puertos no permitidos, proxies de ambiente y redirects automaticos o bloqueados;
- los tests de DNS IPv4/IPv6, DNS rebinding, redirects reales, MIME, timeouts y limite descomprimido quedan como gate obligatorio de Fase 4 antes de habilitar cualquier fetch o headless real.

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

### P2-05 — Evaluation harness, runner y regression suite inicial

**Objetivo:** Preparar evaluación desde la fundación y cumplir en Fase 1 la regression suite inicial exigida por ADR-012 sin fingir readiness beta.

**Entregables:**

- `src/jusnova/modules/evals/runner.py`;
- comando `make eval-smoke`;
- comando `make eval-regression`;
- fixture vacío para smoke, `tests/fixtures/evals/contract-regression-v1.yaml` no vacío y `test_eval_regression.py`;
- estructura de reporte compatible con el reporte versionado exigido por `docs/quality/beta-readiness-gates.md`.

**Criterios de aceptación:**

- corre sin dataset real;
- produce reporte técnico mínimo;
- no evalúa respuestas jurídicas aún.
- el bootstrap de Sprint 1 no satisface beta por sí solo;
- antes de cerrar Fase 1, `make eval-regression` ejecuta la suite contractual inicial no vacía y produce un reporte versionado con `suite_version`, `dataset_version=phase1-contract-regression-v1`, resultados por fixture y `beta_eligible=false`;
- antes de beta, este workstream debe producir reporte versionado con `dataset_version`, versión de golden dataset y métricas/gates de `docs/quality/evaluation-plan-v0.md` y `docs/quality/initial-golden-dataset-spec.md`.

**Dependencias:** P0-01, P1-03. La regression reutiliza el validador de schemas; no crea un segundo motor de validación.

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
- P1-08;
- P1-09;
- P2-01 solo despues de integrar P1-07/P1-08/P1-09 y si aun existe capacidad.

### Obligatorio antes de beta aunque no entre en Sprint 1

- P1-07;
- P1-08;
- P1-09;
- P2-05 en modo beta report versionado con `dataset_version` y resultado de gates 0.12.

### Obligatorio antes de cerrar Fase 1 aunque no entre en Sprint 1

- P1-01 a P1-09 y P2-01 a P2-06 forman el alcance ejecutable de Fase 1 y deben cumplir sus criterios antes de su cierre global; las prioridades P1/P2 permiten diferirlos fuera de Sprint 1, no fuera de Fase 1;
- P2-05 en modo regression suite contractual inicial no vacía; el reporte beta completo conserva su gate separado;
- P2-06, para que `WorkflowGateway` no quede solo como decisión documental.

### No forzar si compromete calidad

- P1-01/P1-02 tablas completas;
- P2-04 OpenSearch connection stub/profile opcional;
- P2-05 como reporte beta completo dentro de Sprint 1; el smoke puede quedar como preparación de Sprint 1 y la regression suite inicial es obligatoria al cierre de Fase 1, pero beta queda bloqueada sin el reporte sobre golden dataset y gates 0.12.

Esta lista aplica solo al cutline de Sprint 1. No reduce el Definition of Done global de Fase 1 ni autoriza cerrar la fase con un item P1/P2 pendiente.

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

### PR-5 — ModelProvider/PromptVersionRegistry + CostGovernor + budgets

Incluye:

- P0-10;
- P0-11;
- manifest/seed técnico y tests de `PromptVersionRegistry`;
- tests P1-06 de budget puro, sin débito ni conciliación.

### PR-6 — UsageLedger + commercial + tracing

Incluye:

- P0-12;
- P0-13;
- persistencia mínima;
- tests P1-06 de débito/conciliación con ambos `trace_id` y `answer_id`.

PR-6 es unidad atómica para research credits: no se mergea el débito `research_credit_used` sin ambos anchors terminales de P0-13.

### PR-7 — Conversation message pipeline

Incluye:

- P1-04 si entra.

### PR-8 — Contratos y foundations P1

Incluye:

- P1-01;
- P1-02;
- P1-03;
- P1-05;
- P1-07;
- P1-08.

### PR-9 — Auth productiva pre-beta

Incluye:

- P1-09;
- decision versionada del adapter;
- tests de token, membership y tenant isolation;
- eliminacion de cualquier ruta de dev auth en configuracion beta/production.

### PR-10 — Stubs y foundations P2

Incluye:

- P2-01 a P2-06 si el sprint lo permite;
- `0004_source_registry_seed.py` despues de `0003_evidence_contract_stubs.py`;
- regression suite contractual inicial no vacia y `WorkflowGateway` local.

PR-10 parte de P1-07/P1-08/P1-09 ya integrados. Esto conserva una sola secuencia post-esqueleto y no permite adelantar trabajo P2 por delante del gate de autenticacion productiva pre-beta.

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
8. usage registrado solo si la ejecución produce resultado publicable con `UsageEvent` válido enlazado;
9. `TraceObject` registrado para el cierre tecnico `answer_blocked`; un fallo temprano conserva solo trazabilidad operativa y no fabrica una traza terminal;
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
