# Source Routing Matrix

**Estado documental:** Accepted
**Fecha:** 2026-05-22
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.5 - Source Routing para Live Legal Search

## Proposito

Definir rutas primarias, secundarias y fallback para los 11 intents canonicos de `docs/schemas/legal-intents.yaml`.

## Matriz

| Intent | Ruta primaria | Ruta secundaria | Fallback | Notas |
|---|---|---|---|---|
| NORMATIVA | Gaceta / SILEP validado / dominios oficiales | entidades sectoriales oficiales | TIER2 con advertencia | Vigencia no se presume. |
| JURISPRUDENCIA | TSJ/Genesis / TCP | discovery restringido a dominios oficiales | TIER2 con advertencia | Intentar tribunal, sala, resolucion y fecha. |
| PROCEDIMIENTO | norma + jurisprudencia | guias oficiales | fuentes secundarias solo contexto | Plazos y requisitos requieren cita. |
| ESTRATEGIA | documentos + norma + jurisprudencia | doctrina secundaria etiquetada | advertencia de inferencia | Separar evidencia de recomendacion. |
| DOC_ANALYSIS | documentos del usuario | busqueda externa si pregunta derecho | abstencion si documento no contiene base | Documento usuario no es norma vigente. |
| REDACCION | norma/documento segun caso | plantillas futuras internas | advertencia de revision humana | Fundamento separado del texto redactado. |
| SECTORIAL | fuente oficial de entidad sectorial | source pack sectorial | TIER2 con advertencia | Identificar emisor y fecha. |
| SUBNACIONAL | gaceta o portal municipal/departamental oficial | source pack subnacional | TIER2 con advertencia fuerte | Advertir cobertura local limitada. |
| VIGENCIA | fuente canonica + fuente modificatoria/derogatoria | fuente estructurada oficial validada | Modo Investigacion o abstencion | No decir vigente sin evidencia explicita. |
| MIXTO | documentos + norma + jurisprudencia | discovery controlado por subpreguntas | respuesta parcial o investigacion | Separar hechos, documentos y derecho. |
| META | no crea LegalSearchQuery si no requiere busqueda viva | documentacion interna del producto | respuesta sin Live Search | `META` existe como intent canonico, no como busqueda obligatoria. |

## Source targets canonicos

| Target | Uso |
|---|---|
| GACETA | normativa canonica nacional |
| SILEP_CANDIDATE | fuente estructurada candidata, sujeta a validacion |
| TSJ_GENESIS | jurisprudencia TSJ |
| TCP | jurisprudencia constitucional |
| OFFICIAL_DOMAIN | portales oficiales bolivianos |
| SECTOR_PACK | entidades publicas sectoriales |
| SUBNATIONAL_OFFICIAL | gobernaciones, municipios y autonomias |
| USER_DOCUMENT | documentos del usuario |
| OPEN_WEB_DISCOVERY | discovery multi-proveedor controlado |
| TIER2_FALLBACK | fallback confiable con advertencia |

## Validacion

- Cada intent de `legal-intents.yaml` debe aparecer exactamente una vez en esta matriz.
- `META` no genera `LegalSearchQuery` si no requiere busqueda viva.
- Ninguna ruta puede convertir una fuente secundaria en fuente primaria sin advertencia.
