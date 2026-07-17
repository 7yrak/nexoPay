# Etapa 1 - Fundacion de ingenieria y contratos

## Estado

- Estado: `DONE`.
- Inicio: 2026-07-16.
- Cierre: 2026-07-16.
- Ambiente: maquina local, Docker Compose y datos sinteticos.
- Dependencia habilitante: etapa 0 completada para Alpha interna.

## Objetivo

Construir contratos v1, toolchain reproducible y entorno local minimo antes de
implementar Payment Core o frontends funcionales.

## Tablero de trabajo

| ID | Entregable | Repositorio | Estado |
| --- | --- | --- | --- |
| E1.1 | Toolchain de contratos y reglas de lint | nexopay-contracts | DONE |
| E1.2 | Schemas comunes de IDs, dinero, errores y tenant | nexopay-contracts | DONE |
| E1.3 | OpenAPI de billing, checkout sessions y payment intents | nexopay-contracts | DONE |
| E1.4 | AsyncAPI y envelope comun de eventos | nexopay-contracts | DONE |
| E1.5 | Docker Compose local y servicios de apoyo | nexopay-platform-infrastructure | DONE |
| E1.6 | Pipeline Jenkins real para contratos | nexopay-contracts | DONE |
| E1.7 | Convenciones de logs, trazas, metricas y health | nexoPay | DONE |

## Orden de implementacion

1. Fijar Node/pnpm y herramientas OpenAPI, AsyncAPI y JSON Schema.
2. Crear schemas comunes sin dependencias de implementacion.
3. Definir billing API para carga y consulta simuladas.
4. Definir checkout sessions y payment intents.
5. Definir eventos y politica de compatibilidad.
6. Levantar PostgreSQL, Kafka, Redis, Keycloak y Jaeger local.
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

- [x] Contratos v1 pasan lint y validacion de ejemplos.
- [x] Existe deteccion automatizada de breaking changes.
- [x] Fake PSP y fake water biller pueden implementarse sin inventar campos.
- [x] Docker Compose inicia dependencias con un comando documentado.
- [x] Jenkins ejecuta lint, compatibilidad, test y build de artefactos.
- [x] Productores y consumidores tienen reglas claras de compatibilidad.
- [x] Bitacora contiene commits y evidencia de validacion.

## Restricciones

- No implementar endpoints reales antes de estabilizar sus contratos.
- No agregar Kubernetes a la ruta critica local.
- No incluir credenciales, RUT ni deuda real.
- No generar clientes SDK hasta que el contrato minimo sea coherente.

## Siguiente accion

Comenzar etapa 2 en `nexopay-payments-platform`: monolito modular Kotlin/Spring,
migraciones, payment intent, idempotencia, maquina de estados, fake PSP, ledger
y transactional outbox sobre los contratos publicados.

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

### E1.3 y E1.4

- Commit contratos: [`0c61eb2`](https://github.com/7yrak/nexopay-contracts/commit/0c61eb2).
- OpenAPI 3.1 de billing y checkout validado por Redocly sin warnings.
- AsyncAPI 3.1 de `payment.status-changed.v1` validado por parser oficial.
- 19 JSON Schemas y 11 ejemplos sinteticos validos.
- Headers de idempotencia/correlacion, OAuth2 y session bearer definidos.
- Montos se derivan de deuda validada; las APIs no aceptan PAN/CVV.

### Compatibilidad

- Commit: [`99c26bd`](https://github.com/7yrak/nexopay-contracts/commit/99c26bd).
- Baseline inmutable: tag `contracts-v1-alpha.1`.
- Comparador conservador para JSON Schema, OpenAPI y AsyncAPI.
- 13 pruebas pasan, incluidas cuatro de compatibilidad compatible/ruptura.

### E1.5

- Commit infraestructura: [`cf1e814`](https://github.com/7yrak/nexopay-platform-infrastructure/commit/cf1e814).
- Hardening: [`c35e4cc`](https://github.com/7yrak/nexopay-platform-infrastructure/commit/c35e4cc).
- `make up` y `make verify` ejecutados correctamente.
- PostgreSQL 17.5, Kafka 4.0.0, Redis 7.4.2 y Keycloak 26.3.1 saludables.
- Topic `nexopay.payment.status-changed.v1` creado con 12 particiones.
- Persistencia Kafka verificada recreando el broker sin perder el topic.
- Logs limitados por contenedor a tres archivos de 10 MiB.
- Jaeger 2.8.0 validado con UI y endpoints OTLP accesibles por loopback.
- Consumo observado de la plataforma base bajo 1 GiB en reposo; limites
  declarados evitan exceder la maquina local.

### E1.6

- Jenkins LTS 2.555.3 local, Node 24.13.0 y plugins fijados.
- Jenkins Configuration as Code crea `nexopay-contracts-local`.
- Build `#3`: `SUCCESS` en 25 segundos.
- Ejecuta install con lockfile, lint, compatibilidad, 13 tests, build de 36
  archivos y archivado con fingerprints.

### E1.7

- [Estandar de observabilidad](../engineering/OBSERVABILITY_STANDARD.md).
- W3C Trace Context, logs JSON, campos prohibidos, metricas, health checks,
  sampling, alertas y validaciones definidos.
