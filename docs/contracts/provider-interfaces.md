# Provider Interfaces - Live Legal Search

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Subfase 0.5 - Live Legal Search Engine Contract

## Proposito

Definir las interfaces documentales que Fase 4 implementara para discovery, adaptadores oficiales, fetch, extraccion, snapshots y ranking. El core no debe importar SDKs concretos de proveedores.

## Reglas

1. Todo provider tiene `name`, timeout, error mapping y feature flag.
2. Ningun provider de discovery es obligatorio.
3. OpenAI Web Search se modela como discovery opcional con `web_search`, no como fuente final ni como cita final.
4. JusNova debe hacer fetch, extraction y snapshot propio antes de construir Evidence Pack.
5. Los presupuestos usados aqui son `SearchBudget` o `RetrievalBudget`; el presupuesto comercial del Cost Governor se define en Subfase 0.8.

## Interfaces

```python
class SearchBudget:
    discovery_calls_max: int
    source_fetches_max: int
    official_adapter_calls_max: int
    max_results_per_provider: int
    tool_rounds_max: int
    timeout_ms: int
```

```python
class SearchDiscoveryProvider:
    name: str

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
class OfficialSourceAdapter:
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
class SourceFetcher:
    def fetch(
        self,
        url: str,
        policy: "FetchPolicy",
    ) -> "FetchedSource":
        ...
```

```python
class EvidenceExtractor:
    def extract(
        self,
        fetched_source: "FetchedSource",
        query: "LegalSearchQuery",
    ) -> list["EvidencePassage"]:
        ...
```

```python
class SnapshotProvider:
    def snapshot(
        self,
        fetched_source: "FetchedSource",
    ) -> "SourceSnapshot":
        ...
```

```python
class LegalRankingProvider:
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

## Criterios de aceptacion

- El core puede cambiar o apagar providers sin cambiar contratos internos.
- Ningun SDK externo aparece como dependencia directa del core.
- Las citas finales no provienen del provider de discovery; provienen de pasajes extraidos y auditables.
- Los errores de provider se registran como parte de `RetrievalRun`.
