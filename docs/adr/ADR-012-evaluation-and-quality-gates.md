# ADR-012 - Evaluation And Quality Gates

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-25
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Subfase 0.12 - Evaluacion inicial y quality gates

## Contexto

JusNova no puede validarse por impresiones. La calidad juridica exige medicion de citas, claims, fuentes, vigencia, retrieval, OCR, seguridad, tenancy, prompt injection y abstencion.

## Problema

Sin evaluacion sistematica, el sistema puede parecer correcto mientras produce citas rotas, claims sin soporte, fuentes secundarias mal usadas o vigencia no confirmada.

## Restricciones

- Citas inexistentes aprobadas deben ser cero.
- Claims criticos sin soporte aprobado deben ser cero.
- Beta y mercado requieren gates bloqueantes.
- Revision humana juridica es necesaria.
- Ninguna metrica blocker puede quedar sin medir para entrar a beta.
- Ninguna waiver puede ocultar cita inexistente, claim critico sin soporte, source tier incorrecto, vigencia mal manejada, document grounding insuficiente, prompt injection blocking no mitigado, fuga tenant o fuga raw data.

## Opciones consideradas

1. Revisar manualmente solo en beta.
2. Confiar en feedback de usuarios.
3. Evaluation harness, golden cases, regression suite y revision humana.

## Decision

No se considera calidad sin evaluacion sistematica, gates de release y revision juridica humana.

Subfase 0.12 acepta los siguientes entregables:

- `docs/quality/evaluation-plan-v0.md`
- `docs/quality/initial-golden-dataset-spec.md`
- `docs/quality/beta-readiness-gates.md`
- `docs/quality/market-readiness-gates.md`
- `docs/quality/phase-0-acceptance-checklist.md`

## Justificacion

Las metricas convierten calidad juridica, retrieval, OCR y seguridad en criterios verificables y bloqueantes.

## Metricas iniciales

- `citation_validity_rate`
- `unsupported_critical_claims`
- `source_tier_correctness`
- `validity_awareness`
- `abstention_accuracy`
- `retrieval_precision`
- `retrieval_recall_benchmark`
- `conflict_detection_rate`
- `document_grounding`
- `prompt_injection_resistance`

## Blockers no waiver

No se puede hacer waiver para beta o mercado sobre:

- Cita inexistente o cita que no resuelve a fuente/pasaje esperado.
- Claim critico publicado sin soporte valido.
- `source_tier_correctness` bajo target, no medible o con fuente secundaria mal clasificada para soporte critico.
- `validity_awareness` bajo target, no medible o con vigencia/incertidumbre/conflicto sin advertencia politica.
- `document_grounding` bajo target, no medible o con documento citado sin pagina/pasaje verificable.
- Prompt injection blocking no mitigado.
- Fuga tenant, acceso cross-tenant o raw data leakage.

## Waiver policy

- Un waiver solo puede aplicar a metricas no blocker.
- Debe nombrar metrica o gate, razon, owner, fecha de expiracion y plan de correccion.
- Debe quedar documentado antes de cualquier release afectado.
- No puede usarse para declarar beta ni mercado listos si falta evidencia de un gate blocker.

## Dependencias posteriores

- Fase 1 implementara evaluation harness, eval runner y regression suite inicial.
- Revision humana debe registrarse antes de mercado.

## No afirma todavia

- No afirma que dataset ya exista.
- No afirma que eval runner este implementado.
- No afirma que metricas iniciales sean definitivas sin calibracion real.
- No afirma que beta o mercado esten listos.

## Riesgos

- Evals demasiado faciles.
- Muestras no representativas.
- Metricas juridicas sin revision humana.
- Evals tardias que revelen errores estructurales.

## Mitigaciones

- Golden dataset especificado desde Fase 0.
- Casos adversariales.
- Portales caidos simulados.
- PDFs escaneados.
- Revision por abogados bolivianos.
- Gates bloqueantes.
- Waivers cerrados y auditables.

## Criterios de aceptacion

- Subfase 0.12 crea `evaluation-plan-v0.md`.
- Subfase 0.12 crea `initial-golden-dataset-spec.md`.
- Existen metricas minimas y bloqueantes.
- Beta y mercado tienen gates explicitos.
- Fase 1 queda obligada a implementar harness/eval runner antes de beta.
- Revision humana juridica queda obligatoria antes de mercado.

## Momento de revision

Revisar al cerrar Subfase 0.12, antes de beta, antes de mercado y en cada release que cambie retrieval, OCR, prompts, modelos, providers, citation auditor, document grounding, prompt injection controls o validity resolver.

## Consecuencias

El avance a beta o mercado queda condicionado por metricas, gates, evidencia verificable y revision juridica, no por percepcion subjetiva.
