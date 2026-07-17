# ADR 0001 - Chile y CLP como mercado inicial

- Estado: Accepted.
- Fecha: 2026-07-16.

## Contexto

El dominio, los proveedores, la regulacion y la residencia de datos dependen de
la jurisdiccion. Soportar varios paises antes del primer piloto aumenta el riesgo
sin validar el producto.

## Decision

El MVP operara solo en Chile y pesos chilenos (`CLP`). Los pagos
transfronterizos y multimoneda quedan fuera de alcance.

## Consecuencias

- Proveedores, contratos y cumplimiento se evaluan bajo normativa chilena.
- El modelo de dinero acepta moneda explicita, aunque solo habilita CLP.
- No se implementa conversion de moneda.
