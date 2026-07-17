# ADR 0010 - Capa anticorrupcion canonica para facturadores

- Estado: Accepted para Alpha sintetica
- Fecha: 2026-07-17

## Contexto

Cada empresa puede exponer REST, SOAP, archivos o consultas batch, con reglas
distintas para buscar, reservar, confirmar y revertir deuda. Acoplar Payment
Core a esos modelos haria que un cambio de proveedor afecte la contabilidad y
que un adaptador lento pueda degradar todos los pagos.

ESVAL es el piloto objetivo, pero `PG-03` sigue abierto: no hay convenio,
especificacion autorizada, credenciales sandbox ni datos de prueba.

## Decision

- `nexopay-contracts` publica capacidades explicitas y modelos canonicos de
  consulta, validacion de snapshot, reserva, confirmacion y reversa.
- `nexopay-billing-connectors` es la unica capa que traduce protocolos y
  estados particulares de una empresa.
- Cada adaptador declara capacidades; una operacion no declarada responde como
  no soportada y nunca se infiere.
- Cada adaptador tiene pool, timeout y estado de circuito independientes.
- La referencia de cliente se envia en el body, se trata como sensible y no se
  guarda en Payment Core ni se incluye en logs o respuestas del portal.
- Payment Core valida un snapshot canonico, guarda IDs, versiones y montos, y
  confirma exactamente ese snapshot despues de capturar.
- La confirmacion usa una clave idempotente derivada del pago. Billing conserva
  estado tecnico y auditoria append-only en una base separada.
- Para la Etapa 4 Alpha la entrega es sincrona y reintentable por replay. Inbox,
  backoff, dead-letter y conciliacion masiva pertenecen a la Etapa 6.

## Consecuencias

- Los canales y Payment Core no dependen de modelos ESVAL ni de otro proveedor.
- Un adaptador lento no consume los ejecutores de otro adaptador.
- Captura PSP y confirmacion de deuda no son una transaccion distribuida; una
  confirmacion puede quedar pendiente y debe reconciliarse.
- El secreto compartido local se reemplazara por identidad de workload antes
  de usar infraestructura o datos reales.
- No se creara un adaptador llamado `esval` hasta cerrar `PG-03` con evidencia
  externa verificable.
