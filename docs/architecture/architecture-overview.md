# Architecture Overview

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** ADR-001 - High-Assurance Modular Core + Distributed Execution Layer; ADR-002 - Stack Backend And Infrastructure; ADR-011 - Security, Privacy and Provider Boundaries
**Enmiendas:** Subfase 0.10 - provider registry, data classification and raw access boundaries; Subfase 0.11 - conceptual domain model, entity ownership and API draft boundaries

## Proposito

Este documento define la vista arquitectonica base que respalda ADR-001. No implementa backend; fija la forma que debe respetar Fase 1 y las fases posteriores.

## Decision de arquitectura

JusNova usara un nucleo backend modular de alta garantia con una capa distribuida de ejecucion para trabajos pesados, lentos, fallables o reintentables.

```txt
Frontend Next.js / React
        |
        v
API Gateway / Backend Core FastAPI
        |
        |-- API Gateway / Request Boundary Module
        |-- Identity / Membership Module
        |-- Conversation Module
        |-- Case Memory Module
        |-- Legal Orchestration Module
        |-- Live Legal Search Module
        |-- Source Registry Module
        |-- Evidence Module
        |-- Answer Contract Module
        |-- Citation Auditor Module
        |-- Claim Verification Module
        |-- Document Registry Module
        |-- Plan / Subscription Module
        |-- Cost Governor Module
        |-- Usage Ledger Module
        |-- Error Handling Module
        |-- Security / Tenancy Module
        |-- Audit / Trace Module
        |
        |--------------------------------|
        |                                |
        v                                v
Workflow / Worker Layer             Streaming Layer
        |                                |
        |-- Live Retrieval Worker        |-- run_queued / run_started
        |-- Official Adapter Worker      |-- retrieval_started / retrieval_progress
        |-- Fetch / Snapshot Worker      |-- document_processing_required
        |-- Extraction Worker            |-- evidence_ready / run_failed
        |                                |-- answer_final / answer_blocked
        |-- OCR Worker
        |-- Indexing Worker
        |-- Provider Audit Worker
        |-- Raw Access Audit Worker
        |-- Evaluation Runner
        |
        v
Data / Retrieval Infrastructure
        |
        |-- PostgreSQL
        |-- Redis
        |-- OpenSearch
        |-- S3-compatible Object Storage
        |-- Observability Stack
        |
        v
External Controlled Providers
        |
        |-- Model Provider
        |-- Official Bolivian Sources
        |-- Search Discovery Providers
        |-- Optional Licensed Legal Sources
        |-- OCR / Embedding / Storage Providers behind registry
```

## Core backend

El core contiene reglas transaccionales, contratos juridicos, seguridad, trazabilidad, costos, orquestacion de respuestas y persistencia. El core no debe depender directamente de SDKs de proveedores externos; usa interfaces internas.

## Modelo conceptual y API draft

Subfase 0.11 agrega documentos conceptuales para que Fase 1 pueda crear migraciones iniciales y endpoints sin inventar estructura:

- `domain-model.md` define entidades, IDs, relaciones, cardinalidades y reglas de refs locales.
- `entity-ownership-matrix.md` define tenant, owner, clasificacion, raw access aplicable y comportamiento de borrado por entidad.
- `api-draft-v0.md` define endpoints minimos, visibilidad, ciclo de vida de runs y uso obligatorio de `ErrorEnvelope`.
- `error-envelope.schema.json` define errores seguros, codigos cerrados, metadata tipada y restricciones contra contenido sensible.

Estos documentos son contractuales/conceptuales; no implementan migraciones, routers, OpenAPI formal, auth runtime ni workers.

## Worker layer

Los workers ejecutan tareas que no deben bloquear el core:

- busqueda viva lenta;
- llamadas a adaptadores oficiales;
- fetch, render y snapshot de fuentes;
- extraccion HTML/PDF;
- OCR;
- indexacion;
- revalidacion;
- evaluaciones;
- rebuilds y tareas reintentables.

## Streaming layer

El streaming muestra progreso, no conclusiones juridicas prematuras. Una respuesta sustantiva solo puede emitirse despues de que existan Evidence Packs o una decision estructurada de abstencion.

## Infraestructura base

- PostgreSQL es fuente transaccional.
- Redis mantiene estado efimero, locks, rate limits y circuit breakers.
- OpenSearch cubre busqueda lexical, filtros y busqueda hibrida inicial si la evaluacion lo justifica.
- Object storage S3-compatible guarda documentos, snapshots, PDFs, imagenes renderizadas y artefactos de OCR.
- Observabilidad registra trazas, metricas y logs estructurados.

## Proveedores externos

Los proveedores externos quedan encapsulados por interfaces:

- `ModelProvider`
- `SearchDiscoveryProvider`
- `OfficialSourceAdapter`
- `FetchProvider`
- `ExtractionProvider`
- `OCRProvider`
- `EmbeddingProvider`
- `StorageProvider`
- `WorkflowProvider`
- `SnapshotProvider`
- `LegalRankingProvider`

La lista canonica coincide con `provider-policy.md`, `provider-interfaces.md` y `provider-registry.yaml`. Los alias historicos son `SourceFetcher = FetchProvider` y `EvidenceExtractor = ExtractionProvider`.

Toda llamada externa a provider debe resolver contra `provider-registry.yaml`, respetar `data-classification.yaml`, declarar `training_use_allowed = false` y producir `ProviderCallAudit`.

## Regla de no conclusion prematura

El core no debe producir una respuesta juridica critica si no existe uno de estos estados:

- Evidence Pack con calidad suficiente.
- Evidence Pack parcial con advertencias.
- Abstencion estructurada.
- Solicitud de dato faltante.
- Propuesta de Modo Investigacion.

## Relacion con ADR-001

ADR-001 acepta esta arquitectura como base. `module-boundaries.md` define limites concretos de modulos, workers y criterios de extraccion futura. Subfase 0.11 ancla esos limites al modelo de dominio, la matriz de ownership y el API draft que Fase 1 debe respetar.
