# Provider Interfaces - Live Legal Search

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.5 - Live Legal Search Engine Contract; Subfase 0.10 - Security, Privacy and Provider Boundaries
**Enmiendas:** Subfase 0.10 - provider registry, provider audit and canonical provider families; Subfase 0.14 - provider route, per-attempt context, phase ownership and SSRF-safe fetch boundary closure

## Proposito

Definir interfaces documentales provider-neutral cuya implementacion se reparte por el roadmap aceptado. Fase 1 implementa foundations/stubs de `ModelProvider`, `StorageProvider` y `WorkflowProvider`/`WorkflowGateway`; Fase 4 implementa discovery, fetch, extraction, captura basica de snapshots asociada a retrieval vivo y ranking; los adaptadores oficiales se amplian en Fase 5; Evidence Cache y Source Snapshot Registry productivos permanecen en Fase 8; OCR productivo permanece en Fase 9. Embeddings y demas integraciones productivas se activan solo en la fase o ADR que las autorice. El core no debe importar SDKs concretos de proveedores.

`AuthProvider` no pertenece a estas 11 familias ni a `provider-registry.yaml`: es un boundary de seguridad separado definido por el handoff P1-09. No se debe agregar al enum de `ProviderCallAudit` ni activar llamadas de autenticacion externas por request sin una decision posterior que cierre su auditoria.

## Reglas

1. Todo provider tiene `name`, timeout, error mapping y feature flag.
2. Ningun provider de discovery es obligatorio.
3. OpenAI Web Search se modela como discovery opcional con `web_search`, no como fuente final ni como cita final.
4. JusNova debe hacer fetch, extraction y snapshot propio antes de construir Evidence Pack.
5. Los presupuestos usados aqui son `SearchBudget` o `RetrievalBudget`; el presupuesto comercial del Cost Governor se define en Subfase 0.8.
6. Toda llamada externa a provider debe crear `ProviderCallAudit`.
7. Todo provider debe estar declarado en `provider-registry.yaml`, con `training_use_allowed = false`.
8. Ningun provider puede recibir clases de datos fuera de su allowlist y de `data-classification.yaml`.
9. `SourceFetcher` es alias historico de `FetchProvider`; `EvidenceExtractor` es alias historico de `ExtractionProvider`.

## Interfaces

Los comentarios `^...$` de los DTOs siguientes son notacion abreviada. La implementacion usa validacion de valor completo (`fullmatch`) con el cierre real definido en `docs/contracts/README.md`, sin trim/coercion previa ni terminadores de linea terminales.

```python
class ProviderMetadata:
    name: str
    provider_family: str
    enabled_feature_flag: str
    kill_switch: str
    default_timeout_ms: int
    retry_policy: "RetryPolicy"
    error_mapping: dict[str, "ProviderAuditErrorCode"]
    data_received_classes: list[str]
    data_returned_classes: list[str]
    external_call: bool
    training_use_allowed: bool
    region_or_residency: str
```

```python
class ProviderRoute:
    route_id: str  # ^proute_[A-Za-z0-9_-]+$
    provider_family: str
    primary_provider_name: str
    fallback_provider_name: str
    organization_id: str | None  # null = ruta global
    allowed_data_classes: list[str]
    allowed_cost_budget_refs: list[str]  # cb_*
    enabled: bool
```

```python
class ProviderCallContext:
    logical_call_id: str  # ^pcall_[A-Za-z0-9_-]+$
    request_id: str  # ^rq_[A-Za-z0-9_-]+$
    correlation_id: str  # ^corr_[A-Za-z0-9_-]+$
    attempt_number: int  # inicia en 1
    attempt_kind: str  # initial | retry | fallback
    operation_idempotent: bool
    idempotency_key_hash: str | None  # sha256:*; nunca key raw
    fallback_route_id: str | None  # proute_* solo para attempt_kind=fallback
```

```python
class BaseProvider:
    metadata: ProviderMetadata
```

`ProviderMetadata.external_call` no se configura de forma ad hoc. Se resuelve desde `provider-registry.yaml`: `always -> true`, `never -> false` y `deployment_configured` exige un booleano explicito del deployment antes de habilitar el provider. La PRL interpreta `retry_policy.max_attempts` como intentos totales incluido el inicial y solo reintenta operaciones idempotentes o protegidas por `idempotency_key_hash`; cualquier fallback requiere un `ProviderRoute` activo, nunca seleccion automatica silenciosa.

`ProviderAuditErrorCode` es el enum interno aceptado por `ProviderCallAudit.error_code`. Los valores actuales de `error_mapping` son `timeout|rate_limited|provider_error|cancelled_by_system`; no son `ErrorEnvelope.error_code` publicos. La PRL realiza el mapping de frontera definido por `provider-policy.md`, y `ModelCall`/`ToolCall` usan sus propios enums contractuales sin copiar un codigo incompatible.

`ProviderRoute` y `ProviderCallContext` son DTOs internos cerrados, no payloads publicos. Los nombres de provider deben resolver en el registry activo, ser distintos y pertenecer a la misma familia; `organization_id`, clases permitidas y `allowed_cost_budget_refs` deben ser compatibles con la ejecucion. El registry aceptado mantiene `fallback_routes: []`: por tanto, `RetrievalPlan.fallback_allowed=true` expresa permiso, no disponibilidad, y la PRL productiva falla con `provider_unavailable` mientras no exista una ruta aceptada. Fase 1 solo prueba el camino positivo con un registry fixture completo, cerrado y exclusivo de tests que agrega dos providers stub de la misma familia y una `ProviderRoute` entre ellos; ambos nombres resuelven dentro de ese fixture, que nunca modifica, extiende ni sustituye `docs/schemas/provider-registry.yaml` en runtime.

## Familias canonicas

La lista canonica debe coincidir exactamente con `provider-policy.md`, `provider-registry.yaml` y `architecture-overview.md`.

- `ModelProvider`
- `SearchDiscoveryProvider`
- `OfficialSourceAdapter`
- `FetchProvider`
- `ExtractionProvider`
- `OCRProvider`
- `EmbeddingProvider`
- `StorageProvider`
- `WorkflowProvider`
- `SnapshotProvider`
- `LegalRankingProvider`

```python
class SearchBudget:
    discovery_calls_max: int
    source_fetches_max: int
    official_adapter_calls_max: int
    max_results_per_provider: int
    tool_rounds_max: int
    timeout_ms: int
```

`FetchPolicy` es un DTO interno cerrado, no un JSON Schema publico ni una taxonomia extensible por usuario o modelo:

```python
class FetchPolicy:
    allowed_schemes: tuple[str, ...] = ("https",)
    allowed_ports: tuple[int, ...] = (443,)
    max_redirects: int = 3
    connect_timeout_ms: int = 5_000
    read_timeout_ms: int = 20_000
    max_decompressed_bytes: int = 52_428_800
    allowed_media_types: tuple[str, ...] = (
        "text/html",
        "text/plain",
        "application/pdf",
        "application/xhtml+xml",
        "application/xml",
        "text/xml",
        "application/json",
    )
    reject_userinfo: bool = True
    reject_ip_literals: bool = True
    require_globally_routable_dns: bool = True
    use_environment_proxy: bool = False
    follow_redirects_automatically: bool = False
```

Antes de abrir un socket, `FetchProvider` valida la URL inicial y cada redirect contra la misma politica versionada. En v0 solo se permite HTTPS por puerto 443, hostname DNS sin userinfo ni literal IP, y todas las direcciones resueltas deben ser globalmente routables. Se rechazan IPv4 e IPv6 loopback, privadas, link-local, multicast, unspecified y reservadas. El cliente fija para la conexion una direccion ya validada, conservando hostname y SNI, para evitar DNS rebinding; no vuelve a resolver por una ruta no validada.

El cliente desactiva proxies heredados del ambiente y redirects automaticos: cada salto se obtiene y autoriza de forma explicita antes de la siguiente conexion, no puede degradar el esquema y el total no supera tres. Un proxy o egress gateway futuro solo puede habilitarse mediante decision aceptada y debe aplicar la misma validacion de destino, DNS y redirects en el punto que abre la conexion final. Timeouts, tamano maximo descomprimido y media types permitidos se aplican antes de extraction o snapshot. Ampliar esquemas, puertos, destinos o limites requiere una decision aceptada; ningun prompt, resultado de discovery ni salida de modelo puede modificarlos.

```python
class SearchDiscoveryProvider(BaseProvider):
    def search(
        self,
        query: str,
        allowed_domains: list[str],
        blocked_domains: list[str],
        max_results: int,
        timeout_ms: int,
    ) -> list["LegalSearchResult"]:
        ...
```

```python
class ModelProvider(BaseProvider):
    def generate(
        self,
        prompt_version: str,
        bounded_context_refs: list[str],
        timeout_ms: int,
    ) -> "ModelProviderResult":
        ...
```

```python
class OfficialSourceAdapter(BaseProvider):
    source_name: str
    supported_entity_types: list[str]

    def resolve(
        self,
        entity: "LegalEntity",
        budget: "SearchBudget",
    ) -> list["LegalSearchResult"]:
        ...

    def search(
        self,
        query: "LegalSearchQuery",
    ) -> list["LegalSearchResult"]:
        ...
```

```python
class FetchProvider(BaseProvider):
    def fetch(
        self,
        url: str,
        policy: "FetchPolicy",
    ) -> "FetchedSource":
        ...
```

```python
class ExtractionProvider(BaseProvider):
    def extract(
        self,
        fetched_source: "FetchedSource",
        query: "LegalSearchQuery",
    ) -> list["EvidencePassage"]:
        ...
```

```python
class OCRProvider(BaseProvider):
    def extract_text(
        self,
        document_ref: str,
        page: int,
    ) -> list["DocumentEvidence"]:
        ...
```

```python
class EmbeddingProvider(BaseProvider):
    def embed(
        self,
        fragment_refs: list[str],
    ) -> "EmbeddingBatchResult":
        ...
```

```python
class StorageProvider(BaseProvider):
    def put_private_object(
        self,
        object_ref: str,
        classification: str,
    ) -> "StoredObjectRef":
        ...
```

```python
class WorkflowProvider(BaseProvider):
    def enqueue(
        self,
        workflow_name: str,
        payload_ref: str,
    ) -> "WorkflowRunRef":
        ...
```

```python
class SnapshotProvider(BaseProvider):
    def snapshot(
        self,
        fetched_source: "FetchedSource",
    ) -> "SourceSnapshot":
        ...
```

```python
class LegalRankingProvider(BaseProvider):
    def rank(
        self,
        results: list["LegalSearchResult"],
        query: "LegalSearchQuery",
    ) -> list["LegalSearchResult"]:
        ...
```

## Adaptadores definidos para Fase 4/5

| Tipo | Nombre | Estado 0.5 |
|---|---|---|
| OfficialSourceAdapter | GacetaAdapter | Especificado, no implementado |
| OfficialSourceAdapter | TCPAdapter | Especificado, no implementado |
| OfficialSourceAdapter | TSJGenesisAdapter | Especificado, no implementado |
| OfficialSourceAdapter | SILEPAdapterCandidate | Candidato, no obligatorio |
| OfficialSourceAdapter | OfficialDomainAdapter | Especificado, no implementado |
| SearchDiscoveryProvider | OpenAIWebSearchDiscoveryProvider | Opcional |
| SearchDiscoveryProvider | BraveSearchDiscoveryProviderCandidate | Opcional |
| SearchDiscoveryProvider | BingSearchDiscoveryProviderCandidate | Opcional |
| SearchDiscoveryProvider | GoogleCSEDiscoveryProviderCandidate | Opcional |
| SearchDiscoveryProvider | SerpAPIDiscoveryProviderCandidate | Opcional |
| SearchDiscoveryProvider | ManualCuratedRoutesProvider | Opcional |

## Alias historicos

Texto canonico para validadores documentales: SourceFetcher = FetchProvider; EvidenceExtractor = ExtractionProvider.

| Alias historico | Familia canonica |
|---|---|
| `SourceFetcher` | `FetchProvider` |
| `EvidenceExtractor` | `ExtractionProvider` |

## Criterios de aceptacion

- El core puede cambiar o apagar providers sin cambiar contratos internos.
- Ningun SDK externo aparece como dependencia directa del core.
- Las citas finales no provienen del provider de discovery; provienen de pasajes extraidos y auditables.
- Los errores de provider se registran como parte de `RetrievalRun`.
- Las llamadas externas se concilian con `ProviderCallAudit`.
- La familia y el nombre de provider de cada llamada resuelven contra `provider-registry.yaml`.
- Una URL bloqueada por `FetchPolicy` nunca alcanza la red, queda rechazada con `reason=access_not_allowed` y solo expone referencias sanitizadas en logs y trazas.
