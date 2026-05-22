# ADR-012 - Evaluation And Quality Gates

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Evaluacion, metricas y gates de calidad

## Contexto

JusNova no puede validarse por impresiones. La calidad juridica exige medicion de citas, claims, fuentes, vigencia, retrieval, OCR, seguridad y abstencion.

## Problema

Sin evaluacion sistematica, el sistema puede parecer correcto mientras produce citas rotas, claims sin soporte, fuentes secundarias mal usadas o vigencia no confirmada.

## Restricciones

- Citas inexistentes aprobadas deben ser cero.
- Claims criticos sin soporte aprobado deben ser cero.
- Beta y mercado requieren gates.
- Revision humana juridica es necesaria.

## Opciones consideradas

1. Revisar manualmente solo en beta.
2. Confiar en feedback de usuarios.
3. Evaluation harness, golden cases, regression suite y revision humana.

## Decision

No se considera calidad sin evaluacion sistematica y revision juridica.

## Justificacion

Las metricas convierten calidad juridica, retrieval, OCR y seguridad en criterios verificables y bloqueantes.

## Metricas iniciales

- Citation validity rate.
- Unsupported critical claims.
- Source tier correctness.
- Validity awareness.
- Abstention accuracy.
- Retrieval precision/recall.
- Conflict detection rate.
- Document grounding.
- Prompt injection resistance.

## Dependencias posteriores

- Subfase 0.12 debe crear evaluation plan, golden dataset spec, beta readiness gates y market readiness gates.
- Fase 12 implementara evaluation harness y regression suite.
- Revision humana debe registrarse antes de mercado.

## No afirma todavia

- No afirma que dataset ya exista.
- No afirma que eval runner este implementado.
- No afirma que metricas iniciales sean definitivas sin calibracion real.

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

## Criterios de aceptacion

- Subfase 0.12 crea `evaluation-plan-v0.md`.
- Subfase 0.12 crea `initial-golden-dataset-spec.md`.
- Existen metricas minimas y bloqueantes.
- Beta y mercado tienen gates explicitos.

## Momento de revision

Revisar al cerrar Subfase 0.12, antes de beta, antes de mercado y en cada release que cambie retrieval, OCR, prompts, modelos, citation auditor o validity resolver.

## Consecuencias

El avance a beta o mercado queda condicionado por metricas y revision juridica, no por percepcion subjetiva.

