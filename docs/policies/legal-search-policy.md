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
