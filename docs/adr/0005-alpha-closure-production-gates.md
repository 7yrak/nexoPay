# ADR 0005 - Cierre Alpha separado de autorizacion productiva

- Estado: Accepted.
- Fecha: 2026-07-16.

## Contexto

NexoPay no tiene cliente, convenio ESVAL, PSP contratado ni plataforma
productiva. La Alpha interna puede avanzar con datos sinteticos, pero opiniones
legales, PCI, contratos y certificaciones no pueden fabricarse ni sustituirse
por implementacion tecnica.

## Decision

Cerrar la etapa 0 para el alcance de Alpha interna cuando las decisiones de
ingenieria esten acotadas, los riesgos documentados y toda dependencia externa
se registre como gate bloqueante en `PRODUCTION_GATES.md`.

Una etapa de descubrimiento `DONE` significa que no quedan ambiguedades para el
siguiente trabajo sintetico. No significa que NexoPay este autorizado, seguro o
listo para produccion. Los gates aplicables deben pasar antes de datos, dinero o
credenciales reales.

## Consecuencias

- Las etapas 1 a 6 pueden desarrollar con simuladores y datos ficticios.
- Etapa 7 no puede habilitar PSP real sin PG-01, PG-02 y PG-04.
- Un adaptador `esval` no puede existir sin PG-03.
- Staging representativo y piloto requieren los gates restantes.
- El plan y la bitacora deben declarar el alcance al marcar etapa 0 `DONE`.
