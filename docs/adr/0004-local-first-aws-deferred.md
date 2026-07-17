# ADR 0004 - Desarrollo local y AWS diferido

- Estado: Accepted para desarrollo y Alpha interna.
- Fecha: 2026-07-16.

## Contexto

NexoPay aun no tiene clientes oficiales ni necesita procesar datos o dinero real.
La maquina Linux actual permite ejecutar un entorno de desarrollo limitado. Una
plataforma productiva ahora agregaria costo y operacion sin validar contratos.

## Decision

Desarrollar y ejecutar la Alpha interna en la maquina local mediante Docker
Compose y datos sinteticos. No publicar servicios ni almacenar datos reales.

Cuando exista el primer cliente oficial se evaluara AWS como primera opcion para
produccion. La decision productiva exigira arquitectura multi-zona, datos
administrados, seguridad, backups, recuperacion, costos y SLO aprobados.

## Consecuencias

- La etapa 1 puede comenzar sin cuenta cloud.
- Docker Compose precede a Kubernetes local.
- Los benchmarks locales solo validan comportamiento, no capacidad productiva.
- Infraestructura debe evitar supuestos que impidan migrar a AWS.
- Ningun cliente ni proveedor puede interpretar la maquina local como ambiente
  productivo o de certificacion.
