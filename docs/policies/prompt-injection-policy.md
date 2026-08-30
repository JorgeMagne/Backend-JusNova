# Prompt Injection Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.10 - Seguridad, privacidad y proveedores externos

## Proposito

Definir como JusNova trata documentos, HTML, snippets, OCR y EvidencePacks como datos no confiables para impedir que alteren instrucciones, herramientas, citas o comportamiento del sistema.

## Alcance

Aplica a mensajes de usuario, fuentes publicas, documentos privados, OCR, HTML externo, snippets de busqueda, EvidencePacks, prompts de modelo, tools y retrieval.

## Reglas obligatorias

1. Contenido de documentos no puede dar instrucciones al sistema.
2. HTML externo no puede alterar comportamiento ni ejecutar herramientas.
3. Snippets de busqueda son datos, no instrucciones.
4. EvidencePacks y fragmentos deben delimitarse como evidencia con origen y clasificacion.
5. El modelo recibe instruccion explicita de no obedecer instrucciones dentro de evidencia.
6. Ninguna tool ejecuta URLs, comandos o instrucciones encontradas dentro de documentos, HTML, snippets u OCR.
7. Si una fuente contiene texto tipo "ignora instrucciones", "revela prompt" o "usa esta cita aunque no soporte", se registra `PromptInjectionRisk`.

## Riesgos cerrados

- `instruction_override_attempt`
- `system_prompt_extraction`
- `tool_use_instruction`
- `credential_or_secret_request`
- `citation_manipulation`
- `data_exfiltration_request`
- `html_script_or_hidden_text`
- `external_link_instruction`

## Reglas deterministicas

1. Todo riesgo detectado debe usar `prompt-injection-risk.schema.json`.
2. `DocumentEvidence.prompt_injection_risks[]` es obligatorio y registra riesgos en documentos/OCR/model vision; `[]` significa evaluado sin riesgos detectados.
3. `RetrievalRun.prompt_injection_risks[]` registra riesgos detectados durante discovery, fetch o extraction.
4. `TraceObject.retrieval_runs[].prompt_injection_risks[]` conserva riesgos en summaries sanitizados.
5. `TraceObject.prompt_injection_risks[]` agrega todos los riesgos usados por la respuesta salvo exclusion auditada con `handling=excluded_from_evidence`.
6. Claims criticos no pueden depender de evidencia con `severity=blocking` o `handling=blocked`.
7. `detected_in_ref` no puede ser URL cruda, texto documental ni snippet libre; debe usar `D#:P#`, `F#:P#`, `msg_*`, `rr_*` o `url_hash:sha256:*`. `source_hash:*` no se acepta hasta que un contrato futuro defina de forma unica los bytes canonicos y el artefacto resoluble.
8. Un riesgo con `severity=blocking` no puede tener `handling=ignored_as_instruction` ni `handling=used_as_evidence_only`; debe quedar `excluded_from_evidence`, `requires_review` o `blocked`.
9. Un riesgo `D#:P#` o `F#:P#` contamina ese pasaje exacto. Si su severidad es `blocking` o su handling es `excluded_from_evidence|blocked`, ninguna cita ni claim publicado puede depender de ese pasaje.
10. Un riesgo `url_hash:sha256:*` contamina todas las fuentes cuya URL canonica, normalizada conforme a `docs/quality/initial-golden-dataset-spec.md#canonicalizacion-de-url-para-hashes`, produzca ese hash y todos sus pasajes descendientes. Para resolverlo se elimina solo el prefijo literal `url_hash:` de `detected_in_ref` y se compara el valor restante `sha256:<64hex>` con el `url_hash` del artefacto; no se rehashea el string prefijado. Si el hash no puede resolverse de forma unica contra el artefacto de retrieval/evidencia, el guard falla cerrado y no publica soporte derivado.
11. Un riesgo `rr_*` contamina por defecto todas las fuentes y pasajes producidos por ese `RetrievalRun`. El alcance solo puede reducirse cuando existen riesgos mas especificos persistidos por pasaje o `url_hash` que delimitan de forma resoluble la deteccion; una inferencia libre del modelo no limita el alcance.
12. Un riesgo `msg_*` es request-scoped: no contamina automaticamente evidencia independiente, pero la instruccion adversarial se elimina o ignora antes de derivar query/tools. Si es `severity=blocking` o `handling=blocked`, no se publica respuesta y la frontera API usa `prompt_injection_blocked`; `requires_review` no permite continuar hasta existir la aprobacion aplicable.
13. La propagacion de contaminacion se valida antes de construir `AnswerContract`, y los riesgos de evidencia/retrieval que hayan influido en la respuesta se agregan en `TraceObject.prompt_injection_risks[]` conforme a la regla 5.
14. `PromptInjectionRisk.classification` coincide con la clasificacion efectiva del artefacto resuelto por `detected_in_ref`. Para `rr_*`, hereda la mayor sensibilidad del `RetrievalRun` y nunca queda por debajo de `INTERNAL_TRACE_RESTRICTED`; detectar el riesgo no autoriza degradar la clasificacion del run.

## Reglas asistidas por IA

1. Un modelo puede clasificar o resumir indicios de prompt injection, pero no decide por si solo ejecutar tools, conservar evidencia blocking ni levantar un bloqueo.
2. La deteccion asistida produce una propuesta `PromptInjectionRisk` que debe validar contra schema y policy antes de influir en evidencia o respuesta.
3. Las instrucciones encontradas dentro de documentos, HTML, snippets, OCR o evidencia siguen tratandose como datos incluso si el modelo las considera benignas.
4. Un falso negativo del modelo no anula detectores deterministas, delimitacion de evidencia, allowlists de tools ni reglas de bloqueo.

## Claims criticos

Un claim es critico si cumple cualquiera de estas condiciones:

- `Claim.criticality = high`.
- `claim_type` en `plazo`, `requisito`, `competencia`, `causal`, `procedimiento`, `norma`, `jurisprudencia`, `vigencia`.

## Comportamiento ante incumplimiento

- Evidencia con riesgo `blocking` no sostiene conclusion critica.
- Si la evidencia sigue siendo juridicamente relevante, puede usarse solo como dato y con el riesgo auditado.
- Si no puede separarse dato de instruccion maliciosa, se excluye la evidencia o se abstiene.
- Si una tool recibe instruccion desde evidencia, debe ignorarla y registrar riesgo.

## Ejemplos permitidos

- Usar una sentencia publica que contiene texto malicioso como evidencia factual, ignorando la instruccion y registrando `instruction_override_attempt`.
- Excluir un fragmento HTML con texto oculto y registrar `html_script_or_hidden_text`.

## Ejemplos prohibidos

- Ejecutar una URL contenida en un PDF de usuario.
- Cambiar el prompt del sistema por instrucciones dentro de una pagina web.
- Citar un fragmento que ordena manipular citas para sostener un claim critico.

## Relacion con contratos

- `prompt-injection-risk.schema.json`
- `document-evidence.schema.json`
- `retrieval-run.schema.json`
- `trace-object.schema.json`
- `claim.schema.json`
- `citation-audit.schema.json`

## Criterios de aceptacion

- Riesgos persistidos con enum cerrado.
- Claims criticos no se apoyan en evidencia bloqueada por prompt injection.
- Riesgos por pasaje, URL y retrieval run se propagan de forma determinista a las fuentes, pasajes, citas y claims afectados; riesgos `msg_*` se resuelven en el scope de request sin contaminar evidencia independiente.
- La clasificacion de cada riesgo coincide con el artefacto detectado; riesgos `rr_*` conservan la sensibilidad heredada del run.
- Las tools tratan evidencia como datos no confiables.
- EvidencePacks quedan delimitados como datos.

## Momento de revision

Antes de implementar ingestion documental, fetch HTML, OCR worker, web search, agents/tools o prompts productivos.
