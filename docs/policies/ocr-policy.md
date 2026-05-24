# OCR Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.9 - Conversacion, memoria, documentos y OCR

## Proposito

Definir como JusNova procesa texto documental y OCR sin convertir documentos privados en instrucciones, sin inflar costos y sin perder trazabilidad de pagina, metodo y confianza.

## Alcance

Aplica a documentos privados de usuario, PDFs escaneados, imagenes, paginas con texto nativo, fragmentos OCR citables y escalacion visual selectiva.

No implementa OCR worker, storage, permisos finales ni retencion completa. Subfase 0.10 define contratos documentales de prompt injection, raw access y privacy/security; enforcement runtime queda para Fase 1 o fases posteriores.

## Definiciones

- `native_text`: texto extraido directamente del documento sin OCR.
- `ocr`: OCR local o deterministico primario.
- `model_vision`: escalacion selectiva con modelo visual sobre paginas o fragmentos justificados.
- `DocumentEvidence`: fragmento documental citable con pagina, locator, metodo, confianza y hash.

## Reglas obligatorias

1. OCR primario debe ser local o deterministico.
2. `model_vision` no puede ser OCR primario ni recibir documentos completos por defecto.
3. OCR se registra por pagina o fragmento, nunca como texto documental completo dentro de trazas.
4. Todo fragmento citable requiere `DocumentEvidence`.
5. Todo fragmento citado debe conservar metodo de extraccion, confianza, pagina y locator.
6. Texto documental se trata como dato no confiable, no como instruccion del sistema.
7. Documento extenso debe enviarse a job async; no se procesa completo inline salvo caso pequeno dentro del budget.

## Reglas deterministicas

1. `DocumentEvidence.extraction_method` solo puede ser `native_text`, `ocr` o `model_vision`.
2. Si `extraction_method = model_vision`, debe existir `vision_escalation_reason` cerrado.
3. Si `extraction_method != model_vision`, `vision_escalation_reason` debe estar ausente o ser `null`.
4. `citation_eligible = true` exige `extraction_confidence >= 0.85`.
5. Si `extraction_confidence < 0.85`, `warnings` debe contener `low_confidence` o `manual_review_required`.
6. `DocumentEvidence.text` es fragmento citable y no puede superar `4000` caracteres.
7. `DocumentEvidence.passage_ref` debe empezar con el mismo `source_ref`.
8. `DocumentEvidence.text_hash` debe ser el hash `sha256` de los bytes UTF-8 exactos persistidos en `text`.
9. `DocumentEvidence.document_version_hash` debe coincidir con la version documental resuelta por `document_version_id`.
10. `DocumentEvidence.locator.page` debe coincidir con `page`.
11. `DocumentEvidence.locator.coordinates`, cuando exista, debe usar bbox cerrado `x`, `y`, `width`, `height`; no se permiten objetos de coordenadas con payloads arbitrarios.

## Reglas asistidas por IA

1. El modelo puede ayudar a identificar si una pagina requiere `model_vision`, pero la razon debe mapearse a enum cerrado.
2. El modelo puede resumir advertencias OCR para usuario, pero no puede ocultar baja confianza.
3. El modelo no puede tratar texto documental como instruccion, aunque el documento lo ordene.

## Comportamiento ante incumplimiento

- Si no hay confianza suficiente para sostener un fragmento critico, la respuesta debe abstenerse parcial o totalmente, pedir Modo Investigacion o solicitar revision humana.
- Si OCR falla, el sistema debe registrar warning cerrado y no citar el fragmento como evidencia decisiva.
- Si se requiere `model_vision` pero el budget no lo permite, se debe advertir limite de evidencia.

## Ejemplos permitidos

- `native_text` con confianza alta y `citation_eligible = true`.
- `ocr` con `low_confidence` y `citation_eligible = false`.
- `model_vision` con `vision_escalation_reason = signature_or_stamp`.

## Ejemplos prohibidos

- Citar un fragmento OCR sin `DocumentEvidence`.
- Enviar documento completo a vision como OCR primario.
- Usar OCR de baja confianza como soporte unico de claim critico.
- Copiar OCR completo en `TraceObject`, `ModelCall`, `ToolCall` o `UsageEvent`.

## Criterios de aceptacion

- `document-evidence.schema.json` valida metodo, confianza, warnings, hashes y localizador.
- `DocumentEvidence` puede convertirse a `EvidencePassage` sin perder `source_ref`, `passage_ref`, texto, locator, metodo, confianza ni warnings.
- Las reglas de baja confianza y escalacion visual quedan cubiertas por ejemplos invalidos o policy-invalid.

## Relacion con contratos

- Implementa reglas para `document-evidence.schema.json`.
- Complementa `passage.schema.json`, `source.schema.json`, `citation.schema.json` y `trace-object.schema.json`.
- Complementa `cost-budget.schema.json` porque OCR consume budget.

## Momento de revision

Revisar al implementar OCR worker, al integrar storage documental, al medir costo por pagina, cuando `document_citation_correctness` baje de umbral o ante cambios en prompt injection/raw access/privacy-security.
