# Non-Negotiable Principles

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Fase 0.1 - Alineacion de principios no negociables  

## Proposito

Este documento convierte los principios no negociables de JusNova en reglas tecnicas verificables. Su objetivo es impedir que los principios queden como declaracion aspiracional o como instrucciones de prompt.

## Regla madre

Ningun principio critico puede depender unicamente del modelo. Cada principio debe tener al menos un mecanismo verificable en contratos, validadores, policies, tests, trazas, budgets, schemas, evaluaciones o controles de runtime.

## Alcance

Esta policy aplica a:

- respuestas juridicas;
- busqueda juridica en vivo;
- fuentes y snapshots;
- evidencia, claims y citas;
- documentos del usuario;
- memoria de caso;
- vigencia y conflictos;
- proveedores externos;
- costos y presupuestos;
- seguridad, privacidad y tenancy;
- evaluacion y readiness gates.

## Estados de cumplimiento

| Estado | Significado |
|---|---|
| Enforced | Existe control deterministico o test automatizable. |
| Guarded | Existe contrato/policy y debe implementarse en Fase 1 o posterior. |
| Blocked | No se puede operar el flujo si falta el control. |

## Principios y reglas tecnicas

### P-NN-001 - Evidencia o silencio

**Principio:** Toda afirmacion juridica critica requiere evidencia o debe bloquearse, abstenerse o declararse incierta.  
**Regla tecnica:** Todo `Claim` con `criticality = high` o `claim_type` en `plazo`, `requisito`, `competencia`, `causal`, `procedimiento`, `norma`, `jurisprudencia`, `vigencia` debe tener al menos una cita valida a un `EvidencePassage` existente, salvo que `verification_status = blocked` y la respuesta use formato de abstencion. Toda afirmacion juridica critica visible debe estar representada por un `Claim`.
**Mecanismo verificable:** `claim.schema.json`, `citation.schema.json`, `evidence-pack.schema.json`, `CitationAuditor`, `ClaimVerifier`, tests de claims criticos sin soporte.  
**No depende solo del prompt:** El auditor debe rechazar programaticamente respuestas con critical claims sin cita valida.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro AI Orchestration Lead.  
**Criterio de aceptacion:** Una respuesta con plazo, requisito, competencia, causal, vigencia o criterio jurisprudencial sin cita valida no puede pasar a `FinalAnswer`.  
**Severidad si se incumple:** Critical.

### P-NN-002 - Bolivia por defecto

**Principio:** JusNova opera con Bolivia como jurisdiccion inicial y predeterminada.  
**Regla tecnica:** `RetrievalPlan`, `LegalSearchQuery`, `EvidencePack`, `SourceClassifier` y `CaseMemory` deben declarar `jurisdiction = BO` por defecto. Derecho extranjero solo puede entrar si se etiqueta como comparativo, contexto, ruido descartado o fuente no aplicable.  
**Mecanismo verificable:** `legal-search-query.schema.json`, `evidence-pack.schema.json`, `case-memory.schema.json`, source jurisdiction scoring, retrieval filters, evals con derecho extranjero como ruido.  
**No depende solo del prompt:** El planner y classifier deben persistir jurisdiccion como campo estructurado, no solo mencionarla en texto.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Retrieval Lead.  
**Criterio de aceptacion:** Una fuente peruana, colombiana, espanola u otra no puede sostener una conclusion de derecho boliviano sin advertencia explicita y clasificacion no primaria.  
**Severidad si se incumple:** Critical.

### P-NN-003 - Sin RAG juridico propio para lanzamiento

**Principio:** El lanzamiento no depende de un corpus juridico boliviano propio indexado masivamente.  
**Regla tecnica:** La fundamentacion juridica debe provenir de Live Legal Search, adaptadores oficiales, discovery controlado, Source Snapshots, Evidence Cache con fecha, o documentos del usuario; nunca de un corpus juridico propio tratado como fuente exhaustiva antes de mercado.  
**Mecanismo verificable:** ADR-004, `no-rag-launch-policy.md`, `source-policy.md`, estrategia de snapshot/hash, `retrieval-run.schema.json`, trace de origen de evidencia.
**No depende solo del prompt:** Cada fuente usada debe tener `source_type`, `tier`, `retrieved_at` y origen recuperable.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Backend Lead.  
**Criterio de aceptacion:** Ninguna respuesta juridica critica puede declarar que proviene de "el corpus JusNova" antes de que exista un ADR posterior que habilite corpus/RAG avanzado.  
**Severidad si se incumple:** High.

### P-NN-004 - Busqueda juridica en vivo como corazon de fundamentacion

**Principio:** La base de fundamentacion del producto de lanzamiento es el JusNova Live Legal Search Engine.  
**Regla tecnica:** Para intents `NORMATIVA`, `JURISPRUDENCIA`, `PROCEDIMIENTO`, `VIGENCIA`, `SECTORIAL`, `SUBNACIONAL`, `MIXTO` y estrategia juridica con fundamento, el orquestador debe intentar recuperar evidencia antes de generar conclusion sustantiva.  
**Mecanismo verificable:** `retrieval-plan.schema.json`, `legal-search-query.schema.json`, `retrieval-run.schema.json`, `EvidencePackBuilder`, `EvidenceQualityEvaluator`, traces de search runs.  
**No depende solo del prompt:** El backend debe exigir `retrieval_run_id` o decision estructurada de abstencion antes de `AnswerDraft`.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Retrieval Lead.  
**Criterio de aceptacion:** Una consulta juridica normativa o jurisprudencial no produce respuesta final si no existe `EvidencePack` o abstencion justificada.  
**Severidad si se incumple:** Critical.

### P-NN-005 - Fuente recuperada no equivale a fuente valida

**Principio:** Un resultado encontrado en internet no es evidencia valida por el solo hecho de existir.  
**Regla tecnica:** Un `LegalSearchResult` solo puede convertirse en fuente citable si fue abierto o procesado, clasificado, snapshotteado o justificado, extraido en pasajes y evaluado por autoridad, jurisdiccion, extractabilidad y calidad.  
**Mecanismo verificable:** `legal-search-result.schema.json`, `source.schema.json`, `source-policy.md`, estrategia de snapshot/hash, `passage.schema.json`, Source Authority Classifier, Evidence Quality Evaluator.
**No depende solo del prompt:** Las fuentes sin pasaje extraido no pueden aparecer como citas inline ni en `Fuentes utilizadas`.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Retrieval Lead.  
**Criterio de aceptacion:** Un snippet de buscador o URL descubierta con `extraction_status = not_fetched` no puede sostener un claim.  
**Severidad si se incumple:** Critical.

### P-NN-006 - Cita decorativa es fallo critico

**Principio:** Las citas deben sostener afirmaciones; no son bibliografia decorativa.  
**Regla tecnica:** Toda cita inline `[F#:P#]` o `[D#:P#]` debe resolver a un `EvidencePassage` existente, y todo claim critico debe apuntar a citas cuyo pasaje soporte el claim de forma directa o inferencial declarada.  
**Mecanismo verificable:** `citation.schema.json`, `citation-audit.schema.json`, `CitationAuditor`, pruebas de cita rota, fuente final derivada de citas reales.  
**No depende solo del prompt:** El auditor debe fallar si una cita no existe, apunta a fuente inexistente, no aparece en `EvidencePack`, o se lista una fuente no citada.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro AI Orchestration Lead.  
**Criterio de aceptacion:** `citation_audit.overall_status = passed` exige cero citas rotas, cero assertions criticas visibles sin `Claim` y cero claims criticos sin soporte.
**Severidad si se incumple:** Critical.

### P-NN-007 - Vigencia no se presume

**Principio:** La vigencia normativa no se afirma sin evidencia positiva suficiente.  
**Regla tecnica:** Toda fuente normativa debe declarar `validity_status`. La frase "vigente" solo puede usarse si `validity_status = VIGENCIA_CONFIRMADA` y existe evidencia de soporte registrada.  
**Mecanismo verificable:** `validity-statuses.yaml`, `source.schema.json`, `validity-policy.md`, `validity_checks`, tests de frases prohibidas, ClaimVerifier para claims de vigencia.  
**No depende solo del prompt:** El formatter debe bloquear o reescribir frases de vigencia no soportada antes de respuesta final.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Retrieval Lead.  
**Criterio de aceptacion:** Por defecto, normativa recuperada queda en `VIGENCIA_NO_CONFIRMADA` si no se ejecuta o no pasa verificacion de vigencia.  
**Severidad si se incumple:** Critical.

### P-NN-008 - Memoria no es verdad juridica

**Principio:** La memoria de conversacion o caso ayuda a continuidad, pero no sustituye evidencia juridica.  
**Regla tecnica:** `CaseMemory` puede aportar hechos, fuentes historicas, preguntas abiertas y advertencias, pero no puede sostener por si sola una afirmacion normativa, jurisprudencial o de vigencia.  
**Mecanismo verificable:** `case-memory.schema.json`, `memory-policy.md`, Context Assembler con tipos de contexto, CitationAuditor que no acepte memoria como fuente legal externa.  
**No depende solo del prompt:** El ensamblador debe marcar memoria como contexto, no como `EvidenceSource` juridica.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Backend Lead.  
**Criterio de aceptacion:** Una respuesta no puede citar "memoria del caso" como fundamento juridico; debe recuperar o reusar fuentes trazables.  
**Severidad si se incumple:** High.

### P-NN-009 - Documento de usuario no es norma vigente

**Principio:** Un documento subido por el usuario puede probar hechos o texto documental, pero no establece derecho vigente por si mismo.  
**Regla tecnica:** `USER_DOCUMENT` y citas `[D#:P#]` solo pueden sostener claims de hechos, contenido de documento, version contractual, expediente o analisis documental. Claims normativos o de vigencia requieren fuentes externas o abstencion.  
**Mecanismo verificable:** `document-evidence.schema.json`, `source.schema.json`, `claim.schema.json`, `ocr-policy.md`, `document-security-policy.md`, ClaimVerifier por tipo de fuente.  
**No depende solo del prompt:** El auditor debe fallar si un claim `norma`, `vigencia`, `plazo`, `competencia` o `jurisprudencia` se sostiene solo con `[D#:P#]`, salvo que el documento sea tratado explicitamente como objeto de analisis y no como derecho vigente.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Document Processing Lead.  
**Criterio de aceptacion:** "El contrato dice X" puede citar `[D#:P#]`; "la ley vigente exige X" no puede citar solo `[D#:P#]`.  
**Severidad si se incumple:** Critical.

### P-NN-010 - Costo se controla por profundidad, no por degradar veracidad

**Principio:** Los planes limitan profundidad, volumen, OCR, retencion y creditos; no autorizan respuestas falsas, sin citas o sin advertencias.  
**Regla tecnica:** `CostGovernor` asigna budgets por complejidad y plan. Cuando el budget no alcanza, el sistema debe responder parcialmente con advertencia, pedir datos, ofrecer Modo Investigacion o abstenerse; nunca eliminar evidencia, citas o warnings para ahorrar.  
**Mecanismo verificable:** `budgets.yaml`, `cost-budget.schema.json`, `usage-event.schema.json`, `cost-governor-policy.md`, Usage Ledger, tests de budget agotado.  
**No depende solo del prompt:** El orquestador debe recibir limites estructurados y registrar excedentes antes de decidir continuacion, degradacion o abstencion.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Backend Lead/Product Lead.  
**Criterio de aceptacion:** Una respuesta limitada por budget debe declarar la limitacion y no presentar confianza alta si la evidencia quedo incompleta.  
**Severidad si se incumple:** High.

### P-NN-011 - Toda respuesta critica debe poder auditarse

**Principio:** Toda respuesta juridica critica debe poder reconstruirse: input, busquedas, fuentes, modelos, claims, citas, costos, latencias y version.  
**Regla tecnica:** `TraceObject` es obligatorio para cada `FinalAnswer`, y `AnswerVersion` preserva cambios sin sobrescritura silenciosa.  
**Mecanismo verificable:** `trace-object.schema.json`, `answer-version.schema.json`, `model-call.schema.json`, `tool-call.schema.json`, `retrieval-run.schema.json`, `usage-event.schema.json`, trace persistence tests.  
**No depende solo del prompt:** El backend debe crear trazas como datos estructurados, no como texto explicativo generado por el modelo.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Backend Lead.  
**Criterio de aceptacion:** No existe respuesta juridica persistida sin `trace_id`, `answer_id`, `answer_version`, `evidence_pack_ids`, `claims`, `citation_audit` y `cost`.  
**Severidad si se incumple:** Critical.

### P-NN-012 - Todo proveedor externo se encapsula

**Principio:** Ningun proveedor externo debe invadir el core juridico o transaccional.  
**Regla tecnica:** El core usa interfaces internas para modelos, discovery, OCR, fetch, storage, embeddings y workflows; no importa SDKs concretos fuera de adaptadores.  
**Mecanismo verificable:** `provider-policy.md`, `provider-interfaces.md`, boundaries de modulos, tests de import boundaries, feature flags por provider.  
**No depende solo del prompt:** La dependencia se controla por arquitectura de codigo y revisiones de importacion, no por instrucciones al modelo.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Backend Lead.  
**Criterio de aceptacion:** Cambiar OpenAI, discovery provider, OCR provider o storage provider no requiere modificar contratos juridicos ni modelos de dominio centrales.  
**Severidad si se incumple:** High.

### P-NN-013 - Nada critico queda acoplado a un proveedor unico

**Principio:** SerpAPI, proveedor OCR, vector DB, buscador web o proveedor unico no pueden ser dependencia critica acoplada directamente.  
**Regla tecnica:** Las capacidades criticas deben tener interfaces, configuracion, feature flags, timeouts, fallback y mapping de errores. SerpAPI y otros proveedores opcionales solo pueden operar como adaptadores intercambiables.  
**Mecanismo verificable:** `provider-policy.md`, ADR-002, ADR-003, ADR-011, provider registry, dependency review, config validation.  
**No depende solo del prompt:** Las dependencias obligatorias deben estar declaradas en config y ADR; un SDK directo en core es fallo de arquitectura.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Backend Lead.  
**Criterio de aceptacion:** El sistema puede ejecutar modo base con adaptadores oficiales y discovery controlado sin requerir SerpAPI, proveedor OCR externo, Qdrant o buscador unico.  
**Severidad si se incumple:** High.

### P-NN-014 - Seguridad y privacidad desde el inicio

**Principio:** JusNova debe asumirse como sistema con secreto profesional, documentos sensibles y datos de clientes.  
**Regla tecnica:** Toda entidad privada debe tener ownership/tenant; documentos, mensajes y trazas sensibles deben clasificarse; logs no deben contener documentos completos; proveedores externos reciben solo datos necesarios y quedan registrados.  
**Mecanismo verificable:** `privacy-security-policy.md`, `data-classification.yaml`, `entity-ownership-matrix.md`, security checklist, tenant isolation tests, log redaction tests.  
**No depende solo del prompt:** La seguridad se implementa con modelos de datos, permisos, middleware, storage policy y logging controls.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro Security/Backend Lead.  
**Criterio de aceptacion:** Fase 1 no puede crear tablas privadas sin `organization_id`/owner o decision documentada que explique por que no aplica.  
**Severidad si se incumple:** Critical.

### P-NN-015 - No se avanza a produccion sin evaluacion medible

**Principio:** La calidad juridica, de busqueda, citacion, OCR, seguridad y abstencion debe medirse.  
**Regla tecnica:** Beta y mercado requieren evaluation plan, golden dataset spec, metricas, blockers y regresion. Una release que incumple blockers no pasa.  
**Mecanismo verificable:** `evaluation-plan-v0.md`, `initial-golden-dataset-spec.md`, `beta-readiness-gates.md`, `market-readiness-gates.md`, eval runner futuro, CI gates.  
**No depende solo del prompt:** Las metricas se calculan con tests/evals y revision humana registrada, no con opinion del modelo.  
**Dueno futuro de implementacion:** Codex / JusNova Chief Backend Architect, futuro QA/Legal Review Lead.  
**Criterio de aceptacion:** No se declara beta si citation validity, unsupported critical claims, source tier correctness, validity awareness, document grounding o prompt injection resistance no tienen medicion inicial.  
**Severidad si se incumple:** Critical.

## Matriz resumida de enforcement

| Principio | Control primario | Control secundario | Fase minima de implementacion |
|---|---|---|---|
| P-NN-001 | CitationAuditor / ClaimVerifier | Evals juridicas | Fase 2 |
| P-NN-002 | Jurisdiction fields / SourceClassifier | Retrieval filters | Fase 4 |
| P-NN-003 | No-RAG policy / Trace origin | ADR-004 | Fase 0 |
| P-NN-004 | RetrievalPlan / EvidencePack gate | Search Trace | Fase 4 |
| P-NN-005 | Fetch/extract/snapshot gate | SourceAuthorityClassifier | Fase 4 |
| P-NN-006 | CitationAuditor | Answer formatter | Fase 2 |
| P-NN-007 | ValidityStatus / ValidityResolver | Forbidden phrase checks | Fase 4/5 |
| P-NN-008 | Context Assembler | Memory policy | Fase 3 |
| P-NN-009 | Claim source rules | Document policy | Fase 9 |
| P-NN-010 | CostGovernor | UsageLedger | Fase 1 |
| P-NN-011 | TraceObject | AnswerVersion | Fase 3 |
| P-NN-012 | Provider interfaces | Import boundary checks | Fase 1 |
| P-NN-013 | Provider registry/config | Feature flags | Fase 1 |
| P-NN-014 | Tenant ownership | Log redaction/security tests | Fase 1 |
| P-NN-015 | Eval gates | Human review records | Fase 1 |

Para P-NN-007, Fase 4 implementa la clasificacion conservadora y el `ValidityResolver` base junto al Source Registry; Fase 5 integra adapters oficiales y amplía verificacion. Ningun control minimo de vigencia queda diferido a Fase 7.

## Anti-patrones bloqueantes

Estos anti-patrones bloquean aprobacion de Fase 0 o avance de fase si aparecen en diseno o implementacion:

1. "El modelo sabra citar."
2. "La vigencia la inferimos."
3. "Primero integramos SerpAPI directo y despues lo separamos."
4. "Guardemos todo el historial y listo."
5. "Peguemos todo el documento al prompt."
6. "Las fuentes al final las ponemos desde URLs."
7. "La trazabilidad la agregamos despues."
8. "La seguridad se ve cuando haya usuarios."
9. "No necesitamos evals hasta beta."
10. "Si el portal oficial cae, usamos cualquier fuente sin advertir."

## Criterios de aceptacion de esta policy

- Los 15 principios tienen regla tecnica.
- Cada regla tiene dueno futuro de implementacion.
- Cada regla declara mecanismo verificable.
- Cada regla indica por que no depende solo del prompt.
- La policy esta en estado `Accepted`.

## Momento de revision

Revisar al cerrar Fase 0, antes de iniciar Fase 1, y cada vez que un ADR cambie arquitectura, proveedor, evidencia, busqueda, seguridad, costos o evaluacion.
