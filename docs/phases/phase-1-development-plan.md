# JusNova — Fase 1 Development Plan Ultra Detallado

**Ruta objetivo en repo:** `/docs/phases/phase-1-development-plan.md`
**Versión:** 1.0
**Fecha:** 2026-05-26
**Estado:** Accepted como plan de Fase 1; ejecucion habilitada por el cierre formal de 0.14/Fase 0
**Fase:** Fase 1 — Fundaciones backend, datos, telemetría y Cost Governor
**Propósito:** instruir el desarrollo completo de Fase 1 sin reinterpretación.

---

## 1. Veredicto de Fase 1

Fase 1 es la fundación operativa del backend de JusNova. No es una fase de IA visible. No es una fase de búsqueda legal. No es una fase de OCR. No es una fase de generación jurídica. Es la fase que impide que todo eso se construya sobre una base débil.

La Fase 1 debe cerrar estas capacidades:

1. **Estructura modular:** el backend nace organizado por dominios y contratos.
2. **Persistencia transaccional:** PostgreSQL y Alembic son obligatorios.
3. **Observabilidad mínima:** logs estructurados, request IDs y health checks.
4. **Trazabilidad mínima:** cada ejecución tiene `run_id` y eventos persistidos.
5. **Presupuesto:** `CostGovernor` existe antes de hacer llamadas caras.
6. **Medición:** `UsageLedger` registra consumo básico.
7. **Proveedor de modelos encapsulado:** no hay llamadas directas a modelos.
8. **Contratos futuros preparados:** Evidence, Claim, Citation, Source Registry y stubs.

El resultado de Fase 1 debe permitir iniciar Fase 2/3/4 sin reconstrucción del core.

---

## 2. Principios de ejecución

### 2.1 No construir inteligencia jurídica todavía

Fase 1 no debe producir análisis jurídico real. Cualquier texto devuelto al usuario debe ser técnico o de estado.

### 2.2 No construir búsqueda viva todavía

Fase 1 puede crear interfaces, seeds y stubs. No debe implementar adaptadores reales a portales oficiales ni discovery web real.

### 2.3 No crear dependencias externas innecesarias

OpenAI, OpenSearch, MinIO y Temporal deben estar encapsulados o desactivables según ambiente. El sistema debe poder pasar tests sin depender de servicios externos pagos.

### 2.4 No omitir trazabilidad

Toda ejecución conversacional mínima debe producir trazabilidad operativa:

- `operational_run`;
- `budget_decision`;
- `run_event`;
- `request_id`;
- `correlation_id`.

`usage_event` solo se produce cuando la ejecución queda publicada y existe un `UsageEvent` válido enlazado; un `run_failed` no fuerza usage ni billing.

### 2.5 No hardcodear budgets

Los límites viven en `docs/schemas/budgets.yaml` y se validan al arranque.

### 2.6 No permitir módulos sin dueño

Cada módulo debe tener responsabilidad definida. Si un componente no pertenece claramente a un módulo, no se implementa hasta decidir su ubicación.

---

## 3. Resultado final esperado de Fase 1

Al cerrar Fase 1, el backend debe estar listo para que Fase 2 implemente Evidence Contract, Citation Auditor y bloqueo de claims críticos; Fase 3 implemente conversación avanzada y versionado de respuestas; y Fase 4 implemente Live Legal Search sin romper estructura.

El equipo debe poder demostrar:

```txt
1. Se levanta el backend localmente.
2. Se aplican migraciones desde cero.
3. Health checks reportan estado real.
4. Se crea una organización y usuario semilla.
5. Se crea una conversación.
6. Se envía un mensaje.
7. El mensaje crea un run.
8. ModelProvider stub se invoca sin llamadas externas reales.
9. El CostGovernor evalúa presupuesto.
10. UsageLedger registra consumo.
11. RunEvents registran pasos.
12. SSE emite estados técnicos.
13. ErrorEnvelope normaliza errores.
14. Evidence/Citation/Claim schemas y modelos/stubs contractuales existen.
15. Source Registry seed y `source_registry_health` existen.
16. Fetcher/Snapshot/OpenSearch/Eval stubs existen.
17. Tests críticos pasan.
18. No se genera respuesta jurídica real.
```

---

## 4. Subfases de Fase 1

## SUBFASE 1.0 — Entry Gate y preparación del equipo

### Objetivo

Asegurar que Fase 1 inicia con documentos, decisiones y restricciones entendidas.

### Entradas requeridas

- ADRs fundacionales aceptados;
- `phase-1-implementation-brief.md`;
- `sprint-1-backlog.md`;
- policies críticas;
- budgets iniciales;
- JSON schemas críticos;
- risk register sin bloqueantes.

### Trabajo

1. Revisar brief completo con el equipo.
2. Confirmar que el plan mínimo es 400 Bs/mes.
3. Confirmar que no se construye RAG.
4. Confirmar que no se construye búsqueda legal real.
5. Confirmar que no se producen respuestas jurídicas reales.
6. Confirmar que toda ejecución debe tener budget y trazabilidad operativa; usage solo se registra para ejecuciones publicadas con `UsageEvent` válido enlazado, y `TraceObject` se crea solo para respuestas finalizadas o bloqueadas.
7. Confirmar stack cerrado.
8. Confirmar que OpenAI real queda bloqueado durante toda Fase 1 aunque existan stubs de `ProviderCallAudit`.

### Entregables

- checklist de inicio firmado o marcado en repo;
- issues creados según backlog;
- responsables asignados;
- rama base creada.

### Criterios de aceptación

```txt
[ ] El equipo entiende alcance y fuera de alcance.
[ ] Las tareas P0 están creadas.
[ ] No hay preguntas abiertas bloqueantes.
[ ] El entorno de desarrollo está definido.
```

### Riesgos

- reinterpretar Fase 1 como chat funcional;
- intentar adelantar búsqueda jurídica;
- ignorar CostGovernor.

### Gate

No se escribe código de Fase 1 hasta cerrar esta subfase.

---

## SUBFASE 1.1 — Estructura de repositorio y tooling base

### Objetivo

Crear un repositorio que soporte desarrollo serio, CI y modularidad.

### Trabajo

1. Crear estructura de carpetas.
2. Crear `pyproject.toml`.
3. Crear `README.md` técnico inicial.
4. Crear `.env.example`.
5. Crear `Makefile`.
6. Configurar lint/typecheck/test básicos.
7. Configurar `docker-compose.yml` con Postgres y Redis.
8. Crear módulos base vacíos con `__init__.py`.

### Decisiones fijas

- Código en `src/jusnova`.
- Tests en `tests`.
- Documentación en `docs`.
- Migraciones en `alembic`.
- Lint con `ruff check`.
- Typecheck con `mypy src tests`.
- No usar estructura improvisada.

### Comandos mínimos

```bash
make dev
make test
make lint
make typecheck
make migrate
make downgrade
make validate-schemas
```

### Entregables

- repo inicial;
- comandos Makefile;
- Docker Compose;
- estructura modular.

### Criterios de aceptación

```txt
[ ] make help funciona.
[ ] make dev intenta levantar app.
[ ] make test ejecuta pytest.
[ ] docker compose levanta Postgres y Redis; OpenSearch queda definido en profile `search` deshabilitado por defecto.
[ ] No hay secretos reales.
```

### Riesgos

- crear estructura excesivamente genérica;
- instalar dependencias innecesarias;
- no bloquear secretos.

---

## SUBFASE 1.2 — FastAPI modular base

### Objetivo

Levantar la API base con composición modular.

### Trabajo

1. Crear `main.py`.
2. Crear `api/router.py`.
3. Crear router `/v1`.
4. Crear router de health.
5. Configurar lifecycle startup/shutdown.
6. Configurar exception handlers base.
7. Configurar CORS básico para dev, no abierto para producción.

### Decisiones fijas

- FastAPI es el backend core.
- Routers no contienen lógica de negocio.
- Todos los endpoints deben usar schemas.
- La API versionada empieza en `/v1`.

### Entregables

- app corriendo;
- endpoint live;
- OpenAPI en dev.

### Criterios de aceptación

```txt
[ ] GET /health/live responde 200.
[ ] API docs disponibles solo en ambientes permitidos.
[ ] Router /v1 existe.
[ ] No hay lógica de dominio en main.py.
```

### Riesgos

- dejar todo en un solo archivo;
- configurar CORS abierto sin control;
- empezar endpoints sin contratos.

---

## SUBFASE 1.3 — Settings, environments y secrets policy

### Objetivo

Cerrar configuración tipada y segura.

### Trabajo

1. Implementar `Settings` con Pydantic.
2. Definir ambientes: `local`, `test`, `staging`, `production`.
3. Validar settings críticos por ambiente.
4. Crear `.env.example`.
5. Implementar helper `is_production`.
6. Implementar feature flags.
7. Documentar reglas de secretos.

### Campos mínimos de Settings

Esta lista debe coincidir con `docs/handoff/phase-1-implementation-brief.md` y no puede reducirse en P0-03.

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

La lista define campos tipados, no env vars requeridas incondicionalmente. `REDIS_URL` solo es obligatorio si `ENABLE_REDIS=true`; `OPENSEARCH_URL` solo si `ENABLE_OPENSEARCH=true`; `S3_ENDPOINT_URL`, `S3_BUCKET`, `S3_ACCESS_KEY_ID` y `S3_SECRET_ACCESS_KEY` solo si `ENABLE_S3=true`. Con flags `false`, tests y boot local no fallan por ausencia de esas URLs/credenciales.

### Criterios de aceptación

```txt
[ ] Settings cargan desde env.
[ ] Tests pueden usar env aislado.
[ ] Producción no permite `ENABLE_OPENAI_REAL_CALLS=true` en Fase 1.
[ ] Producción no permite dev auth.
[ ] No hay acceso disperso a os.environ.
```

### Riesgos

- default inseguro;
- secretos en `.env.example`;
- feature flags ignorados.

---

## SUBFASE 1.4 — Base de datos, sesiones y migraciones

### Objetivo

Establecer PostgreSQL como sistema de verdad.

### Trabajo

1. Configurar engine/session.
2. Configurar Alembic.
3. Crear base model con timestamps.
4. Crear `0001_core_foundation`.
5. Crear `0002_trace_usage_budget`.
6. Crear tests de migración.
7. Documentar comandos DB.

### Decisiones fijas

- PostgreSQL es obligatorio.
- Alembic es obligatorio.
- No usar `metadata.create_all()` en runtime.
- IDs publicos y cross-contract usan los prefijos de `docs/architecture/domain-model.md`; UUID solo puede usarse como surrogate privado interno.
- Timestamps: timezone-aware.

### Migraciones mínimas

- organizations;
- users;
- memberships;
- conversations;
- messages;
- operational_runs;
- run_events;
- model_calls;
- tool_calls;
- answers;
- answer_versions;
- abstention_renders;
- trace_objects;
- budget_decisions;
- usage_events;
- plans;
- subscriptions;
- research_credits;
- research_credit_movements;
- provider_call_audits;
- raw_access_events.

`operational_runs` y `run_events` son tablas de implementacion para orquestar streaming, presupuesto y auditoria tecnica. No agregan entidades canonicas nuevas a 0.11; el `run_id` publico conserva forma `tr_*` y no es `TraceObject` schema-valid hasta la finalizacion.

### Criterios de aceptación

```txt
[ ] alembic upgrade head funciona en DB limpia.
[ ] alembic downgrade -1 funciona para migraciones recientes si es razonable.
[ ] Tests de migración pasan.
[ ] Modelos tienen índices mínimos.
[ ] No hay tablas sin organization_id donde corresponda.
```

### Riesgos

- migraciones no reproducibles;
- constraints insuficientes;
- no indexar por tenant/fecha.

---

## SUBFASE 1.5 — Request context, correlation y ErrorEnvelope

### Objetivo

Normalizar trazabilidad por request y respuestas de error.

### Trabajo

1. Middleware `RequestContextMiddleware`.
2. Generación de `request_id`.
3. Propagación de `correlation_id`.
4. Headers de respuesta.
5. `ErrorEnvelope` Pydantic.
6. Exception handlers.
7. Tests de propagación y error.

### Contrato ErrorEnvelope

```json
{
  "error_code": "validation_error",
  "safe_message_code": "invalid_request",
  "message": "La solicitud no cumple el contrato esperado.",
  "severity": "error",
  "request_id": "rq_phase1_validation",
  "retryable": false,
  "user_visible": true,
  "created_at": "2026-05-26T00:00:00Z",
  "metadata": {}
}
```

`ErrorEnvelope` es top-level y debe coincidir con `docs/contracts/error-envelope.schema.json`; `correlation_id` vive en headers/logs, no como campo requerido del envelope.
`tenant_mismatch` debe estar incluido desde P0 porque Fase 1 crea el esqueleto tenant.

### Criterios de aceptación

```txt
[ ] Todo error HTTP devuelve ErrorEnvelope.
[ ] request_id aparece en headers.
[ ] correlation_id aparece en headers.
[ ] Validation errors están normalizados.
[ ] No se exponen stack traces en producción.
```

### Riesgos

- errores inconsistentes;
- IDs sin sanitizar;
- leaking de detalles internos.

---

## SUBFASE 1.6 — Logging estructurado y observabilidad mínima

### Objetivo

Crear logs consultables, útiles y seguros.

### Trabajo

1. Configurar `structlog`.
2. Emitir logs JSON.
3. Agregar request context a logs.
4. Agregar middleware de latencia.
5. Sanitizar secrets.
6. Preparar OpenTelemetry básico si el equipo lo puede cerrar sin bloquear P0.

### Eventos mínimos

- app_started;
- request_started;
- request_finished;
- request_failed;
- budget_checked;
- usage_event_recorded, solo cuando existe `UsageEvent` válido enlazado;
- trace_event_recorded.

### Criterios de aceptación

```txt
[ ] Logs tienen request_id/correlation_id.
[ ] Logs no tienen API keys.
[ ] Logs no guardan bodies completos por defecto.
[ ] Latencia por request se registra.
```

### Riesgos

- loggear datos sensibles;
- logs demasiado verbosos;
- logs sin correlación.

---

## SUBFASE 1.7 — Health checks y dependency readiness

### Objetivo

Permitir operación y despliegue seguro.

### Trabajo

1. Implementar `/health/live`.
2. Implementar `/health/ready`.
3. Implementar `/health/version`.
4. Check DB.
5. Check Redis si está enabled.
6. Check OpenSearch si está enabled.
7. Check S3 si está enabled.
8. Check budgets YAML válido.

### Respuesta esperada

```json
{
  "status": "HEALTHY",
  "service": "jusnova-backend",
  "version": "0.1.0",
  "environment": "local",
  "dependencies": {
    "database": {"enabled": true, "status": "HEALTHY"},
    "redis": {"enabled": true, "status": "HEALTHY"},
    "opensearch": {"enabled": false, "status": "UNKNOWN"},
    "object_storage": {"enabled": false, "status": "UNKNOWN"},
    "budgets": {"enabled": true, "status": "HEALTHY"}
  }
}
```

Los valores `status` usan `docs/schemas/host-statuses.yaml`; no se introducen enums de health paralelos. Una dependencia deshabilitada por flag usa `enabled=false`, `status=UNKNOWN`.

### Criterios de aceptación

```txt
[ ] live no depende de DB.
[ ] ready falla si DB está caída.
[ ] budgets inválidos hacen ready `DOWN`.
[ ] dependencias apagadas por flag se reportan `enabled=false`, `status=UNKNOWN`, no error.
```

### Riesgos

- readiness falso positivo;
- health checks costosos;
- dependencia opcional bloqueando dev.

---

## SUBFASE 1.8 — Modelos base y tenant skeleton

### Objetivo

Crear la base de organizaciones, usuarios y conversación.

### Trabajo

1. Implementar `organizations`.
2. Implementar `users`.
3. Implementar `memberships`.
4. Implementar tenant context mínimo.
5. Implementar `conversations`.
6. Implementar `messages`.
7. Crear `ConversationService` y `MessageService` internos.
8. Crear seed dev.

### Endpoints mínimos

```http
POST /v1/conversations
GET  /v1/conversations/{conversation_id}
```

`MessageService` debe poder crear mensajes en tests e integrarse con runs, budget, usage y trace. El endpoint público `POST /v1/conversations/{conversation_id}/messages` pertenece a Subfase 1.13/P1-04 junto con SSE; no es requisito de cierre de P0.

### Auth temporal

Fase 1 puede usar `DevAuthProvider` únicamente en `APP_ENV=local|test`.

Headers dev permitidos:

```txt
X-Dev-Organization-Id
X-Dev-User-Id
```

Regla:

- En producción, `DevAuthProvider` debe fallar.
- Esta no es auth real de mercado.

### Criterios de aceptación

```txt
[ ] No hay conversación sin organization_id.
[ ] No hay mensaje sin conversation_id.
[ ] Dev auth no funciona en production.
[ ] Tests validan tenant base.
[ ] `test_tenant_isolation.py` cubre negativos cross-tenant de lectura, escritura, listado y borrado lógico solo donde exista superficie aceptada; no se crean rutas `DELETE` nuevas.
```

### Riesgos

- confundir dev auth con auth real;
- tenant isolation incompleto;
- no asociar usage/trace a organization.

---

## SUBFASE 1.9 — Runs, trazabilidad mínima y auditabilidad base

### Objetivo

Cada ejecución debe ser reconstruible al menos técnicamente.

### Trabajo

1. Crear `operational_runs`.
2. Crear `run_events`.
3. Crear `model_calls`.
4. Crear `tool_calls`.
5. Crear tablas shell `answers`, `answer_versions`, `abstention_renders` y `trace_objects`.
6. Crear mensaje tecnico `assistant_final` como `TraceObject.output_message_id`.
7. Definir FKs circulares como `DEFERRABLE INITIALLY DEFERRED` o una estrategia transaccional equivalente.
8. Crear interfaz/repositorio base de `TraceService`.
9. Crear endpoint trace interno sobre eventos persistidos.
10. Dejar explícito que la integración funcional de P0-13 se completa después de 1.10, 1.11 y 1.12, antes de exponer el pipeline 1.13.

### Eventos estructurales de 1.9

```txt
run_created
message_received
message_persisted
```

Si hay error:

```txt
run_failed
error_envelope_returned
```

Estos nombres son internos de `run_events`; no son eventos publicos SSE. Deben coincidir con el enum operativo cerrado de `phase-1-implementation-brief.md`. El `run_failed` interno de P0 registra fallo técnico cuando ya existe `operational_run`; el `run_failed` público SSE se emite solo cuando Subfase 1.13/P1-04 expone streaming.

`run_events` persiste solo eventos internos en Fase 1. Los SSE publicos se emiten como `StreamEvent`; si se audita una emision publica, se registra como `stream_event_emitted`.

Eventos que dependen de servicios posteriores:

```txt
model_provider_stubbed -- se emite al integrar 1.10
budget_checked         -- se emite al integrar 1.11
usage_event_recorded   -- se emite al integrar 1.12 solo cuando existe UsageEvent valido enlazado
stream_event_emitted   -- se emite al integrar 1.13
run_completed          -- se emite al cerrar el pipeline 1.13
```

Subfase 1.9 crea tablas, DTOs, repositorios base y enum operativo. No cierra P0-13 funcional por sí sola. El cierre P0-13 ocurre después de 1.10, 1.11 y 1.12, cuando ya existen `CostGovernor`, `UsageLedger` y `ModelProvider`, y antes de exponer el pipeline 1.13.

### Criterios de aceptación

```txt
[ ] Checkpoint base: tablas, DTOs, repositorios y enum operativo existen.
[ ] Checkpoint funcional posterior a 1.12: cada invocación del message pipeline interno crea run.
[ ] Checkpoint funcional posterior a 1.12: cada run tiene run_events.
[ ] Errores se asocian a run cuando existe.
[ ] Trace endpoint devuelve eventos ordenados.
[ ] Model/tool calls tienen repository aunque no se usen plenamente.
[ ] Para bloqueos/abstenciones, `AnswerVersion.answer_hash` coincide con `AbstentionRender.render_hash`.
[ ] `AbstentionRender.render_storage_ref` reconstruye `render_body_canonical` DB-local y `render_hash=sha256(canonicalize(render_body_canonical))`; no depende de S3, filesystem ni StorageProvider real.
[ ] `answer_blocked` valida `TraceObject` completo contra `trace-object.schema.json`, incluido `citation_audit` compatible con `policy_blocked`.
[ ] `AbstentionRender` de bloqueos comparte `trace_id`, `answer_id`, `answer_version`, `response_outcome` y `reason_code` con `AnswerVersion`/`TraceObject`; `source_trace_refs` es subconjunto real del `TraceObject`.
```

### Riesgos

- trazabilidad solo en logs;
- runs no asociados a usuario/organización;
- payloads con datos sensibles.

---

## SUBFASE 1.10 — ModelProvider wrapper y stub determinista

### Objetivo

Encapsular modelos desde el inicio.

### Trabajo

1. Definir `ModelProvider` protocol/interface.
2. Definir `ModelRequest`.
3. Definir `ModelResponse`.
4. Implementar `StubModelProvider`.
5. Registrar `model_calls`.
6. Integrar feature flag como bloqueo explicito de llamadas reales en Fase 1.
7. Crear tests.

### Regla

Nadie llama OpenAI directamente. Nunca.

### Stub esperado

Debe poder devolver:

```json
{
  "content": "Fase 1: pipeline técnico ejecutado. No se emite análisis jurídico.",
  "token_usage": {
    "input_tokens": 0,
    "output_tokens": 0,
    "total_tokens": 0
  },
  "estimated_cost": 0,
  "provider": "stub",
  "model": "stub-model"
}
```

### Criterios de aceptación

```txt
[ ] Stub es default.
[ ] Tests no requieren API key.
[ ] Model call se registra conforme a `docs/contracts/model-call.schema.json` si se invoca.
[ ] Feature flag impide proveedor real; activarlo queda fuera de Fase 1 y exige decisión aceptada posterior más `ProviderCallAudit` productivo.
[ ] No hay prompts jurídicos finales.
```

### Riesgos

- acoplar OpenAI al core;
- esconder costos;
- permitir llamadas reales en test.

---

## SUBFASE 1.11 — CostGovernor y budgets YAML

### Objetivo

Controlar costo, profundidad y modos antes de integrar IA real.

### Trabajo

1. Consumir y validar `docs/schemas/budgets.yaml` aceptado en 0.8.
2. Crear loader validado.
3. Crear Pydantic schemas internos para request/decision, resolviendo limites desde `docs/contracts/cost-budget.schema.json` y `docs/schemas/budgets.yaml` sin promover `BudgetDecision` a contrato JSON Schema nuevo.
4. Implementar `CostGovernorService`.
5. Crear `budget_decisions`.
6. Integrar a message pipeline.
7. Crear tests.

### Regla comercial fija

El plan mínimo es:

```txt
PROFESIONAL — 400 Bs/mes
```

No crear plan inferior.

### Complejidades mínimas

- `SIMPLE`;
- `MEDIO`;
- `COMPLEJO`;
- `INVESTIGACION`.

### Criterios de aceptación

```txt
[ ] Budgets cargan desde YAML.
[ ] Plan PROFESIONAL existe.
[ ] No hay plan menor a 400 Bs.
[ ] CostGovernor devuelve allow/degrade/block.
[ ] Toda ejecución tiene budget_decision.
[ ] Tests cubren límites por complejidad.
[ ] Tests de `research_credit_cost` son budget-only; el débito `research_credit_used` se valida en Subfase 1.12 al existir UsageLedger y anchors.
```

### Riesgos

- budget decorativo;
- límites hardcodeados;
- no registrar decisiones.

---

## SUBFASE 1.12 — UsageLedger y commercial mínimo

### Objetivo

Registrar consumo por organización desde el inicio y resolver plan/suscripción activa sin fuentes comerciales paralelas.

### Trabajo

1. Crear `usage_events`.
2. Crear `plans`, `subscriptions`, `research_credits` y `research_credit_movements`.
3. Crear seed `PROFESIONAL`, `monthly_research_credits=8`, suscripción bootstrap y grant mensual por organización de desarrollo.
4. Crear `CommercialService`/repository para resolver `subscription_id`, `plan_code` y límites visibles.
5. Crear `UsageLedgerService`.
6. Registrar eventos del pipeline.
7. Crear endpoint `GET /v1/usage/current`.
8. Crear endpoint `GET /v1/research-credits`.
9. Crear tests de usage, créditos y suscripción activa.

### Eventos mínimos

```txt
standard_query
complex_query
research_credit_used
```

`model_input_tokens` y `model_output_tokens` son eventos contractuales aceptados, pero en Fase 1 se registran solo cuando ya existe `model_call_id` por la integración de ModelProvider/P0-13/P1-05.
`research_credit_used` es obligatorio para ejecuciones publicables `COMPLEJO` e `INVESTIGACION`; si no puede registrarse con quantity `1` o `2` respectivamente, la ejecución debe bloquearse o degradarse.
La prueba de débito conciliable requiere los anchors `trace_id` o `answer_id`; por eso se ejecuta junto con la integración de trazabilidad mínima, no como test aislado de budget puro.

### Criterios de aceptación

```txt
[ ] Usage asociado a organization_id.
[ ] Usage asociado a actor_ref/actor_type sin PII directa.
[ ] Usage asociado a conversation_id, trace_id o model_call_id cuando aplique.
[ ] `subscription_id` se resuelve desde `subscriptions`, no desde `organizations.plan_code`.
[ ] Existe una sola suscripción current por organización.
[ ] `research_credits` mantiene saldo por suscripción y periodo.
[ ] `research_credit_movements` registra grants y débitos enlazados a `usage_event_id` cuando aplique.
[ ] Endpoint devuelve `period_start`, `period_end`, `plan_code`, `subscription_id`, `usage_totals`, `limits` y `research_credits_balance`.
[ ] `GET /v1/research-credits` devuelve balance y movimientos resumidos.
[ ] `research_credit_used` se registra para `COMPLEJO` e `INVESTIGACION` o la ejecución se bloquea/degrada.
[ ] Tests no dependen de logs.
[ ] Tests cubren resolución de suscripción activa y seed `PROFESIONAL`.
[ ] Tests cubren grant inicial y saldo no negativo.
[ ] Budget decisions y stream status quedan en budget_decisions/run_events, no como UsageEvent inventado.
```

### Riesgos

- usage no confiable;
- contabilidad duplicada;
- no tener unidades claras.

---

## SUBFASE 1.13 — Pipeline mínimo de conversación y streaming de estados

### Objetivo

Probar el ciclo completo sin análisis jurídico.

### Trabajo

1. Endpoint de crear conversación.
2. Endpoint de enviar mensaje.
3. Crear run.
4. Aplicar CostGovernor.
5. Persistir mensaje.
6. Invocar ModelProvider stub o simular etapa.
7. Registrar usage.
8. Registrar trace.
9. Exponer SSE de estados.
10. Cerrar run.

### Eventos SSE publicos minimos

```txt
run_queued
run_started
answer_blocked
run_failed
```

`answer_blocked` es el evento terminal publico para una ejecucion tecnica exitosa de Fase 1 que no emite analisis juridico. Debe estar respaldado por `Answer`, `AnswerVersion`, `AbstentionRender`, mensaje `assistant_final` y `TraceObject` schema-valid contra `docs/contracts/trace-object.schema.json`, con `response_outcome=blocked`, `output_message_id`, razon contractual `policy_blocked` y `citation_audit` minimo contract-compatible para `policy_blocked`.

### Criterios de aceptación

```txt
[ ] El frontend puede consumir SSE.
[ ] No se emiten deltas de respuesta ni analisis juridico.
[ ] Cada evento publico SSE usa `event_type`/`status` del API draft.
[ ] Cada hito interno registra run_event si corresponde.
[ ] Run termina completed o failed.
```

### Riesgos

- transmitir texto legal prematuro;
- eventos sin orden;
- no persistir estado.

---

## SUBFASE 1.14 — Evidence, Claim y Citation contracts fundacionales

### Objetivo

Preparar Fase 2 sin implementar auditoría real.

### Trabajo

1. Crear Pydantic schemas y validators desde los JSON Schemas aceptados en `docs/contracts/`.
2. Crear modelos ORM/stubs contractuales para las tablas mínimas de evidencia, claims y citas.
3. Crear `0003_evidence_contract_stubs.py` para tablas stubs contractuales.
4. Crear fixtures.
5. Crear tests de validación.

### Contratos mínimos

- EvidencePack;
- EvidenceSource;
- EvidencePassage;
- Claim;
- Citation.

### Regla

Estos contratos no son RAG. Son estructura de evidencia. `CitationAudit` queda como value object embebido en `TraceObject`; no crear tabla standalone ni JSON Schema paralelo en Fase 1.

### Criterios de aceptación

```txt
[ ] Schemas validan fixtures.
[ ] Referencias F#:P# son validables.
[ ] Referencias D#:P# son validables.
[ ] Claims tienen support_level.
[ ] `0003_evidence_contract_stubs.py` aplica desde cero junto con las migraciones previas.
[ ] No hay generación legal.
```

### Riesgos

- confundir contratos con recuperación real;
- schemas demasiado laxos;
- no preservar localizadores.

---

## SUBFASE 1.15 — Source Registry y stubs de búsqueda/snapshot/OpenSearch

### Objetivo

Preparar Fase 4 sin acceso real a portales.

### Trabajo

1. Crear modelo operativo de seed para Source Registry sin promover `SourceRegistryEntry` a schema nuevo.
2. Crear `0004_source_registry_seed.py`.
3. Crear seed de fuentes.
4. Crear tabla `source_registry_health` 1:1 por `source_registry_entry_id`.
5. Crear Fetcher interface.
6. Crear FetcherStub.
7. Crear Snapshot interface.
8. Crear SnapshotStub.
9. Crear OpenSearch client stub.
10. Crear readiness opcional para OpenSearch.

### Fuentes seed

- Gaceta Oficial de Bolivia;
- TCP;
- TSJ/Génesis;
- SILEP;
- Dominios oficiales `.gob.bo`.

### Criterios de aceptación

```txt
[ ] Seed idempotente.
[ ] `0004_source_registry_seed.py` aplica el seed de forma idempotente.
[ ] `source_registry_health` existe como tabla separada, no columna alternativa, con una fila por fuente seed.
[ ] Health inicial de cada fuente seed es `UNKNOWN` con `last_checked_at=null` porque no hay fetch real en Fase 1; `live_fetch_enabled=false` expresa que está deshabilitada.
[ ] Live fetch disabled por defecto.
[ ] FetcherStub no hace red real.
[ ] SnapshotStub genera `snapshot_id` stub con forma `snap_*`.
[ ] OpenSearch disabled no rompe app.
```

### Riesgos

- implementar scraping antes de política;
- habilitar proveedores externos sin evaluación;
- confundir seed con fuente validada en vivo.

---

## SUBFASE 1.16 — Evaluation runner skeleton

### Objetivo

Dejar preparada la cultura de evaluación desde Fase 1.

### Trabajo

1. Crear `src/jusnova/modules/evals/runner_stub.py`.
2. Crear comando `make eval-smoke`.
3. Crear estructura de reportes.
4. Crear fixture vacío.
5. Documentar que el smoke skeleton no satisface beta.
6. Definir modo pre-beta que produce reporte versionado con `dataset_version`, versión de golden dataset y gates 0.12.

### Criterios de aceptación

```txt
[ ] make eval-smoke corre.
[ ] No requiere dataset jurídico real.
[ ] Produce reporte mínimo.
[ ] No bloquea Sprint 1 si P0 incompleto.
[ ] Beta queda bloqueada mientras no exista reporte versionado con `dataset_version` conforme a `docs/quality/beta-readiness-gates.md`.
```

### Riesgos

- crear evaluación falsa;
- reportar calidad no medida;
- gastar tiempo antes de cerrar P0.

---

## SUBFASE 1.17 — Beta blocker foundations de Fase 1

### Objetivo

Cubrir los gates de beta asignados a Fase 1 sin activar proveedores reales, fetch real, OCR real ni evaluación jurídica completa.

### Trabajo

1. Crear `ProviderReliabilityLayerStub`.
2. Crear `ProviderCallAuditService` y fixtures schema-valid + policy-valid contra `provider-policy.md` y `provider-registry.yaml`.
3. Crear `RawAccessAuditService` y fixtures schema-valid + policy-valid contra `privacy-security-policy.md`.
4. Crear `PromptInjectionGuardStub`.
5. Crear `StorageProvider` interface/stub y validación de límites privados.
6. Crear tests de timeout, rate limit y provider unavailable con `ErrorEnvelope`.
7. Crear tests fail-closed para llamada externa sin audit ref.
8. Crear tests fail-closed para acceso raw/elevado sin `RawAccessEvent`.
9. Crear tests fail-closed de escritura storage sin provider habilitado.
10. Crear tests de prompt injection `blocking` bloqueado o excluido.
11. Cuando `PromptInjectionGuardStub` opere en frontera API, devolver `ErrorEnvelope.error_code=prompt_injection_blocked`.
12. Crear `WorkflowGateway`/`LocalWorkflowGateway` como contrato local de workflows, sin requerir Temporal ni importarlo desde el core.
13. Probar que `ProviderCallAudit` rechaza provider desconocido, familia divergente, payload enviado fuera de allowlist y audit huérfano sin `trace_id|model_call_id|tool_call_id|retrieval_run_id|usage_event_id|cost_report_id`.
14. Probar que `ProviderCallAudit.status=policy_blocked` permite `attempted_data_classes[]` conocidas fuera de allowlist solo cuando `data_sent_classes=[]`.
15. Probar que `RawAccessEvent` rechaza `approved_by_ref == actor_ref`, `expires_at <= accessed_at` y refs documentales inconsistentes con `resource_type`.

### Criterios de aceptación

```txt
[ ] Provider Reliability Layer (PRL) básico devuelve errores controlados.
[ ] ProviderCallAudit fixtures validan contra schema, provider registry y provider policy, incluyendo refs contractuales tenant-compatible.
[ ] RawAccessEvent fixtures validan contra schema y privacy/security policy.
[ ] StorageProviderStub falla cerrado y no escribe documentos reales.
[ ] PromptInjectionGuardStub bloquea riesgo blocking.
[ ] API boundary de prompt injection blocking usa `prompt_injection_blocked`.
[ ] WorkflowGateway/LocalWorkflowGateway existen y `make test` no requiere Temporal.
[ ] Tests no requieren OpenAI, internet ni documentos reales.
```

### Riesgos

- confundir stubs de beta gate con proveedores reales;
- registrar auditoría sin tenant;
- permitir acceso raw sin evento elevado.

---

## SUBFASE 1.18 — Tests, CI y gates técnicos

### Objetivo

Garantizar que la fundación no se rompe silenciosamente.

### Trabajo

1. Configurar pytest.
2. Crear fixtures de DB.
3. Crear tests unitarios P0.
4. Crear tests integration P0.
5. Configurar `ruff check` para lint y `mypy src tests` para typecheck.
6. Configurar CI si el repositorio ya lo permite.
7. Crear comandos Make.

### Tests obligatorios

```txt
test_settings.py
test_error_envelope.py
test_request_context.py
test_health.py
test_migrations.py
test_conversations.py
test_tenant_isolation.py
test_cost_governor.py
test_commercial_service.py
test_usage_ledger.py
test_trace_persistence.py
test_model_provider_stub.py
test_storage_provider_stub.py
test_schema_validator.py
test_streaming_status.py
test_source_registry_seed.py
test_fetcher_stub.py
test_snapshot_stub.py
test_opensearch_stub.py
test_eval_smoke.py
test_prl_stub.py
test_provider_call_audit.py
test_raw_access_event.py
test_prompt_injection_guard_stub.py
```

### Criterios de aceptación

```txt
[ ] make test pasa.
[ ] make lint pasa.
[ ] make typecheck pasa.
[ ] Tests no requieren OpenAI.
[ ] Tests no requieren internet.
[ ] Tests aplican migraciones.
```

### Riesgos

- tests solo unitarios sin DB;
- CI lento;
- tests dependen del orden.

---

## SUBFASE 1.19 — Cierre de Fase 1 y handoff

### Objetivo

Cerrar Fase 1 de forma auditable y preparar Fase 2/3.

### Trabajo

1. Ejecutar checklist de Fase 1.
2. Documentar desviaciones.
3. Actualizar risk register.
4. Actualizar open questions.
5. Exportar OpenAPI contract.
6. Documentar comandos de operación.
7. Preparar backlog de Fase 2/3 según orden decidido.

### Criterios de aceptación

```txt
[ ] Checklist de Fase 1 completo.
[ ] P0 cerrado.
[ ] P1 crítico cerrado o planificado.
[ ] No hay bloqueantes para Fase 2/3.
[ ] Riesgos actualizados.
[ ] OpenAPI exportado.
```

### Riesgos

- cerrar por presión sin tests;
- esconder deuda;
- pasar a búsqueda sin trazabilidad completa.

---

## 5. Orden recomendado por sprints dentro de Fase 1

### Sprint 1 — Foundation Skeleton

Objetivo:

- repo;
- FastAPI;
- settings;
- DB;
- logging;
- request IDs;
- health;
- modelos base;
- ErrorEnvelope;
- ModelProvider stub;
- CostGovernor;
- Plan/Subscription/ResearchCredit mínimo;
- CommercialService;
- UsageLedger;
- trace mínimo.

### Sprint 2 — Conversation Pipeline & Contracts

Objetivo:

- endpoint de mensajes;
- SSE de estados;
- model/tool call persistence;
- Evidence/Citation/Claim schemas, modelos/stubs contractuales y migración `0003`;
- Source Registry seed y `source_registry_health`;
- stubs de retrieval/snapshot/OpenSearch;
- tests ampliados.

### Sprint 3 — Hardening de Fase 1

Objetivo:

- CI;
- docs;
- OpenAPI;
- readiness robusto;
- eval runner skeleton;
- eval report versionado antes de beta;
- revisión de seguridad básica;
- cleanup de deuda;
- cierre de Fase 1.

---

## 6. Criterios de aceptación globales de Fase 1

Fase 1 no se aprueba hasta que se cumpla:

```txt
[ ] Backend modular operativo.
[ ] Docker Compose operativo.
[ ] Migrations reproducibles.
[ ] Settings tipados.
[ ] Health live/ready/version.
[ ] ErrorEnvelope universal.
[ ] request_id/correlation_id en headers/logs.
[ ] Logging JSON.
[ ] Organizations/Users/Memberships.
[ ] Conversations/Messages.
[ ] Operational runs implementados como detalle tecnico, no entidad canonica nueva.
[ ] Run events implementados como detalle tecnico, no sustituto de `TraceObject`.
[ ] `answer_blocked` verificado con las invariantes cross-contract de `AnswerVersion`/`AbstentionRender`/`TraceObject`, no solo con SSE o run events.
[ ] ModelCalls/ToolCalls persistence mínima.
[ ] CostGovernor con budgets YAML.
[ ] UsageLedger básico.
[ ] `research_credit_used` se debita para `COMPLEJO`/`INVESTIGACION` o esas ejecuciones se bloquean/degradan.
[ ] `GET /v1/research-credits` devuelve balance y movimientos resumidos.
[ ] Plan/Subscription/ResearchCredit mínimo y CommercialService operativos.
[ ] Tenant isolation cross-tenant probado para lectura, escritura, listado y borrado lógico solo donde exista superficie aceptada; no se crean rutas `DELETE` nuevas.
[ ] ModelProvider stub default.
[ ] SSE de estados.
[ ] WorkflowGateway/LocalWorkflowGateway sin acoplamiento directo a Temporal.
[ ] Evidence/Citation/Claim schemas, modelos/stubs contractuales y migración `0003_evidence_contract_stubs.py`.
[ ] SourceRegistry seed y `source_registry_health`.
[ ] Fetcher/Snapshot/OpenSearch/Eval stubs.
[ ] Provider Reliability Layer (PRL), ProviderCallAudit, RawAccessEvent y PromptInjectionGuard stubs fail-closed, con ProviderCallAudit/RawAccessEvent policy-valid además de schema-valid y PromptInjectionGuard alineado con `prompt-injection-policy.md`.
[ ] `make test` completo pasa con tests P0/P1/P2 aplicables a los artefactos implementados.
[ ] No hay análisis jurídico real.
[ ] No hay búsqueda viva real.
[ ] No hay RAG.
[ ] No hay secretos reales.
```

---

## 7. Métricas técnicas que deben existir al cierre

Aunque sean básicas, deben poder medirse:

| Métrica | Fuente | Obligatoria |
|---|---|---|
| requests total | logs/metrics | sí |
| request latency | middleware | sí |
| request_id coverage | tests/logs | sí |
| DB health | ready check | sí |
| budget decisions | DB | sí |
| usage events | DB | sí |
| run events por operational run | DB | sí |
| model calls registradas | DB | sí si se invoca stub |
| stream events emitidos | tests | sí |
| errors por code | logs | sí |

---

## 8. Entregables documentales de Fase 1

Deben quedar en repo:

```txt
docs/handoff/phase-1-implementation-brief.md
docs/handoff/sprint-1-backlog.md
docs/phases/phase-1-development-plan.md
docs/schemas/budgets.yaml
docs/contracts/error-envelope.schema.json
docs/contracts/cost-budget.schema.json
docs/contracts/usage-event.schema.json
docs/contracts/trace-object.schema.json
docs/contracts/evidence-pack.schema.json
docs/contracts/claim.schema.json
docs/contracts/citation.schema.json
```

---

## 9. Riesgos globales y mitigaciones

### 9.1 Riesgo: fundación demasiado lenta

Mitigación:

- cerrar P0 primero;
- dejar P1/P2 para Sprint 2;
- no implementar legal search.

### 9.2 Riesgo: fundación demasiado superficial

Mitigación:

- exigir migraciones;
- exigir tests;
- exigir tracing;
- exigir CostGovernor.

### 9.3 Riesgo: dependencia externa prematura

Mitigación:

- stubs por defecto;
- feature flags;
- readiness por dependencia habilitada.

### 9.4 Riesgo: auth dev confundida con producción

Mitigación:

- `DevAuthProvider` bloqueado en producción;
- registrar auth real como deuda de seguridad pre-beta/pre-market, sin asignarla a Fase 8;
- tenant context obligatorio.

### 9.5 Riesgo: costos no medidos

Mitigación:

- UsageLedger desde Sprint 1;
- model calls obligatorias detrás de wrapper;
- budget decision por run.

### 9.6 Riesgo: trazas incompletas

Mitigación:

- tests de trace persistence;
- run obligatorio;
- eventos mínimos por pipeline.

---

## 10. Qué debe revisar el arquitecto antes de aprobar Fase 1

Checklist de revisión senior:

```txt
[ ] ¿La estructura del repo permitirá crecer sin caos?
[ ] ¿Los routers están limpios?
[ ] ¿Los services contienen la lógica correcta?
[ ] ¿Las migraciones son reproducibles?
[ ] ¿Los modelos tienen organization_id donde corresponde?
[ ] ¿Los errores siguen ErrorEnvelope?
[ ] ¿Los logs tienen request_id/correlation_id?
[ ] ¿Hay secretos en logs o repo?
[ ] ¿CostGovernor usa budgets.yaml?
[ ] ¿UsageLedger registra eventos reales?
[ ] ¿CommercialService resuelve suscripción activa sin fuente paralela?
[ ] ¿Cada message pipeline crea run?
[ ] ¿ModelProvider está encapsulado?
[ ] ¿Tests cubren P0, P1/P2 aplicables y aislamiento tenant cross-tenant?
[ ] ¿No hay análisis jurídico prematuro?
[ ] ¿No hay búsqueda viva prematura?
[ ] ¿Stubs están claramente marcados como stubs?
```

---

## 11. Handoff hacia Fase 2/3/4

Al cerrar Fase 1, el sistema queda listo para tres rutas paralelizables con cuidado:

### Ruta A — Evidence/Citation

Base para Fase 2:

- Evidence Pack builder;
- Citation Auditor;
- Claim Extractor;
- Answer Contract renderer.

### Ruta B — Conversation Core avanzado

Base para Fase 3:

- memoria conversacional;
- resúmenes persistentes;
- context assembler;
- streaming avanzado.

### Ruta C — Live Legal Search

Base para Fase 4:

- Provider Reliability Layer (PRL) completo para proveedores reales, extendiendo el stub basico de Fase 1;
- Source Registry real;
- Fetcher real;
- Snapshotter real;
- Discovery providers;
- búsqueda en fuentes oficiales.

Fase 1 no construye esas rutas completas; solo deja sus cimientos.

---

## 12. Decisión final

Fase 1 debe ejecutarse con disciplina.

No se acepta avanzar a funcionalidades de apariencia inteligente si todavía no existen:

- migraciones;
- request context;
- ErrorEnvelope;
- CostGovernor;
- UsageLedger;
- trace events;
- ModelProvider wrapper;
- CommercialService;
- StorageProvider;
- tenant isolation tests;
- tests P0 y tests P1/P2 aplicables a cualquier artefacto implementado.

La regla final de Fase 1:

> **Antes de que JusNova sea brillante, debe ser controlable. Antes de que sea controlable, debe ser trazable. Antes de que sea trazable, debe estar bien estructurado. Fase 1 construye exactamente eso.**
