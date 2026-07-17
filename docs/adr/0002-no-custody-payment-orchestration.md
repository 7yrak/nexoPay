# ADR 0002 - Orquestacion sin custodia de fondos

- Estado: Proposed, pendiente de revision legal y contractual.
- Fecha: 2026-07-16.

## Contexto

NexoPay debe recaudar deudas sin mantener dinero de clientes. La liquidacion y
responsabilidad de pago cambian el perimetro regulatorio y la arquitectura.

## Decision propuesta

NexoPay administra deuda y orquesta el checkout. Un PSP/adquirente regulado
procesa y liquida directamente al comercio o facturador. NexoPay no custodia,
liquida ni mantiene saldos.

## Consecuencias

- Cada integracion debe demostrar quien recibe y liquida fondos.
- El ledger de NexoPay registra obligaciones y eventos, no saldo custodiado.
- Contratos, cobro de comisiones y responsabilidad deben validarse en Chile.
- Si el modelo cambia, este ADR se reemplaza antes de implementar dinero real.
