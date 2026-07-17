# ADR 0003 - Keycloak para identidad B2B local

- Estado: Accepted para desarrollo y Alpha interna.
- Fecha: 2026-07-16.

## Contexto

El entorno inicial es on-premise en una maquina de desarrollo. Management Portal
requiere OIDC, MFA, tenants y futura federacion con identidades empresariales.

## Decision

Usar Keycloak para desarrollo local, con un realm NexoPay y Organizations para
representar tenants. Habilitar OIDC, MFA y service accounts. La autorizacion de
dominio permanece en Management API y se valida en cada request.

## Consecuencias

- No existe dependencia de un cloud para la Alpha.
- Se puede federar OIDC/SAML de empresas en el futuro.
- Keycloak requiere PostgreSQL, actualizaciones y configuracion versionable.
- La decision de produccion se reevalua antes del piloto; autohospedarlo exige
  alta disponibilidad, backups, monitoreo y respuesta a vulnerabilidades.

## Alternativas

- Auth0, Entra External ID, Cognito u otro servicio administrado cuando exista
  proveedor cloud y presupuesto operacional.
