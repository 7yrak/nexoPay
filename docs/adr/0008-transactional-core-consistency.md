# ADR 0008 - Consistencia del nucleo transaccional

- Estado: Accepted para Alpha sintetica.
- Fecha: 2026-07-17.

## Contexto

Payment Core debe mantener idempotencia, trazabilidad contable y eventos
coherentes aun cuando un proveedor o Kafka fallen. La etapa 2 no necesita la
complejidad operacional de transacciones distribuidas ni de separar cada modulo
en un servicio.

NexoPay orquesta cobros sin recibir, custodiar o liquidar fondos. Aun asi,
necesita un registro operacional que permita demostrar que cada resultado de
pago genero una sola obligacion con el facturador.

## Decision

- Implementar Payments Platform como monolito modular Kotlin/Spring desplegable.
- Usar PostgreSQL como unica fuente de verdad para pago, intentos de proveedor,
  idempotencia, ledger y outbox.
- Reservar idempotencia por tenant, operacion y clave, asociada al hash canonico
  del comando.
- Serializar transiciones concurrentes con row locks, version del agregado y
  constraints unicas.
- Invocar proveedores fuera de la transaccion y persistir estados ambiguos
  `INITIATED` o `UNKNOWN` para reconciliarlos.
- Registrar un ledger operacional append-only de partida doble. Una captura
  debita la cuenta por cobrar al PSP y acredita la obligacion con el facturador;
  estos asientos no representan dinero custodiado por NexoPay.
- Crear el evento de cambio de estado en un outbox dentro de la misma transaccion
  que modifica el pago y el ledger.
- Publicar desde el outbox hacia Kafka con semantica al menos una vez. Cada
  consumidor debe deduplicar por `eventId` y no asumir exactly-once end-to-end.
- Mantener Redis fuera de decisiones financieras canonicas.

## Consecuencias

- No puede existir una escritura de pago confirmada sin su evento persistido.
- Una caida de Kafka no revierte el pago; deja un evento pendiente reintentable.
- Es posible repetir una entrega despues de un ACK incierto, por lo que inbox o
  deduplicacion equivalente es obligatoria en consumidores.
- Un timeout de proveedor no se convierte en rechazo y requiere reconciliacion.
- La base de datos concentra la consistencia de la Alpha y puede escalarse antes
  de justificar particionamiento o extraccion de servicios.
- Corregir el ledger requerira asientos compensatorios; `UPDATE` y `DELETE`
  quedan rechazados por la base de datos.

## Alternativas descartadas

- Transaccion distribuida PostgreSQL/Kafka: aumenta acoplamiento y no elimina la
  necesidad de consumidores idempotentes.
- Publicar Kafka despues del commit sin outbox: permite perder eventos.
- Ledger mutable: impide conservar una historia auditable.
- Marcar timeout como fallo: puede duplicar un pago aceptado tardiamente.
- Separar ledger, pagos y outbox en microservicios: introduce coordinacion
  distribuida sin una necesidad de escala medida en la Alpha.
