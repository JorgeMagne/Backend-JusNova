# Legal Search Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.5 - Live Legal Search Engine Contract

## Proposito

Gobernar como el JusNova Live Legal Search Engine convierte una consulta juridica en evidencia recuperada, extraida, normalizada, rankeada y apta para Evidence Pack.

## Alcance

Aplica a `LegalSearchQuery`, `RetrievalPlan`, `RetrievalRun`, `LegalSearchResult`, `EvidenceQuality`, providers, adaptadores oficiales, fetchers, extractors y snapshots.

## Reglas obligatorias

1. Discovery no es evidencia.
2. Un resultado web crudo no es citable.
3. Para citar, el resultado debe tener pasaje extraido.
4. El snapshot es obligatorio salvo imposibilidad documentada con `snapshot_unavailable_reason`.
5. Si no hay pasaje extraido, no hay cita.
6. OpenAI Web Search es provider opcional con `web_search`, no `web_search_preview`, y no es fuente final.
7. JusNova no trata citas del modelo como citas finales.
8. JusNova debe hacer fetch, extract y snapshot propio antes de construir Evidence Pack.
9. La obligatoriedad de cualquier provider queda prohibida; esto incluye OpenAI Web Search, Brave, Bing, Google CSE y SerpAPI.
10. Fuente `TIER3_SECUNDARIO` no puede ser soporte principal de claim normativo critico.
11. Vigencia no se presume.
12. Cache o snapshot no confirman vigencia por si mismos.
13. Derecho extranjero se descarta salvo comparacion solicitada.
14. Si una fuente oficial cae, el fallback se etiqueta con advertencia.
15. Si no hay evidencia suficiente, el sistema debe pedir datos, responder parcialmente con advertencia, proponer Modo Investigacion o abstenerse.
16. Dentro de una `LegalSearchQuery`, cada `entities[].entity_id` debe ser unico.
17. Dentro de un `RetrievalRun`, cada `discovery_results[].result_id` debe ser unico; un ID repetido invalida el run y no puede incrementar precision, recall ni cobertura.
18. Dentro de `RetrievalRun.sources_rejected[]`, cada `url_hash` canonico puede aparecer una sola vez y debe conservar una unica razon final de rechazo. Variantes equivalentes se consolidan antes de persistir el run.
19. `RetrievalPlan.query_id` resuelve una `LegalSearchQuery` del mismo tenant. `RetrievalRun` resuelve ese plan y esa misma query: `RetrievalRun.query_id == RetrievalPlan.query_id` y los tres `organization_id` coinciden.
20. `RetrievalRun -> EvidencePack` es `0..1`. Si `evidence_pack_id` existe, resuelve un pack del mismo tenant/query con `EvidencePack.retrieval_run_id == RetrievalRun.retrieval_run_id`; un pack con run no nulo debe estar enlazado de vuelta por ese run.
21. `RetrievalRun.sources_rejected[].reason` y `TraceObject.sources_rejected[].reason` comparten el mismo enum cerrado. `unsupported_for_critical_claim` identifica una fuente recuperada y tematicamente relevante que no puede soportar el claim critico evaluado; no se reclasifica como `irrelevant`.
22. Una URL descubierta es solo candidata. `FetchProvider` debe aplicar el `FetchPolicy` cerrado de `provider-interfaces.md` antes de cualquier acceso de red.
23. En v0 solo se permite HTTPS por puerto 443, con hostname DNS, sin userinfo ni literal IP. Cualquier ampliacion exige una decision aceptada.
24. La URL inicial y cada redirect deben resolver exclusivamente a direcciones globalmente routables. Se rechazan rangos IPv4 e IPv6 loopback, privados, link-local, multicast, unspecified y reservados; la conexion fija una direccion validada conservando hostname y SNI para impedir DNS rebinding.
25. Redirects automaticos y proxies heredados del ambiente permanecen desactivados. Cada redirect se autoriza de nuevo antes de seguirlo, no puede degradar el esquema y el flujo admite como maximo tres redirects. Headless rendering y adaptadores oficiales usan el mismo guard de egreso; un egress gateway futuro requiere decision aceptada y enforcement equivalente en la conexion final.
26. Cada fetch aplica `connect_timeout_ms=5000`, `read_timeout_ms=20000`, maximo descomprimido de 50 MiB y allowlist de media types: `text/html`, `text/plain`, `application/pdf`, `application/xhtml+xml`, `application/xml`, `text/xml` y `application/json`. El consumo total tambien debe permanecer dentro del `CostBudget` efectivo.
27. Una URL insegura se registra en `sources_rejected[]` con `reason=access_not_allowed` y su `url_hash` canonico, omitiendo la URL cruda; no produce `EvidencePack` ni snapshot y no se confunde con un fallo de snapshot de un host publico permitido. Para otras razones, `url` solo puede conservarse despues de que `FetchPolicy` autorice la URL publica.
28. `RetrievalRun.adapter_calls[].error_code` y `error.error_code` son codigos internos seguros; `error.message` y `warnings[]` son textos sanitizados de una linea, sin controles C0/C1 ni separadores Unicode, y hasta 240 caracteres. Ninguno interpola query, URL, snippet, documento, HTML, OCR, excepcion, stack trace, status line ni respuesta raw del provider.
29. `LegalSearchResult.snippet`, si existe, es contenido no confiable normalizado a texto plano de una sola linea, entre 1 y 1000 caracteres, sin controles C0/C1 ni separadores Unicode. Se elimina HTML, se colapsa whitespace y nunca se persiste la respuesta raw del provider en ese campo; `warnings[]` sigue el perfil sanitizado de 1 a 240 caracteres sin duplicados.

## Reglas deterministicas

1. Jurisdiccion, tier, source type, snapshot, extractabilidad, budget y acceso permitido se validan antes de incorporar una fuente al `EvidencePack`.
2. Las reglas duras de ranking, la existencia de pasaje extraido y los gates de vigencia/fuente no pueden ser anulados por score probabilistico ni por salida de modelo.
3. Discovery, fetch, extraction, snapshot y normalizacion conservan refs auditables; un resultado web crudo nunca se convierte directamente en cita.
4. Si un gate obligatorio falla, el flujo descarta, advierte, responde parcialmente o se abstiene conforme a policy; no inventa evidencia ni vigencia.
5. El validador relacional rechaza `entity_id` o `result_id` duplicados aunque los objetos no sean byte a byte iguales.
6. Antes de descartar o conservar una URL, runtime calcula `sources_rejected[].url_hash` conforme a `docs/quality/initial-golden-dataset-spec.md#canonicalizacion-de-url-para-hashes`; runtime y eval runner comparten la misma implementacion/version, verifican unicidad por hash y no deduplican silenciosamente durante scoring.
7. Antes de ejecutar o puntuar un run, resolver `LegalSearchQuery`, `RetrievalPlan` y las refs opcionales de `EvidencePack`; rechazar cualquier mismatch de tenant, query o enlace reciproco.
8. La canonicalizacion de URL sirve para identidad, hashing y deduplicacion; no autoriza acceso. Runtime, tests y adaptadores comparten una sola version de `FetchPolicy` y del guard de egreso.
9. El adaptador traduce errores de red/provider a codigos internos antes de construir `RetrievalRun`; los tests de contrato rechazan codigo libre, texto multilinea y cualquier mensaje generado mediante copia directa de la excepcion o respuesta externa.
10. El adaptador normaliza `LegalSearchResult.snippet` antes de validar el contrato y trata su contenido como no confiable para prompt injection; un snippet multilinea, sobredimensionado, con HTML, controles o separadores Unicode no se persiste como resultado normalizado.

## Reglas asistidas por IA

1. Un modelo puede ayudar a reformular queries, agrupar resultados o estimar relevancia, siempre dentro del `RetrievalPlan`, budget y rutas permitidas.
2. Una estimacion asistida no decide autoridad, vigencia, acceso legal, tier final ni aptitud de una fuente como soporte critico.
3. Cualquier reranking asistido se somete despues a los filtros deterministas y conserva los scores/refs necesarios para auditoria.
4. Las citas propuestas por un modelo no son citas finales y deben resolver a `EvidencePassage` validado.

## Legalidad de acceso

Permitido:

- Consultar portales publicos.
- Usar APIs publicas o endpoints publicos.
- Usar discovery comercial autorizado.
- Fetch de documentos publicos.
- Headless rendering conservador para paginas publicas.
- Reintentos con backoff y circuit breakers.

Prohibido:

- Evadir autenticacion.
- Romper captchas.
- Simular identidades enganosas.
- Extraer informacion no publica.
- Martillar portales degradados o caidos.
- Tratar fuente secundaria como oficial sin advertencia.
- Acceder mediante `file`, `ftp`, `gopher`, `data`, `javascript`, un esquema o puerto no aprobado, userinfo, literal IP, hostname no globalmente routable o redirect no validado.

## Ranking juridico inicial

```txt
final_score =
  0.35 * authority_score
+ 0.25 * relevance_score
+ 0.15 * jurisdiction_score
+ 0.10 * freshness_score
+ 0.10 * extractability_score
+ 0.05 * source_health_score
```

Reglas duras:

- Si `jurisdiction_score = 0` y no hay comparacion solicitada, descartar.
- Si `tier = TIER3_SECUNDARIO`, no puede ser fundamento principal.
- Si `source_type = norma` y `validity_status = VIGENCIA_NO_CONFIRMADA`, se puede usar solo con advertencia.
- Si `source_type = jurisprudencia`, se debe intentar identificar tribunal, sala, resolucion y fecha.
- Si no hay pasaje extraido, no puede ser cita.

## OpenAI Web Search

OpenAI Web Search puede usarse como `SearchDiscoveryProvider` opcional para encontrar URLs candidatas. Sus resultados y anotaciones no sustituyen los contratos de JusNova. Antes de citar cualquier fuente descubierta por ese provider, JusNova debe:

1. registrar la llamada en `RetrievalRun`;
2. normalizar la URL;
3. hacer fetch permitido;
4. extraer pasajes;
5. crear snapshot o registrar razon cerrada de imposibilidad;
6. clasificar tier, jurisdiccion y source type;
7. pasar por ranking y EvidenceQuality.

## Relacion con abstencion

La busqueda termina en abstencion o respuesta parcial si:

- no hay evidencia encontrada;
- el budget tecnico se agota;
- fuentes oficiales estan caidas y fallback es insuficiente;
- el conflicto no puede resolverse;
- falta informacion del usuario;
- el acceso no esta permitido;
- la extraccion falla sin pasajes citables.

## Criterios de aceptacion

- El motor tiene contrato de entrada y salida.
- Se distinguen adaptadores oficiales, discovery providers, fetchers, extractors, snapshots y ranking.
- El sistema sabe que hacer ante evidencia insuficiente.
- No hay dependencia obligatoria de un proveedor de discovery.
- La cita final nunca nace de resultado web crudo.
- Antes de habilitar fetch real, los tests cubren esquema y puerto, DNS IPv4/IPv6, loopback/privadas/link-local/reservadas, DNS rebinding, redirects, timeouts, MIME no permitido y limite descomprimido; un destino bloqueado no recibe conexion.
- Los errores y warnings persistidos por retrieval usan codigos internos y mensajes sanitizados; los snippets se normalizan y acotan como contenido no confiable, sin filtrar queries, URLs, documentos, excepciones ni payloads de provider.

## Momento de revision

Antes de implementar discovery, fetch, extraction, snapshot, ranking o cualquier provider de busqueda; tambien cuando cambien portales oficiales, reglas de acceso o taxonomias de fuentes.
