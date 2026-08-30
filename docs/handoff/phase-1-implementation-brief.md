# JusNova — Phase 1 Implementation Brief

**Ruta objetivo en repo:** `/docs/handoff/phase-1-implementation-brief.md`
**Versión:** 1.0
**Fecha:** 2026-05-26
**Estado documental:** Accepted
**Condicion de ejecucion:** Habilitada por el cierre formal de 0.14/Fase 0
**Responsable:** Codex / JusNova Chief Backend Architect
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
7. implementar `ModelProvider` como wrapper/stub, no llamadas libres a OpenAI;
8. implementar `CostGovernor` y `UsageLedger` en versión funcional mínima;
9. exponer health checks y readiness checks;
10. dejar stubs controlados para evidencia, búsqueda, snapshots y OpenSearch, junto con evaluation harness/runner y regression suite contractual inicial;
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
- `PromptVersionRegistry` interno, versionado y resoluble desde cada `ModelCall.prompt_version`;
- `CostGovernor` funcional básico;
- `budgets.yaml`;
- `UsageLedger` básico;
- `commercial` mínimo con `Plan`/`Subscription`/`ResearchCredit`;
- `StorageProvider` abstracto con stub y límites privados;
- `AuthProvider` productivo mínimo antes de beta, provider-neutral y tenant-scoped; `DevAuthProvider` solo local/test;
- trazabilidad mínima;
- shells `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject` para `answer_blocked`;
- modelos de Evidence/Citation/Claim como contratos fundacionales;
- Source Registry seed mínimo;
- stubs de fetcher/snapshot/OpenSearch y evaluation harness/runner con regression suite contractual inicial no vacía;
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
- autenticación enterprise completa (SSO corporativo, SCIM, administración avanzada y múltiples IdP); esto no difiere el `AuthProvider` productivo mínimo requerido por P1-09;
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

#### Gates de seguridad estáticos

- `make scan-secrets` ejecuta un secret scanner mantenido y versionado en CI sobre archivos trackeados e historial del rango del PR. La implementación de referencia usa Gitleaks con configuración `.gitleaks.toml`; cualquier alternativa exige evidencia equivalente y una decisión aceptada mediante ADR o una versión `Superseded` de este handoff.
- `make scan-sensitive-surfaces` ejecuta `scripts/scan_sensitive_surfaces.py` y falla si un schema aceptado permite cualquiera de las propiedades de la denylist cerrada de `privacy-security-policy.md` (`raw_prompt`, `raw_output`, `document_text`, `full_document`, `ocr_full_text`, `html_raw`, `user_message`) fuera de colecciones `x-invalid-*`, o si deja un objeto sanitizado abierto capaz de admitirlas sin una excepcion tipada y documentada. Incluye un fixture negativo de `Source.metadata` no permitida. Para detectar fugas aunque cambie el nombre de la propiedad, los tests inyectan en tiempo de ejecución sentinels sintéticos únicos para prompt, mensaje, documento, OCR y payload de proveedor, serializan log/traza y fallan si sobrevive cualquier sentinel.
- `make validate-schemas` recorre todos los nodos de los 29 contratos aceptados y falla si un `type: string` variable carece de `minLength: 1`/`maxLength` o un `type: array` carece de `maxItems`. Los enums/`const` son finitos; cualquier excepcion futura requiere ADR. Pruebas generadas rechazan `""`, `limite + 1` y formatos invalidos, y aceptan el mayor valor que tambien cumpla pattern/format/item constraints, incluidas variantes nullable y subschemas condicionales.
- La allowlist del secret scanner solo puede cubrir placeholders sintéticos concretos, hashes de fixtures y ejemplos inválidos conocidos; no se permiten exclusiones amplias por directorio ni patrón.
- Ambos comandos son gates de CI. `scan-secrets` se instala desde P0-01; `scan-sensitive-surfaces` y sus fixtures son entregables de P1-07 antes de beta.

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
  .gitleaks.toml
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
        canonical_json.py
        config_documents.py
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
        auth/
          __init__.py
          provider.py
          dev_provider.py
          production_adapter.py
          service.py
          schemas.py
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
          prompt_registry.py
          prompts/
            registry.yaml
            phase-1-pipeline-check-v1.txt
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
          claim_completeness.py
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
          runner.py
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
    scan_sensitive_surfaces.py

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
      test_canonical_json.py
      test_config_documents.py
      test_error_envelope.py
      test_budget_loader.py
      test_cost_governor.py
      test_commercial_service.py
      test_usage_ledger.py
      test_request_context.py
      test_model_provider_stub.py
      test_model_tool_call_persistence.py
      test_prompt_version_registry.py
      test_storage_provider_stub.py
      test_workflow_gateway.py
      test_schema_validator.py
      test_prl_stub.py
      test_provider_call_audit.py
      test_raw_access_event.py
      test_prompt_injection_guard_stub.py
      test_sensitive_surface_scans.py
      test_auth_provider.py
      test_source_registry_seed.py
      test_fetcher_stub.py
      test_snapshot_stub.py
      test_opensearch_stub.py
      test_eval_smoke.py
      test_eval_regression.py
      test_evidence_contracts.py
    integration/
      test_health.py
      test_conversations.py
      test_tenant_isolation.py
      test_streaming_status.py
      test_trace_persistence.py
      test_migrations.py
    fixtures/
      evals/
        contract-regression-v1.yaml
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
- En Fase 1, `ENABLE_OPENAI_REAL_CALLS` existe solo como guard rail y debe permanecer `false`; no se permiten llamadas reales a OpenAI ni a integraciones productivas de modelo, búsqueda, extracción, OCR, storage o workflow aunque existan stubs de `ProviderCallAudit`. Esta prohibición no sustituye ni bloquea el `AuthProvider` productivo de P1-09, que conserva su decisión, validaciones y gate pre-beta propios.

### 6.1.1 `core.canonical_json`

Responsabilidad:

- ofrecer una sola implementación de JSON Canonicalization Scheme (JCS, RFC 8785) para todos los hashes contractuales;
- parsear JSON raw con detección de nombres de propiedad duplicados antes de Pydantic, JSON Schema o persistencia;
- devolver bytes UTF-8 sin BOM y calcular `sha256:` en hexadecimal minúsculo sobre esos mismos bytes;
- rechazar `NaN`, `Infinity`, números no representables por JCS/I-JSON, claves no string y cualquier input que la implementación fijada no pueda canonicalizar.

Implementación cerrada de Fase 1:

- dependencia directa `rfc8785==0.1.4`, fijada también por el lockfile;
- `canonicalize_json(value) -> bytes` delega en esa librería sobre un valor JSON ya validado;
- `sha256_jcs(value) -> str` reutiliza exactamente esos bytes, sin volver a serializar;
- `loads_unique_json(raw)` rechaza claves duplicadas y luego entrega el valor a validación; `json.dumps(sort_keys=True)` no es sustituto de JCS;
- arrays conservan su orden y strings no reciben normalización Unicode adicional.

`AnswerContract`, `render_body_canonical`, `ModelCall`, `ToolCall` y `ProviderCallAudit` deben reutilizar este helper. No se admiten canonicalizadores por módulo ni hashes calculados desde una representación distinta del DTO minimizado que se persiste o audita.

### 6.1.2 `core.config_documents`

Responsabilidad:

- cargar los YAML aceptados de `docs/schemas` con una única semántica estricta tanto en CI como en runtime;
- rechazar ambigüedades antes de construir el objeto: claves duplicadas en cualquier nivel, cualquier tag explícito, anchors, aliases, merge keys, múltiples documentos YAML y raíz distinta de mapping;
- validar UTF-8 y después materializar modelos Pydantic cerrados con `extra="forbid"`, tipos estrictos y metadata de gobierno obligatoria (`document_status`, `date`, `responsible`, `related_decision`);
- aplicar las invariants y referencias cruzadas específicas de cada documento sin crear loaders paralelos en `CostGovernor`, PRL, Source Registry u otros módulos.

Implementación cerrada de Fase 1:

- dependencia directa `PyYAML==6.0.3`, fijada también por el lockfile;
- `load_strict_yaml(path_or_bytes) -> dict[str, object]` usa un `SafeLoader` derivado y versionado por el proyecto, con constructor de mappings que rechaza claves duplicadas antes de sobrescribirlas;
- un preflight sobre los eventos YAML rechaza aliases, anchors, cualquier tag explícito y merge keys antes de construir el objeto; no se permite `yaml.load` sin el loader compartido ni `safe_load` directo en módulos consumidores;
- `load_budgets`, `load_provider_registry` y los demás loaders tipados reutilizan esa primitiva y modelos cerrados; `scripts/validate_schemas.py` llama exactamente a los mismos loaders;
- `budgets.yaml` exige una entrada única para cada combinación canónica de plan/complejidad y todos los límites requeridos por `cost-budget.schema.json`;
- `provider-registry.yaml` exige nombres de provider y familias consistentes, clases conocidas, `training_use_allowed=false`, referencias resolubles y `error_mapping_target=provider_call_audit_error_code`; los valores de `error_mapping` se validan contra `ProviderCallAudit.error_code` y se traducen al `ErrorEnvelope` público mediante la tabla cerrada de `provider-policy.md`.

Un YAML inválido, ambiguo o con campos desconocidos impide `make validate-schemas`; si el documento es una dependencia crítica habilitada, también impide readiness. No se corrige silenciosamente, no se aceptan defaults que oculten campos faltantes y no se recrea el archivo canónico desde extractos documentales.

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
research_credit_required
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
  case_id text null -- Fase 1: siempre null; futuro FK tenant-safe a cases.case_id
  created_by_actor_ref text not null -- actor_hash_*|sha256:*, DB-only audit copy
  title text
  status text not null default 'active'
  deleted_at timestamptz null
  created_at timestamptz not null
  updated_at timestamptz not null
```

`created_by_actor_ref` es columna interna DB-only para auditoría. Debe excluirse de serializers y payloads `Conversation` antes de validar contra `docs/contracts/conversation.schema.json`; la superficie contractual usa `owner_actor_ref`/`owner_actor_type`.

Fase 1 no crea tabla ni servicio `Case`. Por tanto, `ConversationCreate.case_id` solo acepta `null` u omitido; cualquier `case_*` devuelve `ErrorEnvelope.error_code=validation_error`. No se permite guardar una ref no resoluble ni confiar en un ID de caso controlado por cliente. Cuando un trabajo posterior implemente `Case`, habilitar `case_*` exige FK/lookup con igualdad de `organization_id`.

### 6.8 `messages`

Responsabilidad:

- persistir mensajes de usuario/sistema/asistente;
- permitir versionado futuro;
- no guardar respuestas jurídicas definitivas aún.

Modelo mínimo:

```txt
messages
  id uuid pk -- surrogate interno opcional
  message_id text unique not null -- msg_*; CHECK char_length(message_id) between 1 and 160
  conversation_id text fk not null -- conv_*; CHECK char_length(conversation_id) between 1 and 160
  organization_id text fk not null -- org_*; CHECK char_length(organization_id) between 1 and 160
  actor_ref text not null -- actor_hash_*|service_*|system|sha256:*; max 160
  actor_type text not null -- user|system|service
  role text not null -- user|assistant|system_status|tool_status
  message_kind text not null -- user_input|assistant_final|assistant_clarification|system_notice|tool_progress
  content text not null -- CHECK char_length(content) between 1 and 20000 code points
  content_hash text not null -- sha256:*
  idempotency_key_hash text null -- sha256:*; DB-only, obligatorio solo para user_input creado por el endpoint canónico de mensajes
  request_fingerprint text null -- sha256:* sobre JCS del request tenant/actor-scoped; DB-only
  answer_id text null -- ans_*
  trace_id text null -- tr_*
  answer_version_ref text null -- av_*
  attachments jsonb not null default '[]' -- valida contra message.schema.json; max 20; no metadata libre
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
  conversation_id text fk not null -- conv_*
  input_message_id text fk unique not null -- msg_* iniciador del run en Fase 1
  actor_ref text not null -- actor_hash_*|service_*|system|sha256:*
  actor_type text not null -- user|system|service; coincide con el Message iniciador
  status text not null -- queued|running|completed|failed|cancelled
  mode text not null default 'STANDARD' -- enum de docs/schemas/response-modes.yaml
  complexity text not null default 'SIMPLE'
  started_at timestamptz
  completed_at timestamptz
  error_code text
  failure_stage text null
  created_at timestamptz not null
```

`idempotency_key_hash` y `request_fingerprint` se excluyen del payload contractual `Message`. Para `user_input` creado por `POST /v1/conversations/{conversation_id}/messages`, ambos son no nulos y la combinación `(organization_id, actor_ref, conversation_id, idempotency_key_hash)` es única; mensajes internos/asistente usan ambos en `null`. El fingerprint es `sha256(canonicalize({organization_id, actor_ref, conversation_id, content_hash, attachments}))` con el helper JCS compartido. `actor_ref` se deriva de auth, no del cliente. La key raw nunca se persiste, loguea ni incluye en trazas.

Maquina de estados operativa cerrada:

- `queued`: `started_at=null`, `completed_at=null`, `error_code=null` y `failure_stage=null`;
- `running`: `started_at >= created_at`, `completed_at=null`, `error_code=null` y `failure_stage=null`;
- `completed`: `completed_at >= started_at >= created_at`, `error_code=null` y `failure_stage=null`;
- `failed`: `error_code`/`failure_stage` no nulos y `completed_at >= created_at`; si el run alcanzó `running`, también exige `completed_at >= started_at >= created_at`, pero un fallo confirmado todavía en cola conserva `started_at=null`;
- `cancelled`: `completed_at >= created_at`, `error_code=null` y `failure_stage` no nulo para registrar el punto de cancelacion; `started_at` es `null` si se cancela en cola o cumple `completed_at >= started_at >= created_at` si ya estaba ejecutando;
- toda transicion usa compare-and-set o lock equivalente y solo admite `queued -> running|failed|cancelled` y `running -> completed|failed|cancelled`; `queued -> failed` representa exclusivamente un error posterior al commit del run pero anterior al inicio de ejecución, y un estado terminal es inmutable.
- En Fase 1 cada run nace de exactamente un mensaje de entrada: `input_message_id` resuelve un `Message` del mismo tenant/conversation y sus `actor_ref`/`actor_type` coinciden. No se persiste `user_id` crudo en esta superficie operativa.
- Cuando el run materializa `TraceObject`, `operational_runs.input_message_id` aparece en `TraceObject.input_message_ids[]`, y tenant, conversation y actor coinciden entre run, mensaje y traza.

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
  run_id text fk not null -- tr_*, DB-only operational relation
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
  run_id text fk not null -- tr_*, DB-only operational relation
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

`model_calls.run_id` y `tool_calls.run_id` son columnas internas para resolver el `operational_run`; se excluyen al serializar y validar `ModelCall`/`ToolCall`, cuyos JSON Schemas son cerrados. La asociacion contractual final se reconstruye desde `TraceObject.model_calls[]`/`tool_calls[]`, sin agregar `run_id` a esos payloads.

En Fase 1, `run_events` persiste solo eventos internos. Los eventos SSE publicos (`run_queued`, `run_started`, `answer_blocked`, `run_failed`, etc.) se emiten como `StreamEvent` y, si se requiere auditoria, se reflejan con un evento interno `stream_event_emitted`; no se mezclan como `event_name` de `run_events`.

Por cada run, `sequence` empieza en `1` con un unico `run_created`, es contigua y estrictamente creciente; `created_at` es no decreciente respecto del evento anterior y nunca anterior a `operational_runs.created_at`. La asignacion de secuencia se serializa con lock/contador transaccional o mecanismo equivalente, de modo que concurrencia y retries no creen duplicados ni huecos confirmados. `message_received` y `message_persisted` son logs de request/repository anteriores a la creacion del run y no pertenecen a `run_events`, porque `operational_runs.input_message_id` ya debe resolver al insertar el run.

Enum operativo cerrado de `run_events.event_name` en Fase 1:

```txt
run_created
budget_checked
run_execution_started
model_provider_stubbed
usage_event_recorded
stream_event_emitted
run_completed
run_failed
run_cancelled
error_envelope_returned
```

La creacion confirmada de `operational_run`, su unico `BudgetDecision`, `run_created` y `budget_checked` ocurre en una sola transaccion; si no puede resolverse/persistirse el budget, no queda un run huerfano. `run_execution_started` se emite una sola vez al confirmar `queued -> running`; una cancelacion o fallo desde `queued` no lo inventa. La presencia de los demas eventos derivados es verificable, no decorativa: `model_provider_stubbed` exige un `ModelCall` enlazado; cada `usage_event_recorded` usa `resource_type=usage_event` y un `resource_ref=ue_*` unico dentro del run que resuelve un `UsageEvent` real; `stream_event_emitted`, una emision SSE real; y `error_envelope_returned`, un `ErrorEnvelope` efectivamente devuelto con solo `error_code`/`safe_message_code` sanitizados y sin payload crudo. Todo run que alcanza estado terminal registra exactamente uno de `run_completed|run_failed|run_cancelled`, coherente con `operational_runs.status`; un fallo previo a otra etapa no inventa el evento de esa etapa.

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
  created_by_ref text not null -- system|service_*|actor_hash_*|support_hash_*|sha256:*

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
  render_storage_ref text not null -- igual a abstention_render_id en Fase 1; ref DB-local unica
  render_body_canonical jsonb not null -- DB-only; objeto cerrado {"content": string}
  redaction_profile text not null -- USER_SUMMARY_SAFE
  created_at timestamptz not null
  created_by_ref text not null -- system|service_*|actor_hash_*|support_hash_*|sha256:*

trace_objects
  trace_id text pk/fk -- tr_* igual a operational_runs.run_id
  organization_id text fk not null -- org_*
  answer_id text fk not null -- ans_*
  answer_version_ref text fk not null -- av_*
  conversation_id text fk not null -- conv_*
  output_message_id text fk not null -- msg_* con message_kind=assistant_final
  response_outcome text not null -- blocked
  abstention_reason text not null -- policy_blocked
  data_classification text not null -- DB-only; enum de data-classification.yaml, max(inputs, INTERNAL_TRACE_RESTRICTED)
  trace_object_payload jsonb not null -- TraceObject schema-valid minimizado, sin prompts/docs raw
  created_at timestamptz not null
```

Regla contractual para el cierre tecnico de Fase 1:

- `answer_blocked` no es un error HTTP; es el evento terminal publico de una ejecucion tecnica completada sin analisis juridico.
- `trace_object_payload` debe validar completo contra `docs/contracts/trace-object.schema.json`; no basta con persistir columnas escalares.
- La proyeccion relacional es exacta: `trace_object_payload.trace_id`, `organization_id`, `answer_id`, `answer_version_ref`, `conversation_id`, `output_message_id`, `response_outcome` y `abstention_reason` coinciden con las columnas homonimas de `trace_objects`; `trace_object_payload.answer_version` coincide con la version del `AnswerVersion` resuelto. Cualquier divergencia invalida la transaccion aunque ambos lados sean localmente validos.
- `trace_objects.data_classification` es DB-only y no se serializa dentro de `TraceObject`, cuyo schema es cerrado. Se calcula deterministicamente como la clase con mayor `sensitivity_rank` entre mensajes, claims, evidencia, retrieval y riesgos resueltos, con minimo `INTERNAL_TRACE_RESTRICTED`; cualquier `RawAccessEvent` para esa traza debe usar exactamente ese valor.
- `TraceObject.response_outcome=blocked`.
- `TraceObject.abstention_reason=policy_blocked`.
- `TraceObject.citation_audit` debe cumplir las reglas condicionales de `trace-object.schema.json` para `blocked/policy_blocked`: `overall_status` en `failed|blocked` y `blocking_failures[]` contiene `failure_code=policy_blocked`.
- `AnswerVersion.response_outcome=blocked`.
- `AnswerVersion.answer_contract_ref=null`.
- `AnswerVersion.abstention_render_ref` debe apuntar a `AbstentionRender`.
- `answers.latest_answer_version_ref` debe resolver a un `AnswerVersion` con el mismo `answer_id` y `organization_id`; en Fase 1 apunta a la version 1 creada en esa misma transaccion.
- Aunque el modelo conceptual permite `Answer -> AnswerVersion 0..n` para estados futuros, Fase 1 no persiste un `Answer` con cero versiones: la fila inicial se crea atomicamente con `AnswerVersion` 1, por lo que `latest_answer_version_ref` es no nulo en este perfil.
- `answers.response_outcome` debe ser igual al `response_outcome` de la version resuelta por `latest_answer_version_ref`.
- Al incorporar versiones posteriores, `latest_answer_version_ref` se actualiza atomicamente a la version vigente con el mayor `answer_version`; no puede quedar atrasado ni apuntar a una version futura.
- La relacion es 1:1: `answer_versions.abstention_render_ref` es unico cuando no es null y `abstention_renders(answer_id, answer_version)` es unico; no se admiten renders huerfanos o multiples para una misma version.
- `AbstentionRender.response_outcome=blocked`.
- `AbstentionRender.reason_code=policy_blocked`.
- `AbstentionRender.source_trace_refs` debe existir en el payload y como columna JSONB si se materializa la tabla.
- `AbstentionRender` debe resolver al mismo `trace_id`, `answer_id`, `answer_version` y `response_outcome` que `AnswerVersion` y `TraceObject`.
- `AbstentionRender.source_trace_refs` debe ser subconjunto real de `TraceObject.evidence_pack_ids[]`, `TraceObject.claims[].claim_id`, citas en `TraceObject.citation_audit.results[]`/`blocking_failures[]` y fuentes usadas, rechazadas o auditadas por `TraceObject`.
- Para bloqueos y abstenciones, `AnswerVersion.answer_hash` debe coincidir con `AbstentionRender.render_hash`, conforme a `answer-versioning-policy.md`.
- En Fase 1, `render_storage_ref == abstention_render_id` y resuelve de forma unica el `render_body_canonical` almacenado en esa misma fila; no apunta a S3, filesystem ni storage externo. Esta persistencia DB de texto seguro de bloqueo no cuenta como escritura real de `StorageProvider`.
- `render_body_canonical` tiene exactamente la forma cerrada `{ "content": <string> }`, sin propiedades adicionales. Su `content` es byte a byte igual a `messages.content` del `Message` resuelto por `TraceObject.output_message_id`.
- `render_hash` es `sha256(canonicalize(render_body_canonical))`; `canonicalize` usa JCS (RFC 8785) sobre el valor JSON validado, UTF-8 sin BOM, conserva el orden de arrays, no normaliza Unicode y rechaza nombres de propiedad duplicados antes de persistir. El valor usa prefijo `sha256:` más digest hexadecimal minusculo. Si el render canónico no puede reconstruirse desde `render_storage_ref`, difiere del mensaje visible o contiene otra shape, `answer_blocked` no puede considerarse válido.
- `TraceObject.output_message_id` debe apuntar al mismo mensaje `assistant_final` tecnico y seguro; su `content_hash` es el SHA-256 de los bytes UTF-8 exactos de ese mismo `content`, conforme a `document-security-policy.md`.
- `trace_summary_url` debe resolver a `/v1/answers/{answer_id}/trace-summary`.
- Estos artefactos son shells seguros de bloqueo; no contienen respuesta legal sustantiva, prompts crudos, documentos crudos ni salidas completas de modelo.

Estrategia transaccional obligatoria:

- Las FK circulares entre `answers.latest_answer_version_ref`, `answer_versions.answer_id`, `answer_versions.trace_id`, `answer_versions.abstention_render_ref`, `abstention_renders.trace_id`, `trace_objects.answer_version_ref` y `messages.answer_version_ref` deben ser `DEFERRABLE INITIALLY DEFERRED` en PostgreSQL.
- La igualdad de answer/tenant del puntero se fuerza con FK compuesta deferrable `(answers.answer_id, answers.latest_answer_version_ref, answers.organization_id) -> answer_versions(answer_id, answer_version_id, organization_id)` o constraint equivalente; la vigencia maxima y el `response_outcome` se validan en la misma transaccion de servicio.
- La creación de `answer_blocked` forma parte de la unica transacción terminal de publicación: reservar IDs `tr_*`, `ans_*`, `av_*`, `abstention_render_*` y `msg_*`; crear el mensaje `assistant_final` tecnico; crear `answers`, `answer_versions`, `abstention_renders` y `trace_objects`; crear los `UsageEvent` obligatorios y el débito/movimiento de crédito cuando aplique; cerrar el run en `completed`; validar todo al commit.
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
  run_id text fk unique not null -- tr_*; exactamente una decision confirmada por operational_run
  organization_id text fk not null -- org_*
  plan_code text not null
  complexity text not null
  mode text not null -- enum de docs/schemas/response-modes.yaml
  cost_budget_ref text not null -- cb_<plan>_<complexity>_vNNN
  cost_budget_version text not null -- cost-budget-v*
  decision text not null -- allow|degrade|block
  decision_reason_code text not null -- within_budget|nonessential_output_capped|budget_exhausted|research_credit_required|policy_blocked
  budget_snapshot jsonb not null
  created_at timestamptz not null
```

`complexity` conserva la complejidad solicitada y efectiva en Fase 1: no existe downgrade de complejidad aprobado. `decision=degrade` solo puede aplicar `decision_reason_code=nonessential_output_capped` y reducir salida no esencial dentro del mismo `CostBudget`; nunca elimina evidencia, citas, warnings o trazabilidad. `allow` exige `within_budget`; `block` exige `budget_exhausted|research_credit_required|policy_blocked`. `budget_snapshot` es una copia exacta y cerrada de los limites del `CostBudget` resuelto por `cost_budget_ref`/`cost_budget_version`, y plan/complejidad deben coincidir con ese budget.

Toda fila confirmada de `budget_decisions` pertenece exactamente a un `operational_run`: comparte `organization_id`, `mode` y `complexity`, y `run_id` es unico/no nulo. En una ejecucion publicada, `organization_id`, `plan_code`, `complexity`, `cost_budget_ref` y `cost_budget_version` coinciden exactamente entre esa decision, el `TraceObject` y cada `UsageEvent` de scope `execution`; el runner no puede reconstruirlos de fuentes distintas. Los unit tests de `CostGovernor` pueden evaluar un DTO en memoria sin run; eso no autoriza persistir una decision huerfana.

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
  organization_id text fk not null -- org_*
  research_credit_id text fk not null -- rc_*
  subscription_id text fk not null -- sub_*
  usage_event_id text fk null -- ue_*, required for debit
  movement_type text not null -- grant|debit|expiry en Fase 1; adjustment reservado y rechazado
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
- La maquina de estados es cerrada: `active|trialing|past_due` exige `ended_at=null`; `cancelled` exige `ended_at` no nulo y `ended_at >= started_at`. `CommercialService` nunca trata una fila `cancelled` como current ni acepta una fila abierta con estado terminal.
- Una suscripcion current solo es utilizable si su `plan_id` resuelve un `plans.status=active`, su `plan_code` pertenece al enum contractual y `cost_budget_version` coincide con el catalogo `budgets.yaml` cargado; cualquier divergencia falla cerrado antes de decidir budget o consumo.
- `plans.cost_budget_version` selecciona la version del catalogo; el `CostGovernor` resuelve por ejecucion `cost_budget_ref=cb_<plan>_<complexity>_vNNN` desde `docs/schemas/budgets.yaml`.
- `plans.monthly_research_credits` es la fuente de verdad para grants mensuales. En fixtures de Fase 1, `PROFESIONAL` usa `8` créditos mensuales, dentro del rango 8-12 de `docs/policies/commercial-plans-v0.md`.
- `research_credits` guarda el saldo por suscripcion y periodo; `research_credit_movements` guarda grants/debitos resumibles por `GET /v1/research-credits`.
- `research_credits.organization_id` debe coincidir con la organizacion de `subscriptions.subscription_id`; cada `research_credit_movements.organization_id` debe coincidir con su `research_credit_id`, `subscription_id` y, cuando existan, `usage_event_id`, `trace_id` y `answer_id`.
- En `UsageEvent`, las columnas DB-only `subscription_id` y `research_credit_id` deben resolver al mismo `organization_id`; para `research_credit_used`, el credito pertenece a esa misma suscripcion y periodo.
- Cada periodo `research_credits` tiene exactamente un movimiento inicial `grant`, creado en la misma transaccion. Su `delta` y `balance_after` coinciden exactamente con `plans.monthly_research_credits`; no se admiten grants acumulativos, retries duplicados ni un monto derivado de otra version de plan.
- Cada debit debe crear `UsageEvent.event_type=research_credit_used` y `research_credit_movements.movement_type=debit` en la misma transaccion terminal de publicacion.
- Regla de movimiento: `debit` de una respuesta publicada implica `usage_event_id`, `trace_id` y `answer_id` no nulos, `delta < 0` y `balance_after >= 0`.
- Conciliacion completa: el primer movimiento del periodo es un `grant` y cumple `0 + delta == balance_after`; cada movimiento posterior satisface `previous.balance_after + delta == balance_after`. `granted_quantity` coincide con la suma de grants, `used_quantity` con la suma absoluta de debitos y `balance_quantity` con el ultimo `balance_after` y con la suma de todos los `delta` del periodo.
- Conciliacion de debito: `abs(research_credit_movements.delta) == UsageEvent.quantity`, `balance_after` coincide con el `research_credits.balance_quantity` resultante y `used_quantity` aumenta exactamente por esa misma cantidad.
- Regla de signo: `grant` implica `delta > 0`; `debit` implica `delta < 0`; `expiry` implica `delta <= 0` y `balance_after=0`. Un `expiry` usa `delta=0` cuando el periodo ya estaba agotado y `delta=-balance_anterior` cuando quedaba remanente. `adjustment` queda reservado pero no ejecutable en Fase 1: el service lo rechaza hasta que un contrato posterior defina motivo cerrado, actor, aprobacion y conciliacion.
- Estado derivado: `active` exige `balance_quantity > 0`; `exhausted` exige `balance_quantity = 0` en un periodo vigente; `expired` exige periodo cerrado, `balance_quantity = 0` y exactamente un movimiento terminal `expiry`. Un debito solo opera sobre `active`; llegar exactamente a cero cambia el estado a `exhausted` en la misma transaccion. Al cerrar el periodo, el service crea el `expiry` unico incluso si su delta es cero, cambia el estado a `expired` y no admite movimientos posteriores. El siguiente grant mensual crea un nuevo registro de periodo. Cualquier reactivacion futura mediante `adjustment` requiere antes el contrato de actor, aprobacion, motivo y conciliacion indicado arriba.
- `billing_period=YYYY-MM` del credito se deriva de `research_credits.created_at` y de su grant inicial en UTC, y representa el intervalo semiabierto desde el primer dia del mes a `00:00:00Z` hasta el primer dia del mes siguiente a `00:00:00Z`; esa regla determina de forma unica si el periodo esta vigente o cerrado en Fase 1.
- `grant` y `debit` usan `created_at` dentro del intervalo del credito. El `expiry` terminal usa `created_at` mayor o igual al limite superior del periodo, porque documenta su cierre, y nunca precede al movimiento anterior. `UsageEvent.billing_period` de un debito coincide con el periodo del credito; `research_credits.updated_at` es mayor o igual que `created_at` del credito y coincide con el ultimo movimiento confirmado, incluido `expiry`.
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

`research_credit_used` es obligatorio cuando la ejecución publicable usa `complexity=COMPLEJO` o `complexity=INVESTIGACION`. Su `quantity` debe coincidir con `CostBudget.research_credit_cost`: `1` para `COMPLEJO` y `2` para `INVESTIGACION`. Si no existe saldo/reserva de créditos o el débito no puede registrarse, `CostGovernor` debe bloquear; no se permite publicar una ejecución de esas complejidades sin el débito conciliable. El payload `UsageEvent` sigue cerrado por `docs/contracts/usage-event.schema.json`; `subscription_id` y `research_credit_id` son columnas internas DB-only y se excluyen antes de validar/exportar el payload contractual.

Cada ejecución publicada crea exactamente un evento de consulta con `quantity=1`: `standard_query` para `SIMPLE|MEDIO` y `complex_query` para `COMPLEJO|INVESTIGACION`. Ese evento y, cuando aplique, `research_credit_used` incluyen ambos `trace_id` y `answer_id`; sus `organization_id`, `plan_code`, `complexity`, `cost_budget_ref` y `cost_budget_version` coinciden con el `BudgetDecision` efectivo y el `TraceObject`. `conversation_id` por sí solo no identifica consumo publicable ni permite idempotencia por run.

Si el saldo es insuficiente, Fase 1 devuelve `ErrorEnvelope.error_code=research_credit_required`, no publica respuesta y no crea `research_credit_used`; no existe una regla aprobada de downgrade de complejidad. `decision=degrade` queda reservado al recorte de salida no esencial dentro de la misma complejidad y no evita el credito exigido. Consultar una suscripcion activa agotada en `GET /v1/research-credits` sigue siendo exitoso con `balance=0`; la ausencia de suscripcion activa devuelve `not_found`.

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
- resolver cada `ModelCall.prompt_version` contra un registro versionado e inmutable;
- permitir stub determinista.

Regla:

- En Fase 1 el provider por defecto es `StubModelProvider`.
- Fase 1 no permite llamadas reales a OpenAI ni a providers productivos de modelo, búsqueda, extracción, OCR, storage o workflow. El `AuthProvider` productivo de P1-09 queda fuera de esta prohibición y no habilita esas integraciones.
- Activar cualquier proveedor real queda fuera de Fase 1; requiere decisión aceptada posterior, `provider_call_audits` productivo contract-compatible con `docs/contracts/provider-call-audit.schema.json` y enlace desde `ModelCall`/`ToolCall`.
- `PromptVersionRegistry` es un registro interno del módulo, no un contrato API ni un JSON Schema público. Se implementa con `prompt_registry.py` y el manifest versionado `prompts/registry.yaml`; no crea tabla ni migración en Fase 1.
- Cada entrada usa exactamente `prompt_version`, `purpose`, `template_path`, `content_hash`, `status` y `created_at`. `purpose` usa el enum de `model-call.schema.json`; `content_hash` usa `sha256:` + 64 hex sobre los bytes exactos UTF-8 sin BOM del template; `status` usa `active|deprecated`.
- `template_path` es una ruta POSIX relativa a `src/jusnova/modules/model_provider/`; no admite ruta absoluta ni segmentos `..`.
- `prompt_version` es único e inmutable: cambiar contenido, purpose o template exige una versión nueva. El loader rechaza duplicados, paths inexistentes, hash divergente y versiones desconocidas antes de invocar el provider o persistir el `ModelCall`.
- Solo una entrada `status=active` puede seleccionarse para una invocación nueva o un `ModelCall` nuevo. Una entrada `deprecated` permanece resoluble únicamente para auditoría, reconstrucción o replay controlado de `ModelCall` históricos y nunca habilita una llamada de provider. La transición permitida es `active -> deprecated`; no se elimina una versión usada ni se reactiva sin crear una versión nueva.
- El seed mínimo es `phase-1-pipeline-check-v1`, con `purpose=other` y contenido técnico determinista sin prompt jurídico, datos de usuario ni payload externo.
- El archivo seed contiene exactamente `Fase 1: pipeline tecnico ejecutado. No se emite analisis juridico.` codificado en UTF-8 sin BOM y terminado por un único LF; el hash incluye ese LF.

Forma mínima del manifest interno:

```yaml
registry_version: prompt-registry-v1
prompts:
  - prompt_version: phase-1-pipeline-check-v1
    purpose: other
    template_path: prompts/phase-1-pipeline-check-v1.txt
    content_hash: sha256:43eab7b2272f23689a9dc0a21c6a4c2e9da2406a008da7d328eb5a90eed90b3d
    status: active
    created_at: "2026-05-26T00:00:00Z"
```

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

### 6.19 `evals`

Responsabilidad:

- implementar el evaluation harness y runner iniciales;
- ejecutar en Fase 1 una regression suite contractual sintética, versionada y no vacía;
- no declarar calidad jurídica ni satisfacer beta hasta ejecutar el golden dataset y los gates completos de 0.12;
- permitir que CI valide determinísticamente contratos, policies y formato de reporte.

Ruta canónica:

- `src/jusnova/modules/evals/runner.py`. El nombre no lleva `stub`: aunque la suite inicial sea sintética y no pruebe calidad jurídica, el runner debe ejecutar fixtures reales y producir resultados deterministas al cierre de Fase 1.

Comandos mínimos:

- `make eval-smoke`: valida arranque y formato con fixture vacío;
- `make eval-regression`: ejecuta `tests/fixtures/evals/contract-regression-v1.yaml`, suite contractual inicial no vacía, y produce un reporte versionado con `suite_version`, `dataset_version=phase1-contract-regression-v1` y resultados por fixture. Ese reporte se marca `beta_eligible=false` hasta usar el golden dataset aprobado y todas las métricas/gates de 0.12.

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

- cubrir los beta blockers asignados a Fase 1 sin activar providers productivos de modelo, búsqueda, extracción, OCR, storage o workflow ni análisis jurídico; el `AuthProvider` productivo de P1-09 conserva su gate separado;
- dejar Provider Reliability Layer (PRL), provider audit, raw access audit y prompt-injection blockers en modo fail-closed;
- hacer que los gates de beta sean verificables desde tests técnicos.

Modelos mínimos:

```txt
provider_call_audits
  provider_call_audit_id text pk -- pca_*
  organization_id text fk not null -- org_*
  logical_call_id text not null -- pcall_*, agrupa todos los intentos de una llamada
  request_id text not null -- rq_*
  correlation_id text not null -- corr_*
  attempt_number integer not null -- >= 1; unico por logical_call_id
  attempt_kind text not null -- initial|retry|fallback
  operation_idempotent boolean not null
  idempotency_key_hash text null -- sha256:*; nunca key raw
  fallback_route_id text null -- proute_* solo para attempt_kind=fallback
  provider_family text not null
  provider_name text not null
  trace_id text null -- tr_*, solo si ya existe un TraceObject schema-valid; nunca un operational run sin traza final
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

- `ProviderReliabilityLayerStub` debe mapear timeout, rate limit y provider unavailable a `ErrorEnvelope` sin llamar proveedores reales. Implementa los DTOs internos `ProviderRoute` y `ProviderCallContext` definidos en `docs/contracts/provider-interfaces.md`. `max_attempts` cuenta el intento inicial; solo reintenta operaciones idempotentes o protegidas por `idempotency_key_hash`, nunca errores de policy/validación.
- `RetrievalPlan.fallback_allowed=true` es permiso, no selector. El fallback exige `ProviderRoute` activo compatible con familia, tenant, allowlist y budget. El registry aceptado contiene `fallback_routes: []`, por lo que la ruta productiva siempre devuelve `provider_unavailable`; el test positivo usa un registry fixture completo, cerrado y exclusivo de tests con dos providers stub distintos de la misma familia y una ruta entre ellos. Los nombres resuelven dentro de ese fixture y este nunca modifica ni sustituye `docs/schemas/provider-registry.yaml` en runtime.
- Cada intento persiste un `ProviderCallAudit` con el mismo `logical_call_id`, `organization_id`, `request_id` y `correlation_id`, y con `(logical_call_id, attempt_number)` único. Retry/fallback no duplica usage ni cargos. El `provider_call_audit_id` singular de `ModelCall`/`ToolCall` apunta al intento terminal; los intentos anteriores se resuelven por la ref dueña y `logical_call_id`.
- La referencia circular entre una llamada externa y su audit terminal se persiste en una sola transacción: se admite insertar `ModelCall`/`ToolCall.provider_call_audit_id=null` solo como estado transitorio no confirmado, crear los audits enlazados por `model_call_id`/`tool_call_id`, asignar el audit terminal y validar ambos payloads antes del commit; una FK diferible equivalente también es válida. Nunca puede quedar confirmada una llamada externa sin audit terminal ni un audit que apunte a una llamada inexistente o de otro tenant.
- `external_call_mode=always|never|deployment_configured` del registry debe resolver de forma determinista `ProviderMetadata.external_call`; `deployment_configured` exige un booleano explícito antes de habilitar el provider.
- `ProviderCallAuditService` debe validar fixtures contra `provider-call-audit.schema.json` y contra `provider-policy.md` + `docs/schemas/provider-registry.yaml`: exige todos los campos de `x-reliability-policy.required_per_attempt`; `provider_name` resuelve en registry; `provider_family` coincide; `data_sent_classes[]` y `data_returned_classes[]` respetan allowlists/clases conocidas; y los estados `policy_blocked|timeout|rate_limited|cancelled|error` cumplen sus `error_code`/campos condicionales. Para `status=success|error|timeout|rate_limited|cancelled`, `attempted_data_classes[]` debe representar el mismo set que `data_sent_classes[]` y respetar allowlist; para `status=policy_blocked`, `attempted_data_classes[]` puede contener clases conocidas fuera de allowlist, pero `data_sent_classes=[]`, `data_returned_classes=[]` y `policy_decision` debe ser `blocked_by_classification|blocked_by_feature_flag|blocked_by_kill_switch`. Si la razón es clase fuera de allowlist, `policy_decision=blocked_by_classification`. Los campos `error_code`, `region_or_residency`, `feature_flag` y `kill_switch` son nullable en tabla pero obligatorios cuando el schema o la policy los exigen. Toda llamada externa futura debe fallar si no puede crear `provider_call_audit_id` policy-valid.
- `ProviderCallAudit` no puede quedar huérfano: al menos uno de `trace_id`, `model_call_id`, `tool_call_id`, `retrieval_run_id`, `usage_event_id` o `cost_report_id` debe existir en fixtures contables, y `organization_id` debe coincidir con cada recurso resuelto. Un `trace_id` no nulo resuelve exclusivamente a un `TraceObject` schema-valid; el `run_id=tr_*` reservado de `operational_runs` no satisface esa referencia mientras no exista la traza final. Si el intento ocurre antes de materializarla o el run falla temprano, `trace_id` queda nulo y el audit se ancla mediante otra ref contractual resoluble.
- Los `null` de columnas DB no se copian ciegamente al payload contractual. Si `trace_id`, `model_call_id`, `tool_call_id`, `retrieval_run_id`, `usage_event_id`, `cost_report_id`, `region_or_residency`, `feature_flag` o `kill_switch` están en `null`, la propiedad se omite al serializar `ProviderCallAudit`; esos campos son opcionales pero no nullable en `provider-call-audit.schema.json`. Solo se serializa `null` donde el schema lo admite expresamente, como `idempotency_key_hash`, `fallback_route_id`, `output_hash` o `error_code`, sin eludir sus reglas condicionales.
- Todo audit enlazado por `model_call_id|tool_call_id` comparte exactamente el `input_hash` del owner; el audit terminal comparte tambien `output_hash`, incluido `null`. El adapter calcula cada hash una sola vez sobre el DTO JSON minimizado con JCS (RFC 8785), UTF-8 sin BOM y rechazo de claves duplicadas, y reutiliza ese valor en ambas filas; Fase 1 no define hashing para payload binario.
- `ProviderCallAudit.completed_at` debe ser mayor o igual que `started_at`; el service rechaza orden temporal inverso aunque el JSON Schema solo pueda validar formato.
- `RawAccessAuditService` debe validar fixtures contra `raw-access-event.schema.json` y `privacy-security-policy.md`: cuando `resource_type=document_evidence`, `document_id`, `document_version_id` y `passage_ref` son obligatorios; cuando `resource_type!=document_evidence`, esos campos deben ser null/ausentes; `approved_by_ref != actor_ref`; `expires_at`, si existe, debe ser posterior a `accessed_at`. Cualquier acceso raw/elevado sin `RawAccessEvent` policy-valid falla.
- `PromptInjectionGuardStub` debe bloquear o excluir fixtures de riesgo `blocking` segun `prompt-injection-policy.md`; no interpreta contenido de documentos, mensajes ni fuentes como instrucciones. Propaga riesgos `D#:P#|F#:P#` al pasaje exacto, `url_hash:sha256:*` a la fuente resoluble y todos sus pasajes, y `rr_*` a todas las fuentes/pasajes del run salvo delimitacion mas especifica persistida; una cita o claim publicado no puede depender de evidencia contaminada con severidad `blocking` o handling `excluded_from_evidence|blocked`. Un riesgo `msg_*` se trata en scope de request: bloquea cuando corresponda y nunca se usa para marcar como contaminada evidencia independiente no derivada de ese mensaje.
- Estos servicios son stubs contractuales de Fase 1; la lógica completa de proveedores, fetch real y evaluación adversarial profunda queda para fases posteriores.

---

## 7. Migraciones iniciales

### 7.0 Reglas transversales de limites en persistencia

Las migraciones `0001` a `0004`, los modelos SQLAlchemy y los repositorios deben conservar los limites cerrados de `docs/contracts/*.schema.json`; validar solo al entrar por HTTP no es suficiente.

- Toda columna escalar que materialice una propiedad contractual con `minLength`/`maxLength` usa `varchar(n)` y/o `CHECK (char_length(...) BETWEEN min AND max)` igual o mas estricto; `NULL` solo se permite si el contrato lo permite y nunca equivale a `""`. Los identificadores y refs DB-only que crucen tablas usan `1..160` caracteres salvo un limite menor ya definido.
- Todo array contractual persistido en JSONB valida `jsonb_typeof(value) = 'array'`, `jsonb_array_length(value) <= maxItems` y el payload completo contra su JSON Schema antes del `INSERT`/`UPDATE`.
- Todo objeto contractual persistido en JSONB se valida completo contra el schema aceptado; las columnas proyectadas de identidad, tenant, version, refs y estado deben coincidir exactamente con el payload.
- Ninguna capa trunca silenciosamente. Un valor sobre limite se rechaza antes de persistir con el error contractual correspondiente.
- Los tests de migracion/repositorio prueban string vacio, limite aceptado y `limite + 1` para strings/arrays representativos de cada tabla, ademas de una divergencia columna-payload.

Estas reglas son invariants de almacenamiento, no una autorizacion para crear DTOs, enums o contratos paralelos.

### 7.1 `0001_core_foundation.py`

Debe crear:

- `organizations`;
- `users`;
- `memberships`;
- `conversations`;
- `messages`;
- extensiones necesarias como `pgcrypto` para UUID si aplica.

`0001_core_foundation.py` no crea `cases`. En Fase 1, `conversations.case_id` permanece `null`; un valor no nulo no puede llegar a persistencia.

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
messages.organization_id,actor_ref,conversation_id,idempotency_key_hash unique where idempotency_key_hash is not null
messages check (message_kind = user_input and idempotency_key_hash is not null and request_fingerprint is not null) or (message_kind <> user_input and idempotency_key_hash is null and request_fingerprint is null)
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

Las columnas escalares de `trace_objects` y los campos equivalentes de `trace_object_payload` deben coincidir exactamente, incluido `trace_id`, tenant, answer/version, conversation, output message, outcome y razon; el test de migracion/servicio rechaza una proyeccion divergente.

Índices mínimos:

```txt
operational_runs.organization_id,created_at
operational_runs.input_message_id unique
operational_runs check de maquina de estados y timestamps segun 6.9
run_events.run_id,created_at
run_events.run_id,sequence unique
run_events (run_id,event_name) unique where event_name in (run_created,budget_checked,run_execution_started)
run_events.run_id unique where event_name in (run_completed,run_failed,run_cancelled)
run_events.run_id,resource_ref unique where event_name = usage_event_recorded
run_events check sequence >= 1
run_events check event_scope = internal
run_events check event_name in (run_created,budget_checked,run_execution_started,model_provider_stubbed,usage_event_recorded,stream_event_emitted,run_completed,run_failed,run_cancelled,error_envelope_returned)
model_calls.run_id,started_at
model_calls check completed_at >= started_at
tool_calls.run_id,started_at
tool_calls check completed_at >= started_at
answers.organization_id,created_at
answer_versions.answer_id,answer_version unique
answer_versions.answer_id,answer_version_id,organization_id unique -- target de FK compuesta desde answers.latest_answer_version_ref
answer_versions.trace_id unique
answer_versions.abstention_render_ref unique where abstention_render_ref is not null
abstention_renders.trace_id unique
abstention_renders.answer_id,answer_version unique
abstention_renders.render_storage_ref unique
abstention_renders check render_storage_ref = abstention_render_id
trace_objects.organization_id,created_at
trace_objects.organization_id,data_classification
trace_objects.trace_id unique/fk operational_runs.run_id
trace_objects.answer_version_ref unique
trace_objects.output_message_id unique
messages.answer_id,trace_id,answer_version_ref
usage_events.organization_id,created_at
usage_events.usage_event_id,organization_id unique
usage_events.subscription_id,billing_period
usage_events.research_credit_id
usage_events.cost_report_id
usage_events.trace_id unique where trace_id is not null and event_type in (standard_query,complex_query)
usage_events.trace_id unique where trace_id is not null and event_type = research_credit_used
usage_events.model_call_id,event_type unique where model_call_id is not null and event_type in (model_input_tokens,model_output_tokens)
research_credits.subscription_id,billing_period unique
research_credits.research_credit_id,subscription_id,organization_id unique
research_credit_movements.research_credit_id,created_at
research_credit_movements.organization_id,created_at
research_credit_movements.usage_event_id unique where usage_event_id is not null
research_credit_movements.research_credit_id unique where movement_type = grant
research_credit_movements.research_credit_id unique where movement_type = expiry
research_credit_movements check movement_type <> debit or (usage_event_id is not null and trace_id is not null and answer_id is not null and delta < 0 and balance_after >= 0)
research_credit_movements check movement_type <> grant or (usage_event_id is null and trace_id is null and answer_id is null and delta > 0 and balance_after >= 0)
research_credit_movements check movement_type <> expiry or (usage_event_id is null and trace_id is null and answer_id is null and delta <= 0 and balance_after = 0)
research_credit_movements check movement_type in (grant,debit,expiry) -- adjustment no ejecutable en Fase 1
research_credits check balance_quantity >= 0 and used_quantity >= 0 and granted_quantity >= 0
research_credits check (status = active and balance_quantity > 0) or (status in exhausted|expired and balance_quantity = 0)
budget_decisions.organization_id,created_at
budget_decisions.run_id unique not null
budget_decisions check decision in (allow,degrade,block)
budget_decisions check decision_reason_code in (within_budget,nonessential_output_capped,budget_exhausted,research_credit_required,policy_blocked)
budget_decisions check (decision = allow and decision_reason_code = within_budget) or (decision = degrade and decision_reason_code = nonessential_output_capped) or (decision = block and decision_reason_code in (budget_exhausted,research_credit_required,policy_blocked))
plans.plan_code unique
subscriptions.organization_id,status
subscriptions.organization_id unique where ended_at is null and status in active|trialing|past_due
subscriptions.subscription_id,organization_id unique
subscriptions check (status in active|trialing|past_due and ended_at is null) or (status = cancelled and ended_at is not null and ended_at >= started_at)
provider_call_audits.organization_id,started_at
provider_call_audits.logical_call_id,attempt_number unique
provider_call_audits.request_id,correlation_id
provider_call_audits check completed_at >= started_at
provider_call_audits.trace_id
provider_call_audits.model_call_id
provider_call_audits.tool_call_id
provider_call_audits.retrieval_run_id
provider_call_audits.usage_event_id
provider_call_audits.cost_report_id
raw_access_events.organization_id,accessed_at
```

Los vinculos tenant-scoped de `research_credits` y `research_credit_movements` usan FKs compuestas o constraints/transaccion de servicio equivalentes: no se admite que un credito, movimiento, suscripcion, `UsageEvent`, `TraceObject` o `Answer` resuelto pertenezca a otra organizacion.

Las igualdades cross-row/cross-table de cantidad, suma de grants/debitos, continuidad de movimientos, saldo, periodo y transición de estado no se delegan al `CHECK` local: `UsageLedgerService` debe verificarlas bajo lock en la misma transaccion y sus tests deben detectar delta, `granted_quantity`, `used_quantity`, `balance_after`, periodo o `status` divergentes.

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
  query_id text not null -- q_*; sin FK DB en Fase 1, resuelto por fixture/manifest contractual
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
- `evidence_packs.organization_id` es obligatorio. En Fase 1, `query_id` debe resolver contra un fixture/manifest versionado de `LegalSearchQuery` schema-valid con el mismo `organization_id`; si `retrieval_run_id` existe, resuelve un fixture `RetrievalRun` schema-valid y su `RetrievalPlan` schema-valid. Query, plan, run y pack comparten tenant, `RetrievalRun.query_id == RetrievalPlan.query_id == EvidencePack.query_id`, `RetrievalRun.evidence_pack_id == EvidencePack.evidence_pack_id` y `EvidencePack.retrieval_run_id == RetrievalRun.retrieval_run_id`.
- `0003_evidence_contract_stubs.py` no crea `legal_search_queries` ni una FK a una tabla sin owner. La persistencia/FK productiva de `LegalSearchQuery` y `RetrievalRun` pertenece a Fase 4; hasta entonces, estas filas son stubs de test sin endpoint productivo y una ref ausente del manifest invalida el fixture.
- `retrieval_run_id` es nullable en la tabla stub porque `EvidencePack -> RetrievalRun` es `0..1` en 0.11; packs manuales o documentales no deben inventar `rr_*` falso.
- `evidence-pack.schema.json` requiere la propiedad `retrieval_run_id`, pero admite valor `null` para packs manuales o documentales; esas filas son `EvidencePack` contractuales siempre que el resto del payload sea schema-valid. No se inventa un `rr_*` para satisfacer el contrato.
- `quality` valida siempre contra `evidence-quality.schema.json`; no se crea una forma inline paralela en ORM, DTO, fixture o migracion.
- `legal_area[]` se trata como conjunto sin duplicados; `sources[].source_ref` y `passages[].passage_ref` son unicos dentro del mismo `EvidencePack` incluso cuando dos payloads distintos validen por separado.
- Todo `EvidencePassage.source_ref` debe resolver exactamente una fuente del mismo `EvidencePack`; un pasaje huerfano o una colision de identidad local invalida el pack completo.
- Toda fuente con `validity_status=VIGENCIA_CONFIRMADA|DEROGADA_CONFIRMADA` debe resolver al menos un `EvidencePassage` del mismo pack y mismo `source_ref`; la validez local del payload `Source` no sustituye este gate relacional.
- `evidence_sources.organization_id` y `evidence_passages.organization_id` son columnas internas de ownership copiadas desde `evidence_packs`; se excluyen de los payloads `Source`/`Passage` antes de validar contra JSON Schema.
- `claims.organization_id` y `citations.organization_id` son obligatorios y nunca se derivan solo del request actual.
- `claims.answer_version_ref` y `citations.answer_version_ref`/`evidence_pack_id` son columnas relacionales DB-only; no se agregan a los payloads cerrados `Claim`/`Citation`.
- `evidence_sources` usa `primary key (evidence_pack_id, source_ref)`; `evidence_passages` usa `primary key (evidence_pack_id, passage_ref)`.
- `citations` usa `primary key (answer_version_ref, citation_ref)` y debe resolver `source_ref` y `passage_ref` dentro del mismo `evidence_pack_id`.
- `citations.passage_ref` debe resolver a un `evidence_passages` del mismo `evidence_pack_id` cuyo `source_ref` sea exactamente igual a `citations.source_ref`; tambien debe cumplirse que el prefijo de `passage_ref` antes de `:P#` coincida con ese `source_ref`.
- Todo `Claim` referenciado por `Citation.supports_claim_ids[]` debe existir con el mismo `organization_id` y el mismo `answer_version_ref` que la cita; no se permite soporte cross-tenant, cross-version ni contra claims con `answer_version_ref=null`.
- Todo `Claim.claim_payload.citations[]` no vacio debe apuntar a `citations.citation_ref` existentes en la misma `answer_version_ref`; toda `Citation` contractual tiene `supports_claim_ids[]` no vacio y aparece en al menos un `Claim.claim_payload.citations[]` de esa misma version, cualquiera sea su `status`.
- `Claim <-> Citation` es una relacion `n:m`; no se modela con un `claim_id` escalar en `citations`. `Claim.citations[]` y `Citation.supports_claim_ids[]` son conjuntos sin duplicados y representan exactamente los mismos pares dentro de cada `answer_version_ref`.
- Todo claim serializado dentro de `AnswerContract.claims[]` debe tener `verification_status=passed` y `support_level=direct|inferential`; claims con soporte `weak|unsupported` o estado `failed|needs_review|blocked` pueden persistirse para traza, pero no forman parte del contrato final visible.
- Dentro de una misma `answer_version_ref`, `claim_id` y `citation_ref` son unicos; `AnswerContract.sources_used[]` tampoco admite duplicados. El validador relacional rechaza colisiones de identidad aunque los payloads distintos validen individualmente contra JSON Schema.
- Cada `CitationAudit.results[]` usado como fixture de Fase 1 resuelve exactamente una `Citation` de la misma `answer_version_ref`; `(claim_id, citation_ref)` es unico, `passage_ref`/`source_ref` coinciden con esa cita y el par existe en `Claim.citations[]`/`Citation.supports_claim_ids[]`. Esta validacion fundacional no adelanta el Citation Auditor productivo, que permanece en Fase 2.
- Las columnas canonicas y los payloads JSONB deben coincidir: `source_payload.source_ref == evidence_sources.source_ref`, `passage_payload.passage_ref/source_ref == evidence_passages.passage_ref/source_ref`, `claim_payload.claim_id/organization_id/citations == claims.claim_id/organization_id` y refs de citas resueltas, y `citation_payload.citation_ref/organization_id/passage_ref/source_ref/supports_claim_ids == citations.*` y claims resueltos.
- Los fixtures P1 deben validar `source_payload`, `passage_payload`, `claim_payload` y `citation_payload` contra `source.schema.json`, `passage.schema.json`, `claim.schema.json` y `citation.schema.json`.
- `source_payload.metadata` usa exclusivamente la forma cerrada de `source.schema.json`; no admite metadata libre ni claves raw.
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
- Eventos operativos internos como `budget_checked` se guardan en `run_events` con `event_scope=internal`; `message_received`/`message_persisted` son logs estructurados anteriores al run y ninguno es taxonomia publica SSE.
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
  "mode": "STANDARD",
  "complexity": "SIMPLE",
  "requested_capabilities": ["conversation"],
  "estimated_input_tokens": 0,
  "estimated_output_tokens": 0
}
```

`BudgetRequest` es un DTO interno y no acepta `plan_code`, `actor_ref` ni `actor_type` controlados por el cliente. El caller resuelve `actor_ref`/`actor_type` desde el contexto autenticado y `plan_code` desde la suscripción current de la misma organización mediante `CommercialService`; el valor queda como snapshot de la decisión y no puede sobreescribir la fuente comercial.

### 8.4 `BudgetDecision`

```json
{
  "budget_decision_id": "bd_phase1_demo",
  "run_id": "tr_phase1_demo",
  "organization_id": "org_demo",
  "decision": "allow",
  "plan_code": "PROFESIONAL",
  "mode": "STANDARD",
  "complexity": "SIMPLE",
  "cost_budget_ref": "cb_profesional_simple_v080",
  "cost_budget_version": "cost-budget-v0.8.0",
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
  "decision_reason_code": "within_budget",
  "created_at": "2026-05-26T00:00:00Z"
}
```

`BudgetDecision` es un DTO interno para `budget_decisions`; no sustituye `CostBudget` ni crea un JSON Schema canonico nuevo. `limits` se persiste como `budget_snapshot` exacto, y sus `cost_budget_ref`/`cost_budget_version` identifican el budget efectivo que produjo la decision. `decision_reason_code` usa exclusivamente el enum cerrado de 6.12; no se persiste texto libre.

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
  "organization_id": "org_demo",
  "event_scope": "internal",
  "event_name": "budget_checked",
  "sequence": 2,
  "resource_type": "budget_decision",
  "resource_ref": "bd_phase1_demo",
  "safe_message_code": null,
  "error_code": null,
  "payload_hash": "sha256:cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc",
  "created_at": "2026-05-26T00:00:00Z"
}
```

El UUID surrogate `run_events.id` se asigna en persistencia y no forma parte del DTO. `organization_id` es obligatorio y debe coincidir con `operational_runs.organization_id`.

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

En el perfil de Fase 1, `case_id` debe ser `null` u omitirse. La forma `case_*` permanece reservada para el contrato futuro y no se habilita hasta existir un agregado `Case` tenant-scoped resoluble.

### 8.9 `MessageCreate`

```json
{
  "content": "Necesito revisar una consulta jurídica.",
  "attachments": []
}
```

`Idempotency-Key` es header obligatorio de la operación, no campo de `MessageCreate`; el DTO no admite una key raw ni metadata libre.

En el perfil de Fase 1, `attachments` debe ser `[]` u omitirse porque todavía no existen `Document`/`DocumentVersion` ni procesamiento documental tenant-scoped. Un array no vacío devuelve `ErrorEnvelope.error_code=document_processing_required` antes de persistir mensaje o run; no se aceptan `doc_*`/`docv_*` libres controlados por cliente.

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

P0 implementa `ConversationService`, `MessageService` y el pipeline interno que crea mensaje, run, budget decision y trazabilidad operativa; crea usage solo cuando el resultado es publicable y existe un `UsageEvent` schema-valid con ancla contractual. P1-04 expone `POST /v1/conversations/{conversation_id}/messages` como endpoint público y lo conecta a SSE; antes de P1-04, la validación P0 puede ejercitar el pipeline por service/integration tests.

`POST /v1/conversations/{conversation_id}/messages` debe:

1. prevalidar auth, tenant, conversation, `Idempotency-Key` ASCII de `1..128`, body JSON de hasta `131072` bytes decodificados, `content` de `1..20000` code points y `attachments` de hasta `20`; un exceso devuelve `payload_too_large` y cualquier error en esta etapa ocurre antes de persistir mensaje, reservar IDs o crear run;
2. reservar en memoria los IDs `msg_*` de entrada/salida, `tr_*`, `ans_*`, `av_*` y `abstention_render_*`;
3. resolver la suscripcion current y aplicar `CostGovernor` con el `run_id` reservado, sin confirmar todavia filas parciales;
4. calcular `idempotency_key_hash` y `request_fingerprint`; en una unica transaccion, reclamar su combinación tenant/actor/conversation, persistir el mensaje de usuario y luego `operational_run`, su unico `BudgetDecision`, `run_created` y `budget_checked`; el FK `input_message_id` resuelve al mensaje ya insertado dentro de esa misma transaccion y cualquier fallo revierte el conjunto. Un conflicto concurrente con el mismo fingerprint relee y devuelve el resultado original; con fingerprint distinto devuelve `conflict` sin crear filas nuevas;
5. si `BudgetDecision.decision=allow|degrade`, confirmar `queued -> running` junto con `run_execution_started` antes de ejecutar etapas posteriores; si `decision=block`, cerrar `queued -> failed`, persistir `run_failed` y, solo cuando la frontera lo devuelve, `error_envelope_returned` con el codigo que corresponde a `decision_reason_code`, sin crear `run_execution_started`, `ModelCall`, usage, `Answer`, `AnswerVersion`, `AbstentionRender` ni `TraceObject`;
6. solo para `allow|degrade`, usar `ModelProvider` stub si corresponde y persistir su `ModelCall`/evento real;
7. construir y validar en memoria el mensaje tecnico `assistant_final`, los shells `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject` y los `UsageEvent` requeridos, sin exponer todavía un resultado terminal;
8. en una unica transaccion terminal, volver a bloquear y validar la fila de creditos cuando aplique; persistir el evento de consulta, el `research_credit_used` y su movimiento si corresponde; persistir mensaje/answer/version/render/trace; confirmar `running -> completed`; y agregar un `usage_event_recorded` por cada `UsageEvent` junto con `run_completed`;
9. si la transaccion terminal falla, revertir íntegramente respuesta, traza, usage, movimiento y cierre exitoso. Una transaccion de fallo separada lleva `running -> failed`, agrega `run_failed` y, solo si la frontera devuelve el error, `error_envelope_returned`; una carrera por el último crédito devuelve `research_credit_required` sin alterar retroactivamente el `BudgetDecision` de admisión ni fabricar artefactos finales;
10. emitir `answer_blocked` o `run_failed` por SSE solo después del commit terminal correspondiente y devolver metadata del run.

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

`answer_blocked` es el evento terminal de exito tecnico en Fase 1 cuando el pipeline corre pero no emite analisis juridico. Debe estar respaldado por `TraceObject.response_outcome=blocked`, `TraceObject.abstention_reason=policy_blocked`, `TraceObject.citation_audit` contract-compatible para `policy_blocked`, `AnswerVersion.response_outcome=blocked`, `AnswerVersion.abstention_render_ref` no nulo y `AbstentionRender.reason_code=policy_blocked`. `run_failed` solo se usa si la ejecucion falla antes de crear `AnswerVersion`/`TraceObject`. Eventos como `budget_checked`, `model_provider_stubbed` o `run_completed` son internos y se registran en `run_events` con `event_scope=internal`; `message_received`/`message_persisted` permanecen en logging estructurado de request/repository y ninguno de ellos es taxonomia publica SSE.

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
  "usage_totals": {
    "standard_query": 0,
    "complex_query": 0,
    "research_credit_used": 0
  },
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
  "research_credits_balance": 8
}
```

Regla:

- `usage_totals` son agregados visibles, no filas raw de `usage_events`.
- `usage_totals` es un objeto cerrado que solo permite keys de `UsageEvent.event_type`, con cantidades no negativas y key omitida equivalente a cero.
- `limits` es el objeto cerrado de nueve campos del `CostBudget` efectivo y se deriva de `docs/schemas/budgets.yaml` y la suscripción activa.
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
  "updated_at": "2026-05-01T00:00:00Z",
  "next_cursor": null,
  "has_more": false
}
```

Regla:

- `movements[]` son movimientos resumidos, no filas raw de `UsageEvent`.
- `balance` se calcula desde `research_credits.balance_quantity`.
- `movements[]` se calcula desde `research_credit_movements`, no desde `UsageEvent` raw.
- `movements[]` pagina con `limit` default 50/maximo 100 y cursor opaco; cada item es el objeto cerrado `date`, `type=grant|debit|expiry`, `delta`, `balance_after` definido por `api-draft-v0.md`.
- Cada movimiento `debit` de una respuesta publicada debe enlazar `usage_event_id` de un `UsageEvent.event_type=research_credit_used` y compartir sus mismos `trace_id` y `answer_id`.
- El resumen respeta el estado derivado: saldo positivo implica `active`; saldo cero vigente implica `exhausted`; un periodo cerrado aparece `expired` solo despues de su movimiento terminal `expiry`, que drena el remanente o usa `delta=0` si el saldo ya estaba agotado.
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
- `CostGovernor` bloquea `COMPLEJO`/`INVESTIGACION` sin saldo con `research_credit_required`, sin publicar ni crear debito ficticio; `degrade` solo recorta salida no esencial dentro de la misma complejidad;
- `CommercialService` resuelve suscripción activa desde `subscriptions`, valida seed `PROFESIONAL` y rechaza doble suscripción current;
- `ErrorEnvelope` serializa con `request_id`;
- request middleware genera `request_id` si falta;
- request middleware respeta `X-Correlation-Id` si llega;
- `StubModelProvider` devuelve salida determinista;
- `PromptVersionRegistry` carga el manifest, verifica unicidad/path/hash y resuelve `phase-1-pipeline-check-v1`;
- `PromptVersionRegistry` rechaza versión desconocida, duplicada, path inexistente o contenido con hash divergente;
- `PromptVersionRegistry` rechaza una versión `deprecated` para llamadas/`ModelCall` nuevos, pero puede resolverla sin invocar provider para auditoría histórica;
- `UsageLedger` crea evento válido;
- `UsageLedger` crea exactamente un evento de consulta por ejecución publicada: `standard_query` para `SIMPLE|MEDIO` o `complex_query` para `COMPLEJO|INVESTIGACION`, nunca ambos para la misma traza;
- el evento de consulta y cada `research_credit_used` comparten `trace_id`, `answer_id`, tenant y los cuatro campos de budget con `BudgetDecision`/`TraceObject`; cualquier divergencia falla cerrado;
- `UsageLedger` registra `research_credit_used` con quantity `1` para `COMPLEJO` y `2` para `INVESTIGACION`;
- `research_credit_used` crea movimiento `debit` en `research_credit_movements` enlazado al `usage_event_id`, con cantidad, `used_quantity` y `balance_after` conciliados en la misma transaccion;
- `GET /v1/research-credits` devuelve balance y movimientos resumidos; una suscripcion activa agotada devuelve `balance=0` y la ausencia de suscripcion activa devuelve `not_found`;
- saldo insuficiente devuelve `research_credit_required` en el endpoint de mensajes, sin publicar respuesta ni registrar un debito ficticio;
- `StorageProviderStub` existe, no escribe documentos reales y falla cerrado con `storage_unavailable`;
- `TraceService` crea evento válido.
- `test_canonical_json.py` valida vectores de RFC 8785, equivalencia ante distinto orden de propiedades, preservación de orden de arrays y Unicode sin normalización adicional;
- `test_canonical_json.py` rechaza claves duplicadas antes de validar, `NaN`/`Infinity`, números fuera del dominio JCS/I-JSON y demuestra que todos los hashes auditables reutilizan `core.canonical_json` en vez de `json.dumps(sort_keys=True)`.
- `test_config_documents.py` valida los 10 YAML canónicos con el loader compartido y rechaza UTF-8 inválido, claves duplicadas anidadas, alias/anchors, merge keys, tags explícitos, múltiples documentos, raíz no mapping, metadata ausente, campos desconocidos, enums duplicados y referencias cruzadas no resolubles;
- `test_config_documents.py` prueba además que `budgets.yaml` cubre exactamente planes/complejidades canónicos y que `provider-registry.yaml` separa `ProviderCallAudit.error_code` de la traducción pública a `ErrorEnvelope`.

### 11.2 Integration tests P0

Obligatorios:

- `GET /health/live` devuelve 200 sin DB;
- `GET /health/ready` devuelve 200 con DB sana;
- migraciones aplican en DB limpia;
- `POST /v1/conversations` crea conversación;
- `POST /v1/conversations` acepta `case_id=null` u omitido y rechaza `case_*` con `validation_error` mientras no exista `Case` tenant-scoped;
- `POST /v1/conversations/{conversation_id}/messages` acepta `attachments=[]` u omitido y rechaza cualquier adjunto con `document_processing_required` antes de persistir mensaje/run mientras no exista el pipeline documental tenant-scoped;
- `POST /v1/conversations/{conversation_id}/messages` exige `Idempotency-Key`: falta/formato inválido devuelve `validation_error`; retries secuenciales y concurrentes con igual fingerprint reutilizan el resultado, mientras la misma key con otro payload devuelve `conflict`;
- `POST /v1/conversations/{conversation_id}/messages` rechaza con `payload_too_large`, antes de persistencia o reserva de IDs, un body decodificado mayor a `131072` bytes, `content` mayor a `20000` code points o mas de `20` attachments; los boundary tests aceptan el limite exacto y rechazan `limite + 1`;
- ningún retry idempotente duplica `Message`, `OperationalRun`, `BudgetDecision`, `UsageEvent` o movimiento de crédito, y la key raw no aparece en DB, logs, trazas o errores;
- `test_tenant_isolation.py` cubre negativos cross-tenant de lectura, escritura, listado y borrado lógico solo donde exista superficie aceptada; no se crean rutas `DELETE` nuevas;
- antes de beta, `AuthProvider` productivo rechaza firma/token invalido o expirado, issuer/audience incorrectos, membership inactiva y tenant contradictorio; `DevAuthProvider` falla fuera de local/test;
- `MessageService`/message pipeline interno crea mensaje, run, budget decision y trace events; solo crea `UsageEvent` cuando la ejecución llega a un resultado publicable y existe un ancla contractual válida;
- `test_terminal_settlement.py` demuestra que respuesta/traza, usage obligatorio, débito y `run_completed` se confirman en una sola transacción; un fallo en cualquier escritura revierte todos los artefactos finales y solo deja el cierre `run_failed` de la transacción de fallo separada;
- dos ejecuciones concurrentes que compiten por el último crédito producen como máximo una publicación/débito; la otra devuelve `research_credit_required` sin `Answer`/`TraceObject` ni evento terminal de éxito;
- retries no duplican el evento de consulta, `research_credit_used` ni los eventos de tokens anclados al mismo `model_call_id`;
- `GET /v1/usage/current` devuelve `period_start`, `period_end`, `plan_code`, `subscription_id`, `usage_totals`, `limits` y `research_credits_balance`;
- error de validación devuelve ErrorEnvelope;
- respuesta incluye `X-Request-Id` y `X-Correlation-Id`.

### 11.3 Tests P1/P2 y beta foundations aplicables

- EvidencePack schema valida fixture mínimo;
- `test_evidence_contracts.py` cubre identidad local, ownership tenant, refs same-pack/same-version, reciprocidad `Claim <-> Citation`, coincidencia columna/payload y evidencia de vigencia confirmada;
- Citation schema valida valores contractuales `F1:P1` y `D1:P1`; los corchetes pertenecen solo al render visible;
- Claim schema clasifica `criticality` y rechaza degradar a `low|medium` los tipos jurídicos críticos cerrados por el contrato;
- fixture de `ClaimCompletenessValidator` detecta una afirmación jurídica crítica visible omitida de `claims[]` como `critical_assertion_unmapped`; el enforcement productivo queda en Fase 2;
- fixture del oracle semántico compara `Claim.text` contra `expected_claim_safe_text`/hash/modo aprobado y no acepta la autoevaluación del sistema como oracle;
- Source Registry seed y `source_registry_health` existen con health inicial `UNKNOWN` y `live_fetch_enabled=false`;
- OpenSearch stub no rompe readiness si está disabled;
- Fetcher stub no hace red real;
- `make eval-smoke` corre con fixture vacío y `make eval-regression` ejecuta una regression suite contractual inicial no vacía; su reporte versionado declara `beta_eligible=false` hasta ejecutar el golden dataset y gates completos de 0.12;
- Provider Reliability Layer (PRL) stub mapea timeout/rate limit/provider unavailable a `ErrorEnvelope`;
- PRL prueba un retry idempotente; no reintenta policy/validación; prueba fallback con fixture local cerrado; y verifica que el registry productivo sin rutas falla con `provider_unavailable`, sin duplicar usage/cargos;
- `external_call_mode` resuelve `ProviderMetadata.external_call` antes de habilitar providers;
- ProviderCallAudit fixture valida contra schema y contra `provider-policy.md` + `provider-registry.yaml`, incluyendo contexto completo por intento, unicidad `(logical_call_id, attempt_number)`, orden temporal, igualdad de hashes con el owner, enlace terminal de `ModelCall`/`ToolCall` y traducción cerrada de `error_mapping` interno al `ErrorEnvelope` público;
- `test_model_tool_call_persistence.py` valida payloads completos contra sus schemas, reglas condicionales de status/error/audit, ownership tenant y `completed_at >= started_at`;
- `test_trace_persistence.py` rechaza cualquier divergencia entre las columnas de `trace_objects` y los campos equivalentes de `trace_object_payload`, incluso si la fila y el JSON son validos por separado;
- RawAccessEvent fixture valida contra schema y contra `privacy-security-policy.md`;
- PromptInjectionGuardStub bloquea riesgo `blocking`, propaga contaminacion de pasaje/URL/run hasta fuentes, pasajes, citas y claims, y usa `ErrorEnvelope.error_code=prompt_injection_blocked` cuando opera en frontera API.
- `test_schema_validator.py` recorre los `pattern` positivos que expresan valores completos, parte de valores validos y rechaza las variantes con `\r`, `\n`, `\r\n`, `U+2028` y `U+2029` terminales; los detectores anidados bajo `not` se prueban por separado como denylists, y no se aplica trim, normalizacion ni coercion antes de JSON Schema.
- `test_schema_validator.py` inserta NUL, TAB, ESC, NEL y otros controles C0/C1 dentro y al final de cada warning, nota, mensaje de error o snippet sanitizado, y exige rechazo antes de persistencia/log/render.
- `test_schema_validator.py` falla si cualquier schema aceptado deja un string variable sin `minLength: 1`/`maxLength` o un array sin `maxItems`; rechaza string vacio, `limite + 1` y formato invalido, y acepta el mayor valor generable que tambien satisfaga las demas keywords, sin mantener fixtures manuales gigantes.
- `make scan-secrets` rechaza un fixture con clave sintética y acepta únicamente placeholders/hash allowlisted de forma puntual;
- `make scan-sensitive-surfaces` rechaza la denylist cerrada de `privacy-security-policy.md` en schemas aceptados fuera de `x-invalid-*`; sus fixtures inyectan sentinels sintéticos únicos en prompt, mensaje, documento, OCR, payload de proveedor e `Idempotency-Key` raw y prueban que ninguno sobrevive en logs/traces serializados, incluso bajo otra clave;
- Si P1-04 entra, `POST /v1/conversations/{conversation_id}/messages` crea mensaje, run, budget decision y trace events; crea `UsageEvent` solo para una ejecución publicable con ancla contractual válida, nunca por el mero intento ni por un fallo temprano;
- si P1-04 entra, `GET /v1/conversations/{conversation_id}/runs/{run_id}/stream` emite `run_queued`, `run_started` y terminal `answer_blocked` respaldado por `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject`;
- si P1-04 entra, `GET /v1/answers/{answer_id}/trace-summary` resuelve la URL devuelta por `answer_blocked`.

### 11.4 Regla de CI

No se acepta PR si:

- `make test` completo falla;
- tests P0/P1/P2 aplicables al alcance del PR fallan;
- migraciones no aplican;
- lint/type-check falla;
- `make scan-secrets` falla o hay secretos en repo/historial del PR;
- `make scan-sensitive-surfaces` falla;
- `make validate-schemas` detecta un contrato sin cotas `minLength: 1`/`maxLength`/`maxItems` o falla cualquier boundary/format test;
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
   En el perfil de Fase 1, toda conversación usa `case_id=null`; no existen refs `case_*` huérfanas.
9. Cada `Message` iniciador confirmado (`message_kind=user_input`, o trigger system/service explicitamente tipado en una fase futura) crea exactamente un `operational_run`; mensajes `assistant_final|assistant_clarification|system_notice|tool_progress` nunca inician otro run.
   Retries con la misma `Idempotency-Key` y fingerprint reutilizan el mensaje/run original; una key reutilizada con otro request devuelve `conflict`.
10. Tests negativos de aislamiento tenant cubren lectura, escritura, listado y borrado lógico cross-tenant solo para superficies implementadas.
11. `CostGovernor` evalúa presupuesto desde `budgets.yaml`.
12. Cada ejecución registra `budget_decision`.
13. `UsageLedger` registra exactamente un evento de consulta por ejecución publicada, con tipo derivado de la complejidad e identidad/budget iguales a `BudgetDecision` y `TraceObject`.
14. `plans`, `subscriptions`, `research_credits` y `CommercialService` resuelven plan/suscripción activa y créditos sin fuente paralela.
15. `research_credit_used` se registra obligatoriamente para `COMPLEJO` e `INVESTIGACION` y concilia exactamente cantidad, delta, `used_quantity`, saldo, `trace_id`, `answer_id` y budget efectivo; publicación, usage, débito y cierre del run son atómicos. Sin saldo, incluida una carrera por el último crédito, se devuelve `research_credit_required` sin publicar ni crear debito ficticio, y no se degrada la complejidad en Fase 1.
16. `GET /v1/research-credits` devuelve balance y movimientos resumidos.
17. `StorageProvider` interface/stub existe y falla cerrado sin escritura real.
18. `WorkflowGateway` interface y `LocalWorkflowGateway` existen; el core no importa Temporal directamente.
19. Trazabilidad mínima persiste `run_events`, `model_calls` y/o `tool_calls` cuando aplique.
20. `ModelProvider` existe como wrapper, el stub es el default y todo `ModelCall.prompt_version` resuelve contra `PromptVersionRegistry` antes de invocación o persistencia.
   Solo prompts `active` pueden seleccionarse para llamadas nuevas; prompts `deprecated` quedan disponibles exclusivamente para reconstrucción histórica sin invocar provider.
21. El endpoint SSE emite estados técnicos.
22. `answer_blocked` queda verificado cross-contract: `TraceObject` valida completo contra schema, `TraceObject.citation_audit` cumple `policy_blocked`, `AbstentionRender` comparte `trace_id`/`answer_id`/`answer_version`/`response_outcome`, `reason_code == TraceObject.abstention_reason`, `AnswerVersion.answer_hash == AbstentionRender.render_hash`, `render_storage_ref` reconstruye `render_body_canonical` local DB, `source_trace_refs` es subconjunto real del `TraceObject` y `trace_summary_url` resuelve a la vista segura del mismo answer/version/trace. El SSE terminal solo se emite tras confirmar atómicamente esos artefactos, usage y cierre del run.
23. Provider Reliability Layer (PRL), ProviderCallAudit, RawAccessEvent y PromptInjectionGuard existen y fallan cerrado; PRL no activa fallback productivo con `fallback_routes: []`, RawAccessEvent valida schema + `privacy-security-policy.md` y PromptInjectionGuard aplica propagacion determinista por pasaje, URL y retrieval run conforme a `prompt-injection-policy.md`.
24. `ProviderCallAuditService` valida schema + policy/registry y contexto completo por intento; no acepta provider desconocido, familia divergente, audit sin ref contractual, intento duplicado, orden temporal inverso, hashes divergentes respecto del owner ni clases enviadas/devueltas fuera de allowlist. `policy_blocked` permite `attempted_data_classes[]` conocidas fuera de allowlist solo si no se envió payload; el ref singular model/tool apunta al intento terminal y comparte su `output_hash`.
25. `make scan-secrets` y `make scan-sensitive-surfaces` pasan y bloquean respectivamente secretos y payloads raw sensibles en schemas/logs/traces.
26. No hay llamadas externas reales obligatorias.
27. Evidence/Citation/Claim schemas y modelos/stubs contractuales existen, y `0003_evidence_contract_stubs.py` aplica las tablas stub mínimas.
28. Source Registry seed mínimo y `source_registry_health` 1:1 existen.
29. Fetcher/Snapshot/OpenSearch stubs existen; evaluation harness/runner ejecuta una regression suite contractual inicial no vacía y versionada, sin atribuirse cumplimiento beta.
30. `make test` completo pasa en CI/local, incluyendo tests P0/P1/P2 aplicables a los artefactos implementados.

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
- P1-09 implementa autenticacion productiva provider-neutral antes de beta, con decision versionada del adapter;
- token firmado, issuer, audience, expiracion y membership activa resuelven el tenant; headers/body del cliente no lo sustituyen; la baseline verifica con material configurado/cached, no persiste token raw y no hace introspeccion de red por request sin decision/auditoria especifica.

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
- `rfc8785==0.1.4`;
- `PyYAML==6.0.3`;
- SQLAlchemy 2.x;
- Alembic;
- pytest;
- pytest-asyncio;
- httpx;
- ruff;
- mypy;
- Gitleaks o scanner equivalente aprobado mediante ADR aceptado o una versión `Superseded` de este handoff;
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
15. Implementar ModelProvider stub y `PromptVersionRegistry` con su manifest/seed técnico versionado.
16. Implementar CostGovernor.
17. Implementar Plan/Subscription/ResearchCredit y `CommercialService`.
18. Implementar UsageLedger.
19. Completar P0-13 funcional: `OperationalRun`/`RunEvent` services, repos de model/tool calls, shells `answer_blocked` y endpoint interno de trace sobre las tablas de `0002`.
20. Implementar endpoint message pipeline.
21. Implementar SSE de estados.
22. Implementar P1-03: `scripts/validate_schemas.py` y `make validate-schemas` como el único motor compartido de validación contractual.
23. Implementar schemas, modelos/stubs contractuales y migración `0003_evidence_contract_stubs` al ejecutar P1-01/P1-02 para Evidence/Citation/Claim.
24. Implementar Provider Reliability Layer (PRL), ProviderCallAudit, RawAccessEvent, PromptInjectionGuard y los gates estáticos de P1-07.
25. Implementar `StorageProvider` interface/stub fail-closed al ejecutar P1-08, antes de beta y antes de cualquier upload/document storage.
26. Implementar el `AuthProvider` productivo provider-neutral de P1-09 y bloquear `DevAuthProvider` fuera de local/test antes de beta.
27. Implementar Source Registry seed y los stubs de P2-01 a P2-04; `0004_source_registry_seed` pertenece a P2-01.
28. Implementar evaluation harness/runner y ejecutar la regression suite contractual inicial no vacía con reporte `beta_eligible=false`.
29. Implementar `WorkflowGateway`/`LocalWorkflowGateway` al ejecutar P2-06, sin importar Temporal desde el core.
30. Implementar tests P0/P1/P2 aplicables.
31. Documentar cierre y handoff.

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
[ ] El equipo entiende que `ModelCall.prompt_version` debe resolver contra `PromptVersionRegistry`; un string libre no satisface ADR-005.
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
[ ] El endpoint de mensajes exige `Idempotency-Key`, conserva solo hash/fingerprint DB-only y no duplica efectos ante retries secuenciales o concurrentes.
[ ] RunEvent operativo implementado como DTO interno, sin sustituir `TraceObject`.
[ ] `answer_blocked` respaldado por `Answer`/`AnswerVersion`/`AbstentionRender`/`TraceObject` schema-valid y por las invariantes cross-contract de `answer-versioning-policy.md` para bloqueos; su `trace_summary_url` resuelve a la vista segura del mismo answer/version/trace.
[ ] ModelCall/ToolCall persistence mínima implementada.
[ ] ModelProvider stub implementado.
[ ] PromptVersionRegistry y manifest/seed técnico implementados; toda versión usada resuelve con path/hash válidos e identidad inmutable.
[ ] Solo prompts `active` crean llamadas nuevas; prompts `deprecated` se resuelven únicamente para auditoría histórica y no invocan provider.
[ ] StorageProvider interface/stub implementado y fail-closed.
[ ] `docs/schemas/budgets.yaml` integrado y validado.
[ ] CostGovernor funcional.
[ ] Plan/Subscription/ResearchCredit y CommercialService mínimos resuelven suscripción activa y saldo de créditos.
[ ] UsageLedger funcional.
[ ] Cada ejecución publicada tiene exactamente un `standard_query` o `complex_query` según su complejidad, con `trace_id`, `answer_id`, tenant y budget iguales a `BudgetDecision`/`TraceObject`; constraints y tests impiden duplicados por retry.
[ ] `research_credit_used` se registra para `COMPLEJO`/`INVESTIGACION` y concilia cantidad, delta, `used_quantity` y saldo en la misma transaccion terminal que respuesta, traza y cierre exitoso; sin saldo, incluida concurrencia por el último crédito, se devuelve `research_credit_required` sin publicar ni crear debito ficticio, y Fase 1 no reduce la complejidad para evitar el credito.
[ ] `GET /v1/research-credits` devuelve balance y movimientos resumidos.
[ ] Tests de aislamiento tenant base cubren lectura, escritura, listado y borrado lógico cross-tenant solo en recursos con superficie aceptada; no existen rutas `DELETE` inventadas.
[ ] AuthProvider productivo pre-beta valida token, issuer/audience/expiry y membership activa; DevAuthProvider solo funciona en local/test.
[ ] SSE de estados implementado.
[ ] WorkflowGateway/LocalWorkflowGateway implementados; Temporal no está acoplado al core.
[ ] Evidence/Citation/Claim schemas y modelos/stubs contractuales existen, respaldados por `0003_evidence_contract_stubs.py`.
[ ] Source Registry seed y `source_registry_health` existen.
[ ] Fetcher/Snapshot/OpenSearch stubs existen; evaluation harness/runner y regression suite contractual inicial no vacía pasan con reporte versionado `beta_eligible=false`.
[ ] Provider Reliability Layer (PRL)/ProviderCallAudit/RawAccessEvent/PromptInjectionGuard stubs existen y fallan cerrado; PRL no activa fallback productivo sin ruta, ProviderCallAudit conserva contexto por intento, orden temporal e igualdad de hashes con su owner/enlace terminal conforme a schema + provider policy/registry, RawAccessEvent valida schema + privacy/security policy y PromptInjectionGuard propaga riesgos por pasaje/URL/run conforme a prompt-injection policy.
[ ] `make scan-secrets` y `make scan-sensitive-surfaces` pasan como gates de CI y sus fixtures negativos prueban que ambos fallan cerrado.
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
