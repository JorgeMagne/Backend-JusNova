# Prompt Injection Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.10 - Seguridad, privacidad y proveedores externos

## Proposito

Definir como JusNova trata documentos, HTML, snippets, OCR y EvidencePacks como datos no confiables para impedir que alteren instrucciones, herramientas, citas o comportamiento del sistema.

## Alcance

Aplica a fuentes publicas, documentos privados, OCR, HTML externo, snippets de busqueda, EvidencePacks, prompts de modelo, tools y retrieval.

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
7. `detected_in_ref` no puede ser URL cruda, texto documental ni snippet libre; debe usar refs o hashes cerrados.
8. Un riesgo con `severity=blocking` no puede tener `handling=ignored_as_instruction` ni `handling=used_as_evidence_only`; debe quedar `excluded_from_evidence`, `requires_review` o `blocked`.

## Claims criticos

Un claim es critico si cumple cualquiera de estas condiciones:

- `Claim.criticality = high`.
- `claim_type` en `norma`, `vigencia`, `plazo`, `competencia`, `jurisprudencia`.

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
- Las tools tratan evidencia como datos no confiables.
- EvidencePacks quedan delimitados como datos.

## Momento de revision

Antes de implementar ingestion documental, fetch HTML, OCR worker, web search, agents/tools o prompts productivos.
