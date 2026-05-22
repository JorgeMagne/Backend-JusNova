# ADR-002 - Stack Backend And Infrastructure

**Estado:** Accepted  
**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Stack base para Fase 1 y evolucion posterior

## Contexto

JusNova combina IA, recuperacion, documentos, OCR, busqueda, trazabilidad, evaluacion y seguridad. El stack debe favorecer contratos claros, pruebas, procesamiento documental y operacion gradual.

## Problema

El stack debe permitir construir rapido sin encerrar el sistema en proveedores, bases especializadas prematuras o una plataforma operacional demasiado pesada para el arranque.

## Restricciones

- Backend centrado en IA, documentos y retrieval.
- Necesidad de OpenAPI y streaming.
- Persistencia transaccional fuerte.
- Busqueda lexical y filtros juridicos.
- Tareas reintentables.
- Observabilidad desde Fase 1.

## Opciones consideradas

1. NestJS como core principal.
2. Python/FastAPI/Pydantic como core principal.
3. Base vectorial dedicada como dependencia inicial.
4. OpenSearch como busqueda inicial lexical/filtros/hibrida evaluable.
5. Orquestacion de contenedores avanzada desde el inicio.
6. Despliegue simple con servicios gestionados o contenedores hasta que haya presion operativa real.

## Decision

- Backend: Python, FastAPI y Pydantic.
- Base transaccional: PostgreSQL.
- Cache, locks y circuit breakers: Redis.
- Busqueda e indexacion inicial: OpenSearch.
- Object storage: interfaz `StorageProvider` S3-compatible.
- Workflows: Temporal como meta profesional; cola durable inicial permitida si Temporal bloquea Fase 1.
- Observabilidad: OpenTelemetry, logs estructurados y request/correlation IDs.

## Justificacion

Python ofrece el ecosistema mas fuerte para IA, OCR, PDF, retrieval y evaluacion. FastAPI entrega contratos HTTP claros y streaming. PostgreSQL cubre verdad transaccional. Redis cubre estado efimero. OpenSearch permite BM25, filtros y busqueda hibrida inicial sin introducir una base adicional antes de medir. Storage S3-compatible separa archivos grandes de la base transaccional.

## Dependencias posteriores

- Fase 1 debe crear settings, migraciones, logging y health checks.
- Fase 1 debe definir `StorageProvider` sin fijar un proveedor unico.
- Fase 1 debe preparar docker compose o equivalente local para PostgreSQL, Redis y OpenSearch.
- Fases posteriores revisaran Temporal y busqueda hibrida con metricas.

## No afirma todavia

- No afirma que Temporal se use obligatoriamente desde el primer sprint.
- No afirma que una base vectorial dedicada sea necesaria para lanzamiento.
- No afirma que Kubernetes sea requisito de beta o mercado.
- No afirma que un proveedor S3-compatible especifico quede elegido.

## Riesgos

- Python puede requerir disciplina fuerte de tipos y boundaries.
- OpenSearch puede no alcanzar objetivos vectoriales futuros.
- Temporal puede elevar carga operacional.
- Object storage mal elegido puede afectar costo, licencia u operacion.

## Mitigaciones

- Usar Pydantic, type checking, tests y boundaries.
- Evaluar retrieval antes de introducir otra base.
- Permitir cola durable inicial con ruta clara a Temporal.
- Mantener `StorageProvider` e investigar licencias/costos antes de produccion.

## Criterios de aceptacion

- Cada tecnologia tiene proposito especifico.
- Cada alternativa importante fue considerada.
- Redis no es fuente de verdad.
- OpenSearch no se trata como solucion permanente sin evaluacion.
- La adopcion de Kubernetes requiere metrica operacional, no preferencia.

## Momento de revision

Revisar al cerrar Fase 1, antes de beta, y cuando las metricas de retrieval, latencia, costos u operacion indiquen que OpenSearch, Temporal o storage requieren cambio.

## Consecuencias

Fase 1 debe iniciar con FastAPI, Pydantic, PostgreSQL, Redis, OpenSearch, storage abstracto y observabilidad basica.

