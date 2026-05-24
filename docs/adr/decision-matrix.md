# ADR Decision Matrix

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Subfase 0.3 - Canon de ADRs fundacionales

## Proposito

Esta matriz fija la lista canonica de ADR-001 a ADR-012 para Fase 0. El orden resumido del Plan v3 queda como referencia historica no canonica cuando contradiga esta lista.

## Lista canonica

| ADR | Decision canonica | Alternativa principal descartada | Razon de descarte | Dependencia posterior | Criterio de revision futura |
|---|---|---|---|---|---|
| ADR-001 | High-Assurance Modular Core + Distributed Execution Layer. | Microservices mesh inicial o monolito plano. | La malla agrega complejidad prematura; el monolito plano rompe boundaries. | Fase 1 estructura modular; workers por fases. | Revisar al cerrar Fase 1 o si un modulo demuestra cuello operativo. |
| ADR-002 | Python/FastAPI/Pydantic, PostgreSQL, Redis, OpenSearch, StorageProvider S3-compatible, Temporal como meta, OpenTelemetry. | NestJS como core, base vectorial dedicada inicial, orquestacion avanzada inicial. | Python alinea IA/OCR/retrieval; dependencias adicionales requieren metricas. | Fase 1 settings, migraciones, observabilidad y servicios locales. | Revisar con metricas de retrieval, costo, latencia u operacion. |
| ADR-003 | Live Legal Search Engine como corazon de fundamentacion. | Buscador web generico o memoria del modelo. | No garantizan autoridad, jurisdiccion, vigencia ni citas auditables. | Subfase 0.5 contratos; Fases 4-8 implementacion. | Revisar con recall, precision y costo por Evidence Pack. |
| ADR-004 | Lanzamiento sin RAG juridico propio. | Esperar corpus completo o vender cache como corpus. | Aumenta riesgo y demora salida sin evidencia de demanda/cobertura. | `no-rag-launch-policy.md` aceptada en Subfase 0.6; Fase 8 cache/snapshots. | Revisar tras beta con demanda, permisos y cobertura medidos. |
| ADR-005 | OpenAI como proveedor IA inicial detras de ModelProvider. | SDK del proveedor dentro del core o multi-proveedor temprano. | Acopla dominio; multi-proveedor temprano aumenta complejidad. | Fase 1 ModelProvider stub; Subfase 0.10 provider policy. | Revisar con evals, costo, latencia o cambios contractuales. |
| ADR-006 | OCR local/deterministico primario; vision solo escalacion selectiva. | Vision como OCR primario o ignorar PDFs escaneados. | Vision primaria eleva costo/privacidad; ignorar PDFs rompe cobertura juridica. | Subfase 0.9 OCR/document policies; Fase 9 implementacion. | Revisar con calidad OCR, costo por pagina y citation correctness. |
| ADR-007 | Evidence, Answer, Citation y Claim contracts para respuestas criticas. | Respuesta libre con fuentes al final. | Permite citas decorativas y claims sin soporte. | Subfase 0.4 schemas/policies; Fase 2 auditor. | Revisar si citation validity baja de 1.0 o aparece unsupported critical claim. |
| ADR-008 | Cost Governor con plan base 400 Bs y budgets por profundidad. | Sin limites o solo limite de tokens. | No controla fetches/OCR/discovery ni experiencia comercial. | Subfase 0.8 budgets/contracts/policies aceptados; Fase 1 CostGovernor/UsageLedger. | Revisar mensualmente en beta con costo real por consulta. |
| ADR-009 | Source Registry, tiers, snapshots y vigencia conservadora. | Lista informal de URLs o clasificacion libre por modelo. | No controla autoridad, cache, conflicto ni vigencia. | Source, validity, conflict y uncertainty policies aceptadas en Subfase 0.6; Fase 4/5 registry/adapters. | Revisar con source tier correctness, validity awareness y conflicts. |
| ADR-010 | TraceObject y AnswerVersion por respuesta critica. | Logs generales o solo respuesta final. | No permite reconstruccion juridica ni auditoria. | Subfase 0.7 schemas/policies aceptados; Fase 3 versionado basico. | Revisar ante incidentes, citation failures o cambios de trazabilidad. |
| ADR-011 | Seguridad, privacidad y provider boundaries desde el inicio. | Seguridad despues de beta. | Rehacer datos, logs, storage y providers seria costoso y riesgoso. | Subfase 0.10 policies/checklist; Fase 1 ownership. | Revisar ante cambios de proveedor, storage, logging o tenancy. |
| ADR-012 | Evaluacion sistematica y revision juridica como gates. | Validacion subjetiva o feedback tardio. | No detecta regresiones juridicas ni fallos de citas. | Subfase 0.12 eval plan/dataset/gates; Fase 12 harness. | Revisar en cada release que cambie retrieval, OCR, prompts o auditoria. |

## Canonicalizacion de numeracion

La numeracion canonica para Fase 0 es:

```txt
ADR-001 - High-Assurance Modular Core
ADR-002 - Stack Backend And Infrastructure
ADR-003 - Live Legal Search Engine
ADR-004 - Launch Without Own Legal RAG
ADR-005 - AI Provider And Model Policy
ADR-006 - Document OCR Policy
ADR-007 - Evidence, Answer, Citation And Claim Contracts
ADR-008 - Cost Governor And Commercial Budgets
ADR-009 - Source Registry And Validity Policy
ADR-010 - Traceability And Answer Versioning
ADR-011 - Security, Privacy And Provider Boundaries
ADR-012 - Evaluation And Quality Gates
```

## Semantica de Accepted

`Accepted` en estos ADRs significa decision arquitectonica aceptada. No significa que los contracts, policies, schemas, workers, providers o evals posteriores ya esten implementados o aprobados.
