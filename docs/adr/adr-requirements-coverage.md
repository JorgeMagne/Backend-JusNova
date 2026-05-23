# ADR Requirements Coverage

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Subfase 0.3 - Cobertura de requisitos por ADR

## Proposito

Esta matriz permite revisar cada ADR contra los puntos obligatorios que debe cerrar. Si una fila queda sin cobertura, Subfase 0.3 no debe aceptarse.

| ADR | Debe cerrar | Donde queda cubierto | Estado | Dependencia posterior | Riesgo si falta |
|---|---|---|---|---|---|
| ADR-001 | Por que no microservices mesh desde el dia uno. | ADR-001: Problema, Opciones consideradas, Justificacion. | Accepted | Ninguna para decision. | Complejidad distribuida prematura. |
| ADR-001 | Por que no monolito plano. | ADR-001: Problema, Opciones consideradas, Justificacion. | Accepted | Ninguna para decision. | Mezcla de responsabilidades criticas. |
| ADR-001 | Modulos dentro del core. | ADR-001: Limites aprobados; `module-boundaries.md`. | Accepted | Fase 1 estructura modular. | Core sin ownership ni boundaries. |
| ADR-001 | Tareas en workers. | ADR-001: Limites aprobados; `architecture-overview.md`; `module-boundaries.md`. | Accepted | Fases posteriores workers/workflows. | OCR/retrieval bloquean core. |
| ADR-001 | Criterios para extraer microservicios futuros. | `module-boundaries.md`: Criterios de extraccion futura. | Accepted | ADR nuevo ante extraccion. | Servicios prematuros o inconsistentes. |
| ADR-001 | Diagrama y limites modulares. | `architecture-overview.md`; `module-boundaries.md`. | Accepted | Ninguna para decision. | ADR sin respaldo visual/operativo. |
| ADR-002 | Python/FastAPI/Pydantic sobre NestJS. | ADR-002: Contexto, Decision, Justificacion. | Accepted | Fase 1 scaffold. | Stack no alineado a IA/OCR/retrieval. |
| ADR-002 | OpenSearch cubre lexical/filtros/vector inicial evaluable. | ADR-002: Decision, Justificacion. | Accepted | Fase 1 OpenSearch; Fase 12 evals. | Base de busqueda prematura o insuficiente. |
| ADR-002 | Base vectorial dedicada no obligatoria en lanzamiento. | ADR-002: Opciones consideradas, No afirma todavia. | Accepted | Revisar tras metricas retrieval. | Dependencia extra sin evidencia. |
| ADR-002 | Redis no es fuente de verdad. | ADR-002: Decision; `architecture-overview.md`. | Accepted | Fase 1 cache/locks. | Perdida de datos transaccionales. |
| ADR-002 | Object storage obligatorio. | ADR-002: Decision, Justificacion. | Accepted | Fase 1 StorageProvider. | PDFs/snapshots en base transaccional. |
| ADR-002 | Condicion para adoptar Kubernetes. | ADR-002: No afirma todavia, Criterios de aceptacion, Revision. | Accepted | Metricas operativas futuras. | Plataforma sobredimensionada. |
| ADR-003 | Que significa busqueda juridica viva. | ADR-003: Decision, Justificacion, Componentes aprobados; `docs/policies/legal-search-policy.md`. | Accepted | Fase 4 implementacion. | Busqueda generica sin estructura juridica. |
| ADR-003 | Que fuentes consulta. | ADR-003: Contexto; ADR-009 fuentes iniciales; `docs/policies/source-routing-matrix.md`; `docs/policies/source-policy.md`. | Accepted | Fase 5 adapters. | Cobertura Bolivia-first incompleta. |
| ADR-003 | Que adaptadores existen. | ADR-003: Componentes aprobados; `docs/contracts/provider-interfaces.md`. | Accepted | Fase 5 adapters. | Portales oficiales no modelados. |
| ADR-003 | Rol de discovery web. | ADR-003: Problema, Decision, No afirma todavia; `docs/policies/legal-search-policy.md`. | Accepted | Fase 4 provider implementation. | Discovery tratado como fuente final. |
| ADR-003 | Rol de snapshots. | ADR-003: Componentes aprobados. | Accepted | Fase 8 Source Snapshot Registry. | No reproducibilidad de fuentes. |
| ADR-003 | Rol de Evidence Cache. | ADR-003: Componentes aprobados; ADR-004. | Accepted | Fase 8 Evidence Cache. | Cache confundido con corpus o vigencia. |
| ADR-003 | Que no es RAG. | ADR-003: No afirma todavia; ADR-004; `docs/policies/no-rag-launch-policy.md`. | Accepted | Fase 8 cache/snapshots. | Promesa comercial incorrecta. |
| ADR-003 | Como se generan Evidence Packs. | ADR-003: Componentes aprobados; `retrieval-run.schema.json`; `evidence-pack.schema.json`. | Accepted | Fase 4 implementation. | Respuestas desde texto crudo. |
| ADR-003 | Como se evalua calidad de busqueda. | ADR-003: Mitigaciones, Revision; `evidence-quality.schema.json`; ADR-012. | Accepted | Subfase 0.12 evaluation plan. | Calidad subjetiva del motor. |
| ADR-003 | No depender de un unico proveedor de discovery. | ADR-003: Restricciones, No afirma todavia, Criterios; `provider-interfaces.md`. | Accepted | Fase 4 provider implementation. | Fragilidad y costo por proveedor unico. |
| ADR-003 | No devolver texto crudo sin normalizacion. | ADR-003: Criterios de aceptacion; `legal-search-policy.md`; `legal-search-result.schema.json`. | Accepted | Fase 4 implementation. | Modelo cita texto no confiable. |
| ADR-004 | Fuera de alcance antes de lanzamiento. | ADR-004: Fuera de lanzamiento. | Accepted | Ninguna para decision. | Alcance inflado. |
| ADR-004 | Que si se construye. | ADR-004: Dentro de lanzamiento. | Accepted | Fases 4-9. | No hay sustituto al corpus propio. |
| ADR-004 | Evitar vender Evidence Cache como corpus. | ADR-004: Problema, Decision, No afirma todavia; `docs/policies/no-rag-launch-policy.md`. | Accepted | Fase 8 cache/snapshots. | Riesgo comercial/juridico. |
| ADR-004 | Cuando reconsiderar RAG propio. | ADR-004: Momento de revision. | Accepted | ADR futuro post-mercado. | Corpus prematuro sin permisos. |
| ADR-005 | Que tareas usa IA. | ADR-005: Tareas que pueden usar IA. | Accepted | Fase 1 ModelProvider. | Uso IA indefinido. |
| ADR-005 | Que tareas no dependen solo de IA. | ADR-005: Tareas que no dependen solo de IA. | Accepted | Fase 1/2 validators/auditors. | Prompt como control unico. |
| ADR-005 | Versionado de prompts. | ADR-005: Restricciones, Mitigaciones. | Accepted | Fase 1 prompt registry. | Imposible comparar outputs. |
| ADR-005 | Evaluacion de modelos. | ADR-005: Mitigaciones, Revision. | Accepted | Subfase 0.12 eval plan. | Cambios de modelo sin evidencia. |
| ADR-005 | Datos enviados a proveedor. | ADR-005: Restricciones, Dependencias posteriores. | Accepted | Subfase 0.10 provider policy. | Exposicion de datos sensibles. |
| ADR-005 | Minimizar datos sensibles. | ADR-005: Restricciones, Mitigaciones. | Accepted | Subfase 0.10 data classification. | Riesgo privacidad. |
| ADR-005 | Cambiar proveedor futuro. | ADR-005: Decision, Justificacion. | Accepted | Provider interfaces. | Dependencia rigida. |
| ADR-006 | Pipeline documental. | ADR-006: Pipeline aprobado. | Accepted | Subfase 0.9 contracts. | Documentos sin flujo confiable. |
| ADR-006 | Deteccion de escaneo. | ADR-006: Pipeline aprobado. | Accepted | Fase 9 implementacion. | PDFs imagen sin texto. |
| ADR-006 | OCR progresivo. | ADR-006: Decision, Mitigaciones. | Accepted | Fase 9 budgets/async. | Costos y latencia altos. |
| ADR-006 | Limites por plan/modo. | ADR-006: Restricciones; ADR-008. | Accepted | Subfase 0.8/0.9. | OCR sin control de costo. |
| ADR-006 | Seguridad documental. | ADR-006: PDFs publicos vs privados; Mitigaciones. | Accepted | Subfase 0.10. | Fuga de documentos. |
| ADR-006 | Prompt injection documental. | ADR-006: Restricciones, Riesgos, Mitigaciones. | Accepted | Subfase 0.10 prompt injection policy. | Documento controla comportamiento. |
| ADR-006 | Citas [D#:P#]. | ADR-006: Restricciones, Pipeline aprobado. | Accepted | Subfase 0.4/0.9 schemas. | Hechos sin locator. |
| ADR-007 | Estructura visible al abogado. | ADR-007: Estructura visible aprobada. | Accepted | Subfase 0.4 answer contract. | Respuestas inconsistentes. |
| ADR-007 | Estructura interna de Evidence Pack. | ADR-007: Decision, Dependencias posteriores. | Accepted | Subfase 0.4 evidence schema. | Evidencia no auditable. |
| ADR-007 | Formato [F#:P#], [D#:P#]. | ADR-007: Restricciones. | Accepted | Subfase 0.4 schemas. | Citas ambiguas. |
| ADR-007 | Tipos de claims. | ADR-007: Dependencias posteriores. | Accepted | Subfase 0.4 claim schema. | Auditoria incompleta. |
| ADR-007 | Soporte directo/inferencial/debil/no soportado. | ADR-007: Dependencias posteriores. | Accepted | Subfase 0.4 claim/citation policy. | Claims mal clasificados. |
| ADR-007 | Reglas de bloqueo. | ADR-007: Decision, Mitigaciones, Criterios. | Accepted | Fase 2 Citation Auditor. | Respuestas sin soporte. |
| ADR-007 | Existen schemas JSON como condicion de Fase 0 completa. | ADR-007: Dependencias posteriores, Criterios. | Accepted | Subfase 0.4. | ADR aceptado sin contratos posteriores. |
| ADR-008 | Budgets por complejidad. | ADR-008: Presupuestos conceptuales. | Accepted | Subfase 0.8 budgets YAML. | Costos invisibles. |
| ADR-008 | Limites de discovery/fetch/OCR/tool rounds/output. | ADR-008: Decision, Dependencias posteriores. | Accepted | Subfase 0.8. | Presupuesto incompleto. |
| ADR-008 | Creditos de investigacion. | ADR-008: Decision, Presupuestos conceptuales. | Accepted | Subfase 0.8. | Modo profundo sin control. |
| ADR-008 | Usage Ledger. | ADR-008: Decision, Dependencias posteriores. | Accepted | Fase 1 UsageLedger. | Consumo no trazable. |
| ADR-008 | Respuesta ante excedentes. | ADR-008: Justificacion, Mitigaciones. | Accepted | Subfase 0.8 cost policy. | Silencio ante limites. |
| ADR-008 | Plan de entrada 400 Bs. | ADR-008: Contexto, Restricciones, Decision. | Accepted | Subfase 0.8 commercial plans. | Restriccion comercial no reflejada. |
| ADR-008 | No reducir citacion para ahorrar costo. | ADR-008: Restricciones, Criterios. | Accepted | Subfase 0.8 cost policy. | Respuestas menos verificables por costo. |
| ADR-009 | Fuentes iniciales prioritarias. | ADR-009: Fuentes iniciales prioritarias. | Accepted | Fase 5 adapters. | Cobertura sin prioridades. |
| ADR-009 | Campos minimos de registro. | ADR-009: Restricciones; `docs/contracts/source.schema.json`; `docs/policies/source-policy.md`. | Accepted | Source Registry Entry schema posterior. | Fuente sin metadata. |
| ADR-009 | Tiers. | ADR-009: Decision, Mitigaciones; `docs/schemas/source-tiers.yaml`; `docs/policies/source-policy.md`. | Accepted | Fase 4 registry. | Fuentes debiles sin etiqueta. |
| ADR-009 | Validity statuses. | ADR-009: Restricciones, Decision; `docs/schemas/validity-statuses.yaml`; `docs/policies/validity-policy.md`. | Accepted | Fase 4 validity resolver. | Vigencia presumida. |
| ADR-009 | Policy TIER2/TIER3. | ADR-009: Criterios, Mitigaciones; `docs/policies/source-policy.md`. | Accepted | Fase 4 registry. | Fuente secundaria como primaria. |
| ADR-009 | Conflictos. | ADR-009: Decision, Mitigaciones; `docs/policies/conflict-policy.md`. | Accepted | Conflict Resolver futuro. | Contradicciones ocultas. |
| ADR-009 | Snapshots. | ADR-009: Decision, Mitigaciones. | Accepted | Fase 8 snapshots. | Fuentes no reproducibles. |
| ADR-009 | Source, validity, conflict y uncertainty policies requeridas. | ADR-009: Dependencias posteriores, Criterios; `docs/policies/source-policy.md`; `docs/policies/validity-policy.md`; `docs/policies/conflict-policy.md`; `docs/policies/uncertainty-policy.md`. | Accepted | Fase 4/5 implementacion. | Decision sin politicas operativas. |
| ADR-010 | Que se registra. | ADR-010: Trazabilidad aprobada; `trace-object.schema.json`; `model-call.schema.json`; `tool-call.schema.json`; `cost-report.schema.json`. | Accepted | Fase 3 persistencia. | Respuesta no reconstruible. |
| ADR-010 | Que es visible al usuario. | ADR-010: Restricciones; `trace-visibility-policy.md`. | Accepted | Subfase 0.10 permisos finales. | Exposicion excesiva. |
| ADR-010 | Que es interno. | ADR-010: Restricciones, Mitigaciones; `trace-visibility-policy.md`. | Accepted | Subfase 0.10 controles de acceso. | Soporte sin datos o con datos excesivos. |
| ADR-010 | Contenido de answer version. | ADR-010: Decision, Justificacion; `answer-version.schema.json`; `abstention-render.schema.json`. | Accepted | Fase 3 persistencia. | Sobrescritura silenciosa o abstencion no reconstruible. |
| ADR-010 | Cuando crear nueva version. | ADR-010: Decision, Criterios; `answer-versioning-policy.md`. | Accepted | Fase 3 implementacion. | Cambios invisibles. |
| ADR-010 | Proteger datos sensibles en trazas. | ADR-010: Riesgos, Mitigaciones; `trace-object.schema.json`; `trace-visibility-policy.md`. | Accepted | Subfase 0.10 privacy policy. | Fuga de datos en trazas. |
| ADR-010 | TraceObject futuro para respuesta juridica critica. | ADR-010: Decision, Trazabilidad aprobada, Consecuencias; `trace-object.schema.json`. | Accepted | Fase 3. | Respuesta no auditable. |
| ADR-011 | Tenancy. | ADR-011: Reglas aprobadas. | Accepted | Fase 1 ownership. | Acceso cruzado. |
| ADR-011 | Roles minimos. | ADR-011: Reglas aprobadas. | Accepted | Fase 11 hardening. | Permisos ambiguos. |
| ADR-011 | Logs. | ADR-011: Restricciones, Reglas, Mitigaciones. | Accepted | Subfase 0.10. | Datos sensibles en logs. |
| ADR-011 | Secrets. | ADR-011: Restricciones. | Accepted | Fase 1 settings/secrets. | Secretos en repo. |
| ADR-011 | Object storage. | ADR-011: Reglas aprobadas. | Accepted | Fase 1 StorageProvider. | Archivos expuestos. |
| ADR-011 | Document permissions. | ADR-011: Reglas aprobadas. | Accepted | Fase 11. | Documentos sin control. |
| ADR-011 | Provider minimization. | ADR-011: Reglas aprobadas. | Accepted | Subfase 0.10 provider policy. | Datos excesivos a terceros. |
| ADR-011 | Prompt injection. | ADR-011: Reglas aprobadas. | Accepted | Subfase 0.10 prompt injection policy. | Evidencia como instrucciones. |
| ADR-011 | Retencion/eliminacion. | ADR-011: Reglas aprobadas. | Accepted | Subfase 0.10. | Datos retenidos sin politica. |
| ADR-012 | Metricas iniciales. | ADR-012: Metricas iniciales. | Accepted | Subfase 0.12. | Calidad subjetiva. |
| ADR-012 | Dataset inicial minimo. | ADR-012: Dependencias posteriores. | Accepted | Subfase 0.12 dataset spec. | Evals sin cobertura. |
| ADR-012 | Golden cases. | ADR-012: Decision, Mitigaciones. | Accepted | Subfase 0.12. | Regresiones no detectadas. |
| ADR-012 | Evals automaticas. | ADR-012: Decision, Mitigaciones. | Accepted | Fase 12 harness. | Releases sin control. |
| ADR-012 | Revision humana. | ADR-012: Decision, Mitigaciones. | Accepted | Subfase 0.12 y beta. | Calidad juridica sin abogado. |
| ADR-012 | Gates beta/mercado. | ADR-012: Decision, Criterios. | Accepted | Subfase 0.12 gates. | Salida a mercado insegura. |
