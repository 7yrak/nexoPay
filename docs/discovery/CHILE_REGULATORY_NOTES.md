# Notas regulatorias iniciales - Chile

Estado: investigacion preliminar. Este documento no constituye asesoria legal ni
aprueba el modelo regulatorio.

## Modelo declarado

NexoPay administra deuda, presenta el checkout y orquesta un PSP. No custodia,
liquida ni mantiene saldos. El PSP regulado procesa y abona directamente al
facturador/comercio.

## Hallazgos

### Operacion y liquidacion de tarjetas

El Banco Central de Chile regula los sistemas de pago y la operacion de tarjetas.
En 2024 actualizo el marco de PSP, incluyendo registro mas temprano para quienes
liquidan pagos a comercios. La CMF describe la autorizacion de Operadores para
entidades que prestan liquidacion y/o pago de prestaciones adeudadas a entidades
afiliadas.

Inferencia preliminar: no liquidar fondos aleja a NexoPay del supuesto mas claro
de Operador, pero no basta para concluir que queda fuera de regulacion. Deben
revisarse afiliacion, responsabilidad de pago, contratos y servicios efectivos.

Fuentes:

- [Banco Central: actualizacion de regulacion de tarjetas](https://www.bcentral.cl/contenido/-/details/bcch-actualiza-regulacion-tarjetas-pago)
- [Capitulo III.J.2 - Operacion de tarjetas](https://www.bcentral.cl/documents/33528/115568/CapIIIJ2.pdf)
- [CMF: autorizacion de Operadores de Tarjetas](https://www.cmfchile.cl/portal/principal/613/w3-article-29353.html)

### Iniciacion de pagos y finanzas abiertas

La CMF modifico en junio de 2026 la NCG 514 del Sistema de Finanzas Abiertas e
incorporo especificaciones de iniciacion de pagos, con entrada gradual desde
julio de 2027.

Si NexoPay mas adelante inicia transferencias desde cuentas mediante las APIs del
Sistema de Finanzas Abiertas, debe evaluarse especificamente su clasificacion y
obligaciones como proveedor de iniciacion. Integrar un PSP por su API comercial
no debe confundirse automaticamente con ese modelo.

Fuente:

- [CMF: modificacion del Sistema de Finanzas Abiertas](https://www.cmfchile.cl/portal/prensa/625/w4-article-110881.html)

### Datos personales

NexoPay tratara numero de cliente, RUT potencial, deuda, historial y usuarios
administrativos. La Ley 21.719 entra en vigencia el 1 de diciembre de 2026 y
refuerza derechos, seguridad, responsabilidad, minimizacion y limites de
conservacion.

El diseno debe cumplir desde el inicio el estandar que estara vigente antes de
un piloto razonable, incluyendo finalidad, base de licitud, contratos de
encargado/responsable, derechos de titulares, retencion y respuesta a incidentes.

Fuente:

- [BCN: Ley 21.719 de proteccion de datos personales](https://www.bcn.cl/leychile/navegar?idNorma=1209272)

## Preguntas para asesoria legal

1. Si el modelo sin custodia constituye solo servicio tecnologico bajo los
   contratos previstos.
2. Si NexoPay afilia comercios o actua por cuenta de un Operador regulado.
3. Si el flujo con Khipu u otro medio cae en iniciacion de pagos regulada.
4. Responsabilidad por pagos, fraude, contracargos y confirmacion de deuda.
5. Roles de responsable y encargado respecto de datos entregados por ESVAL.
6. Base de licitud, informacion al titular, retencion y ejercicio de derechos.
7. Requisitos contractuales y de seguridad para subencargados y proveedores.

## Condicion de aprobacion D0.2/D0.9

Obtener una opinion escrita de abogado chileno especializado en fintech y
privacidad, junto con revision PCI por un profesional competente antes de usar
datos o medios de pago reales.
