# Commercial Plans v0

**Estado documental:** Accepted
**Fecha:** 2026-05-24
**Responsable:** Codex / JusNova Chief Backend Architect
**Decision relacionada:** Subfase 0.8 - Cost Governor, planes y presupuestos

## Proposito

Definir los planes comerciales iniciales de JusNova sin convertir precio, cuota o credito en una regla que degrade veracidad juridica.

## Alcance

Aplica a nomenclatura de planes, cuotas mensuales, creditos de investigacion, OCR mensual, documentos activos, retencion y prioridad operativa.

No define facturacion real, Stripe, impuestos, descuentos, trials, metodos de pago, SLA contractual final ni enforcement en runtime.

## Planes v0

| Plan | Codigo canonico | Usuario objetivo | Precio inicial |
|---|---|---|---|
| Plan Profesional | `PROFESIONAL` | Abogado individual | 400 Bs/mes |
| Plan Pro Plus | `PRO_PLUS` | Abogado intensivo | 700-900 Bs/mes |
| Plan Estudio | `ESTUDIO` | Estudios juridicos | 1.500-2.500 Bs/mes |
| Enterprise / Institucional | `ENTERPRISE` | Organizaciones con requisitos especiales | Personalizado |

No se propone plan inferior a `PROFESIONAL` en Fase 0.

## Hipotesis de inclusion

### PROFESIONAL

- 300-400 consultas estandar `SIMPLE` o `MEDIO` al mes como hipotesis inicial a validar.
- 8-12 creditos de investigacion.
- 150-200 paginas OCR por mes.
- Memoria de caso basica.
- Limite moderado de documentos activos.
- Fuentes, citas y trazas auditables.
- Retencion estandar.

### PRO_PLUS

- Mayor numero de consultas mensuales.
- 20-30 creditos de investigacion.
- Mas OCR mensual.
- Mas documentos activos.
- Mayor retencion.
- Prioridad de procesamiento.
- Reportes exportables.

### ESTUDIO

- Multiples usuarios.
- Tenant u organizacion.
- Documentos compartidos.
- Auditoria por usuario.
- Mas creditos.
- Soporte preferente.

### ENTERPRISE

- SLA.
- Despliegue dedicado si corresponde.
- Retencion custom.
- Integraciones.
- Controles avanzados.
- Revision contractual.

## Reglas deterministicas

1. `plan_code` usa codigos neutrales: `PROFESIONAL`, `PRO_PLUS`, `ESTUDIO`, `ENTERPRISE`.
2. No se usa `PROFESIONAL_400`, `BASIC`, `FREE`, `BASIC_100` ni variantes con precio embebido como codigo canonico.
3. Los planes superiores aumentan cuotas mensuales, creditos, OCR mensual, documentos, retencion, usuarios y prioridad.
4. En v0 los planes superiores no alteran los budgets internos por complejidad definidos en `budgets.yaml`.
5. El plan limita cantidad, profundidad, OCR, retencion y creditos; no limita veracidad, citas, advertencias, vigencia, auditoria ni trazabilidad.
6. Toda ejecucion conserva el `plan_code` neutral para auditoria, pero vistas de usuario final no deben mostrar presupuesto interno.
7. Ninguna comunicacion comercial debe describir Evidence Cache como corpus juridico exhaustivo.

## Reglas asistidas por IA

1. El modelo puede explicar diferencias entre planes usando solo esta policy y textos comerciales aprobados.
2. El modelo no puede inventar cuotas definitivas distintas de las hipotesis v0.
3. El modelo no puede ofrecer plan inferior a `PROFESIONAL` salvo nueva decision documentada.
4. El modelo no puede prometer vigencia, cobertura exhaustiva o resultado juridico por efecto de un plan superior.

## Criterios de aceptacion

- `PROFESIONAL` queda reflejado como plan base de 400 Bs/mes.
- Los cuatro `plan_code` canonicos existen y no contienen precio.
- `docs/contracts/usage-event.schema.json` rechaza planes inferiores o no reconocidos.
- `docs/contracts/trace-object.schema.json` usa `plan_code` neutral y no embebe precio ni contrato comercial completo.
- `docs/schemas/budgets.yaml` queda independiente del plan superior en v0.

## Relacion con contratos

- `usage-event.schema.json` registra `plan_code`, `billing_period`, scope y consumo.
- `cost-budget.schema.json` aplica el mismo budget por complejidad a todos los planes en v0.
- `trace-object.schema.json` conserva `plan_code`, `complexity`, `cost_budget_ref` y `cost_budget_version` como escalares cerrados.
- `cost-report.schema.json` sigue siendo reporte tecnico observado, no plan comercial.

## Momento de revision

Revisar al cerrar Subfase 0.8, durante beta con costo real por consulta, al cambiar cuotas comerciales, al agregar trials o descuentos, y antes de publicar precios definitivos.
