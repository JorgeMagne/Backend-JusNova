# Contract Governance

**Estado documental:** Draft  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Gobierno de contratos internos y JSON Schemas

## Proposito

Los contratos definen las estructuras que Fase 1 implementara en codigo. Un modulo critico no debe implementarse sin contrato aprobado.

## Reglas

- Todo contrato debe tener responsable.
- Todo contrato debe tener estado documental.
- Todo JSON Schema debe usar draft `2020-12`.
- Todo contrato critico debe incluir ejemplos validos e invalidos antes de cerrar Fase 0.
- Todo contrato que referencie otro schema debe usar rutas relativas estables.
- Los nombres de enums deben venir de `docs/schemas/` cuando exista una taxonomia aprobada.

## Contratos aceptados en Subfase 0.4

Estos contratos quedan aceptados como base documental del flujo `EvidencePack -> AnswerDraft -> ClaimExtractor -> CitationAuditor -> ClaimVerifier -> FinalAnswer`. Esta aceptacion no implementa backend funcional ni `citation-audit.schema.json`.

| Contrato | Estado | Archivo |
|---|---|---|
| Answer Contract | Accepted | `answer-contract.schema.json` |
| Evidence Pack | Accepted | `evidence-pack.schema.json` |
| Source | Accepted | `source.schema.json` |
| Passage | Accepted | `passage.schema.json` |
| Citation | Accepted | `citation.schema.json` |
| Claim | Accepted | `claim.schema.json` |

## Fuera de alcance de Subfase 0.4

`citation-audit.schema.json` queda delegado a Subfase 0.7 junto con trazabilidad, auditoria y versionado. Subfase 0.4 define la forma de `Citation` y las policies de citacion/abstencion, pero no el resultado completo del auditor.

## Schemas minimos esperados

```txt
answer-contract.schema.json
claim.schema.json
citation.schema.json
citation-audit.schema.json
cost-budget.schema.json
document-evidence.schema.json
evidence-pack.schema.json
evidence-quality.schema.json
error-envelope.schema.json
legal-search-query.schema.json
legal-search-result.schema.json
message.schema.json
model-call.schema.json
passage.schema.json
retrieval-plan.schema.json
retrieval-run.schema.json
source.schema.json
source-registry-entry.schema.json
source-snapshot.schema.json
trace-object.schema.json
tool-call.schema.json
usage-event.schema.json
case-memory.schema.json
conversation.schema.json
```
