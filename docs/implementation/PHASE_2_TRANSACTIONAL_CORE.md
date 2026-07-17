# Etapa 2 - Nucleo transaccional

## Estado

- Estado: `DONE`.
- Inicio: 2026-07-16.
- Cierre: 2026-07-17.
- Ambiente: maquina local, Docker Compose y datos sinteticos.
- Baseline contractual: tag inmutable `contracts-v1-alpha.1`.

## Objetivo

Implementar una fuente de verdad transaccional ejecutable sin PSP, facturador,
datos ni dinero real, y demostrar sus garantias de consistencia mediante pruebas
automatizadas y un pipeline Jenkins limpio.

## Tablero de trabajo

| ID | Entregable | Repositorio | Estado |
| --- | --- | --- | --- |
| E2.1 | Toolchain Kotlin/Spring y monolito modular | nexopay-payments-platform | DONE |
| E2.2 | Migracion Flyway y modelo tenant-aware | nexopay-payments-platform | DONE |
| E2.3 | Payment intent, checkout session e idempotencia | nexopay-payments-platform | DONE |
| E2.4 | Maquina de estados y fake PSP determinista | nexopay-payments-platform | DONE |
| E2.5 | Ledger operacional append-only de partida doble | nexopay-payments-platform | DONE |
| E2.6 | Transactional outbox y entrega Kafka | nexopay-payments-platform | DONE |
| E2.7 | Logs, metricas, trazas y health checks | nexopay-payments-platform | DONE |
| E2.8 | Compose, pruebas de contrato y Jenkins real | nexopay-platform-infrastructure | DONE |

## Arquitectura implementada

`nexopay-payments-platform` es un monolito modular desplegable. Sus paquetes
separan API, pagos, checkout, proveedor, ledger, outbox e idempotencia sin
introducir transacciones distribuidas ni microservicios prematuros.

PostgreSQL almacena payment intents, sesiones, intentos de proveedor, registros
de idempotencia, ledger y outbox. Kafka distribuye eventos, pero no es fuente de
verdad. Redis no interviene en decisiones financieras.

El flujo de confirmacion evita mantener una transaccion de base de datos durante
una llamada externa:

1. Una transaccion bloquea el pago, registra un intento y avanza a `PROCESSING`.
2. El adaptador sintetico se invoca fuera de la transaccion.
3. Una nueva transaccion aplica la respuesta y escribe pago, ledger y outbox.
4. Un intento `INITIATED` o `UNKNOWN` se recupera mediante reconciliacion.

## Garantias verificadas

- La API deriva el monto desde una deuda sintetica determinista y nunca acepta
  el monto informado por el cliente.
- `Idempotency-Key`, el hash del request y las constraints detectan replay o
  reutilizacion conflictiva por tenant y operacion.
- Bloqueos `FOR UPDATE`, version del agregado y claves unicas evitan capturas
  concurrentes duplicadas.
- Una captura crea exactamente dos asientos balanceados: cuenta por cobrar al
  PSP y obligacion con el facturador. No representa custodia de fondos.
- Triggers de PostgreSQL rechazan `UPDATE` y `DELETE` del ledger.
- Cada transicion crea `payment.status-changed.v1` en la misma transaccion del
  pago; la entrega Kafka es al menos una vez y exige deduplicar por `eventId`.
- Un timeout queda `UNKNOWN`, nunca se interpreta como rechazo, y puede terminar
  posteriormente en captura.
- Los tokens de checkout se almacenan solo como SHA-256.
- Los logs JSON y las trazas no registran bodies, tokens, PAN, CVV ni secretos.

## PSP sintetico

| Provider | Respuesta inicial | Reconciliacion |
| --- | --- | --- |
| `fake-success` | `CAPTURED` | `CAPTURED` |
| `fake-decline` | `FAILED` | `FAILED` |
| `fake-timeout` | `UNKNOWN` | permanece `UNKNOWN` |
| `fake-late-success` | `UNKNOWN` | `CAPTURED` |

## Criterios de salida

- [x] Un pago simulado recorre creacion, procesamiento y estado final.
- [x] Repetir un comando con la misma clave no duplica pago ni ledger.
- [x] Los timeouts quedan reconciliables y no producen rechazo falso.
- [x] Evento y escritura de dominio son atomicos mediante outbox.
- [x] Pruebas unitarias, integracion y contrato pasan en Jenkins.

## Evidencia

### Payments Platform

- Commit: [`c0c655a`](https://github.com/7yrak/nexopay-payments-platform/commit/c0c655a).
- Aislamiento de jobs programados en pruebas:
  [`3bcfa6e`](https://github.com/7yrak/nexopay-payments-platform/commit/3bcfa6e).
- Integraciones externas siempre ejecutadas en CI:
  [`1f52175`](https://github.com/7yrak/nexopay-payments-platform/commit/1f52175).
- Version `0.1.0-alpha.1` con Java 21, Kotlin 2.3.21, Spring Boot 4.1.0,
  Gradle Wrapper 9.5.1 y dependency locking.
- Migracion Flyway con constraints tenant-aware, ledger inmutable y outbox.
- API v1 de payment intents, checkout sessions, confirmacion y consulta.
- Fake PSP con exito, rechazo, timeout y exito tardio reproducibles.
- Imagen Docker no-root construida y servicio Compose saludable en
  `127.0.0.1:18081`.

### Pruebas

- 7 pruebas unitarias de estados, deuda sintetica, IDs y proveedor.
- 12 pruebas de integracion y contrato sobre PostgreSQL y Kafka reales.
- 19 pruebas totales, 0 fallos.
- Concurrencia de 16 comandos sobre la misma clave produce un solo pago.
- Replay conflictivo, caida de Kafka, recuperacion desde `PROCESSING`, timeout,
  respuesta tardia, balance e inmutabilidad del ledger quedan cubiertos.
- El contrato se valida contra una extraccion exacta del tag
  `contracts-v1-alpha.1`, no contra archivos de trabajo mutables.

### Infraestructura y CI

- Integracion inicial:
  [`212165d`](https://github.com/7yrak/nexopay-platform-infrastructure/commit/212165d).
- Bootstrap determinista de jobs:
  [`d8ff00b`](https://github.com/7yrak/nexopay-platform-infrastructure/commit/d8ff00b).
- Publicacion JUnit:
  [`e2b869d`](https://github.com/7yrak/nexopay-platform-infrastructure/commit/e2b869d).
- Jenkins crea `nexopay-payments-platform-local` mediante Configuration as Code.
- El pipeline usa un checkout limpio, ejecuta pruebas unitarias, integracion y
  contrato, construye el JAR y lo archiva con fingerprint.
- Build Jenkins `nexopay-payments-platform-local #4`: `SUCCESS` en 90 segundos,
  con 19 pruebas, 0 fallos y 0 omisiones.
- Readiness `UP`, metricas Prometheus expuestas y trazas visibles en Jaeger con
  `traceId` y `spanId` correlacionados en logs estructurados.

## Incidencias resueltas

- El puerto host `8081` estaba ocupado por un proceso ajeno a NexoPay; Compose
  publica Payments Platform en `18081` y conserva `8081` dentro del contenedor.
- La primera carga de un Job DSL nuevo podia terminar despues del escaneo de
  jobs de Jenkins; el runner espera el job y reinicia el controlador una vez
  cuando necesita aplicar la nueva configuracion.
- El primer build detecto una carrera entre el scheduler y el test de caida de
  Kafka; los jobs programados se separaron de sus servicios y se deshabilitan
  de forma explicita en el perfil de pruebas.
- Jenkins no incluia el publisher JUnit; se fijo el plugin y se validaron los
  19 resultados publicados.
- Gradle podia restaurar pruebas externas desde build cache; `integrationTest`
  ahora siempre se ejecuta contra PostgreSQL y Kafka.
- El exporter OTLP de metricas no tenia colector local; se deshabilito y se
  mantuvo Prometheus. Las trazas OTLP continuan hacia Jaeger.

## Limites y riesgos abiertos

- Solo se permiten identidades, deudas y pagos sinteticos en loopback.
- Los headers Alpha no reemplazan OAuth2/OIDC y quedan bloqueados si se
  deshabilita el perfil local.
- El ledger es operacional; NexoPay no recibe, custodia ni liquida dinero.
- No existen devoluciones, disputas, conciliacion bancaria ni PSP real.
- PostgreSQL, Kafka y la aplicacion single-node no demuestran SLO, RTO, RPO,
  alta disponibilidad ni capacidad de 2.000 TPS.
- Los gates productivos PG-01 a PG-09 permanecen abiertos.

## Siguiente accion

Iniciar la etapa 3 con Checkout Web, SDK Web Component e iframe seguro, Payment
Portal y pruebas Playwright end-to-end sobre este nucleo sintetico.
