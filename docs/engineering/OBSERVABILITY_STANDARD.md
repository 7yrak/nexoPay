# Estandar de observabilidad

## Objetivo

Definir telemetria interoperable, util para operar pagos y segura frente a datos
personales, secretos y alcance PCI. Aplica a backends, workers, frontends,
conectores y jobs de plataforma.

## Correlacion y trazas

- Propagar W3C Trace Context mediante `traceparent` y `tracestate`.
- Aceptar `X-Correlation-Id` valido o generar un ID opaco; devolverlo en la
  respuesta y copiarlo a eventos.
- Mantener `correlation_id` durante todo el flujo de negocio y `causation_id`
  para relacionar comando/evento causal.
- Usar OpenTelemetry SDK/OTLP. No inventar headers de trace por servicio.
- El baggage no contiene tenant, RUT, numero de cliente, monto ni secretos.

## Logs estructurados

Formato JSON, UTF-8, un evento por linea. Campos minimos:

| Campo | Regla |
| --- | --- |
| `timestamp` | RFC 3339 UTC con milisegundos |
| `level` | `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR` |
| `service` | Nombre canonico del repositorio/componente |
| `service_version` | Commit SHA o version inmutable |
| `environment` | `local`, `dev`, `staging`, `production` |
| `message` | Descripcion estable y sin payloads completos |
| `trace_id`, `span_id` | Cuando exista span activo |
| `correlation_id` | Flujo de negocio, obligatorio en requests/eventos |
| `tenant_id` | ID opaco, solo despues de autenticacion |
| `event_name` | Nombre estable para busqueda/alerta |
| `error_type` | Clase/codigo estable, nunca stack como unico mensaje |

Se permiten IDs opacos de pago, checkout, import y evento. Los stack traces van
en campo separado y solo para errores inesperados.

### Campos prohibidos

- PAN, CVV, token de pago o secreto de proveedor.
- Password, cookie, bearer token, API key o header `Authorization`.
- RUT, numero de cliente, nombre, correo, telefono o direccion.
- Body completo de billing/checkout, URL firmada o query string sensible.
- Archivo de deuda, comprobante o webhook completo.

El logging usa allowlist, no una blacklist dependiente de recordar cada secreto.

## Metricas

Prometheus/OpenTelemetry con prefijo `nexopay_`, unidad en el nombre y labels de
baja cardinalidad. Nunca usar `tenant_id`, `payment_id`, URL completa o mensaje
de error como label.

Minimo por servicio HTTP:

- Rate, errores y duracion por operacion/ruta normalizada.
- Requests en vuelo, saturacion de pools y conexiones.
- Disponibilidad/readiness y version desplegada.

Minimo transaccional/asincrono:

- Payment intents por estado y transiciones invalidas.
- Edad de pagos `PROCESSING`/`UNKNOWN`, sin IDs como labels.
- Outbox pendiente, publish failures, consumer lag e inbox duplicates.
- Webhook attempts, edad, DLQ y conciliaciones divergentes.
- Latencia/error/timeout por proveedor con nombre de proveedor controlado.

## Health checks

- `liveness`: confirma que el proceso no esta bloqueado; no depende de PSP,
  Kafka ni base remota para evitar restart storms.
- `readiness`: confirma que puede aceptar trabajo con dependencias obligatorias.
- `startup`: protege inicializaciones y migraciones lentas.
- Los endpoints no revelan versiones de dependencia, hosts, secretos ni stack.
- Frontends exponen un recurso de build/version y synthetic checks externos.

Backends Spring usaran Actuator con grupos `liveness` y `readiness`, publicados
solo por la interfaz/segmento operacional que corresponda.

## Sampling y retencion

- Alpha: trazas locales al 100% por volumen bajo y solo datos sinteticos.
- Produccion: tail sampling que conserva errores, timeouts y transacciones
  anormales; la tasa normal se define mediante PG-06/PG-07.
- Metricas agregadas no reemplazan auditoria financiera.
- Retencion Alpha sigue `ALPHA_DATA_RETENTION.md`; produccion requiere PG-05.

## Alertas iniciales

- Error rate/SLO burn, p95/p99 y saturacion.
- Aumento de estados `UNKNOWN` o pagos estancados.
- Outbox/consumer lag y DLQ con edad creciente.
- Fallos de confirmacion de deuda, webhook o conciliacion.
- Readiness sin instancias disponibles.
- Fallo de backup, restore test o expiracion de certificado/secreto.

Toda alerta accionable enlaza a un runbook, tiene owner y evita incluir datos
sensibles en nombre, descripcion o notificacion.

## Validacion

- Tests verifican propagacion de trace/correlation y ausencia de campos
  prohibidos.
- Contract tests verifican metadata de eventos.
- CI rechaza configuraciones con logging de body/headers sensibles.
- Las etapas 2, 3, 5 y 6 deben agregar dashboards/runbooks al introducir cada
  senal; Jaeger local solo demuestra transporte de trazas, no operabilidad.
