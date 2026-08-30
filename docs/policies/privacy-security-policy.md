# Privacy And Security Policy

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.10 - Seguridad, privacidad y proveedores externos

## Proposito

Definir los limites minimos de privacidad, seguridad, acceso raw, logs, storage, documentos y visibilidad que condicionan Fase 1.

## Alcance

Aplica a conversaciones, mensajes, documentos, OCR, evidencia documental, memoria de caso, trazas, llamadas de modelo, tools, provider audit, usage ledger, storage y workflows.

## Reglas obligatorias

1. Toda entidad privada requiere `organization_id` o owner tecnico equivalente.
2. Todo documento privado tiene tenant, owner tecnico, version y hash de version.
3. Logs, traces, usage, provider audit, model calls y tool calls no guardan documentos completos, OCR completo, HTML bruto, prompts completos, mensajes completos ni salidas completas.
4. Secrets no se hardcodean en codigo, docs ejecutables, configs versionadas ni ejemplos reales.
5. Object storage usa buckets privados, rutas no adivinables, URLs firmadas de corta vida y claves sin nombres humanos de archivo, cliente o caso.
6. Todo archivo subido requiere allowlist de tipo/tamano, MIME sniffing, control de extension, validacion de contenido y estado cerrado de rechazo.
7. Documentos, HTML, snippets, OCR y evidencia externa son datos no confiables.
8. Eliminar documento privado implica borrar o tombstonear objeto original, OCR, fragmentos, embeddings, indices, snapshots privados y derivados.
9. `TraceObject` y `AnswerVersion` pueden conservar hashes, refs y marcas de eliminacion para auditoria; no conservan contenido raw.
10. Todo acceso raw o elevado se registra con `RawAccessEvent`.
11. Todo payload o artefacto derivado hereda la clasificacion con mayor `sensitivity_rank` de sus entradas, incluyendo snapshots, embeddings, trazas, indices, queries reformuladas, provider payloads y cualquier derivado persistido. La unica excepcion cerrada son los value objects/registros contractualmente metadata-only identificados en `entity-ownership-matrix.md` (`CitationAudit`, `ModelCall`, `ToolCall` y summaries de retrieval); la excepcion clasifica ese objeto sanitizado como `INTERNAL_TRACE_RESTRICTED`, no el recurso referenciado ni un contenedor como `TraceObject`.

## Matriz de visibilidad

| Actor | Puede ver | No puede ver por defecto | Acceso raw |
|---|---|---|---|
| `end_user` | Respuestas propias, fuentes visibles, advertencias, documentos propios, resumen de trazas | Provider audit interno, prompts completos, mensajes de otros usuarios, documentos de otros usuarios | No |
| `organization_admin` | Uso agregado, plan, miembros, configuracion del tenant | Documentos o mensajes de otros usuarios sin permiso futuro explicito | No |
| `support` | `SUPPORT_VIEW` redacted: errores, latencia, refs, estado, plan neutral | Raw prompts, documentos, mensajes, OCR completo, model outputs | Solo como `incident_responder` o `security_auditor` mediante `RawAccessEvent` y `RAW_INCIDENT_ACCESS` |
| `security_auditor` | Auditoria interna con hashes, refs, provider audit y eventos de acceso | Contenido raw sin evento aprobado | Si, mediante `RawAccessEvent` |
| `service_worker` | Minimo necesario por job y allowlist de clasificacion | Navegacion libre por tenant o documentos no relacionados | Solo con `access_role=system_job` cuando el job lo exige |
| `provider` | Payload minimo permitido por `provider_family_rules` | Clases fuera de allowlist, documentos completos salvo `StorageProvider` autorizado | No aplica; recibe payload filtrado |

## Reglas deterministicas

1. `RawAccessEvent.actor_type=support` no puede usar `access_role=support_operator`.
2. `SUPPORT_VIEW` no es raw access y no se registra como `RawAccessEvent`.
3. `RawAccessEvent.approved_by_ref` no puede ser igual a `actor_ref`.
4. `RawAccessEvent.expires_at`, si existe, debe ser posterior a `accessed_at`.
5. `RawAccessEvent.resource_type=document_evidence` debe resolver a `DocumentEvidence` con mismo `organization_id`, `document_evidence_id`, `document_id`, `document_version_id` y `passage_ref`.
6. Si `resource_type != document_evidence`, `document_id`, `document_version_id` y `passage_ref` deben estar ausentes o ser `null`.
7. Cualquier log que contenga `raw_prompt`, `raw_output`, `document_text`, `full_document`, `ocr_full_text`, `html_raw`, `user_message` o mensaje completo falla revision de seguridad salvo que aparezca como ejemplo invalido.
8. `provider_registry.training_use_allowed` y `ProviderCallAudit.training_use_allowed` deben ser `false` en v0.10.
9. La herencia de clasificacion por `sensitivity_rank` se calcula antes de redaccion o minimizacion; la redaccion puede reducir el payload, pero no permite declarar una clase menos sensible salvo la excepcion cerrada metadata-only de la regla obligatoria 11. Agregar contenido, texto derivado o una propiedad libre invalida esa excepcion y obliga a heredar.
10. Los objetos de contratos sanitizados usados en logs, trazas, evidencia o respuestas deben ser cerrados con `additionalProperties=false`, salvo mapas tipados aprobados y documentados. Un objeto sin limites se considera capaz de transportar la denylist raw aunque esas claves no aparezcan declaradas.
11. Las keys de idempotencia son credenciales de repeticion de una mutacion: solo se persiste/loguea su hash `sha256:*`; el header o valor raw no entra en mensajes, eventos, trazas, errores ni provider payloads.
12. Los errores, warnings y notas de auditoria persistidos usan codigos internos cuando existe taxonomia y texto sanitizado de una linea, entre 1 y 240 caracteres, sin controles C0/C1 ni separadores Unicode de linea. Esto aplica como minimo a `Source`, `EvidencePassage`, `EvidenceQuality`, `LegalSearchResult`, `Citation`, `CitationAudit`, `AnswerContract`, `TraceObject` y `RetrievalRun`; los arrays no admiten duplicados. Ninguno interpola excepciones, stacks, status lines, URLs, queries, snippets, documentos ni respuestas raw de providers.
13. `RawAccessEvent.classification` debe coincidir exactamente con la clasificacion resuelta del recurso accedido. Para `resource_type=trace|retrieval_run`, el valor hereda la mayor sensibilidad de sus entradas y nunca puede quedar por debajo de `INTERNAL_TRACE_RESTRICTED`.
14. IDs, refs, hashes, codigos y valores de fecha con `pattern` se validan como valor completo y rechazan `CR`, `LF`, `U+2028`, `U+2029` o whitespace terminal no declarado. Las regex contractuales no dependen de `$` como unica garantia de fin de cadena.
15. Todo string variable de un contrato aceptado declara `minLength: 1`/`maxLength` y todo array declara `maxItems`, conforme a los perfiles de `docs/contracts/README.md`. `format` no sustituye el rechazo estructural de string vacio. API, serializers y persistencia aplican las mismas cotas o unas mas estrictas antes de reservar recursos; truncar, recortar o aceptar primero para corregir despues esta prohibido.
16. Todo endpoint JSON aplica un limite de body en bytes antes del parseo y rechaza exceso con `payload_too_large` cuando ese codigo esta permitido por el endpoint. El endpoint de mensajes usa el perfil exacto de `api-draft-v0.md`; ninguna compresion, encoding o capa proxy puede eludir la medicion sobre bytes decodificados.
17. Ningun endpoint multipart/binario se habilita sin un limite entero positivo y una allowlist de tipo versionados. Proxy y aplicacion cuentan el stream decodificado con el mismo limite o uno mas estricto; el exceso se aborta antes de reservar IDs, crear shells, cuarentenas u objetos, y cualquier fragmento parcial se elimina. `Content-Length`, nombre, extension, MIME declarado o `size_bytes` del cliente nunca sustituyen la medicion y el sniffing del servidor.

## Reglas asistidas por IA

1. Un modelo puede proponer redacciones, summaries o clasificaciones, pero no autoriza acceso raw, cambia tenant, reduce sensibilidad ni decide retencion/borrado.
2. Toda salida asistida pasa por validadores deterministas de ownership, clasificacion, minimizacion y denylist antes de persistirse, loguearse o enviarse a un provider.
3. El modelo no recibe secrets ni material raw fuera de una ruta explicitamente autorizada y auditada; una sugerencia de redaccion no sustituye `RawAccessEvent` ni controles de acceso.

## Comportamiento ante incumplimiento

- Si una accion requiere exponer raw material y no existe `RawAccessEvent` valido, se bloquea.
- Si un proveedor no esta en registry o no permite la clase de datos, la llamada queda `policy_blocked`.
- Si un documento no cumple validacion minima, no entra a OCR, EvidencePack ni memoria.
- Si un borrado documental no puede borrar un derivado, debe registrarse tombstone y razon cerrada.

## Relacion con contratos

- `raw-access-event.schema.json` define auditoria raw/elevada.
- `provider-call-audit.schema.json` define auditoria de proveedores.
- `data-classification.yaml` define clases y reglas por provider family.
- `data-classification.yaml` define el orden de sensibilidad usado para herencia de clasificacion en derivados.
- `document-evidence.schema.json`, `message.schema.json`, `source.schema.json`, `trace-object.schema.json` y `usage-event.schema.json` no deben almacenar material raw fuera de su responsabilidad primaria.

## Criterios de aceptacion

- Se define que ve cada actor.
- Se define como se audita acceso raw.
- Se prohiben raw prompts, documentos completos, mensajes completos y OCR completo en logs/traces/provider audit.
- Se prohíbe persistir o registrar keys de idempotencia raw; retries solo usan hashes tenant-scoped.
- Se cierran las superficies de objetos sanitizados para impedir que metadata libre reintroduzca material raw bajo otra clave.
- Se cierran errores, warnings y notas de auditoria a texto sanitizado, acotado, de una linea, sin controles y sin duplicados para impedir que excepciones o respuestas externas reintroduzcan contenido raw o secuencias de terminal/renderizado.
- Los patrones contractuales rechazan terminadores de linea terminales en IDs, refs, hashes, codigos y fechas; no existe una variante normalizada silenciosamente despues de validar.
- `RawAccessEvent` conserva la clasificacion efectiva del recurso, incluidos `TraceObject` y `RetrievalRun` confidenciales, sin degradarla a una clase fija de traza.
- Se define borrado/tombstone de documentos y derivados.
- Fase 1 puede crear tablas y workers con ownership, logging y storage compatibles.

## Momento de revision

Antes de implementar auth, soporte, storage, OCR worker, provider SDKs o retencion productiva.
