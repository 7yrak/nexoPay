# Clasificacion y retencion de datos

## Decision para Alpha interna

Solo se permiten datos sinteticos. Los volumenes locales son descartables y no
son backup. `make clean` en `nexopay-platform-infrastructure` elimina las bases,
Kafka, Redis y Jenkins local asociados al proyecto.

| Clase | Ejemplos Alpha | Retencion maxima | Eliminacion |
| --- | --- | ---: | --- |
| Efimero | Checkout token y cache Redis | 1 hora | TTL automatico |
| Transaccional sintetico | Pagos, deuda y eventos fake | 30 dias | Reset de volumen |
| Telemetria local | Logs | 30 MiB por contenedor | Rotacion Docker `local` |
| Trazas locales | Jaeger en memoria | Hasta reinicio | Reinicio/contenedor |
| Artefactos CI | Builds Jenkins y fingerprints | 50 builds | Politica del job |
| Auditoria sintetica | Acciones administrativas fake | 90 dias | Reset programado |

No se garantiza recuperacion de datos locales. El codigo, contratos y
configuracion no sensible se conservan en Git; los datos de ejecucion no.

## Clasificacion objetivo

- `PUBLIC`: documentacion y contratos publicables.
- `INTERNAL`: identificadores tecnicos y telemetria no sensible.
- `CONFIDENTIAL`: pagos, montos, deuda, identidad y auditoria.
- `RESTRICTED`: secretos, tokens de proveedor y datos sujetos a PCI.

`RESTRICTED` nunca se publica en eventos de dominio ni telemetria. PAN/CVV deben
permanecer fuera de NexoPay mediante captura alojada/tokenizacion PSP.

## Baseline productiva no aprobada

La retencion legal y operacional real no se fija por este documento. Antes de
datos reales, PG-05 debe definir por categoria: finalidad, base legal,
propietario, sistema de registro, residencia, cifrado, acceso, retencion,
eliminacion y legal hold. La revision debe considerar la normativa chilena
vigente en la fecha de lanzamiento, incluida la entrada en vigencia de la Ley
21.719.

Hasta que PG-05 pase, ningun plazo productivo puede interpretarse como politica
legal aprobada.
