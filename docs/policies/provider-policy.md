# Provider Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.10 - Seguridad, privacidad y proveedores externos
**Enmienda:** Subfase 0.14 - route resolution fail-closed y auditoria por intento

## Proposito

Definir familias de proveedores, registry, limites de datos enviados, auditoria y fronteras para que el core juridico no dependa de SDKs externos ni envie datos fuera de politica.

## Familias canonicas

1. `ModelProvider`
2. `SearchDiscoveryProvider`
3. `OfficialSourceAdapter`
4. `FetchProvider`
5. `ExtractionProvider`
6. `OCRProvider`
7. `EmbeddingProvider`
8. `StorageProvider`
9. `WorkflowProvider`
10. `SnapshotProvider`
11. `LegalRankingProvider`

Aliases historicos:

- `SourceFetcher = FetchProvider`
- `EvidenceExtractor = ExtractionProvider`

`provider-policy.md`, `provider-interfaces.md` y `architecture-overview.md` deben conservar esta misma lista canonica.

## Semantica de `restricted`

Cuando `data-classification.yaml` marca una clase como `restricted` para una familia de provider, significa simultaneamente:

- payload minimizado al fragmento, query o ref estrictamente necesario;
- redaccion aplicada cuando la clase pueda contener datos de cliente no necesarios para la tarea;
- `ProviderCallAudit` obligatorio;
- feature flag activo y kill switch inactivo;
- `training_use_allowed=false`;
- provider declarado en `provider-registry.yaml`;
- clase incluida en `data_received_classes` o `data_returned_classes` del provider resuelto, segun corresponda.

## Registry

Todo provider permitido en v0.10 debe declararse en `docs/schemas/provider-registry.yaml` con:

- `provider_name`
- `provider_family`
- `external_call_mode` (`always|never|deployment_configured`)
- `feature_flag`
- `kill_switch`
- `timeout_ms`
- `retry_policy`
- `error_mapping`
- `cost_tracking`
- `data_received_classes`
- `data_returned_classes`
- `stores_customer_data`
- `training_use_allowed`
- `region_or_residency`
- `subprocessors`
- `operational_logs`

## Reglas obligatorias

1. El core legal no importa SDKs de proveedores directamente.
2. Toda llamada externa genera `ProviderCallAudit`.
3. Ningun provider recibe clases no declaradas en su allowlist.
4. Ningun provider puede declarar una clase prohibida por `data-classification.yaml`.
5. `training_use_allowed=true` queda prohibido en v0.10.
6. Todo provider tiene `feature_flag` y `kill_switch`.
7. Todo provider tiene timeout, retry policy y error mapping cerrado.
8. Todo provider registra costos o declara `cost_tracking=false` de forma explicita.
9. Todo provider declara si almacena datos de cliente.

## Reglas por familia

| Familia | Datos permitidos por defecto | Restriccion |
|---|---|---|
| `ModelProvider` | Fragmentos minimos, queries clasificadas, evidencia delimitada | No recibe documentos completos por defecto |
| `SearchDiscoveryProvider` | `PUBLIC_LEGAL_QUERY`, `DERIVED_LEGAL_QUERY_RESTRICTED` si esta autorizado | No recibe documentos privados |
| `OfficialSourceAdapter` | Query juridica y refs publicas | No recibe documentos privados |
| `FetchProvider` | Fuentes publicas o refs autorizadas | Trata HTML como no confiable |
| `ExtractionProvider` | Fuente publica o documento autorizado | No ejecuta instrucciones en contenido |
| `OCRProvider` | Documento privado autorizado | Local/deterministico preferido |
| `EmbeddingProvider` | Chunks clasificados y minimizados | No recibe documento completo por defecto |
| `StorageProvider` | Raw documents bajo ownership | Rutas privadas no adivinables |
| `WorkflowProvider` | IDs, refs y payload minimo | No recibe contenido confidencial salvo allowlist |
| `SnapshotProvider` | Fuente publica | No snapshot de documento privado salvo politica cerrada |
| `LegalRankingProvider` | Resultados y evidencia minimizada | No decide veracidad final |

## Reglas deterministicas

1. `ProviderCallAudit.provider_name` debe resolver a un provider del registry.
2. `ProviderCallAudit.provider_family` debe coincidir con el registry.
3. `ProviderCallAudit.data_sent_classes[]` debe ser subset de `data_received_classes[]` del provider.
4. `ProviderCallAudit.data_returned_classes[]` describe el payload devuelto por el provider, no solo metadata operacional; debe heredar la clase de mayor sensibilidad del contenido devuelto y ser subset de `data_returned_classes[]` del provider.
5. `ProviderCallAudit.status=policy_blocked` significa que no se envio payload al provider.
6. Para `status=success|error|timeout|rate_limited|cancelled`, `attempted_data_classes[]` y `data_sent_classes[]` deben representar el mismo set de clases.
7. `attempted_data_classes[]` puede violar allowlist solo cuando `status=policy_blocked`; nunca puede contener clases desconocidas.
8. `external_call=true` exige `region_or_residency`, `feature_flag` y `training_use_allowed=false`.
9. `policy_decision=blocked_by_feature_flag` exige `feature_flag`.
10. `policy_decision=blocked_by_kill_switch` exige `kill_switch`.
11. `ProviderCallAudit.status=timeout` exige `error_code=timeout`; `rate_limited` exige `error_code=rate_limited`; `cancelled` exige `cancelled_by_user|cancelled_by_system`; `error` exige `provider_error|unmapped_error`.
12. `ModelCall.provider = openai` representa proveedor externo en v0.10; debe usar `external_provider_call=true` y `provider_call_audit_id` no nulo.
13. `external_call_mode=always` resuelve `ProviderMetadata.external_call=true`; `never` resuelve `false`; `deployment_configured` exige una decision booleana explicita de deployment antes de habilitar el provider. No existe default implicito.
14. Si el valor resuelto de `external_call` es `true`, cada intento externo crea o actualiza un `ProviderCallAudit` y ninguna llamada sale sin audit ref.
15. Cada llamada logica usa un `logical_call_id`; cada intento conserva el mismo `organization_id`, `request_id` y `correlation_id`, y usa un `attempt_number` unico y creciente desde `1`.
16. `attempt_kind=initial` exige `attempt_number=1` y `fallback_route_id=null`; `retry` exige `attempt_number>=2`; `fallback` exige `attempt_number>=2` y un `fallback_route_id=proute_*` resoluble.
17. Todo `retry|fallback` exige `operation_idempotent=true` o `idempotency_key_hash=sha256:*`; nunca se persiste la idempotency key raw.
18. La pareja `(logical_call_id, attempt_number)` es unica. En `ModelCall` o `ToolCall`, el `provider_call_audit_id` singular apunta al intento terminal; los intentos previos se resuelven por `model_call_id|tool_call_id` y `logical_call_id`. Para retrieval, todos los intentos se resuelven por `retrieval_run_id` y `logical_call_id`.

## Provider Reliability Layer

- `retry_policy.max_attempts` cuenta el intento inicial. `max_attempts=2` permite como maximo un retry.
- Un retry solo se ejecuta para una operacion idempotente o protegida por idempotency key y para un codigo listado en `retry_on`.
- Errores de validacion, clasificacion, feature flag, kill switch, autorizacion o policy no se reintentan.
- `RetrievalPlan.fallback_allowed=true` solo concede permiso; no selecciona provider ni demuestra que exista alternativa.
- El fallback solo se permite cuando `provider-registry.yaml` contiene un `ProviderRoute` activo compatible en familia, tenant, clases permitidas y budget. El provider primario y el alternativo deben ser distintos y resolver en el registry.
- El registry congelado mantiene `fallback_routes: []`; por tanto no existe fallback productivo activo en Fase 1. El camino positivo se prueba solo con un fixture local cerrado de dos stubs y el camino productivo sin ruta falla con `provider_unavailable`.
- Si no existe fallback explicito o este viola policy/budget, la PRL devuelve `provider_unavailable` mediante `ErrorEnvelope`; no selecciona un provider por heuristica silenciosa.
- Cada intento crea un `ProviderCallAudit` separado con `ProviderCallContext` completo; retries y fallback no pueden duplicar usage ni cargos.

## Comportamiento ante incumplimiento

- Provider desconocido: bloquear llamada y registrar policy-invalid en validadores.
- Clase prohibida: `ProviderCallAudit.status=policy_blocked`, `policy_decision=blocked_by_classification`, `data_sent_classes=[]`.
- Feature flag apagado: `policy_decision=blocked_by_feature_flag`.
- Kill switch activo: `policy_decision=blocked_by_kill_switch`.
- Error tecnico del provider: `status=error|timeout|rate_limited|cancelled`, con `policy_decision=allowed|redacted`.
- Retry agotado o fallback no permitido: `ErrorEnvelope.error_code=provider_unavailable`, conservando la causa tecnica solo en auditoria interna segura.

## Relacion con contratos

- `provider-call-audit.schema.json`
- `data-classification.yaml`
- `provider-registry.yaml`
- `model-call.schema.json`
- `tool-call.schema.json`
- `retrieval-run.schema.json`

## Criterios de aceptacion

- Las 11 familias aparecen en policy, interfaces y arquitectura.
- Todo provider tiene registry cerrado.
- `external_call` se resuelve de forma determinista para cada deployment.
- Retry seguro, fallback explicito mediante fixture cerrado y ausencia de ruta productiva se prueban sin duplicar usage ni cargos; cada intento conserva auditoria propia y la llamada logica mantiene una sola contabilizacion.
- Toda llamada externa puede auditar clases enviadas/devueltas sin payload raw.
- El core legal queda desacoplado de SDKs concretos.

## Momento de revision

Antes de implementar cualquier provider externo, storage productivo, web search, OCR externo, embeddings o workflow durable.
