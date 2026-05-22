# JusNova Backend Docs

**Estado documental:** Draft  
**Fecha:** 2026-05-22  
**Responsable:** Codex / JusNova Chief Backend Architect  
**Decision relacionada:** Fase 0.0 - Preparacion y gobierno documental

## Proposito

Esta carpeta contiene el paquete rector tecnico de JusNova Backend. Las decisiones vinculantes de Fase 0 viven aqui, versionadas en git, no en conversaciones, archivos externos o entendimiento verbal.

## Estructura

```txt
docs/
  architecture/  Arquitectura, limites modulares, modelo conceptual y decision pack.
  adr/           Architecture Decision Records.
  contracts/     JSON Schemas y contratos internos.
  policies/      Politicas operativas y juridicas del sistema.
  schemas/       Taxonomias YAML/JSON compartidas.
  quality/       Gates, evaluacion, checklists y criterios de aceptacion.
  handoff/       Backlog, riesgos, preguntas y transferencia a Fase 1.
```

## Regla de gobierno

Una decision critica no es vinculante hasta que este documentada en esta carpeta, tenga responsable, estado documental y criterios de aceptacion cuando correspondan.

