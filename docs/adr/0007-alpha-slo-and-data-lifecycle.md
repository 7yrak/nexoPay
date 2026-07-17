# ADR 0007 - Baseline de SLO y ciclo de datos para Alpha

- Estado: Accepted como objetivo de ingenieria, no SLA contractual.
- Fecha: 2026-07-16.

## Contexto

La arquitectura necesita objetivos cuantificados para modelar capacidad y
recuperacion, aunque aun no exista cliente, cloud ni impacto comercial validado.
La Alpha local usa datos sinteticos y no ofrece disponibilidad.

## Decision

Adoptar como baseline de diseno futuro:

- Checkout API/Payment Core y Checkout Web: 99,95% mensual.
- Management API/Portal: 99,9% mensual.
- Creacion/consulta interna p95 menor a 500 ms, excluyendo terceros.
- RTO 60 minutos y RPO 5 minutos como objetivos iniciales de desastre.
- Cero tolerancia a ledger duplicado o inconsistente.
- Capacidad objetivo 10M pagos/dia, 2.000 TPS peak y prueba a 4.000 TPS.

Para Alpha, la retencion maxima y eliminacion siguen
`ALPHA_DATA_RETENTION.md`. No existe SLA, RTO o RPO local.

## Consecuencias

- El diseno puede avanzar con presupuestos de latencia y consistencia claros.
- Los valores productivos solo se aprueban al medir costo, carga, backup y
  restore mediante PG-06/PG-07.
- La retencion de datos reales permanece bloqueada por PG-05.
- Incumplir integridad financiera no se compensa con error budget.
