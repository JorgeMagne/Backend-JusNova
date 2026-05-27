# JusNova — Phase 1 Implementation Brief

**Ruta objetivo en repo:** `/docs/handoff/phase-1-implementation-brief.md`
**Versión:** 1.0
**Fecha:** 2026-05-26
**Estado:** Accepted como handoff de Fase 1; ejecución condicionada al cierre formal de 0.14/Fase 0
**Propietario técnico:** JusNova Chief Backend Architect
**Fase origen:** Subfase 0.13 — Plan de Fase 1, backlog y handoff
**Fase destino:** Fase 1 — Fundaciones backend, datos, telemetría y Cost Governor

---

## 1. Veredicto ejecutivo

Fase 1 debe construir el **esqueleto productivo del backend** de JusNova. No debe construir todavía el motor jurídico completo, no debe implementar búsqueda legal viva real, no debe resolver consultas jurídicas complejas y no debe intentar producir respuestas legales definitivas.

La responsabilidad de Fase 1 es dejar listo el terreno técnico para que las siguientes fases puedan implementar evidencia, búsqueda, documentos, OCR, verificación y generación legal sin reescribir el core.

La decisión de arquitectura que se mantiene es:

> **High-Assurance Modular Core + Distributed Execution Layer**

En Fase 1 esto significa:

1. crear un backend FastAPI modular, no una app plana;
2. cerrar estructura de paquetes y límites internos;
3. conectar PostgreSQL con migraciones reales;
4. instalar logging estructurado con `request_id` y `correlation_id`;
5. crear modelos mínimos de organización, usuario, conversación y mensaje;
6. crear contratos de error, trazabilidad, usage y presupuesto;
7. implementar `CostGovernor` y `UsageLedger` en versión funcional mínima;
8. implementar `ModelProvider` como wrapper/stub, no llamadas libres a OpenAI;
9. exponer health checks y readiness checks;
10. dejar stubs controlados para evidencia, búsqueda, snapshots, OpenSearch y evaluación;
11. entregar tests mínimos suficientes para bloquear deuda peligrosa.

Fase 1 no será una demo. Será una fundación de producción. Si se hace mal, todo lo posterior quedará contaminado: costos invisibles, trazas incompletas, módulos acoplados, respuestas imposibles de auditar y migraciones frágiles.

---

## 2. Objetivo de Fase 1

Crear la base backend necesaria para ejecutar, medir, auditar y controlar cada interacción futura de JusNova.

Al cerrar Fase 1, el sistema debe poder:

- levantar en entorno local con Docker;
- aplicar migraciones de base de datos;
- crear organizaciones y usuarios semilla;
- crear conversaciones y mensajes;
- emitir eventos de streaming de estado sin conclusión legal;
- registrar request/correlation IDs;
- generar logs estructurados;
- persistir trazas mínimas;
- registrar llamadas simuladas a modelos y herramientas;
- calcular y aplicar presupuestos por plan/complejidad;
- registrar consumo en `UsageLedger`;
- responder health/readiness checks;
- correr tests automatizados de fundación;
- dejar contratos listos para Fase 2/3/4.

La frase clave para el equipo:

> **Fase 1 no busca que JusNova “responda bien legalmente”; busca que todo lo que JusNova haga después sea medible, trazable, presupuestado y estructuralmente mantenible.**

---

## 3. Alcance de Fase 1

### 3.1 Dentro de alcance

Fase 1 incluye:

- estructura inicial del repositorio backend;
- configuración Python/FastAPI;
- configuración de settings por ambiente;
- Docker Compose base;
- PostgreSQL;
- Alembic;
- SQLAlchemy 2.x;
- Pydantic v2;
- health checks;
- request context;
- logging estructurado;
- error envelope;
- entidades base;
- conversaciones y mensajes mínimos;
- streaming de estados;
- `ModelProvider` stub;
- `CostGovernor` funcional básico;
- `budgets.yaml`;
- `UsageLedger` básico;
- `commercial` mínimo con `Plan`/`Subscription`/`ResearchCredit`;
- `StorageProvider` abstracto con stub y límites privados;
- trazabilidad mínima;
- shells `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject` para `answer_blocked`;
- modelos de Evidence/Citation/Claim como contratos fundacionales;
- Source Registry seed mínimo;
- stubs de fetcher/snapshot/OpenSearch/eval runner;
- `beta_gate_foundations`: Provider Reliability Layer (PRL), ProviderCallAudit, RawAccessEvent y PromptInjectionGuard fail-closed;
- tests mínimos;
- documentación operativa de cómo levantar y validar la fase.

### 3.2 Fuera de alcance

Fase 1 **no debe** implementar:

- búsqueda jurídica viva real;
- adaptadores reales a Gaceta, TCP, TSJ/Génesis o SILEP;
- OCR real;
- carga de documentos reales;
- análisis legal con fundamento;
- Citation Auditor real completo;
- Claim Verification real;
- motor de respuesta jurídica final;
- memoria de caso avanzada;
- Source Snapshotter real;
- Modo Investigación real;
- pagos/facturación real;
- autenticación enterprise completa;
- RAG jurídico propio;
- embeddings sobre corpus jurídico;
- crawling masivo;
- scraping agresivo;
- microservicios separados para el core.

### 3.3 Fuera de alcance absoluto por ahora

No se debe construir nada que aparente ser respuesta jurídica confiable sin Evidence Pack y Citation Auditor. Si el endpoint de chat devuelve texto en Fase 1, debe ser un mensaje de estado o eco técnico, no una conclusión jurídica.

Ejemplo permitido:

```txt
Tu consulta fue recibida. En esta fase el sistema solo valida el pipeline fundacional y no emite análisis jurídico.
```

Ejemplo prohibido:

```txt
Según la legislación boliviana, corresponde interponer apelación en X días...
```

---

## 4. Stack cerrado para Fase 1

### 4.1 Lenguaje y runtime

**Decisión:** Python 3.12.

Justificación:

- ecosistema maduro para IA, extracción documental, OCR futuro, evaluación y recuperación;
- compatibilidad con FastAPI, Pydantic, SQLAlchemy y OpenTelemetry;
- permite mantener el core de IA/documentos en el mismo lenguaje.

Regla:

- No usar versiones antiguas de Python.
- No mezclar Node/NestJS en backend core durante Fase 1.
- El frontend Next.js consumirá OpenAPI/REST/SSE desde este backend.

### 4.2 Framework API

**Decisión:** FastAPI.

Componentes obligatorios:

- routers versionados `/v1`;
- dependency injection explícita;
- middlewares para request context;
- schemas Pydantic separados de modelos ORM;
- OpenAPI habilitado en dev/staging;
- OpenAPI restringido o protegido en producción.

### 4.3 Validación y contratos

**Decisión:** Pydantic v2.

Uso:

- request/response schemas;
- `BaseSettings` vía `pydantic-settings`;
- contratos internos de eventos;
- budgets YAML validados;
- ErrorEnvelope;
- StreamEvent publico alineado a `api-draft-v0.md`;
- BudgetDecision;
- UsageEvent;
- RunEvent operativo.

Regla de contratos:

- JSON Schemas aceptados viven en `docs/contracts/`.
- Taxonomias y budgets YAML aceptados viven en `docs/schemas/`.
- `StreamEvent` publico debe usar la taxonomia SSE aceptada en `docs/architecture/api-draft-v0.md`.
- `BudgetDecision` y `RunEvent` son DTOs internos de Fase 1 salvo que una subfase posterior los promueva a contrato JSON Schema aceptado.

### 4.4 ORM y migraciones

**Decisión:** SQLAlchemy 2.x + Alembic.

Reglas:

- SQLAlchemy en estilo 2.x;
- modelos ORM separados de schemas API;
- migraciones Alembic obligatorias;
- no usar `metadata.create_all()` en app runtime;
- no modificar DB manualmente fuera de migraciones versionadas;
- cada migración debe tener downgrade razonable mientras sea posible.

### 4.5 Base de datos

**Decisión:** PostgreSQL 16.

Uso en Fase 1:

- organizations;
- users;
- memberships;
- conversations;
- messages;
- operational_runs;
- model_calls;
- tool_calls;
- run_events;
- answers;
- answer_versions;
- abstention_renders;
- trace_objects;
- budget_decisions;
- usage_events;
- plans;
- subscriptions mínimos;
- research_credits;
- research_credit_movements;
- provider_call_audits;
- raw_access_events;
- evidence_packs;
- evidence_sources;
- evidence_passages;
- claims;
- citations;
- source_registry seed mínimo.

### 4.6 Cache/estado efímero

**Decisión:** Redis 7 como dependencia de infraestructura, pero uso limitado en Fase 1.

Uso inicial:

- readiness check;
- futuro rate limiting;
- futuro lock/idempotency;
- no usar como fuente de verdad.

Regla:

- Si Redis cae, el readiness debe reflejar degradación.
- No persistir datos jurídicos o trazabilidad crítica en Redis.

### 4.7 Búsqueda e indexación

**Decisión:** OpenSearch 2.x como motor objetivo de búsqueda; en Fase 1 solo se implementa conexión/stub.

Uso en Fase 1:

- health check opcional;
- cliente inicial;
- contrato de indexación;
- no crear todavía índices jurídicos reales salvo índice técnico de prueba.

Regla:

- No usar OpenSearch para simular un RAG.
- No indexar corpus jurídico en Fase 1.

### 4.8 Object storage

**Decisión:** S3-compatible; MinIO para desarrollo/staging.

Uso en Fase 1:

- configuración y health check opcional;
- no cargar documentos reales aún;
- preparar `StorageProvider` interface;
- implementar `StorageProviderStub` fail-closed para rutas privadas;
- validar límites básicos de storage privado antes de permitir escrituras futuras.

### 4.9 Workflows

**Decisión:** Temporal es el workflow engine objetivo, pero no es P0 de Sprint 1.

Fase 1 debe entregar:

- interfaz `WorkflowGateway`;
- stub local para tests;
- documentación de cómo se integrará Temporal en fases posteriores;
- no introducir acoplamiento directo a Temporal en el core de conversación.

Temporal puede activarse en una subfase posterior de Fase 1 si el equipo ya sabe operarlo. Si no, se deja listo el contrato y se pospone la operación real. El owner ejecutable de esta decisión es `P2-06 WorkflowGateway y LocalWorkflowGateway`; OQ-002 no queda cerrado por documentación solamente.

### 4.10 Observabilidad

**Decisión:** OpenTelemetry + `structlog`.

Uso mínimo:

- logs JSON;
- `request_id`;
- `correlation_id`;
- `organization_id` cuando exista;
- `actor_ref` cuando exista;
- `conversation_id` cuando exista;
- `run_id` cuando exista;
- latencia por request;
- error codes;
- trace IDs para model/tool calls futuros.

### 4.11 Testing

**Decisión:** pytest + pytest-asyncio + httpx.

Uso:

- unit tests;
- API tests;
- schema tests;
- DB migration smoke tests;
- CostGovernor tests;
- middleware tests;
- trace persistence tests.

### 4.12 Lint y typecheck

**Decisión:** `ruff` para lint/format-check y `mypy` para typecheck.

Uso:

- `make lint` ejecuta `ruff check`;
- `make typecheck` ejecuta `mypy src tests`;
- reglas iniciales pueden ser conservadoras, pero no se permite dejar el comando sin herramienta definida.

### 4.13 Packaging y dependencias

**Decisión:** `pyproject.toml` con gestor moderno. Recomendación: `uv` para velocidad y reproducibilidad, manteniendo compatibilidad con `pip`.

Reglas:

- lockfile obligatorio;
- no dependencias innecesarias;
- no SDKs externos hardcodeados sin wrapper;
- no secretos en repo;
- no claves reales en `.env.example`.

---

## 5. Estructura inicial del repositorio

La estructura objetivo para Fase 1 es:

```txt
jusnova-backend/
  README.md
  pyproject.toml
  uv.lock
  .env.example
  .gitignore
  docker-compose.yml
  Makefile

  docs/
    adr/
      ADR-001-high-assurance-modular-core.md
      ADR-002-stack-backend-and-infrastructure.md
      ADR-003-live-legal-search-engine.md
      ...
    handoff/
      phase-1-implementation-brief.md
      sprint-1-backlog.md
    phases/
      phase-1-development-plan.md
    contracts/
      error-envelope.schema.json
      evidence-pack.schema.json
      claim.schema.json
      citation.schema.json
      usage-event.schema.json
      trace-object.schema.json
      model-call.schema.json
      tool-call.schema.json
    policies/
      source-policy.md
      validity-policy.md
      abstention-policy.md
      provider-policy.md
      cost-governor-policy.md
    schemas/
      budgets.yaml
      legal-intents.yaml
      source-tiers.yaml
      validity-statuses.yaml

  src/
    jusnova/
      __init__.py
      main.py

      api/
        __init__.py
        deps.py
        health.py
        router.py
        v1/
          __init__.py
          conversations.py
          usage.py
          traces.py

      core/
        __init__.py
        config.py
        errors.py
        logging.py
        request_context.py
        security_context.py
        ids.py
        time.py
        pagination.py

      db/
        __init__.py
        base.py
        session.py
        migrations.py

      modules/
        organizations/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
        users/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
        conversations/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
        messages/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
        runs/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
        tracing/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
        answer_trace_shells/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
        cost_governor/
          __init__.py
          budgets.py
          models.py
          schemas.py
          service.py
          repository.py
        commercial/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
          seed.py
        usage_ledger/
          __init__.py
          models.py
          schemas.py
          repository.py
          service.py
        model_provider/
          __init__.py
          schemas.py
          provider.py
          stub.py
        storage/
          __init__.py
          provider.py
          stub.py
          service.py
          schemas.py
        evidence/
          __init__.py
          models.py
          schemas.py
        citations/
          __init__.py
          models.py
          schemas.py
        source_registry/
          __init__.py
          models.py
          schemas.py
          seed.py
        retrieval/
          __init__.py
          fetcher_stub.py
          snapshot_stub.py
          opensearch_stub.py
        evals/
          __init__.py
          runner_stub.py
        beta_gate_foundations/
          __init__.py
          prl_stub.py
          provider_call_audit.py
          raw_access_audit.py
          prompt_injection_guard_stub.py

      workers/
        __init__.py
        app.py
        workflow_gateway.py
        local_workflow_gateway.py

  scripts/
    validate_schemas.py

  alembic/
    env.py
    script.py.mako
    versions/
      0001_core_foundation.py
      0002_trace_usage_budget.py
      0003_evidence_contract_stubs.py
      0004_source_registry_seed.py

  tests/
    conftest.py
    unit/
      test_settings.py
      test_error_envelope.py
      test_budget_loader.py
      test_cost_governor.py
      test_commercial_service.py
      test_usage_ledger.py
      test_request_context.py
      test_model_provider_stub.py
      test_storage_provider_stub.py
      test_workflow_gateway.py
      test_schema_validator.py
      test_prl_stub.py
      test_provider_call_audit.py
      test_raw_access_event.py
      test_prompt_injection_guard_stub.py
      test_source_registry_seed.py
      test_fetcher_stub.py
      test_snapshot_stub.py
      test_opensearch_stub.py
      test_eval_smoke.py
    integration/
      test_health.py
      test_conversations.py
      test_tenant_isolation.py
      test_streaming_status.py
      test_trace_persistence.py
      test_migrations.py
```

### 5.1 Regla de módulos

Cada módulo debe separar:

- `models.py`: modelos ORM;
- `schemas.py`: Pydantic contracts;
- `repository.py`: acceso a datos;
- `service.py`: lógica de aplicación;
- `api/health.py`: handlers no versionados de `/health/*`;
- `api/v1/*.py`: handlers HTTP.

Queda prohibido:

- lógica de negocio extensa en routers;
- queries SQL dispersas en handlers;
- llamadas directas a OpenAI desde routers;
- dependencias cíclicas entre módulos;
- usar objetos ORM como response models;
- mezclar configuración, datos y lógica de dominio.

---

## 6. Módulos a crear en Fase 1

### 6.1 `core.config`

Responsabilidad:

- cargar settings;
- validar ambiente;
- exponer configuración tipada;
- proteger secretos;
- definir defaults seguros.

Campos mínimos de `Settings`:

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

`OPENAI_API_KEY` no es setting mínimo de Fase 1. Si aparece en `.env.example`, debe quedar marcado como futuro/opcional y no requerido por settings, CI ni production boot de Fase 1.

La lista anterior define atributos tipados de `Settings`, no variables de entorno obligatorias de forma incondicional. `DATABASE_URL`, `APP_ENV`, `APP_NAME`, `APP_VERSION`, `API_PREFIX`, `LOG_LEVEL`, `LOG_FORMAT`, `REQUEST_ID_HEADER`, `CORRELATION_ID_HEADER`, `BUDGETS_FILE` y los flags `ENABLE_*` deben tener valor efectivo por default seguro o env. `REDIS_URL` solo es obligatorio si `ENABLE_REDIS=true`; `OPENSEARCH_URL` solo si `ENABLE_OPENSEARCH=true`; `S3_ENDPOINT_URL`, `S3_BUCKET`, `S3_ACCESS_KEY_ID` y `S3_SECRET_ACCESS_KEY` solo si `ENABLE_S3=true`. Con flags `false`, boot/CI no debe fallar por ausencia de URLs o secretos de esas dependencias.

Regla:

- `ENABLE_OPENAI_REAL_CALLS=false` por defecto.
- Si faltan secretos en dev, el sistema debe arrancar con stubs cuando sea permitido.
- En producción, no debe arrancar si faltan dependencias críticas configuradas.
- En Fase 1, `ENABLE_OPENAI_REAL_CALLS` existe solo como guard rail y debe permanecer `false`; no se permiten llamadas reales a OpenAI ni otros proveedores externos aunque existan stubs de `ProviderCallAudit`.

### 6.2 `core.errors`

Responsabilidad:

- definir `ErrorEnvelope`;
- mapear excepciones internas a errores HTTP;
- asegurar `request_id` en toda respuesta de error;
- evitar filtración de detalles sensibles.

Códigos iniciales:

```txt
validation_error
auth_required
forbidden
tenant_mismatch
not_found
conflict
rate_limited
budget_exhausted
provider_unavailable
storage_unavailable
timeout
internal_error
policy_blocked
prompt_injection_blocked
```

Fase 1 no puede emitir codigos fuera de `docs/contracts/error-envelope.schema.json` ni fuera del subconjunto permitido por endpoint en `docs/architecture/api-draft-v0.md`.

### 6.3 `core.request_context`

Responsabilidad:

- crear/propagar `request_id`;
- crear/propagar `correlation_id`;
- almacenar contexto por request;
- inyectar headers de respuesta.

Headers:

```txt
X-Request-Id
X-Correlation-Id
```

### 6.4 `core.logging`

Responsabilidad:

- configurar logs JSON;
- anexar request/correlation IDs;
- anexar usuario/organización cuando existan;
- evitar secrets;
- definir helper para logs de dominio.

Campos mínimos en log:

```json
{
  "timestamp": "2026-05-01T00:00:00Z",
  "level": "info",
  "service": "jusnova-backend",
  "env": "dev",
  "request_id": "rq_demo",
  "correlation_id": "corr_demo",
  "organization_id": "org_demo",
  "actor_ref": "actor_hash_demo",
  "event": "request_received",
  "message": "request received"
}
```

### 6.5 `organizations`

Responsabilidad:

- representar tenants;
- ser raíz de aislamiento futuro;
- asociar usuarios, conversaciones, usage y trazas.

Modelo mínimo:

```txt
organizations
  id uuid pk -- surrogate interno opcional
  organization_id text unique not null -- org_*
  name text not null
  slug text unique not null
  status text not null default 'active'
  created_at timestamptz not null
  updated_at timestamptz not null
```

Regla comercial:

- `organizations` no guarda `plan_code` como fuente de verdad comercial.
- Fase 1 debe crear `plans` y `subscriptions` mínimos alineados con `Plan -> Subscription -> Organization` de 0.11.
- `plan_code` puede existir como snapshot en `UsageEvent`, `BudgetDecision` y `TraceObject`, pero se resuelve desde la suscripción activa.

### 6.6 `users`

Responsabilidad:

- representar usuarios internos;
- asociar membresía con organización;
- no implementar auth completa aún.

Modelo mínimo:

```txt
users
  id uuid pk -- surrogate interno opcional
  user_id text unique not null -- usr_*
  email text unique not null
  full_name text
  status text not null default 'active'
  created_at timestamptz not null
  updated_at timestamptz not null

memberships
  id uuid pk -- surrogate interno opcional
  membership_id text unique not null -- mbr_*
  organization_id text fk not null -- org_*
  user_id text fk not null -- usr_*
  role text not null default 'member'
  status text not null default 'active'
  created_at timestamptz not null
  unique(organization_id, user_id)
```

### 6.7 `conversations`

Responsabilidad:

- crear y persistir conversaciones;
- asociar a organización y usuario;
- preparar relación futura con case memory.

Modelo mínimo:

```txt
conversations
  id uuid pk -- surrogate interno opcional
  conversation_id text unique not null -- conv_*
  organization_id text fk not null -- org_*
  owner_actor_ref text not null -- actor_hash_*|sha256:*
  owner_actor_type text not null default 'user'
  case_id text null -- case_*
  created_by_actor_ref text not null -- actor_hash_*|sha256:*, DB-only audit copy
  title text
  status text not null default 'active'
  deleted_at timestamptz null
  created_at timestamptz not null
  updated_at timestamptz not null
```

`created_by_actor_ref` es columna interna DB-only para auditoría. Debe excluirse de serializers y payloads `Conversation` antes de validar contra `docs/contracts/conversation.schema.json`; la superficie contractual usa `owner_actor_ref`/`owner_actor_type`.

### 6.8 `messages`

Responsabilidad:

- persistir mensajes de usuario/sistema/asistente;
- permitir versionado futuro;
- no guardar respuestas jurídicas definitivas aún.

Modelo mínimo:

```txt
messages
  id uuid pk -- surrogate interno opcional
  message_id text unique not null -- msg_*
  conversation_id text fk not null -- conv_*
  organization_id text fk not null -- org_*
  actor_ref text not null -- actor_hash_*|service_*|system|sha256:*
  actor_type text not null -- user|system|service
  role text not null -- user|assistant|system_status|tool_status
  message_kind text not null -- user_input|assistant_final|assistant_clarification|system_notice|tool_progress
  content text not null
  content_hash text not null -- sha256:*
  answer_id text null -- ans_*
  trace_id text null -- tr_*
  answer_version_ref text null -- av_*
  attachments jsonb not null default '[]' -- valida contra message.schema.json; no metadata libre
  created_at timestamptz not null
```

### 6.9 `runs`

Responsabilidad:

- representar una ejecución de respuesta;
- conectar conversación, presupuesto, trazas y streaming.

Modelo mínimo:

```txt
operational_runs
  run_id text pk -- tr_* reservado operacional; no es TraceObject schema-valid hasta finalizacion
  organization_id text fk not null -- org_*
  conversation_id text fk null -- conv_*
  user_id text fk null -- usr_*
  status text not null -- queued|running|completed|failed|cancelled
  mode text not null default 'standard'
  complexity text not null default 'SIMPLE'
  started_at timestamptz
  completed_at timestamptz
  error_code text
  failure_stage text null
  created_at timestamptz not null
```

### 6.10 `tracing`

Responsabilidad:

- registrar eventos críticos;
- persistir model/tool calls;
- enlazar con respuesta futura.

Modelos mínimos:

```txt
run_events
  id uuid pk -- surrogate interno, no contrato externo
  run_id text fk not null -- tr_*
  organization_id text fk not null -- org_*
  event_scope text not null -- internal
  event_name text not null -- enum operativo cerrado de Fase 1
  sequence int not null
  resource_type text null
  resource_ref text null
  safe_message_code text null
  error_code text null
  payload_hash text null -- sha256:* si existe payload externo; no guarda payload crudo
  created_at timestamptz not null

model_calls
  model_call_id text pk -- mc_*
  run_id text fk not null -- tr_*
  organization_id text fk not null -- org_*
  purpose text not null -- enum de model-call.schema.json
  provider text not null
  model text not null
  prompt_version text not null
  input_hash text not null -- sha256:*
  output_hash text null -- sha256:* o null
  token_usage jsonb not null -- input_tokens/output_tokens/total_tokens; shape cerrado
  started_at timestamptz not null
  completed_at timestamptz not null
  status text not null -- success|error|timeout|cancelled
  error_code text null
  external_provider_call boolean not null
  provider_call_audit_id text null -- pca_* si external_provider_call=true

tool_calls
  tool_call_id text pk -- tc_*
  run_id text fk not null -- tr_*
  organization_id text fk not null -- org_*
  tool_name text not null
  purpose text not null
  input_hash text not null -- sha256:*
  output_hash text null -- sha256:* o null
  status text not null -- success|error|timeout|rate_limited|cancelled
  started_at timestamptz not null
  completed_at timestamptz not null
  cost_units jsonb not null -- requests/pages/tokens/estimated_cost; shape cerrado
  external_provider_call boolean not null
  provider_call_audit_id text null -- pca_* si external_provider_call=true
  error_code text
```

En Fase 1, `run_events` persiste solo eventos internos. Los eventos SSE publicos (`run_queued`, `run_started`, `answer_blocked`, `run_failed`, etc.) se emiten como `StreamEvent` y, si se requiere auditoria, se reflejan con un evento interno `stream_event_emitted`; no se mezclan como `event_name` de `run_events`.

Enum operativo cerrado de `run_events.event_name` en Fase 1:

```txt
run_created
message_received
message_persisted
budget_checked
model_provider_stubbed
usage_event_recorded
stream_event_emitted
run_completed
run_failed
error_envelope_returned
```

### 6.11 `answer_trace_shells`

Responsabilidad:

- materializar el cierre contractual `answer_blocked` del stream publico;
- crear los artefactos minimos `Answer`, `AnswerVersion`, `AbstentionRender` y `TraceObject`;
- evitar que `run_id=tr_*` se use como traza valida antes de existir respuesta/version/bloqueo.

Modelos minimos:

```txt
answers
  answer_id text pk -- ans_*
  organization_id text fk not null -- org_*
  conversation_id text fk not null -- conv_*
  latest_answer_version_ref text not null -- av_*
  response_outcome text not null -- blocked en cierre tecnico Fase 1
  created_at timestamptz not null

answer_versions
  answer_version_id text pk -- av_*; expuesto como answer_version_ref en API/SSE
  answer_id text fk not null -- ans_*
  organization_id text fk not null -- org_*
  trace_id text fk not null -- tr_*
  answer_version int not null default 1
  previous_answer_version int null
  response_outcome text not null -- blocked
  answer_hash text not null -- sha256:* de render seguro canonico
  answer_contract_ref text null -- null cuando response_outcome=blocked
  abstention_render_ref text not null -- abstention_render_*
  version_reason text not null -- initial_answer
  created_at timestamptz not null
  created_by_ref text not null -- system|service_*|actor_hash_*|sha256:*

abstention_renders
  abstention_render_id text pk -- abstention_render_*
  organization_id text fk not null -- org_*
  trace_id text fk not null -- tr_*
  answer_id text fk not null -- ans_*
  answer_version int not null default 1
  response_outcome text not null -- blocked
  reason_code text not null -- policy_blocked
  source_trace_refs jsonb not null -- shape de abstention-render.schema.json
  render_hash text not null -- sha256:*
  render_storage_ref text not null -- render_abstention_* DB-local en Fase 1
  render_body_canonical jsonb not null -- DB-only, no sale en AbstentionRender
  redaction_profile text not null -- USER_SUMMARY_SAFE
  created_at timestamptz not null
  created_by_ref text not null -- system|service_*|actor_hash_*|sha256:*

trace_objects
  trace_id text pk/fk -- tr_* igual a operational_runs.run_id
  organization_id text fk not null -- org_*
  answer_id text fk not null -- ans_*
  answer_version_ref text fk not null -- av_*
  conversation_id text fk not null -- conv_*
  output_message_id text fk not null -- msg_* con message_kind=assistant_final
  response_outcome text not null -- blocked
  abstention_reason text not null -- policy_blocked
  trace_object_payload jsonb not null -- TraceObject schema-valid minimizado, sin prompts/docs raw
  created_at timestamptz not null
```

Regla contractual para el cierre tecnico de Fase 1:

- `answer_blocked` no es un error HTTP; es el evento terminal publico de una ejecucion tecnica completada sin analisis juridico.
- `trace_object_payload` debe validar completo contra `docs/contracts/trace-object.schema.json`; no basta con persistir columnas escalares.
- `TraceObject.response_outcome=blocked`.
- `TraceObject.abstention_reason=policy_blocked`.
- `TraceObject.citation_audit` debe cumplir las reglas condicionales de `trace-object.schema.json` para `blocked/policy_blocked`: `overall_status` en `failed|blocked` y `blocking_failures[]` contiene `failure_code=policy_blocked`.
- `AnswerVersion.response_outcome=blocked`.
- `AnswerVersion.answer_contract_ref=null`.
- `AnswerVersion.abstention_render_ref` debe apuntar a `AbstentionRender`.
- `AbstentionRender.response_outcome=blocked`.
- `AbstentionRender.reason_code=policy_blocked`.
- `AbstentionRender.source_trace_refs` debe existir en el payload y como columna JSONB si se materializa la tabla.
- `AbstentionRender` debe resolver al mismo `trace_id`, `answer_id`, `answer_version` y `response_outcome` que `AnswerVersion` y `TraceObject`.
- `AbstentionRender.source_trace_refs` debe ser subconjunto real de `TraceObject.evidence_pack_ids[]`, `TraceObject.claims[].claim_id`, citas en `TraceObject.citation_audit.results[]`/`blocking_failures[]` y fuentes usadas, rechazadas o auditadas por `TraceObject`.
- Para bloqueos y abstenciones, `AnswerVersion.answer_hash` debe coincidir con `AbstentionRender.render_hash`, conforme a `answer-versioning-policy.md`.
- En Fase 1, `render_storage_ref` debe apuntar a un render canónico seguro almacenado localmente en la misma fila DB de `abstention_renders.render_body_canonical`; no apunta a S3, filesystem ni storage externo. Esta persistencia DB de texto seguro de bloqueo no cuenta como escritura real de `StorageProvider`.
- `render_hash` es `sha256(canonicalize(render_body_canonical))`; si el render canónico no puede reconstruirse desde `render_storage_ref`, `answer_blocked` no puede considerarse válido.
- `TraceObject.output_message_id` debe apuntar a un mensaje `assistant_final` tecnico y seguro.
- `trace_summary_url` debe resolver a `/v1/answers/{answer_id}/trace-summary`.
- Estos artefactos son shells seguros de bloqueo; no contienen respuesta legal sustantiva, prompts crudos, documentos crudos ni salidas completas de modelo.

Estrategia transaccional obligatoria:

- Las FK circulares entre `answers.latest_answer_version_ref`, `answer_versions.answer_id`, `answer_versions.trace_id`, `answer_versions.abstention_render_ref`, `abstention_renders.trace_id`, `trace_objects.answer_version_ref` y `messages.answer_version_ref` deben ser `DEFERRABLE INITIALLY DEFERRED` en PostgreSQL.
- La creación de `answer_blocked` ocurre en una sola transacción: reservar IDs `tr_*`, `ans_*`, `av_*`, `abstention_render_*` y `msg_*`; crear el mensaje `assistant_final` tecnico; crear `answers`, `answer_versions`, `abstention_renders` y `trace_objects`; validar constraints al commit.
- No se permite confirmar filas parciales de respuesta/traza si falta cualquier artefacto requerido.

### 6.12 `cost_governor`

Responsabilidad:

- cargar `budgets.yaml`;
- resolver plan + complejidad + modo;
- devolver límites;
- bloquear ejecución si excede presupuesto;
- registrar decisión de presupuesto.

Modelo mínimo:

```txt
budget_decisions
  id uuid pk -- decision operacional, no contrato JSON Schema nuevo
  budget_decision_id text unique not null -- bd_*
  run_id text fk null -- tr_*
  organization_id text fk not null -- org_*
  plan_code text not null
  complexity text not null
  mode text not null
  cost_budget_ref text not null -- cb_<plan>_<complexity>_vNNN
  cost_budget_version text not null -- cost-budget-v*
  decision text not null -- allow|degrade|block
  reason text
  budget_snapshot jsonb not null
  created_at timestamptz not null
```

### 6.13 `commercial`

Responsabilidad:

- representar el catalogo minimo de planes;
- asignar una suscripcion activa a cada organizacion;
- evitar dos fuentes de verdad entre `organizations.plan_code` y suscripcion.

Modelos minimos:

```txt
plans
  plan_id text pk -- plan_*
  plan_code text unique not null -- PROFESIONAL|PRO_PLUS|ESTUDIO|ENTERPRISE
  status text not null -- active|inactive
  cost_budget_version text not null -- cost-budget-v*
  monthly_research_credits numeric(18,6) not null
  created_at timestamptz not null

subscriptions
  subscription_id text pk -- sub_*
  organization_id text fk not null -- org_*
  plan_id text fk not null -- plan_*
  status text not null -- active|trialing|past_due|cancelled
  started_at timestamptz not null
  ended_at timestamptz null
  created_at timestamptz not null

research_credits
  research_credit_id text pk -- rc_*
  organization_id text fk not null -- org_*
  subscription_id text fk not null -- sub_*
  billing_period text not null -- YYYY-MM
  granted_quantity numeric(18,6) not null
  used_quantity numeric(18,6) not null
  balance_quantity numeric(18,6) not null
  status text not null -- active|exhausted|expired
  created_at timestamptz not null
  updated_at timestamptz not null

research_credit_movements
  research_credit_movement_id text pk -- rcm_*
  research_credit_id text fk not null -- rc_*
  subscription_id text fk not null -- sub_*
  usage_event_id text fk null -- ue_*, required for debit
  movement_type text not null -- grant|debit|adjustment|expiry
  delta numeric(18,6) not null
  balance_after numeric(18,6) not null
  trace_id text null -- tr_*
  answer_id text null -- ans_*
  created_at timestamptz not null
```

Regla:

- Sprint 1 usa `subscriptions` como fuente de verdad comercial.
- `UsageEvent.plan_code`, `BudgetDecision.plan_code` y `TraceObject.plan_code` son snapshots derivados, no owners.
- No crear una tabla paralela de asignaciones comerciales si ya existe `subscriptions`.
- Debe existir una sola suscripcion current por organizacion: `ended_at is null` y `status in active|trialing|past_due`.
- `plans.cost_budget_version` selecciona la version del catalogo; el `CostGovernor` resuelve por ejecucion `cost_budget_ref=cb_<plan>_<complexity>_vNNN` desde `docs/schemas/budgets.yaml`.
- `plans.monthly_research_credits` es la fuente de verdad para grants mensuales. En fixtures de Fase 1, `PROFESIONAL` usa `8` créditos mensuales, dentro del rango 8-12 de `docs/policies/commercial-plans-v0.md`.
- `research_credits` guarda el saldo por suscripcion y periodo; `research_credit_movements` guarda grants/debitos resumibles por `GET /v1/research-credits`.
- Cada grant mensual se crea en la misma transaccion que el periodo `research_credits` activo.
- Cada debit debe crear `UsageEvent.event_type=research_credit_used` y `research_credit_movements.movement_type=debit` en la misma transaccion.
- Regla de movimiento: `debit` implica `usage_event_id` no nulo, `trace_id` o `answer_id` no nulo, `delta < 0` y `balance_after >= 0`.
- Regla de signo: `grant` implica `delta > 0`; `expiry` implica `delta <= 0`; `adjustment` exige `balance_after >= 0` y motivo operativo auditado en el service.
- El service debe bloquear la fila `research_credits` del periodo (`SELECT ... FOR UPDATE` o equivalente), verificar saldo suficiente, actualizar `used_quantity`/`balance_quantity` e insertar el movimiento en una sola transaccion. No se permite un `UsageEvent` de débito sin movimiento conciliable ni un movimiento de débito sin `UsageEvent`.

### 6.14 `usage_ledger`

Responsabilidad:

- registrar consumo por organización y usuario;
- preparar límites mensuales;
- no calcular facturación real todavía.

Modelo mínimo:

```txt
usage_events
  usage_event_id text pk -- ue_*
  organization_id text fk not null -- org_*
  actor_ref text not null -- actor_hash_*|support_hash_*|service_*|system|sha256:*
  actor_type text not null -- user|system|support|service
  subscription_id text null -- sub_*, DB-only reconciliation; not serialized in UsageEvent JSON
  research_credit_id text null -- rc_*, DB-only reconciliation for research_credit_used
  plan_code text not null -- PROFESIONAL|PRO_PLUS|ESTUDIO|ENTERPRISE
  billing_period text not null -- YYYY-MM
  event_scope text not null -- execution|organization_period
  event_type text not null -- enum de usage-event.schema.json
  unit text not null -- enum de usage-event.schema.json
  quantity numeric(18,6) not null
  complexity text null -- SIMPLE|MEDIO|COMPLEJO|INVESTIGACION
  cost_budget_ref text null -- cb_*
  cost_budget_version text null -- cost-budget-v*
  conversation_id text null -- conv_*
  trace_id text null -- tr_*
  answer_id text null -- ans_*
  retrieval_run_id text null -- rr_*
  cost_report_id text null -- cr_*
  model_call_id text null -- mc_*
  tool_call_id text null -- tc_*
  estimated_cost numeric(18,6) null
  currency text null -- BOB|USD|NONE
  created_at timestamptz not null
```

Eventos mínimos emitidos por P0:

```txt
standard_query
complex_query
research_credit_used
```

`research_credit_used` es obligatorio cuando la ejecución publicable usa `complexity=COMPLEJO` o `complexity=INVESTIGACION`. Su `quantity` debe coincidir con `CostBudget.research_credit_cost`: `1` para `COMPLEJO` y `2` para `INVESTIGACION`. Si no existe saldo/reserva de créditos o el débito no puede registrarse, `CostGovernor` debe bloquear o degradar; no se permite publicar una ejecución de esas complejidades sin el débito conciliable. El payload `UsageEvent` sigue cerrado por `docs/contracts/usage-event.schema.json`; `subscription_id` y `research_credit_id` son columnas internas DB-only y se excluyen antes de validar/exportar el payload contractual.

Eventos contractuales aceptados que Fase 1 puede emitir solo cuando exista el ancla correspondiente:

```txt
model_input_tokens
model_output_tokens
discovery_call
fetch
ocr_page
document_processed
storage_mb_day
```

Las decisiones de presupuesto se guardan en `budget_decisions`; los hitos tecnicos de pipeline y streaming se guardan en `run_events`. No se deben inventar `UsageEvent.event_type` fuera del enum aceptado.
`model_input_tokens` y `model_output_tokens` solo se emiten cuando existe `model_call_id`; no son requisito de P0-12 antes de P0-13/P1-05.

### 6.15 `model_provider`

Responsabilidad:

- encapsular proveedor de modelos;
- impedir llamadas directas a proveedores desde módulos;
- registrar model calls;
- permitir stub determinista.

Regla:

- En Fase 1 el provider por defecto es `StubModelProvider`.
- Fase 1 no permite llamadas reales a OpenAI ni proveedores externos.
- Activar cualquier proveedor real queda fuera de Fase 1; requiere decisión aceptada posterior, `provider_call_audits` productivo contract-compatible con `docs/contracts/provider-call-audit.schema.json` y enlace desde `ModelCall`/`ToolCall`.

### 6.16 `evidence` y `citations`

Responsabilidad:

- definir modelos y schemas fundacionales;
- no ejecutar auditoría real todavía;
- preparar Fase 2 de Citation Auditor y bloqueo de claims críticos.

Entidades mínimas P1:

```txt
evidence_packs
evidence_sources
evidence_passages
claims
citations
```

`CitationAudit` permanece como value object embebido en `TraceObject` segun 0.11. Fase 1 no debe crear una tabla standalone para auditorias de cita salvo que agregue explicitamente `organization_id`, `trace_id`, `answer_version_ref` y unicidad por `trace_id`.

En Sprint 1 estas pueden quedar como schemas, modelos/stubs contractuales y migracion P1, no necesariamente endpoint.

### 6.17 `source_registry`

Responsabilidad:

- registrar fuentes conocidas;
- seed mínimo sin fetch real;
- preparar Fase 4.

Seed mínimo:

- Gaceta Oficial de Bolivia;
- Tribunal Constitucional Plurinacional;
- TSJ/Génesis;
- SILEP como fuente a validar;
- dominio `.gob.bo` genérico como categoría oficial;
- proveedores de discovery como `disabled` por defecto.

### 6.18 `retrieval` stubs

Responsabilidad:

- definir interfaces de fetch/snapshot/OpenSearch;
- evitar que Fase 1 acople recuperación a implementación real;
- preparar pruebas de integración futuras.

Stubs:

```txt
FetcherStub
SnapshotStub
OpenSearchClientStub
SearchDiscoveryProviderStub
```

### 6.19 `evals` stub

Responsabilidad:

- preparar estructura de evaluación;
- no ejecutar eval jurídica real todavía;
- permitir que CI reconozca contrato.

Ruta canónica:

- `src/jusnova/modules/evals/runner_stub.py`.

### 6.20 `storage`

Responsabilidad:

- definir `StorageProvider` S3-compatible sin fijar proveedor único;
- entregar `StorageProviderStub` para desarrollo/tests;
- aplicar límites de storage privado antes de cualquier escritura futura;
- no subir documentos reales en Fase 1.

Regla:

- Toda operación de escritura real queda fuera de Fase 1.
- Cualquier intento de escritura sin provider configurado debe fallar con `ErrorEnvelope.error_code=storage_unavailable`.
- Las rutas/keys de storage no pueden exponer tenant, usuario, documento raw ni nombres de archivo originales.

### 6.21 `beta_gate_foundations`

Responsabilidad:

- cubrir los beta blockers asignados a Fase 1 sin activar proveedores reales ni análisis jurídico;
- dejar Provider Reliability Layer (PRL), provider audit, raw access audit y prompt-injection blockers en modo fail-closed;
- hacer que los gates de beta sean verificables desde tests técnicos.

Modelos mínimos:

```txt
provider_call_audits
  provider_call_audit_id text pk -- pca_*
  organization_id text fk not null -- org_*
  provider_family text not null
  provider_name text not null
  trace_id text null -- tr_*, final TraceObject/operational run anchor when available
  model_call_id text null -- mc_*
  tool_call_id text null -- tc_*
  retrieval_run_id text null -- rr_*
  usage_event_id text null -- ue_*
  cost_report_id text null -- cr_*
  external_call boolean not null default false
  data_sent_classes jsonb not null -- array de data-classification.yaml
  data_returned_classes jsonb not null -- array de data-classification.yaml
  attempted_data_classes jsonb not null -- minItems 1
  input_hash text not null -- sha256:*
  output_hash text null -- sha256:* o null
  status text not null
  error_code text null
  region_or_residency text null
  feature_flag text null
  kill_switch text null
  training_use_allowed boolean not null default false
  policy_decision text not null
  started_at timestamptz not null
  completed_at timestamptz not null

raw_access_events
  raw_access_event_id text pk -- rae_*
  organization_id text fk not null -- org_*
  resource_type text not null
  resource_ref text not null
  classification text not null
  actor_ref text not null
  actor_type text not null
  access_role text not null
  reason text not null
  document_id text null -- doc_* requerido si resource_type=document_evidence
  document_version_id text null -- docv_* requerido si resource_type=document_evidence
  passage_ref text null -- D#:P# requerido si resource_type=document_evidence
  accessed_at timestamptz not null
  expires_at timestamptz null
  visibility_level text not null
  access_case_ref text not null
  approved_by_ref text not null
```

Servicios mínimos:

```txt
ProviderReliabilityLayerStub
ProviderCallAuditService
RawAccessAuditService
PromptInjectionGuardStub
```

Reglas:

- `ProviderReliabilityLayerStub` debe mapear timeout, rate limit y provider unavailable a `ErrorEnvelope` sin llamar proveedores reales.
- `ProviderCallAuditService` debe validar fixtures contra `provider-call-audit.schema.json` y contra `provider-policy.md` + `docs/schemas/provider-registry.yaml`: `provider_name` resuelve en registry, `provider_family` coincide, `data_sent_classes[]` y `data_returned_classes[]` respetan allowlists/clases conocidas, y los estados `policy_blocked|timeout|rate_limited|cancelled|error` cumplen sus `error_code`/campos condicionales. Para `status=success|error|timeout|rate_limited|cancelled`, `attempted_data_classes[]` debe representar el mismo set que `data_sent_classes[]` y respetar allowlist; para `status=policy_blocked`, `attempted_data_classes[]` puede contener clases conocidas fuera de allowlist, pero `data_sent_classes=[]`, `data_returned_classes=[]` y `policy_decision` debe ser `blocked_by_classification|blocked_by_feature_flag|blocked_by_kill_switch`. Si la razón es clase fuera de allowlist, `policy_decision=blocked_by_classification`. Los campos `error_code`, `region_or_residency`, `feature_flag` y `kill_switch` son nullable en tabla pero obligatorios cuando el schema o la policy los exigen. Toda llamada externa futura debe fallar si no puede crear `provider_call_audit_id` policy-valid.
- `ProviderCallAudit` no puede quedar huérfano: al menos uno de `trace_id`, `model_call_id`, `tool_call_id`, `retrieval_run_id`, `usage_event_id` o `cost_report_id` debe existir en fixtures contables, y `organization_id` debe coincidir con cada recurso resuelto.
- `RawAccessAuditService` debe validar fixtures contra `raw-access-event.schema.json` y `privacy-security-policy.md`: cuando `resource_type=document_evidence`, `document_id`, `document_version_id` y `passage_ref` son obligatorios; cuando `resource_type!=document_evidence`, esos campos deben ser null/ausentes; `approved_by_ref != actor_ref`; `expires_at`, si existe, debe ser posterior a `accessed_at`. Cualquier acceso raw/elevado sin `RawAccessEvent` policy-valid falla.
- `PromptInjectionGuardStub` debe bloquear o excluir fixtures de riesgo `blocking` segun `prompt-injection-policy.md`; no interpreta contenido de documentos, mensajes ni fuentes como instrucciones.
- Estos servicios son stubs contractuales de Fase 1; la lógica completa de proveedores, fetch real y evaluación adversarial profunda queda para fases posteriores.

---

## 7. Migraciones iniciales

### 7.1 `0001_core_foundation.py`

Debe crear:

- `organizations`;
- `users`;
- `memberships`;
- `conversations`;
- `messages`;
- extensiones necesarias como `pgcrypto` para UUID si aplica.

Regla de identidad:

- Los UUID son surrogates internos opcionales.
- Los identificadores que cruzan API, contratos y tablas relacionadas deben respetar `docs/architecture/domain-model.md`: `org_*`, `usr_*`, `mbr_*`, `conv_*`, `msg_*`.

Índices mínimos:

```txt
organizations.slug unique
users.email unique
memberships.organization_id,user_id unique
conversations.organization_id,created_at
messages.conversation_id,created_at
messages.organization_id,created_at
```

### 7.2 `0002_trace_usage_budget.py`

Debe crear:

- `operational_runs`;
- `run_events`;
- `model_calls`;
- `tool_calls`;
- `answers`;
- `answer_versions`;
- `abstention_renders`;
- `trace_objects`;
- `budget_decisions`;
- `usage_events`;
- `plans`;
- `subscriptions`;
- `research_credits`;
- `research_credit_movements`;
- `provider_call_audits`;
- `raw_access_events`.

`operational_runs` y `run_events` son tablas operativas de Fase 1, no entidades canonicas nuevas de 0.11. El `run_id` expuesto conserva forma `tr_*`; solo se considera `TraceObject` valido cuando existe fila en `trace_objects` con `answer_id`, `answer_version_ref`, `response_outcome=blocked`, `abstention_reason=policy_blocked` y `trace_object_payload` validado contra `docs/contracts/trace-object.schema.json`, incluido el `CitationAudit` minimo requerido para `policy_blocked`.

Índices mínimos:

```txt
operational_runs.organization_id,created_at
run_events.run_id,created_at
run_events.run_id,sequence unique
model_calls.run_id,created_at
tool_calls.run_id,created_at
answers.organization_id,created_at
answer_versions.answer_id,answer_version unique
answer_versions.trace_id unique
abstention_renders.trace_id unique
abstention_renders.answer_id,answer_version,trace_id unique
trace_objects.organization_id,created_at
trace_objects.trace_id unique/fk operational_runs.run_id
trace_objects.answer_version_ref unique
trace_objects.output_message_id unique
messages.answer_id,trace_id,answer_version_ref
usage_events.organization_id,created_at
usage_events.subscription_id,billing_period
usage_events.research_credit_id
usage_events.cost_report_id
research_credits.subscription_id,billing_period unique
research_credit_movements.research_credit_id,created_at
research_credit_movements.usage_event_id unique where usage_event_id is not null
research_credit_movements check movement_type <> debit or (usage_event_id is not null and (trace_id is not null or answer_id is not null) and delta < 0 and balance_after >= 0)
research_credit_movements check movement_type <> grant or (usage_event_id is null and delta > 0 and balance_after >= 0)
research_credit_movements check movement_type <> expiry or (delta <= 0 and balance_after >= 0)
research_credits check balance_quantity >= 0 and used_quantity >= 0 and granted_quantity >= 0
budget_decisions.organization_id,created_at
plans.plan_code unique
subscriptions.organization_id,status
subscriptions.organization_id unique where ended_at is null and status in active|trialing|past_due
provider_call_audits.organization_id,started_at
provider_call_audits.trace_id
provider_call_audits.model_call_id
provider_call_audits.tool_call_id
provider_call_audits.retrieval_run_id
provider_call_audits.usage_event_id
provider_call_audits.cost_report_id
raw_access_events.organization_id,accessed_at
```

### 7.3 `0003_evidence_contract_stubs.py`

Debe crear, aunque sea sin endpoints:

- `evidence_packs`;
- `evidence_sources`;
- `evidence_passages`;
- `claims`;
- `citations`.

Shape mínimo cerrado:

```txt
evidence_packs
  evidence_pack_id text pk -- ep_*
  organization_id text fk not null -- org_*
  query_id text not null -- q_*
  retrieval_run_id text null -- rr_*, opcional segun EvidencePack -> RetrievalRun 0..1
  jurisdiction text not null -- BO
  legal_area jsonb not null
  quality jsonb not null
  created_at timestamptz not null

evidence_sources
  evidence_pack_id text fk not null -- ep_*
  organization_id text fk not null -- org_*, copied from evidence_packs
  source_ref text not null -- F#|D#
  source_type text not null
  tier text not null
  validity_status text not null
  retrieved_at timestamptz not null
  source_payload jsonb not null -- Source schema-valid

evidence_passages
  evidence_pack_id text fk not null -- ep_*
  organization_id text fk not null -- org_*, copied from evidence_packs
  passage_ref text not null -- F#:P#|D#:P#
  source_ref text not null -- F#|D#
  passage_payload jsonb not null -- Passage schema-valid, fixture-only in Fase 1

claims
  claim_id text pk -- cl_*
  organization_id text fk not null -- org_*
  answer_version_ref text null -- av_*, nullable until answer shell links it
  claim_type text not null
  criticality text not null
  support_level text not null
  verification_status text not null
  requires_human_review boolean not null
  claim_payload jsonb not null -- Claim schema-valid

citations
  answer_version_ref text not null -- av_*
  citation_ref text not null -- C#
  organization_id text fk not null -- org_*
  evidence_pack_id text fk not null -- ep_*
  passage_ref text not null -- F#:P#|D#:P#
  source_ref text not null -- F#|D#
  status text not null
  citation_payload jsonb not null -- Citation schema-valid
```

Regla:

- Estas tablas no implican RAG.
- Son contratos de evidencia para Fase 2 y búsqueda viva futura.
- `evidence_packs.organization_id` es obligatorio y debe coincidir con `LegalSearchQuery.organization_id` resuelto por `query_id`; si `retrieval_run_id` existe, tambien debe coincidir con `RetrievalRun.organization_id`.
- `retrieval_run_id` es nullable en la tabla stub porque `EvidencePack -> RetrievalRun` es `0..1` en 0.11; packs manuales o documentales no deben inventar `rr_*` falso.
- Mientras `evidence-pack.schema.json` requiera `retrieval_run_id`, solo filas con `retrieval_run_id` no nulo pueden serializarse o validarse como payload contractual `EvidencePack`; filas con `retrieval_run_id=null` son contenedores DB internos/draft y no deben emitirse como `EvidencePack` schema-valid ni contarse como fixture contractual.
- `evidence_sources.organization_id` y `evidence_passages.organization_id` son columnas internas de ownership copiadas desde `evidence_packs`; se excluyen de los payloads `Source`/`Passage` antes de validar contra JSON Schema.
- `claims.organization_id` y `citations.organization_id` son obligatorios y nunca se derivan solo del request actual.
- `evidence_sources` usa `primary key (evidence_pack_id, source_ref)`; `evidence_passages` usa `primary key (evidence_pack_id, passage_ref)`.
- `citations` usa `primary key (answer_version_ref, citation_ref)` y debe resolver `source_ref` y `passage_ref` dentro del mismo `evidence_pack_id`.
- `citations.passage_ref` debe resolver a un `evidence_passages` del mismo `evidence_pack_id` cuyo `source_ref` sea exactamente igual a `citations.source_ref`; tambien debe cumplirse que el prefijo de `passage_ref` antes de `:P#` coincida con ese `source_ref`.
- Todo `Claim` referenciado por `Citation.supports_claim_ids[]` debe existir con el mismo `organization_id` y el mismo `answer_version_ref` que la cita; no se permite soporte cross-tenant, cross-version ni contra claims con `answer_version_ref=null`.
- Todo `Claim.claim_payload.citations[]` no vacio debe apuntar a `citations.citation_ref` existentes en la misma `answer_version_ref`, y toda `Citation.status=valid` incluida en la respuesta debe aparecer en al menos un `Claim.claim_payload.citations[]` de esa misma version.
- Las columnas canonicas y los payloads JSONB deben coincidir: `source_payload.source_ref == evidence_sources.source_ref`, `passage_payload.passage_ref/source_ref == evidence_passages.passage_ref/source_ref`, `claim_payload.claim_id/organization_id/citations == claims.claim_id/organization_id` y refs de citas resueltas, y `citation_payload.citation_ref/organization_id/passage_ref/source_ref/supports_claim_ids == citations.*` y claims resueltos.
- Los fixtures P1 deben validar `source_payload`, `passage_payload`, `claim_payload` y `citation_payload` contra `source.schema.json`, `passage.schema.json`, `claim.schema.json` y `citation.schema.json`.
- `CitationAudit` no es tabla standalone en esta migracion; queda embebido en `TraceObject` hasta una decision posterior con owner/trace constraints.

### 7.4 `0004_source_registry_seed.py`

Debe crear:

- `source_registry`;
- `source_registry_health`;
- seed de fuentes bolivianas principales.

Shape minimo cerrado:

```txt
source_registry
  source_registry_entry_id text pk -- src_*
  code text unique not null
  name text not null
  source_type text not null
  tier text not null
  live_fetch_enabled boolean not null default false
  created_at timestamptz not null
  updated_at timestamptz not null

source_registry_health
  source_registry_entry_id text pk/fk -- src_*
  health_status text not null -- HEALTHY|DEGRADED|DOWN|UNKNOWN
  last_checked_at timestamptz null
  last_error_code text null
  updated_at timestamptz not null
```

Reglas:

- Fase 1 no crea columna alternativa de health en `source_registry`; usa la tabla `source_registry_health`.
- `source_registry_health.source_registry_entry_id` es 1:1 con `source_registry.source_registry_entry_id`.
- `source_registry_health.health_status` usa el vocabulario canónico de `docs/schemas/host-statuses.yaml`; no introduce enums paralelos.
- Como `live_fetch_enabled=false` en todo el seed de Fase 1, cada fila inicial de health usa `health_status=UNKNOWN`, `last_checked_at=null` y `last_error_code=null`. El estado disabled se expresa por `source_registry.live_fetch_enabled=false`, no por un valor nuevo de health.
- La migracion es idempotente por `source_registry_entry_id` y `code`; re-ejecutarla no duplica fuentes ni health rows.

Seed recomendado:

```yaml
- code: gob_gaceta_oficial
  source_registry_entry_id: src_gob_gaceta_oficial
  name: Gaceta Oficial de Bolivia
  source_type: norma
  tier: TIER1_CANONICO
  live_fetch_enabled: false

- code: tcp_bolivia
  source_registry_entry_id: src_tcp_bolivia
  name: Tribunal Constitucional Plurinacional
  source_type: jurisprudencia
  tier: TIER1_CANONICO
  live_fetch_enabled: false

- code: tsj_genesis
  source_registry_entry_id: src_tsj_genesis
  name: Tribunal Supremo de Justicia / Génesis
  source_type: jurisprudencia
  tier: TIER1_CANONICO
  live_fetch_enabled: false

- code: silep
  source_registry_entry_id: src_silep
  name: SILEP
  source_type: norma
  tier: TIER1_STRUCTURED
  live_fetch_enabled: false

- code: official_gob_bo
  source_registry_entry_id: src_official_gob_bo
  name: Dominios oficiales .gob.bo
  source_type: institucional
  tier: TIER1_OFICIAL
  live_fetch_enabled: false
```

`source_registry_entry_id` debe usar forma `src_*` para alinearse con `docs/architecture/domain-model.md`. `code` es alias operativo/idempotency key, no reemplaza el ID canónico. `source_type` y `tier` deben usar los enums de `docs/contracts/source.schema.json`; el seed no crea `source-registry-entry.schema.json` en 0.13.

---

## 8. Contratos que deben implementarse primero

### 8.1 `ErrorEnvelope`

```json
{
  "error_code": "budget_exhausted",
  "safe_message_code": "budget_limit_reached",
  "message": "El presupuesto disponible para esta ejecucion fue excedido.",
  "severity": "warning",
  "request_id": "rq_phase1_budget",
  "retryable": false,
  "user_visible": true,
  "created_at": "2026-05-26T00:00:00Z",
  "metadata": {}
}
```

Reglas:

- Todo error HTTP no 2xx debe devolver exactamente el formato top-level de `docs/contracts/error-envelope.schema.json`.
- No exponer stack trace.
- No exponer secretos.
- `request_id` obligatorio.
- `correlation_id` se propaga en headers/logs, pero no es campo requerido del `ErrorEnvelope` aceptado.

### 8.2 `StreamEvent`

```json
{
  "run_id": "tr_phase1_demo",
  "event_type": "run_started",
  "status": "running",
  "sequence": 1,
  "created_at": "2026-05-26T00:00:00Z"
}
```

Eventos publicos permitidos por `GET /v1/conversations/{conversation_id}/runs/{run_id}/stream`:

```txt
run_queued
run_started
retrieval_started
retrieval_progress
document_processing_required
evidence_ready
run_failed
answer_final
answer_blocked
```

Estados publicos permitidos:

```txt
queued
running
waiting_for_document_processing
failed
finalized
blocked
```

Regla:

- El stream publico usa `event_type`/`status` del API draft, no `type/stage` ni eventos paralelos.
- Eventos operativos internos como `budget_checked` o `message_persisted` se guardan en `run_events` con `event_scope=internal`; no son taxonomia publica SSE.
- Fase 1 no debe emitir deltas de respuesta ni analisis juridico.
- En Fase 1, el cierre tecnico exitoso del stream publico debe emitir `answer_blocked`, no un evento paralelo `run_completed`.
- `answer_blocked` debe incluir `answer_id`, `answer_version_ref`, `trace_summary_url` y `run_id` resoluble a `TraceObject`, conforme a `api-draft-v0.md`.
- Fase 1 debe crear los artefactos minimos `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject` necesarios para `answer_blocked`.
- La razon contractual del bloqueo tecnico de Fase 1 es `policy_blocked` en `TraceObject.abstention_reason` y `AbstentionRender.reason_code`.

### 8.3 `BudgetRequest`

```json
{
  "organization_id": "org_demo",
  "actor_ref": "actor_hash_demo",
  "actor_type": "user",
  "plan_code": "PROFESIONAL",
  "mode": "standard",
  "complexity": "SIMPLE",
  "requested_capabilities": ["conversation"],
  "estimated_input_tokens": 0,
  "estimated_output_tokens": 0
}
```

`BudgetRequest` no usa `user_id` crudo en la frontera de decisión; el caller debe resolver `actor_ref`/`actor_type` desde el contexto autenticado.

### 8.4 `BudgetDecision`

```json
{
  "decision": "allow",
  "plan_code": "PROFESIONAL",
  "mode": "standard",
  "complexity": "SIMPLE",
  "limits": {
    "discovery_calls_max": 1,
    "fetch_urls_max": 3,
    "ocr_pages_inline_max": 0,
    "reformulations_max": 0,
    "tool_rounds_max": 2,
    "time_budget_ms": 45000,
    "output_tokens_cap": 900,
    "research_credit_cost": 0,
    "async_allowed": false
  },
  "reason": null
}
```

`BudgetDecision` es un DTO interno para `budget_decisions`; no sustituye `CostBudget` ni crea un JSON Schema canonico nuevo.

Decisiones posibles:

```txt
allow
degrade
block
```

### 8.5 `UsageEvent`

```json
{
  "usage_event_id": "ue_phase1_model_input",
  "organization_id": "org_demo",
  "actor_ref": "actor_hash_demo",
  "actor_type": "user",
  "plan_code": "PROFESIONAL",
  "billing_period": "2026-05",
  "event_scope": "execution",
  "event_type": "model_input_tokens",
  "unit": "token",
  "quantity": 1200,
  "complexity": "SIMPLE",
  "cost_budget_ref": "cb_profesional_simple_v080",
  "cost_budget_version": "cost-budget-v0.8.0",
  "conversation_id": "conv_demo",
  "trace_id": "tr_demo",
  "model_call_id": "mc_demo",
  "estimated_cost": 0,
  "currency": "NONE",
  "created_at": "2026-05-26T00:00:00Z"
}
```

### 8.6 `RunEvent`

```json
{
  "run_id": "tr_phase1_demo",
  "event_scope": "internal",
  "event_name": "budget_checked",
  "sequence": 3,
  "resource_type": "budget_decision",
  "resource_ref": "bd_phase1_demo",
  "safe_message_code": null,
  "error_code": null,
  "payload_hash": "sha256:cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc",
  "created_at": "2026-05-26T00:00:00Z"
}
```

### 8.7 `ModelCallRecord`

```json
{
  "model_call_id": "mc_phase1_stub",
  "organization_id": "org_demo",
  "purpose": "other",
  "provider": "stub",
  "model": "stub-model",
  "prompt_version": "phase-1-pipeline-check-v1",
  "input_hash": "sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
  "output_hash": "sha256:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
  "token_usage": {
    "input_tokens": 0,
    "output_tokens": 0,
    "total_tokens": 0
  },
  "started_at": "2026-05-26T00:00:00Z",
  "completed_at": "2026-05-26T00:00:01Z",
  "status": "success",
  "error_code": null,
  "external_provider_call": false,
  "provider_call_audit_id": null
}
```

### 8.8 `ConversationCreate`

```json
{
  "title": "Consulta laboral",
  "case_id": null
}
```

### 8.9 `MessageCreate`

```json
{
  "content": "Necesito revisar una consulta jurídica.",
  "attachments": []
}
```

### 8.10 `HealthResponse`

```json
{
  "status": "HEALTHY",
  "service": "jusnova-backend",
  "version": "0.1.0",
  "environment": "local",
  "dependencies": {
    "database": {"enabled": true, "status": "HEALTHY"},
    "redis": {"enabled": true, "status": "HEALTHY"},
    "budgets": {"enabled": true, "status": "HEALTHY"},
    "opensearch": {"enabled": false, "status": "UNKNOWN"},
    "object_storage": {"enabled": false, "status": "UNKNOWN"}
  }
}
```

`status` y `dependencies.*.status` usan `docs/schemas/host-statuses.yaml` (`HEALTHY|DEGRADED|DOWN|UNKNOWN`). Una dependencia apagada por flag se reporta con `enabled=false` y `status=UNKNOWN`; no se introduce `disabled` como enum de health.

---

## 9. `budgets.yaml` obligatorio

Debe existir en:

```txt
docs/schemas/budgets.yaml
```

No se debe crear una carpeta alternativa de budgets. El archivo aceptado en 0.8 ya es la fuente canonica de limites por complejidad; Fase 1 debe validarlo al arrancar y mapearlo con `docs/policies/commercial-plans-v0.md` para el plan comercial minimo `PROFESIONAL`.

Extracto de límites esperados:

El archivo canónico completo conserva la metadata de gobierno (`document_status`, `date`, `responsible`, `related_decision`) definida en `docs/schemas/budgets.yaml`. El bloque siguiente muestra solo el extracto de límites que Fase 1 debe cargar y validar; no debe usarse para recrear el archivo sin metadata.

```yaml
cost_budget_version: cost-budget-v0.8.0
budgets:
  SIMPLE:
    discovery_calls_max: 1
    fetch_urls_max: 3
    ocr_pages_inline_max: 0
    reformulations_max: 0
    tool_rounds_max: 2
    time_budget_ms: 45000
    output_tokens_cap: 900
    research_credit_cost: 0
    async_allowed: false
  MEDIO:
    discovery_calls_max: 2
    fetch_urls_max: 6
    ocr_pages_inline_max: 2
    reformulations_max: 1
    tool_rounds_max: 3
    time_budget_ms: 90000
    output_tokens_cap: 1400
    research_credit_cost: 0
    async_allowed: false
  COMPLEJO:
    discovery_calls_max: 4
    fetch_urls_max: 10
    ocr_pages_inline_max: 10
    reformulations_max: 2
    tool_rounds_max: 4
    time_budget_ms: 180000
    output_tokens_cap: 2200
    research_credit_cost: 1
    async_allowed: false
  INVESTIGACION:
    discovery_calls_max: 8
    fetch_urls_max: 20
    ocr_pages_inline_max: 20
    reformulations_max: 3
    tool_rounds_max: 6
    time_budget_ms: 300000
    output_tokens_cap: 4000
    research_credit_cost: 2
    async_allowed: true
```

Reglas:

- El plan mínimo es `PROFESIONAL`.
- No crear planes debajo de 400 Bs.
- El sistema debe validar YAML al arrancar.
- Si el YAML es inválido, readiness debe fallar.
- No hardcodear límites dentro de servicios.
- Las claves de complejidad son `SIMPLE`, `MEDIO`, `COMPLEJO` e `INVESTIGACION`, no una taxonomia paralela en ingles.

---

## 10. Endpoints mínimos de Fase 1

### 10.1 Health

```http
GET /health/live
GET /health/ready
GET /health/version
```

Criterios:

- `live` no debe depender de DB;
- `ready` debe revisar DB y dependencias activas;
- `version` debe devolver versión de app, commit si está disponible y ambiente.
- el router de health se monta en raíz como `/health/*`, no bajo `/v1`.

### 10.2 Conversations

```http
POST /v1/conversations
GET  /v1/conversations/{conversation_id}
POST /v1/conversations/{conversation_id}/messages
```

P0 implementa `ConversationService`, `MessageService` y el pipeline interno que crea mensaje, run, budget decision, usage y trace. P1-04 expone `POST /v1/conversations/{conversation_id}/messages` como endpoint público y lo conecta a SSE; antes de P1-04, la validación P0 puede ejercitar el pipeline por service/integration tests.

`POST /v1/conversations/{conversation_id}/messages` debe:

1. crear `operational_run` y reservar IDs `ans_*`, `av_*`, `abstention_render_*` y `msg_*` de salida;
2. aplicar `CostGovernor`;
3. persistir mensaje de usuario;
4. crear mensaje técnico `assistant_final` seguro con `answer_id`, `trace_id` y `answer_version_ref`;
5. crear shells `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject` para `answer_blocked`;
6. registrar run events;
7. usar solo `ModelProvider` stub si corresponde;
8. registrar usage básico;
9. devolver metadata de run;
10. permitir streaming de estados.

### 10.3 Streaming

```http
GET /v1/conversations/{conversation_id}/runs/{run_id}/stream
```

Debe usar SSE.

Eventos publicos minimos de Fase 1:

```txt
run_queued
run_started
answer_blocked
run_failed
```

`answer_blocked` es el evento terminal de exito tecnico en Fase 1 cuando el pipeline corre pero no emite analisis juridico. Debe estar respaldado por `TraceObject.response_outcome=blocked`, `TraceObject.abstention_reason=policy_blocked`, `TraceObject.citation_audit` contract-compatible para `policy_blocked`, `AnswerVersion.response_outcome=blocked`, `AnswerVersion.abstention_render_ref` no nulo y `AbstentionRender.reason_code=policy_blocked`. `run_failed` solo se usa si la ejecucion falla antes de crear `AnswerVersion`/`TraceObject`. Eventos como `message_received`, `budget_checked`, `message_persisted`, `model_provider_stubbed` o `run_completed` son internos y se registran en `run_events` con `event_scope=internal`; no son eventos publicos SSE.

### 10.4 Usage

```http
GET /v1/usage/current
```

Debe devolver la shape mínima del API draft:

```json
{
  "period_start": "2026-05-01T00:00:00Z",
  "period_end": "2026-05-31T23:59:59Z",
  "plan_code": "PROFESIONAL",
  "subscription_id": "sub_phase1_demo",
  "usage_totals": {},
  "limits": {},
  "research_credits_balance": 8
}
```

Regla:

- `usage_totals` son agregados visibles, no filas raw de `usage_events`.
- `limits` se derivan de `docs/schemas/budgets.yaml` y la suscripción activa.
- `subscription_id` puede ser `null` solo si la organización todavía no tiene suscripción activa durante bootstrap controlado.
- `research_credits_balance` es numérico cuando `subscription_id` no es `null`; solo puede ser `null` en el mismo caso controlado en que no exista suscripción activa.

### 10.5 Research credits

```http
GET /v1/research-credits
```

Debe devolver la shape mínima del API draft:

```json
{
  "balance": 8,
  "currency": "NONE",
  "movements": [
    {
      "date": "2026-05-01T00:00:00Z",
      "type": "grant",
      "delta": 8,
      "balance_after": 8
    }
  ],
  "updated_at": "2026-05-01T00:00:00Z"
}
```

Regla:

- `movements[]` son movimientos resumidos, no filas raw de `UsageEvent`.
- `balance` se calcula desde `research_credits.balance_quantity`.
- `movements[]` se calcula desde `research_credit_movements`, no desde `UsageEvent` raw.
- Cada movimiento `debit` debe enlazar `usage_event_id` de un `UsageEvent.event_type=research_credit_used` y compartir `trace_id` o `answer_id`.
- Si la organización no tiene suscripción activa, debe responder con `ErrorEnvelope` y no inventar balance.

### 10.6 Traces

```http
GET /v1/answers/{answer_id}/trace-summary
GET /internal/runs/{run_id}/events
```

`GET /v1/answers/{answer_id}/trace-summary` debe existir porque `answer_blocked.trace_summary_url` lo referencia. Devuelve resumen seguro, nunca `TraceObject` completo.

`GET /internal/runs/{run_id}/events` debe ser endpoint interno o protegido, no parte del API publico v1. En Fase 1 sirve para validacion tecnica y solo devuelve eventos operativos sanitizados.

---

## 11. Tests mínimos

### 11.1 Unit tests P0

Obligatorios:

- settings cargan desde `.env`;
- `budgets.yaml` válido carga correctamente;
- `CostGovernor` devuelve `allow` para `SIMPLE` estándar;
- `CostGovernor` devuelve `block` o `degrade` cuando excede límites;
- `CostGovernor` bloquea o degrada `COMPLEJO`/`INVESTIGACION` si no puede registrar `research_credit_used`;
- `CommercialService` resuelve suscripción activa desde `subscriptions`, valida seed `PROFESIONAL` y rechaza doble suscripción current;
- `ErrorEnvelope` serializa con `request_id`;
- request middleware genera `request_id` si falta;
- request middleware respeta `X-Correlation-Id` si llega;
- `StubModelProvider` devuelve salida determinista;
- `UsageLedger` crea evento válido;
- `UsageLedger` registra `research_credit_used` con quantity `1` para `COMPLEJO` y `2` para `INVESTIGACION`;
- `research_credit_used` crea movimiento `debit` en `research_credit_movements` enlazado al `usage_event_id`;
- `GET /v1/research-credits` devuelve balance y movimientos resumidos;
- `StorageProviderStub` existe, no escribe documentos reales y falla cerrado con `storage_unavailable`;
- `TraceService` crea evento válido.

### 11.2 Integration tests P0

Obligatorios:

- `GET /health/live` devuelve 200 sin DB;
- `GET /health/ready` devuelve 200 con DB sana;
- migraciones aplican en DB limpia;
- `POST /v1/conversations` crea conversación;
- `test_tenant_isolation.py` cubre negativos cross-tenant de lectura, escritura, listado y borrado lógico solo donde exista superficie aceptada; no se crean rutas `DELETE` nuevas;
- `MessageService`/message pipeline interno crea mensaje, run, budget decision, usage event y trace events;
- `GET /v1/usage/current` devuelve `period_start`, `period_end`, `plan_code`, `subscription_id`, `usage_totals`, `limits` y `research_credits_balance`;
- error de validación devuelve ErrorEnvelope;
- respuesta incluye `X-Request-Id` y `X-Correlation-Id`.

### 11.3 Tests P1/P2 y beta foundations aplicables

- EvidencePack schema valida fixture mínimo;
- Citation schema valida valores contractuales `F1:P1` y `D1:P1`; los corchetes pertenecen solo al render visible;
- Claim schema clasifica criticality;
- Source Registry seed y `source_registry_health` existen con health inicial `UNKNOWN` y `live_fetch_enabled=false`;
- OpenSearch stub no rompe readiness si está disabled;
- Fetcher stub no hace red real;
- Evaluation runner skeleton corre sin dataset y declara que no satisface el gate beta hasta producir reporte versionado con `dataset_version`;
- Provider Reliability Layer (PRL) stub mapea timeout/rate limit/provider unavailable a `ErrorEnvelope`;
- ProviderCallAudit fixture valida contra schema y contra `provider-policy.md` + `provider-registry.yaml`;
- RawAccessEvent fixture valida contra schema y contra `privacy-security-policy.md`;
- PromptInjectionGuardStub bloquea riesgo `blocking` y usa `ErrorEnvelope.error_code=prompt_injection_blocked` cuando opera en frontera API.
- Si P1-04 entra, `POST /v1/conversations/{conversation_id}/messages` crea mensaje, run, budget decision, usage event y trace events;
- si P1-04 entra, `GET /v1/conversations/{conversation_id}/runs/{run_id}/stream` emite `run_queued`, `run_started` y terminal `answer_blocked` respaldado por `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject`;
- si P1-04 entra, `GET /v1/answers/{answer_id}/trace-summary` resuelve la URL devuelta por `answer_blocked`.

### 11.4 Regla de CI

No se acepta PR si:

- `make test` completo falla;
- tests P0/P1/P2 aplicables al alcance del PR fallan;
- migraciones no aplican;
- lint/type-check falla;
- hay secretos en repo;
- hay llamadas reales a OpenAI en tests;
- hay generación de respuesta jurídica en Fase 1.

---

## 12. Criterios de aceptación de Fase 1

Fase 1 está aceptada cuando:

1. `docker compose up` levanta backend + Postgres + Redis, y el compose define OpenSearch en profile `search` deshabilitado por defecto.
2. Las migraciones aplican desde cero.
3. `GET /health/live` y `GET /health/ready` funcionan.
4. Existe estructura modular conforme a este brief.
5. `request_id` y `correlation_id` aparecen en headers y logs.
6. Los errores usan `ErrorEnvelope`.
7. Se pueden crear organizaciones/usuarios semilla.
8. Se pueden crear conversaciones y mensajes.
9. Cada mensaje crea un `operational_run`.
10. Tests negativos de aislamiento tenant cubren lectura, escritura, listado y borrado lógico cross-tenant solo para superficies implementadas.
11. `CostGovernor` evalúa presupuesto desde `budgets.yaml`.
12. Cada ejecución registra `budget_decision`.
13. `UsageLedger` registra eventos básicos.
14. `plans`, `subscriptions`, `research_credits` y `CommercialService` resuelven plan/suscripción activa y créditos sin fuente paralela.
15. `research_credit_used` se registra obligatoriamente para `COMPLEJO` e `INVESTIGACION`, o esas complejidades se bloquean/degradan.
16. `GET /v1/research-credits` devuelve balance y movimientos resumidos.
17. `StorageProvider` interface/stub existe y falla cerrado sin escritura real.
18. `WorkflowGateway` interface y `LocalWorkflowGateway` existen; el core no importa Temporal directamente.
19. Trazabilidad mínima persiste `run_events`, `model_calls` y/o `tool_calls` cuando aplique.
20. `ModelProvider` existe como wrapper y el stub es el default.
21. El endpoint SSE emite estados técnicos.
22. `answer_blocked` queda verificado cross-contract: `TraceObject` valida completo contra schema, `TraceObject.citation_audit` cumple `policy_blocked`, `AbstentionRender` comparte `trace_id`/`answer_id`/`answer_version`/`response_outcome`, `reason_code == TraceObject.abstention_reason`, `AnswerVersion.answer_hash == AbstentionRender.render_hash`, `render_storage_ref` reconstruye `render_body_canonical` local DB y `source_trace_refs` es subconjunto real del `TraceObject`.
23. Provider Reliability Layer (PRL), ProviderCallAudit, RawAccessEvent y PromptInjectionGuard existen y fallan cerrado; RawAccessEvent valida schema + `privacy-security-policy.md` y PromptInjectionGuard aplica `prompt-injection-policy.md`.
24. `ProviderCallAuditService` valida schema + policy/registry; no acepta provider desconocido, familia divergente, audit sin ref contractual ni clases enviadas/devueltas fuera de allowlist. `policy_blocked` permite `attempted_data_classes[]` conocidas fuera de allowlist solo si no se envió payload.
25. No hay llamadas externas reales obligatorias.
26. Evidence/Citation/Claim schemas y modelos/stubs contractuales existen, y `0003_evidence_contract_stubs.py` aplica las tablas stub mínimas.
27. Source Registry seed mínimo y `source_registry_health` 1:1 existen.
28. Fetcher/Snapshot/OpenSearch/Eval stubs existen.
29. `make test` completo pasa en CI/local, incluyendo tests P0/P1/P2 aplicables a los artefactos implementados.

---

## 13. Riesgos de Fase 1

### Riesgo 1: construir una app plana

Síntoma:

- routers con lógica;
- módulos mezclados;
- imports cíclicos.

Mitigación:

- respetar estructura modular;
- revisión arquitectónica por PR;
- prohibir lógica de dominio en routers.

### Riesgo 2: costos invisibles

Síntoma:

- llamadas a modelos sin `UsageLedger`;
- uso no atribuido a organización;
- budget hardcodeado.

Mitigación:

- wrapper obligatorio `ModelProvider`;
- `CostGovernor` antes de ejecución;
- tests de usage.

### Riesgo 3: trazabilidad insuficiente

Síntoma:

- mensajes sin run;
- model/tool calls sin registro;
- errores sin correlation.

Mitigación:

- `operational_run` obligatorio por mensaje;
- middleware de request context;
- run events en cada etapa.

### Riesgo 4: overbuild de IA en Fase 1

Síntoma:

- prompts legales complejos;
- búsqueda real antes del Source Registry;
- respuestas jurídicas sin auditor.

Mitigación:

- `StubModelProvider` default;
- bloquear `answer_delta` legal;
- etiquetar endpoint como pipeline técnico.

### Riesgo 5: dependencia externa prematura

Síntoma:

- OpenAI hardcodeado;
- SerpAPI hardcodeado;
- OpenSearch requerido para todo;
- Temporal obligatorio antes de operar.

Mitigación:

- interfaces;
- stubs;
- feature flags;
- readiness por dependencia activada.

### Riesgo 6: auth falsa confundida con producción

Síntoma:

- headers dev tratados como seguridad real;
- ausencia de tenant context.

Mitigación:

- `AuthProvider` stub explícito;
- `APP_ENV=production` rechaza dev auth;
- registrar deuda controlada de auth real como blocker pre-beta/pre-market, sin asignarla a Fase 8.

---

## 14. Dependencias

### 14.1 Dependencias técnicas obligatorias

- Python 3.12;
- Docker/Docker Compose;
- PostgreSQL 16;
- Redis 7;
- FastAPI;
- Pydantic v2;
- pydantic-settings;
- SQLAlchemy 2.x;
- Alembic;
- pytest;
- pytest-asyncio;
- httpx;
- ruff;
- mypy;
- structlog;
- OpenTelemetry packages mínimos.

### 14.2 Dependencias opcionales en Fase 1

- OpenSearch container en profile `search`;
- MinIO container;
- Temporal container;
- OpenAI SDK solo como dependencia opcional futura; no se usa en Fase 1. Su uso queda para una decisión posterior con `ProviderCallAudit` productivo.

### 14.3 Dependencias documentales

Deben existir antes de cerrar Fase 1:

- ADRs fundacionales aceptados;
- `budgets.yaml`;
- policies críticas;
- JSON schemas críticos;
- `sprint-1-backlog.md`;
- `phase-1-development-plan.md`;
- risk register actualizado;
- open questions sin bloqueantes.

---

## 15. Qué NO construir en Fase 1

El equipo debe leer esta sección antes de escribir código.

No construir:

1. motor de búsqueda jurídica real;
2. adaptadores reales a portales oficiales;
3. crawler;
4. OCR;
5. upload documental real;
6. answer generator legal;
7. Citation Auditor real;
8. Claim Verifier real;
9. RAG;
10. embeddings;
11. sistema de pagos;
12. dashboard complejo;
13. microservicios;
14. frontend premium;
15. prompts jurídicos finales;
16. Web search provider real como dependencia obligatoria.

Razón:

> Antes de generar o recuperar derecho, JusNova debe poder medir, limitar, registrar, auditar y versionar cada ejecución.

---

## 16. Secuencia de implementación recomendada

Orden exacto:

1. Crear repo y estructura base.
2. Crear FastAPI app mínima.
3. Crear settings tipados.
4. Crear Docker Compose con Postgres/Redis y servicio OpenSearch definido en profile `search` deshabilitado por defecto.
5. Configurar Alembic.
6. Crear migración `0001_core_foundation`.
7. Crear migración `0002_trace_usage_budget`.
8. Crear logging y request context.
9. Crear ErrorEnvelope.
10. Crear health endpoints.
11. Crear Organizations/Users/Memberships.
12. Crear Conversations/Messages.
13. Preparar tablas/DTOs base de trazabilidad creados por `0002_trace_usage_budget`; esto no cierra P0-13 funcional.
14. Integrar y validar `docs/schemas/budgets.yaml` existente.
15. Implementar ModelProvider stub.
16. Implementar CostGovernor.
17. Implementar Plan/Subscription/ResearchCredit y `CommercialService`.
18. Implementar UsageLedger.
19. Completar P0-13 funcional: `OperationalRun`/`RunEvent` services, repos de model/tool calls, shells `answer_blocked` y endpoint interno de trace sobre las tablas de `0002`.
20. Implementar endpoint message pipeline.
21. Implementar SSE de estados.
22. Implementar schemas, modelos/stubs contractuales y migración `0003_evidence_contract_stubs` al ejecutar P1-01/P1-02 para Evidence/Citation/Claim.
23. Implementar `StorageProvider` interface/stub fail-closed al ejecutar P1-08, antes de beta y antes de cualquier upload/document storage.
24. Implementar Source Registry seed, stubs y migración `0004_source_registry_seed` al ejecutar P2-01.
25. Implementar `WorkflowGateway`/`LocalWorkflowGateway` al ejecutar P2-06, sin importar Temporal desde el core.
26. Implementar Provider Reliability Layer (PRL)/ProviderCallAudit/RawAccessEvent/PromptInjectionGuard stubs de beta blockers.
27. Implementar tests P0/P1/P2 aplicables.
28. Documentar cierre y handoff.

---

## 17. Handoff Checklist

```txt
[ ] Todos los ADRs fundacionales están Accepted.
[ ] Todos los JSON Schemas críticos existen.
[ ] `docs/schemas/budgets.yaml` existe y valida al arrancar.
[ ] policies críticas existen.
[ ] phase-1-implementation-brief.md existe.
[ ] sprint-1-backlog.md existe.
[ ] phase-1-development-plan.md existe.
[ ] risk-register.md actualizado.
[ ] open-questions.md no contiene bloqueantes.
[ ] El equipo entiende qué NO construir en Fase 1.
[ ] El equipo entiende que Fase 1 no genera análisis jurídico real.
[ ] El equipo entiende que CostGovernor y UsageLedger son P0, no extras.
[ ] El equipo entiende que ModelProvider debe ser wrapper, no llamada directa.
[ ] El equipo entiende que toda ejecución debe tener `run_id` y trazabilidad operativa; solo respuestas finalizadas o bloqueadas crean `TraceObject`.
```

---

## 18. Definition of Done de la Subfase 0.13

La Subfase 0.13 queda cerrada cuando:

- este brief está en `/docs/handoff/phase-1-implementation-brief.md`;
- el backlog Sprint 1 está en `/docs/handoff/sprint-1-backlog.md`;
- el plan completo de Fase 1 está en `/docs/phases/phase-1-development-plan.md`;
- no quedan ambigüedades sobre stack inicial;
- no quedan ambigüedades sobre qué se implementa primero;
- no quedan ambigüedades sobre qué no se debe construir;
- tras el cierre formal de 0.14/Fase 0, el equipo puede iniciar Fase 1 sin reunión de reinterpretación.

---

## 19. Definition of Done de Fase 1

Fase 1 queda cerrada cuando:

```txt
[ ] Backend FastAPI modular operativo.
[ ] Docker Compose operativo.
[ ] PostgreSQL conectado.
[ ] Redis conectado o degradación explícita.
[ ] Alembic migraciones 0001-0004 aplican desde cero.
[ ] Settings tipados operativos.
[ ] ErrorEnvelope aplicado a errores HTTP.
[ ] request_id/correlation_id funcionan.
[ ] Logging estructurado JSON funciona.
[ ] Health checks funcionan.
[ ] Organization/User/Conversation/Message implementados.
[ ] OperationalRun implementado.
[ ] RunEvent operativo implementado como DTO interno, sin sustituir `TraceObject`.
[ ] `answer_blocked` respaldado por `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject` schema-valid y por las invariantes cross-contract de `answer-versioning-policy.md` para bloqueos.
[ ] ModelCall/ToolCall persistence mínima implementada.
[ ] ModelProvider stub implementado.
[ ] StorageProvider interface/stub implementado y fail-closed.
[ ] `docs/schemas/budgets.yaml` integrado y validado.
[ ] CostGovernor funcional.
[ ] Plan/Subscription/ResearchCredit y CommercialService mínimos resuelven suscripción activa y saldo de créditos.
[ ] UsageLedger funcional.
[ ] `research_credit_used` se registra para `COMPLEJO`/`INVESTIGACION` o esas complejidades se bloquean/degradan.
[ ] `GET /v1/research-credits` devuelve balance y movimientos resumidos.
[ ] Tests de aislamiento tenant base cubren lectura, escritura, listado y borrado lógico cross-tenant solo en recursos con superficie aceptada; no existen rutas `DELETE` inventadas.
[ ] SSE de estados implementado.
[ ] WorkflowGateway/LocalWorkflowGateway implementados; Temporal no está acoplado al core.
[ ] Evidence/Citation/Claim schemas y modelos/stubs contractuales existen, respaldados por `0003_evidence_contract_stubs.py`.
[ ] Source Registry seed y `source_registry_health` existen.
[ ] Fetcher/Snapshot/OpenSearch/Evaluation stubs existen.
[ ] Provider Reliability Layer (PRL)/ProviderCallAudit/RawAccessEvent/PromptInjectionGuard stubs existen y fallan cerrado; ProviderCallAudit valida schema + provider policy/registry, RawAccessEvent valida schema + privacy/security policy y PromptInjectionGuard aplica prompt-injection policy.
[ ] `make test` completo pasa, incluyendo tests P0/P1/P2 aplicables a los artefactos implementados.
[ ] CI básico pasa.
[ ] No hay generación jurídica real.
[ ] No hay búsqueda legal real hardcodeada.
[ ] No hay dependencia externa obligatoria innecesaria.
```

---

## 20. Decisión final condicionada para iniciar Fase 1

Este handoff queda aceptado como instrucción de implementación, pero Fase 1 solo puede iniciar tras el cierre formal de 0.14 y de Fase 0.

La prioridad no es velocidad aparente. La prioridad es construir una base que permita que JusNova escale hacia un motor jurídico vivo, auditable y comercialmente vendible sin reconstruir desde cero.

La regla de ejecución para el equipo:

> **Cada componente implementado en Fase 1 debe mejorar una de estas cinco capacidades: estructura, medición, trazabilidad, presupuesto o persistencia. Si no mejora ninguna, no pertenece a Fase 1.**
