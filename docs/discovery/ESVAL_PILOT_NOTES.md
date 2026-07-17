# Notas del piloto ESVAL

Estado: `VALIDATING`. ESVAL es el facturador objetivo, pero no existe aun
evidencia de convenio, API B2B o acceso a datos de prueba.

## Evidencia publica

El sitio oficial permite:

- Consulta y pago rapido con numero de cliente.
- Pago por RUT para usuarios de Oficina Virtual.
- Para otros servicios, seleccion de una o varias facturas.
- Pago por transferencia con Khipu o tarjetas mediante Webpay.
- Envio de comprobante por correo.
- Gestion de casos donde el pago no rebajo la deuda.

Fuentes:

- [ESVAL: portal de personas y pago rapido](https://www.esval.cl/personas/inicio)
- [ESVAL: pago de otros servicios](https://www.esval.cl/personas/informaci%C3%B3n-no-regulados/pago/)

## Integracion objetivo

```text
findOutstandingBills(customerReference)
validateBillVersion(billId, version, amount)
confirmPayment(paymentReference, bills[])
findConfirmationStatus(paymentReference)
reversePayment(paymentReference) // solo si ESVAL lo soporta
```

## Modos de deuda

### Carga por archivo

ESVAL entrega snapshot completo o deltas firmados. Cada fila requiere ID de
deuda, referencia de cliente, monto CLP, vencimiento, estado, version y checksum.
La ingestion genera lote, totales de control, rechazos y reconciliacion.

### Consulta online

NexoPay consulta un servicio autorizado de ESVAL. Se definen timeout, SLA,
limites, idempotencia y comportamiento ante indisponibilidad.

### Recomendacion

Soportar ambos mediante capacidades del adaptador, sin mezclarlos hasta que el
contrato defina la fuente autoritativa. Para desarrollo se implementara
`fake-water-biller` con ambos modos y datos sinteticos.

## Preguntas para ESVAL

1. Existe API, archivo o ambos para consulta de deuda.
2. Cual es la clave: numero de cliente, RUT u otra.
3. Se permiten multiples facturas en un pago de servicio sanitario regulado.
4. Como se bloquea o versiona una deuda mientras se paga.
5. Como se confirma, consulta y revierte un pago.
6. Que identificador evita confirmaciones duplicadas.
7. Como se entrega conciliacion y totales de control.
8. Que SLA, ventanas, limites y ambientes de prueba existen.
9. Que datos personales puede almacenar NexoPay y por cuanto tiempo.
10. Quien contrata y entrega credenciales Webpay/Khipu.

## Restricciones

- No automatizar ni extraer datos del sitio publico.
- No usar RUT, deuda o datos de clientes reales sin convenio y base legal.
- No modelar endpoints privados mediante ingenieria inversa.
- No asumir reversa, reserva o confirmacion online hasta documentarlas.
