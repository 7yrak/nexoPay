# Plan de desarrollo de NexoPay

## Proposito

Este documento ordena el desarrollo desde la definicion del negocio hasta un
piloto productivo. Es la fuente de verdad del estado de cada etapa. La bitacora
en `DEVELOPMENT_LOG.md` conserva el historial cronologico y la evidencia.

El plan prioriza un vertical slice verificable antes de ampliar integraciones o
dividir servicios. Ninguna etapa se considera terminada solo porque el codigo
existe: debe cumplir sus criterios de salida.

## Supuestos de planificacion

Las duraciones son orientativas para un equipo base de dos personas backend,
dos frontend, una de plataforma y apoyo parcial de QA, seguridad y producto. No
son compromisos hasta definir equipo, pais, proveedores y alcance regulatorio.

- Una semana significa una semana de trabajo del equipo, no una persona-semana.
- Integraciones con terceros pueden extender el calendario por certificaciones
  y accesos fuera del control de NexoPay.
- Seguridad, pruebas, observabilidad y documentacion son parte del entregable,
  no fases posteriores opcionales.
- Solo se desarrollan simultaneamente etapas cuya dependencia ya este estable.

## Objetivos de entrega

| Entrega | Etapas | Resultado |
| --- | --- | --- |
| Alpha interna | 0 a 3 | Pago completo con fake biller y fake PSP. |
| Beta controlada | 4 a 6 | Facturacion, gestion, webhooks y conciliacion operables. |
| Candidato a piloto | 7 y 8 | Proveedor real, seguridad y carga validadas. |
| Piloto productivo | 9 | Primer comercio/facturador operando con soporte formal. |

## Estado general

Estados permitidos: `NOT_STARTED`, `IN_PROGRESS`, `BLOCKED` y `DONE`.

| Etapa | Nombre | Estado | Dependencias | Estimacion |
| --- | --- | --- | --- | --- |
| 0 | Definicion para Alpha y gates productivos | DONE | Ninguna | Completada 2026-07-16 |
| 1 | Fundacion de ingenieria y contratos | DONE | Etapa 0 | Completada 2026-07-16 |
| 2 | Nucleo transaccional | DONE | Etapa 1 | Completada 2026-07-17 |
| 3 | Checkout vertical slice | DONE | Etapa 2 | Completada 2026-07-17 |
| 4 | Facturacion e integraciones de empresas | DONE | Etapas 1-3 | Alpha sintetica completada 2026-07-17; sandbox real diferido PG-03 |
| 5 | Plano de gestion | NOT_STARTED | Etapas 1-2 | 4-6 semanas |
| 6 | Eventos, webhooks y conciliacion | NOT_STARTED | Etapas 2, 4 y 5 | 3-5 semanas |
| 7 | PSP real, cumplimiento y seguridad | NOT_STARTED | Etapas 3 y 6 | 4-8 semanas |
| 8 | Escala, resiliencia y recuperacion | NOT_STARTED | Etapa 7 | 3-5 semanas |
| 9 | Piloto productivo y estabilizacion | NOT_STARTED | Etapa 8 | 2-4 semanas |

Las etapas 4 y 5 pueden avanzar en paralelo despues de estabilizar los contratos
y modelos de la etapa 2.

## Etapa 0 - Definicion para Alpha y gates productivos

Expediente de cierre: [Etapa 0 - Descubrimiento](discovery/PHASE_0_DISCOVERY.md).

### Objetivo

Eliminar ambiguedades para la Alpha sintetica y convertir toda validacion
externa pendiente en un gate explicito que bloquee produccion.

### Trabajo

- Elegir pais inicial, monedas y tipos de comercio/facturador.
- Definir si NexoPay actua como gateway, agregador, subadquirente u otro rol.
- Identificar candidatos a adquirente/PSP y gate de seleccion/certificacion.
- Definir liquidacion, comisiones, devoluciones y responsabilidad por disputas.
- Definir estrategia PCI, tokenizacion y alcance de datos personales.
- Elegir proveedor de identidad y estrategia de usuarios B2B.
- Estimar volumen diario, peak TPS, tamanos de payload y crecimiento.
- Proponer SLO, RTO, RPO, retencion y residencia de datos.
- Elaborar threat model inicial y mapa de datos sensibles.
- Registrar decisiones estructurales como ADR.

### Repositorios

`nexoPay`, `nexopay-contracts`, `nexopay-platform-infrastructure`.

### Criterios de salida

- Alcance MVP aprobado y fuera de alcance explicito.
- Rol funcional/PCI de Alpha acotado y revisiones externas registradas como
  gates que impiden dinero o datos reales.
- Fake PSP/facturador definidos y candidatos reales identificados sin afirmar
  convenio inexistente.
- Capacidad y SLO iniciales cuantificados.
- ADR de identidad, cloud, tenancy, datos y gates creados.
- Threat model inicial con owners, hitos y revision calendarizada.

## Etapa 1 - Fundacion de ingenieria y contratos

Evidencia de cierre: [Etapa 1 - Fundacion](implementation/PHASE_1_FOUNDATION.md).

### Objetivo

Crear una base reproducible para que todos los equipos implementen contratos y
calidad de la misma forma.

### Trabajo

- Definir schemas comunes: dinero, tenant, merchant, biller, error e IDs.
- Crear OpenAPI de checkout sessions, payment intents y billing queries.
- Crear AsyncAPI del ciclo de estados de pago.
- Configurar lint y deteccion de breaking changes en contratos.
- Acordar envelope de eventos, idempotencia y correlation IDs.
- Crear entorno local con PostgreSQL, Kafka, Redis, Keycloak y observabilidad.
- Reservar contratos para fake PSP/fake biller; sus implementaciones pertenecen
  a las etapas 2 y 4 para evitar servicios ficticios sin dominio consumidor.
- Fijar toolchains, convenciones, estrategia de ramas y versionamiento.
- Implementar Jenkins Shared Library minima o pipelines equivalentes probados.
- Definir formato de logs, trazas, metricas y health checks.

### Repositorios

`nexopay-contracts`, `nexopay-platform-infrastructure` y todos los consumidores
para validar los contratos.

### Criterios de salida

- Contratos v1 pasan lint, ejemplos y pruebas de compatibilidad.
- Entorno local arranca mediante un procedimiento documentado.
- Al menos un pipeline real ejecuta lint, test y build de un proyecto minimo.
- Baseline v1 es aceptada para iniciar productores/consumidores; cada
  implementacion agrega contract tests contra esa baseline.

## Etapa 2 - Nucleo transaccional

Evidencia de cierre: [Etapa 2 - Nucleo transaccional](implementation/PHASE_2_TRANSACTIONAL_CORE.md).

### Objetivo

Implementar la fuente de verdad del pago sin depender de proveedores reales.

### Trabajo

- Crear monolito modular Kotlin/Spring y migraciones Flyway.
- Implementar checkout session y payment intent.
- Implementar maquina de estados y reglas de transicion.
- Persistir idempotencia y restricciones contra duplicados.
- Implementar fake PSP determinista con exito, rechazo, timeout y respuesta
  tardia.
- Implementar ledger append-only de partida doble.
- Implementar transactional outbox y publicacion Kafka.
- Instrumentar logs, metricas y trazas sin datos sensibles.
- Probar concurrencia, reintentos y recuperacion tras fallos.

### Repositorios

`nexopay-payments-platform`, `nexopay-contracts` y
`nexopay-platform-infrastructure`.

### Criterios de salida

- Un pago simulado recorre creacion, procesamiento y estado final.
- Repetir un comando con la misma clave no duplica pago ni ledger.
- Timeouts quedan reconciliables y no producen rechazo falso.
- Evento y escritura de dominio son atomicos mediante outbox.
- Pruebas unitarias, integracion y contrato pasan en Jenkins.

## Etapa 3 - Checkout vertical slice

Evidencia de cierre: [Etapa 3 - Checkout vertical slice](implementation/PHASE_3_CHECKOUT_VERTICAL_SLICE.md).

### Objetivo

Completar una experiencia de pago end-to-end usando el nucleo y proveedores
simulados.

### Trabajo

- Construir Checkout Web con estados accesibles y recuperables.
- Construir SDK Web Component con iframe y validacion de `postMessage`.
- Construir Payment Portal con seleccion de empresa y deuda simulada.
- Crear sesiones exclusivamente desde backend autorizado.
- Implementar CSP, allowlist de origen, expiracion y proteccion contra replay.
- Probar redirect, modal/iframe, refresh, back button y navegadores soportados.
- Crear comprobante simulado y consulta posterior del estado.

### Repositorios

`nexopay-payment-portal`, `nexopay-checkout-web`, `nexopay-checkout-sdk`,
`nexopay-payments-platform` y `nexopay-contracts`.

### Criterios de salida

- Demo automatizada desde deuda simulada hasta ledger y comprobante.
- El SDK funciona en HTML y al menos React, Angular y Vue de ejemplo.
- Ningun secreto o dato financiero sensible cruza hacia el parent del iframe.
- Playwright cubre exito, rechazo, timeout, cierre y reingreso.
- Alpha interna desplegada en ambiente `dev`.

## Etapa 4 - Facturacion e integraciones de empresas

Evidencia de cierre interno: [Etapa 4 - Facturacion e integraciones](implementation/PHASE_4_BILLING_INTEGRATIONS.md).

El estado `DONE` cubre el alcance Alpha sintetico controlable por NexoPay. No
declara Beta real: el sandbox del primer proveedor permanece diferido y bloquea
esa entrega mediante `PG-03`.

### Objetivo

Integrar consulta y confirmacion de obligaciones sin acoplar Payment Core a los
modelos particulares de cada empresa.

### Trabajo

- Implementar interfaz canonica de capacidades de biller.
- Crear fake biller con consulta, reserva, confirmacion y reversa.
- Implementar aislamiento de recursos, timeouts y circuit breakers.
- Modelar confirmaciones tardias y reversas no soportadas.
- Integrar el primer facturador real en sandbox.
- Probar datos personales, errores, limites y reconciliacion del facturador.

### Repositorios

`nexopay-billing-connectors`, `nexopay-payments-platform`,
`nexopay-contracts` y `nexopay-event-workers`.

### Criterios de salida

- Fake biller pasa contract tests para todas las capacidades declaradas.
- Un adaptador lento no agota recursos de otros adaptadores.
- Primer proveedor real completa el flujo en sandbox.
- Confirmacion de deuda es idempotente y auditable.

## Etapa 5 - Plano de gestion

### Objetivo

Permitir operar tenants y comercios sin acceso directo al nucleo financiero.

### Trabajo

- Integrar OIDC y modelar roles/permissions.
- Implementar tenant, merchant, sucursal y usuarios.
- Gestionar branding, origenes permitidos y destinos webhook.
- Gestionar referencias de credenciales mediante secret manager.
- Crear auditoria para acciones sensibles.
- Crear proyecciones de pagos para consulta y exportacion acotada.
- Implementar Management Portal con estados de permisos y errores.

### Repositorios

`nexopay-management-api`, `nexopay-management-portal`,
`nexopay-event-workers` y `nexopay-contracts`.

### Criterios de salida

- Aislamiento multi-tenant probado contra referencias cruzadas.
- RBAC se valida en backend y existe auditoria consultable.
- Un comercio configura checkout y webhook sin tocar Payment Core.
- Operaciones sensibles requieren confirmacion reforzada.

## Etapa 6 - Eventos, webhooks y conciliacion

### Objetivo

Operar efectos secundarios y detectar divergencias financieras de forma
repetible y observable.

### Trabajo

- Implementar inbox/deduplicacion de consumidores.
- Entregar webhooks firmados con backoff, reintento y dead-letter topic.
- Construir proyecciones de lectura idempotentes.
- Conciliar Payment Core, ledger, PSP y facturador.
- Crear flujo operacional para investigar y resolver diferencias.
- Definir alertas de lag, edad, fallos y pagos pendientes.
- Crear runbooks de reintento, replay y recuperacion.

### Repositorios

`nexopay-event-workers`, `nexopay-payments-platform`,
`nexopay-management-api`, `nexopay-contracts` y
`nexopay-platform-infrastructure`.

### Criterios de salida

- Reentrega de eventos no duplica efectos.
- Webhooks verificables sobreviven indisponibilidad temporal del comercio.
- Una diferencia simulada aparece, se investiga y se resuelve con auditoria.
- Dashboards y alertas identifican lag y pagos estancados.

## Etapa 7 - PSP real, cumplimiento y seguridad

### Objetivo

Reemplazar el proveedor simulado por una integracion certificable y validar los
controles necesarios para dinero real.

### Trabajo

- Integrar tokenizacion y PSP/adquirente en sandbox.
- Implementar autenticacion adicional como 3DS cuando aplique.
- Implementar refund, void y consulta de estado real.
- Completar threat model, revision de arquitectura y matriz PCI.
- Ejecutar SAST, SCA, secret scanning, SBOM y escaneo de imagenes.
- Realizar pruebas de autorizacion, tenancy, abuso y seguridad de iframe.
- Preparar evidencias, politicas y procedimientos operacionales.
- Ejecutar pentest independiente antes del piloto.

### Repositorios

Todos, con foco en `nexopay-payments-platform`, `nexopay-checkout-web`,
`nexopay-checkout-sdk` y `nexopay-platform-infrastructure`.

### Criterios de salida

- Flujo sandbox certificado por el proveedor.
- Alcance PCI documentado y controles requeridos implementados.
- No quedan hallazgos criticos o altos sin mitigacion aprobada.
- Devoluciones, timeouts y respuestas tardias pasan pruebas end-to-end.

## Etapa 8 - Escala, resiliencia y recuperacion

### Objetivo

Demostrar con mediciones que el sistema soporta carga, fallos y recuperacion
segun los SLO definidos.

### Trabajo

- Crear modelo de carga con peak, burst y distribucion realista.
- Probar API, base de datos, Kafka, workers y proveedores simulados.
- Ajustar indices, pools, particiones, autoscaling y backpressure.
- Ejecutar fallos de pods, nodos, red, Kafka, PostgreSQL y proveedores.
- Validar backups, restore, RTO y RPO.
- Ejecutar pruebas de soak y detectar fugas o acumulacion de lag.
- Definir capacity plan, limites por tenant y margen de seguridad.

### Repositorios

`nexopay-platform-infrastructure`, todos los servicios y un conjunto versionado
de pruebas de rendimiento.

### Criterios de salida

- Peak objetivo sostenido con SLO y margen acordados.
- No existen pagos o movimientos contables duplicados bajo fallos.
- Restore y recuperacion cumplen RTO/RPO medidos.
- Alertas y runbooks funcionan durante ejercicios controlados.
- Capacity report aprobado para el piloto.

## Etapa 9 - Piloto productivo y estabilizacion

### Objetivo

Operar un alcance reducido con dinero real, soporte y capacidad de rollback.

### Trabajo

- Seleccionar comercio/facturador piloto, limites y criterios de suspension.
- Migrar configuracion y credenciales mediante procesos auditados.
- Ejecutar go-live checklist y smoke test productivo controlado.
- Activar monitoreo, guardia, soporte y comunicacion de incidentes.
- Conciliar diariamente y revisar pagos pendientes.
- Medir conversion, latencia, rechazo, diferencias e incidentes.
- Corregir hallazgos antes de aumentar volumen o comercios.

### Repositorios

Todos los repositorios desplegables y el repositorio general para evidencia y
decisiones.

### Criterios de salida

- Ventana piloto completada sin perdida ni duplicacion financiera.
- Conciliacion y liquidacion cierran con diferencias explicadas.
- SLO y metricas de producto se cumplen durante la ventana acordada.
- Runbooks e incident response fueron utilizados o ensayados.
- Go/no-go documentado para ampliar volumen.

## Lineas de trabajo permanentes

Estas lineas se ejecutan en todas las etapas:

- Seguridad y privacidad por diseno.
- Automatizacion de pruebas y contratos.
- Observabilidad y runbooks.
- Documentacion y ADR.
- Gestion de dependencias y vulnerabilidades.
- Experiencia accesible y responsive.
- Control de costos y capacidad.

## Definition of Ready

Una tarea puede comenzar cuando tiene objetivo, repositorio responsable,
contrato conocido, dependencias disponibles, criterios de aceptacion y riesgos
identificados. Si cambia dinero, datos sensibles o permisos, requiere revision
de seguridad antes de implementarse.

## Definition of Done

Una tarea termina cuando:

- Codigo y documentacion estan versionados y revisados.
- Contratos y migraciones son compatibles.
- Pruebas automatizadas relevantes pasan en Jenkins.
- Logs, metricas, trazas y alertas necesarias existen.
- No se introducen secretos ni vulnerabilidades altas sin excepcion aprobada.
- Existe evidencia enlazable en la bitacora.
- El criterio de aceptacion fue demostrado en el ambiente correspondiente.

## Protocolo de seguimiento

Al iniciar trabajo de una etapa:

1. Cambiar su estado a `IN_PROGRESS` en este documento.
2. Crear una entrada de bitacora con objetivo y alcance de la sesion.
3. Registrar bloqueos como `BLOCKED`, responsable y condicion de desbloqueo.
4. Al terminar, enlazar commits, PR, pruebas, dashboards o documentos.
5. Cambiar a `DONE` solo si todos los criterios de salida tienen evidencia.

La bitacora es append-only: no se reescribe el pasado para ocultar cambios de
direccion. Las correcciones se agregan como una entrada nueva.

## Backlog inmediato de la etapa 0

1. Confirmar pais y moneda de la primera operacion.
2. Definir rol comercial/regulatorio de NexoPay.
3. Seleccionar candidatos a PSP/adquirente y facturador piloto.
4. Estimar transacciones diarias, peak TPS y disponibilidad objetivo.
5. Elegir proveedor cloud e identidad mediante ADR.
6. Definir estrategia PCI y tokenizacion con revision especializada.
7. Aprobar el alcance funcional de la Alpha interna.
