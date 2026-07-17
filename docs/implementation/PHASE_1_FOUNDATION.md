# Etapa 1 - Fundacion de ingenieria y contratos

## Estado

- Estado: `IN_PROGRESS`.
- Inicio: 2026-07-16.
- Ambiente: maquina local, Docker Compose y datos sinteticos.
- Dependencia habilitante: etapa 0 parcial completada para Alpha interna.

## Objetivo

Construir contratos v1, toolchain reproducible y entorno local minimo antes de
implementar Payment Core o frontends funcionales.

## Tablero de trabajo

| ID | Entregable | Repositorio | Estado |
| --- | --- | --- | --- |
| E1.1 | Toolchain de contratos y reglas de lint | nexopay-contracts | DONE |
| E1.2 | Schemas comunes de IDs, dinero, errores y tenant | nexopay-contracts | DONE |
| E1.3 | OpenAPI de billing, checkout sessions y payment intents | nexopay-contracts | IN_PROGRESS |
| E1.4 | AsyncAPI y envelope comun de eventos | nexopay-contracts | NOT_STARTED |
| E1.5 | Docker Compose local y servicios de apoyo | nexopay-platform-infrastructure | NOT_STARTED |
| E1.6 | Pipeline Jenkins real para contratos | nexopay-contracts | NOT_STARTED |
| E1.7 | Convenciones de logs, trazas, metricas y health | nexoPay | NOT_STARTED |

## Orden de implementacion

1. Fijar Node/pnpm y herramientas OpenAPI, AsyncAPI y JSON Schema.
2. Crear schemas comunes sin dependencias de implementacion.
3. Definir billing API para carga y consulta simuladas.
4. Definir checkout sessions y payment intents.
5. Definir eventos y politica de compatibilidad.
6. Levantar PostgreSQL, Kafka y Keycloak local.
7. Ejecutar contratos y ejemplos en Jenkins.

## Decisiones de trabajo

- Contratos en ingles; documentacion de negocio puede permanecer en espanol.
- IDs opacos con prefijos solo en representacion externa.
- Montos como enteros en unidad minima y moneda ISO 4217.
- Fechas en RFC 3339 UTC; fechas de vencimiento civil se modelan por separado.
- Errores con codigo estable, mensaje seguro y correlation ID.
- Tenant ID obligatorio en contexto autenticado, nunca confiado solo desde body.
- Ejemplos contienen exclusivamente datos ficticios.
- Breaking changes requieren nueva version mayor.

## Criterios de salida

- [ ] Contratos v1 pasan lint y validacion de ejemplos.
- [ ] Existe deteccion automatizada de breaking changes.
- [ ] Fake PSP y fake water biller pueden implementarse sin inventar campos.
- [ ] Docker Compose inicia dependencias con un comando documentado.
- [ ] Jenkins ejecuta lint, test y build de documentacion/artefactos.
- [ ] Productores y consumidores tienen reglas claras de compatibilidad.
- [ ] Bitacora contiene commits y evidencia de validacion.

## Restricciones

- No implementar endpoints reales antes de estabilizar sus contratos.
- No agregar Kubernetes a la ruta critica local.
- No incluir credenciales, RUT ni deuda real.
- No generar clientes SDK hasta que el contrato minimo sea coherente.

## Siguiente accion

Implementar E1.3 en `nexopay-contracts`: comenzar por billing batch/online y
continuar con checkout sessions y payment intents sobre los schemas comunes.

## Evidencia

### E1.1 y E1.2

- Commit: [`595d921`](https://github.com/7yrak/nexopay-contracts/commit/595d921).
- Node.js 24 y pnpm 11.13.1 fijados.
- Cinco schemas JSON Schema 2020-12 y cuatro ejemplos sinteticos.
- Cinco pruebas automatizadas exitosas.
- Lint, validacion de referencias y build reproducible exitosos.
- Artefacto con manifiesto SHA-256 generado.
- Auditoria de dependencias sin vulnerabilidades conocidas.
- Dependencias reducidas de 1.417 a 168 al reemplazar AsyncAPI CLI por parser.
- Scripts transitivos no autorizados permanecen bloqueados por politica pnpm.
