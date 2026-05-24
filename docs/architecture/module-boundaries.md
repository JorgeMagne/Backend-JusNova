# Module Boundaries

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** ADR-001 - High-Assurance Modular Core + Distributed Execution Layer
**Enmiendas:** Subfase 0.11 - domain model, API boundary and entity ownership matrix

## Proposito

Este documento define que vive dentro del core, que vive en workers y bajo que condiciones una pieza puede convertirse en servicio independiente.

## Modulos dentro del core

| Modulo | Responsabilidad | No debe hacer |
|---|---|---|
| API Gateway / Request Boundary Module | Autenticacion de request, tenant context, path/body validation, stream coordination y `ErrorEnvelope`. | Saltarse tenancy, devolver payloads raw o convertir un run en progreso en `TraceObject` valido. |
| Identity / Membership Module | Organizaciones, usuarios internos, membresias, tenant context y actor refs. | Exponer `user_id` crudo en contratos de auditoria o reemplazar `actor_ref` pseudonimo. |
| Conversation Module | Conversaciones, mensajes, runs y streaming coordinado. | Ejecutar OCR, crawling o indexacion pesada. |
| Case Memory Module | Memoria estructurada de caso, hechos, pendientes y fuentes historicas. | Tratar memoria como verdad juridica. |
| Legal Orchestration Module | Clasificacion, planificacion, coordinacion de tools, respuesta y abstencion. | Llamar SDKs externos directamente. |
| Live Legal Search Module | Crear planes de busqueda, coordinar adapters/providers y producir retrieval runs. | Tratar resultados crudos como evidencia. |
| Source Registry Module | Fuente, tier, autoridad, vigencia, snapshots y health metadata. | Validar juridicamente por intuicion del modelo. |
| Evidence Module | Evidence Packs, sources, passages y quality states. | Generar texto final sin auditoria. |
| Answer Contract Module | Formato de respuesta, secciones, fuentes finales y warnings. | Inventar citas o fuentes. |
| Citation Auditor Module | Validar cita -> pasaje -> fuente y claims criticos. | Depender solo de evaluacion generativa. |
| Claim Verification Module | Verificar soporte, criticidad, bloqueo y repair controlado. | Agregar nuevos claims durante repair. |
| Document Registry Module | Documentos, versiones, paginas, ownership y estado de procesamiento. | Procesar OCR largo inline. |
| Plan / Subscription Module | Planes, suscripciones, creditos de investigacion y estado comercial no juridico. | Usar informacion comercial como evidencia o fuente juridica. |
| Cost Governor Module | Budgets, creditos, limites por plan y decisiones de degradacion controlada. | Reducir veracidad o citacion para ahorrar. |
| Usage Ledger Module | Eventos de uso, costos estimados, tokens, fetches, OCR y creditos. | Ser fuente de verdad juridica. |
| Error Handling Module | Catalogo de errores, `safe_message_code`, sanitizacion de mensajes y metadata cerrada. | Interpolar contenido de usuario, documentos, provider payloads o stack traces. |
| Security / Tenancy Module | Organizaciones, usuarios, permisos, isolation, data classification. | Delegar seguridad a prompts. |
| Audit / Trace Module | TraceObject, model calls, tool calls, answer versions y hashes. | Guardar documentos completos en logs generales. |

## Workers y workflows

| Worker | Responsabilidad | Motivo de separacion |
|---|---|---|
| Live Retrieval Worker | Rondas de busqueda viva y retries controlados. | Latencia y fallos externos. |
| Official Adapter Worker | Consultas a Gaceta, TSJ, TCP y fuentes oficiales. | Portales lentos o inestables. |
| Fetch / Snapshot Worker | HTTP fetch, render publico, descarga PDF, snapshot y hash. | IO externo, timeouts y reintentos. |
| Extraction Worker | HTML/PDF parse, metadata y pasajes. | CPU/IO variable. |
| OCR Worker | OCR por pagina y jobs async. | CPU alto y latencia. |
| Indexing Worker | OpenSearch indexing de documentos, snapshots y evidence cache. | Trabajo reintentable y no transaccional. |
| Evaluation Runner | Golden tests, search benchmarks, regression runs. | Ejecucion batch y costosa. |

## Limites de dependencia

- Core puede depender de contratos, repositorios internos e interfaces de proveedores.
- Workers pueden depender de adaptadores concretos, librerias de OCR, clientes HTTP y SDKs de infraestructura.
- Proveedores externos no se importan en modelos de dominio ni reglas juridicas centrales.
- Redis no guarda verdad transaccional.
- OpenSearch no reemplaza PostgreSQL.
- Evidence Cache no reemplaza Source Registry ni Validity Resolver.
- La capa API no puede omitir tenancy ni devolver `CaseMemory`, documentos, OCR, trazas o provider payloads crudos.
- Workers documentales pueden producir estado, fragmentos y evidencia documental, pero no escriben respuestas juridicas finales directamente.
- Usage Ledger, planes, suscripciones y creditos no son evidencia juridica.
- `ErrorEnvelope` es la unica forma aceptada para errores HTTP no 2xx en el API draft.

## Criterios de extraccion futura a servicio independiente

Un modulo puede extraerse solo si cumple al menos cuatro criterios:

1. escala de forma independiente;
2. tiene SLA distinto;
3. tiene carga pesada o perfil de recursos propio;
4. su contrato de datos esta maduro;
5. observabilidad demuestra cuello de botella;
6. equipo u ownership separado;
7. falla sin comprometer transaccion juridica central;
8. puede versionarse sin romper Evidence/Trace contracts.

## Candidatos futuros

- Live Legal Search Service.
- Document Processing Service.
- Evaluation Service.
- Billing / Usage Service.
- Enterprise Tenant Gateway.

## Regla de estabilidad

Antes de extraer un microservicio, debe existir ADR nuevo con metrica operativa que justifique la extraccion.
