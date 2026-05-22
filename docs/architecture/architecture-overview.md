# Architecture Overview

**Estado documental:** Accepted  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** ADR-001 - High-Assurance Modular Core + Distributed Execution Layer

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
        |-- Cost Governor Module
        |-- Usage Ledger Module
        |-- Security / Tenancy Module
        |-- Audit / Trace Module
        |
        |--------------------------------|
        |                                |
        v                                v
Workflow / Worker Layer             Streaming Layer
        |                                |
        |-- Live Retrieval Worker        |-- status events
        |-- Official Adapter Worker      |-- source_found events
        |-- Fetch / Snapshot Worker      |-- verification events
        |-- Extraction Worker            |-- answer deltas after gates
        |-- OCR Worker
        |-- Indexing Worker
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
```

## Core backend

El core contiene reglas transaccionales, contratos juridicos, seguridad, trazabilidad, costos, orquestacion de respuestas y persistencia. El core no debe depender directamente de SDKs de proveedores externos; usa interfaces internas.

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
- `StorageProvider`
- `WorkflowProvider`

## Regla de no conclusion prematura

El core no debe producir una respuesta juridica critica si no existe uno de estos estados:

- Evidence Pack con calidad suficiente.
- Evidence Pack parcial con advertencias.
- Abstencion estructurada.
- Solicitud de dato faltante.
- Propuesta de Modo Investigacion.

## Relacion con ADR-001

ADR-001 acepta esta arquitectura como base. `module-boundaries.md` define limites concretos de modulos, workers y criterios de extraccion futura.

