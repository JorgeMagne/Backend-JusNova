# ADR-006 - Document OCR Policy

**Estado:** Accepted
**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** OCR, documentos y PDFs escaneados

## Contexto

Bolivia contiene fuentes juridicas publicas y documentos privados en PDF digital, PDF escaneado, imagenes, tablas y texto OCR deficiente. JusNova debe trabajar con documentos sin tratar texto no confiable como instruccion.

## Problema

Usar LLM como OCR primario es costoso, menos deterministico y riesgoso para privacidad. Procesar todo inline puede romper presupuestos y latencia.

## Restricciones

- OCR debe respetar budgets.
- Documentos privados requieren ownership y permisos.
- PDFs publicos usados como fuente requieren snapshot y hash.
- El contenido documental no puede modificar instrucciones del sistema.
- Las citas documentales usan `[D#:P#]`.

## Opciones consideradas

1. Enviar documentos completos a vision como OCR primario.
2. Ignorar PDFs escaneados.
3. OCR local/deterministico primario con vision solo como escalacion selectiva.

## Decision

JusNova usara OCR primario local/deterministico. Vision con modelo queda permitida solo como escalacion selectiva para paginas criticas o baja confianza, con presupuesto, trazabilidad y minimizacion.

## Justificacion

El OCR local reduce exposicion de documentos sensibles, mantiene costos mas predecibles y permite registrar confianza por pagina o pasaje. La vision queda como herramienta de escalacion porque puede mejorar paginas criticas, sellos, tablas o baja calidad visual, pero no debe convertirse en OCR primario ni recibir documentos completos sin justificacion.

## Pipeline aprobado

- Validacion de archivo.
- Hash y version documental.
- Deteccion de texto nativo.
- Deteccion de pagina escaneada.
- OCR por pagina cuando aplique.
- Extraccion de locators.
- Chunking con pagina/coordenadas cuando sea posible.
- Indexacion posterior.
- Citas `[D#:P#]`.

## PDFs publicos legales vs documentos privados

- PDF publico legal: se trata como fuente externa, requiere URL, snapshot, hash, source tier y fecha de recuperacion.
- Documento privado del usuario: se trata como evidencia documental privada, requiere tenant, permisos, version y retencion.

## Dependencias posteriores

- Subfase 0.9 crea `document-evidence.schema.json`, `ocr-policy.md` y una `document-security-policy.md` minima para ownership, hashes, referencias internas y minimizacion.
- Subfase 0.10 acepta privacy/security policy, provider boundaries, provider registry, raw access audit, prompt injection policy y clasificacion de datos. Permisos runtime, retencion automatizada e incident process productivo quedan para Fase 1 y fases posteriores.
- Fase 9 implementara Document Evidence Search y OCR progresivo.

## No afirma todavia

- No afirma que OCR completo sea inline.
- No afirma que vision pueda recibir documentos completos.
- No afirma que OCR de baja confianza sostenga claims de alta confianza.

## Riesgos

- OCR de baja calidad puede inducir errores juridicos.
- Documentos privados pueden filtrarse si no hay ownership.
- PDFs grandes pueden romper presupuesto.
- Texto malicioso puede intentar prompt injection.

## Mitigaciones

- Confidence por pagina/pasaje.
- Warnings cuando OCR sostiene evidencia decisiva.
- Jobs async para documentos extensos.
- Delimitacion de documentos como datos no confiables.
- Tenant isolation y log minimization.

## Criterios de aceptacion

- Subfase 0.9 acepta contratos base de conversacion, mensajes, memoria, evidencia documental y OCR.
- Subfase 0.10 agrega `document_evidence_id`, clasificacion de evidencia documental, raw access y riesgos de prompt injection compartidos.
- No se usa LLM como OCR primario.
- No se envian documentos completos a vision salvo caso justificado y trazado.
- Todo documento procesado conserva hash, version y localizadores.
- Todo fragmento citado conserva metodo de extraccion y confianza.

## Momento de revision

Revisar al implementar Fase 9, antes de beta, y cuando `ocr_extraction_quality`, `document_citation_correctness` o costo por pagina OCR queden fuera de umbral.

## Consecuencias

OCR y documentos se disenan como pipeline controlado, no como carga directa al prompt.
