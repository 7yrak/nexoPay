# Bitacora de desarrollo

Esta bitacora conserva contexto entre sesiones. Cada entrada debe indicar
decisiones, trabajo terminado, riesgos conocidos y siguiente paso verificable.

## 2026-07-17 - Cierre de etapa 2

### Decisiones

- Payments Platform se implementa como monolito modular Kotlin/Spring; no se
  crean microservicios sin una necesidad medida.
- PostgreSQL es fuente de verdad para pagos, intentos, idempotencia, ledger y
  outbox. Kafka distribuye eventos y Redis no decide estado financiero.
- El ledger es operacional, append-only y de partida doble; no representa
  fondos recibidos o custodiados por NexoPay.
- Una llamada al PSP ocurre fuera de la transaccion. Un timeout queda `UNKNOWN`
  y se reconcilia, nunca se convierte automaticamente en rechazo.
- Outbox entrega al menos una vez; cada consumidor deduplica por `eventId`.

### Trabajo completado

- Payment intent y checkout session v1 con identidad Alpha, token hasheado,
  monto derivado de deuda sintetica e idempotencia persistida.
- Maquina de estados y fake PSP determinista para exito, rechazo, timeout y
  exito tardio.
- Migracion Flyway tenant-aware, ledger inmutable, transactional outbox y
  dispatcher Kafka con retry.
- Reconciliacion de intentos `INITIATED`/`UNKNOWN` y recuperacion tras fallo.
- Logs JSON, metricas Prometheus, trazas OpenTelemetry, readiness y Docker
  no-root.
- Compose y Jenkins integrados para Payments Platform.
- ADR 0008 y expediente tecnico de la etapa creados.

### Validaciones

- `nexopay-payments-platform` commits `c0c655a`, `3bcfa6e` y `1f52175`
  publicados.
- `nexopay-platform-infrastructure` commits `212165d`, `d8ff00b` y `e2b869d`
  publicados.
- Dos ejecuciones locales consecutivas confirmaron que las integraciones no se
  restauran desde cache.
- Suite final: 7 pruebas unitarias y 12 de integracion/contrato, 19 totales y 0
  fallos contra el tag exacto `contracts-v1-alpha.1`.
- Jenkins `nexopay-payments-platform-local #4`: `SUCCESS` en 90 segundos, 19
  resultados publicados y JAR archivado con fingerprint.
- Jenkins `nexopay-contracts-local #4`: `SUCCESS`, 13 pruebas, 19 schemas, 11
  ejemplos y 36 artefactos sin breaking changes.
- Imagen Docker construida; Payments Platform saludable en `127.0.0.1:18081`,
  metricas Prometheus y trazas correlacionadas visibles en Jaeger.

### Incidencias resueltas

- Puerto `8081` ocupado: Compose publica la aplicacion en `18081`.
- Race entre scheduler y test de caida de Kafka: jobs separados y schedulers
  deshabilitados explicitamente bajo el perfil de pruebas.
- Reporte JUnit ausente en Jenkins: plugin fijado y publisher validado.
- Integracion restaurable desde Gradle cache: cache/up-to-date deshabilitados
  solo para `integrationTest`.
- Alta inicial de un Job DSL posterior al escaneo de Jenkins: runner con espera
  y un reinicio controlado cuando el job aun no existe.

### Riesgos abiertos

- PG-01 a PG-09 siguen `OPEN`; no se autorizan datos, credenciales ni dinero
  real.
- La identidad por headers y la deuda son exclusivamente sinteticas.
- No existen PSP/facturador real, refunds, disputas o conciliacion bancaria.
- La topologia local single-node no valida SLO, RTO, RPO, alta disponibilidad ni
  capacidad de 2.000 TPS.

### Siguiente paso

Iniciar etapa 3: Checkout Web, SDK Web Component con iframe seguro, Payment
Portal y pruebas Playwright end-to-end sobre el nucleo sintetico.

## 2026-07-16 - Cierre de etapas 0 y 1

### Decisiones

- Etapa 0 queda `DONE` exclusivamente para Alpha interna sintetica.
- Cerrar discovery no equivale a autorizacion productiva; nueve gates externos
  bloquean datos, credenciales o dinero real.
- Se adopta multi-tenancy con schema compartido, `tenant_id` obligatorio,
  constraints tenant-aware y defensa en profundidad.
- Los SLO 99,95%/99,9%, RTO 60 min y RPO 5 min son baseline de ingenieria, no
  compromisos contractuales.
- La retencion Alpha es corta y descartable; la retencion real requiere revision
  legal mediante PG-05.
- `contracts-v1-alpha.1` es la baseline inmutable de contratos v1.

### Trabajo completado

- Etapa 0: alcance/fuera de alcance, threat model, mapa de datos, retencion,
  tenancy, SLO, owners de riesgo y gates de produccion documentados.
- Etapa 1: OpenAPI billing/checkout, AsyncAPI de estados, 19 schemas y 11
  ejemplos sinteticos.
- Comparador de breaking changes para JSON Schema, OpenAPI y AsyncAPI.
- Docker Compose local con PostgreSQL, Kafka, Redis, Keycloak y Jaeger.
- Jenkins LTS local reproducible mediante Dockerfile y Configuration as Code.
- Estandar de logs, trazas, metricas, health checks, sampling y alertas.

### Validaciones

- `nexopay-contracts` commits `0c61eb2`, `7d0b24a` y `99c26bd` publicados.
- Tag `contracts-v1-alpha.1` publicado.
- Cierre documental de `nexoPay` publicado en commit `436d12b`.
- 13 pruebas, lint OpenAPI/AsyncAPI y compatibilidad pasan localmente.
- `nexopay-platform-infrastructure` commit `cf1e814` publicado.
- Hardening de persistencia/logs publicado en commit `c35e4cc`.
- `make up` y `make verify`: PostgreSQL, Kafka, Redis y Keycloak saludables.
- Topic de pago creado con 12 particiones; Jaeger UI/OTLP accesibles.
- Broker Kafka recreado y topic conservado; rotacion de logs inspeccionada.
- Jenkins build `nexopay-contracts-local #3`: `SUCCESS` en 25 segundos.
- Jenkins archivo 36 artefactos con fingerprints y confirmo ausencia de breaking
  changes contra la baseline.

### Incidencias resueltas

- Puerto local `8080` ocupado: Keycloak se movio a `8180`.
- PostgreSQL host ya usaba `5432`: NexoPay se movio a `55432`.
- Jenkins existente usaba `8090`: controlador NexoPay se movio a `18090`.
- Mirror Jenkins inicial no respondia: se fijo un mirror alternativo verificado.
- Jenkins rechazo checkout local por seguridad: se habilito solo para el repo
  hermano montado read-only y se documento que no aplica a CI compartido.
- Crumb Jenkins requirio cookie de sesion: el runner conserva cookie y exige
  resultado `SUCCESS` real.
- Kafka usaba un directorio fuera del volumen: se fijo `KAFKA_LOG_DIRS` y se
  comprobo persistencia tras recrear el broker.

### Riesgos abiertos

- PG-01 a PG-09 siguen `OPEN`: no existe opinion legal, PSP contratado, convenio
  ESVAL, revision PCI/privacidad, plataforma productiva, DR ni pentest.
- La maquina local y Kafka/PostgreSQL single-node no validan disponibilidad ni
  2.000 TPS.
- Los simuladores fake PSP/fake water biller se implementan en etapas 2 y 4.

### Siguiente paso

Iniciar etapa 2 en `nexopay-payments-platform` con monolito modular
Kotlin/Spring, migraciones, idempotencia, maquina de estados, fake PSP, ledger y
transactional outbox, consumiendo `contracts-v1-alpha.1`.

## 2026-07-16 - Etapa 1: toolchain y schemas comunes

### Decisiones

- Node.js 24 y pnpm 11.13.1 forman el toolchain inicial de contratos.
- Los schemas usan JSON Schema 2020-12 e IDs canonicos bajo
  `https://schemas.nexopay.cl`.
- Montos se representan con `amountMinor` entero y `currency`; v1 permite CLP.
- IDs externos son opacos, prefijados y respaldados por ULID.
- Errores se basan en Problem Details e incluyen correlation ID obligatorio.
- Se usa `@asyncapi/parser` en vez de `@asyncapi/cli` para reducir supply chain.

### Trabajo completado

- E1.1 Toolchain de contratos: `DONE`.
- E1.2 Schemas comunes: `DONE`.
- Jenkinsfile activado con Corepack y lockfile congelado.
- Scripts de lint, validacion, ejemplos y build reproducible.
- Schemas de entity ID, money, tenant context, resource metadata y problem.
- Manifiesto de artefacto con SHA-256.

### Validaciones

- Commit `nexopay-contracts`: `595d921`.
- Cinco schemas y cuatro ejemplos validados.
- Cinco pruebas automatizadas pasan.
- Auditoria de dependencias: sin vulnerabilidades conocidas.
- Peer dependencies: sin conflictos.
- Footprint limpio: 168 paquetes y 69 MiB.
- Script de telemetria transitivo `@scarf/scarf` denegado explicitamente.

### Riesgos abiertos

- OpenAPI y AsyncAPI aun no existen; sus validadores omiten correctamente hasta
  que E1.3/E1.4 agreguen contratos.
- El dominio `schemas.nexopay.cl` aun no esta publicado; los IDs son canonicos y
  se resuelven localmente durante esta etapa.
- Breaking-change detection se activara cuando exista una version base publicada.

### Siguiente paso

Implementar E1.3: OpenAPI de carga/consulta de deuda, checkout sessions y
payment intents usando los schemas comunes.

## 2026-07-16 - Transicion a etapa 1 con entorno local

### Decisiones

- La maquina actual se confirma como ambiente inicial mientras no existan
  clientes oficiales.
- Todo desarrollo y prueba usara datos, identidades, deudas y pagos sinteticos.
- AWS se evaluara como primera opcion productiva cuando exista cliente oficial.
- La etapa 0 permanece abierta para legal, PCI, proveedores y produccion.
- La etapa 1 comienza en paralelo porque cuenta con las decisiones parciales
  necesarias para una Alpha interna.
- El simulador se llamara `fake-water-biller`; un adaptador `esval` solo se
  creara con convenio o especificaciones autorizadas.

### Trabajo completado

- ADR 0004 para desarrollo local y AWS diferido.
- Definicion de alcance incluido y excluido de la Alpha local.
- Creacion del tablero E1.1 a E1.7 de la etapa 1.
- Cambio de etapa 1 a `IN_PROGRESS` en el plan maestro.

### Validaciones

- La etapa 1 depende solo de decisiones ya aprobadas para trabajo sintetico.
- Ningun requisito pendiente obliga a detener contratos o infraestructura local.
- Las validaciones pendientes siguen siendo gates antes de dinero o datos reales.

### Riesgos abiertos

- La topologia AWS, SLO, RTO y RPO productivos siguen pendientes.
- La maquina local no representa capacidad ni disponibilidad productiva.
- Continuan pendientes convenio ESVAL, contratos PSP y revision legal/PCI.

### Siguiente paso

Implementar toolchain y schemas comunes en `nexopay-contracts`, luego levantar
las dependencias locales minimas en `nexopay-platform-infrastructure`.

## 2026-07-16 - Etapa 0: mercado, piloto, capacidad e identidad

### Decisiones

- Chile y CLP quedan aprobados como unico mercado y moneda del MVP.
- NexoPay administrara deuda y orquestara pagos sin custodiar ni liquidar fondos;
  el modelo queda en validacion legal y contractual.
- ESVAL queda seleccionado como facturador piloto objetivo, pendiente de convenio
  y acceso tecnico.
- Se aprueba como objetivo a tres anos 10 millones de pagos diarios, 2.000 TPS
  peak y prueba de plataforma a 4.000 TPS.
- La maquina Linux actual sera solo entorno de desarrollo, no produccion.
- Keycloak queda aprobado para desarrollo y Alpha interna; su uso productivo se
  reevalua antes del piloto.

### Trabajo completado

- Inventario de CPU, memoria, discos, Docker, Java, Node y herramientas locales.
- Evaluacion de aptitud y limites de la maquina on-premise.
- Revision preliminar de fuentes oficiales del BCCh, CMF y legislacion de datos.
- Revision del flujo publico ESVAL y sus medios Webpay/Khipu.
- Definicion de estrategias de deuda por carga, online e hibrida.
- Creacion de ADR para mercado, flujo sin custodia e identidad local.

### Validaciones

- La maquina tiene 8 CPU logicos, 13 GiB RAM, NVMe con espacio suficiente y
  Docker operativo: apta para desarrollo con perfiles limitados.
- No es apta para produccion, alta disponibilidad ni certificar 2.000 TPS.
- ESVAL publica consulta/pago por numero de cliente o RUT y un flujo de otras
  facturas con Khipu/Webpay, pero no se encontro API B2B publica.
- Keycloak soporta OIDC, MFA, WebAuthn, identity brokering y Organizations B2B.

### Riesgos abiertos

- Falta opinion legal chilena sobre el modelo contractual efectivo.
- Falta contacto, convenio, API/archivos y sandbox de ESVAL.
- Falta confirmar quien contrata y posee credenciales Webpay/Khipu.
- Produccion, SLO, RTO, RPO y retencion siguen sin aprobarse.
- La Ley 21.719 entra en vigencia el 1 de diciembre de 2026 y debe incorporarse
  desde el diseno.

### Siguiente paso

Confirmar relacion y mecanismo tecnico con ESVAL, evaluar Webpay/Khipu como
integrador multiempresa y aprobar SLO/RTO/RPO y volumen de lanzamiento.

## 2026-07-16 - Inicio operativo de la etapa 0

### Decisiones

- Se abre `docs/discovery/PHASE_0_DISCOVERY.md` como expediente vigente.
- Las decisiones usaran estados `OPEN`, `PROPOSED`, `VALIDATING` y `APPROVED`.
- Se propone para el MVP operar como gateway/orquestador sin custodia de fondos,
  sujeto a validacion legal en la jurisdiccion inicial.
- Se propone dimensionar inicialmente 10 millones de pagos diarios y 2.000 TPS
  peak, con prueba de carga a 2x, sujeto a proyeccion comercial.

### Trabajo completado

- Apertura del tablero D0.1 a D0.10.
- Definicion de preguntas sobre pais, flujo de fondos, PSP y facturador piloto.
- Creacion de escenarios iniciales de capacidad.
- Propuesta preliminar de SLO, RTO y RPO.
- Creacion del primer mapa de datos sensibles y checklist de cierre.

### Validaciones

- Ninguna propuesta fue marcada como aprobada sin evidencia.
- La implementacion de contratos transaccionales permanece dependiente de las
  decisiones que cambian regulacion, seguridad y flujo de fondos.

### Riesgos abiertos

- Jurisdiccion, moneda, rol regulatorio y proveedores no confirmados.
- Los objetivos de capacidad y disponibilidad aun son hipotesis.
- La estrategia PCI requiere revision profesional externa.

### Siguiente paso

Obtener el primer bloque de respuestas del propietario del producto y, con la
jurisdiccion confirmada, investigar fuentes regulatorias y proveedores
oficiales antes de aprobar D0.1 a D0.3.

## 2026-07-16 - Plan de desarrollo por etapas

### Decisiones

- Se adopta `docs/DEVELOPMENT_PLAN.md` como fuente de verdad del roadmap.
- La bitacora sera append-only y registrara evidencia de cada avance.
- Las etapas usan estados `NOT_STARTED`, `IN_PROGRESS`, `BLOCKED` y `DONE`.
- Una etapa solo pasa a `DONE` cuando cumple todos sus criterios de salida.
- El primer objetivo de entrega es una Alpha interna completa con fake biller y
  fake PSP; proveedores reales se incorporan despues de validar el nucleo.

### Trabajo completado

- Definicion de diez etapas desde descubrimiento hasta piloto productivo.
- Identificacion de dependencias, repositorios, entregables y criterios de
  salida por etapa.
- Definicion de Alpha interna, Beta controlada, candidato a piloto y piloto.
- Definicion de Definition of Ready, Definition of Done y protocolo de
  seguimiento.

### Estado de etapas

- Etapa 0 - Definicion de producto, regulacion y capacidad: `IN_PROGRESS`.
- Etapas 1 a 9: `NOT_STARTED`.

### Validaciones

- El plan respeta los limites de repositorios definidos en la arquitectura.
- La ruta critica comienza por decisiones que afectan regulacion, seguridad,
  contratos y capacidad antes de implementar pagos reales.
- Gestion y facturacion pueden avanzar en paralelo despues del nucleo.

### Riesgos abiertos

- La duracion depende del equipo definitivo y tiempos de certificacion externa.
- Aun faltan pais, rol regulatorio, PSP, facturador piloto, cloud, SLO y alcance
  PCI confirmados.

### Siguiente paso

Resolver el backlog inmediato de la etapa 0 y registrar cada decision como ADR
o entrada de bitacora con su evidencia.

## 2026-07-16 - Fundacion de la plataforma

### Decisiones

- `nexoPay` queda como repositorio general de arquitectura e incubacion.
- Cada componente se versiona en un repositorio privado independiente.
- Se separan plano transaccional, gestion, integraciones y procesamiento
  asincrono.
- Portal y checkout embebible usan el mismo Payment Core.
- El producto embebible sera un SDK TypeScript que abre un iframe alojado.
- Backend base: Kotlin, Java 21, Spring Boot y Gradle.
- Frontend base: TypeScript, React y Next.js.
- Integracion entre repositorios exclusivamente mediante contratos versionados.

### Trabajo completado

- Creacion de diez repositorios privados bajo la cuenta `7yrak`.
- Definicion inicial de responsabilidades y dependencias permitidas.
- Documentacion de arquitectura, seguridad, consistencia y operacion.
- README tecnico y Jenkinsfile base planificados para cada componente.
- Clones locales ubicados en `components/`, fuera del versionamiento general.

### Pendientes inmediatos

1. Definir pais inicial, moneda, adquirente y proveedor de identidad.
2. Crear OpenAPI minimo para checkout sessions y payment intents.
3. Crear AsyncAPI para ciclo de vida de pagos.
4. Definir modelo de tenant, merchant, biller y credenciales.
5. Implementar un vertical slice sin PSP real usando un proveedor simulado.
6. Levantar entorno local con PostgreSQL, Kafka y observabilidad basica.

### Riesgos abiertos

- Alcance regulatorio y PCI aun no validado para el pais de operacion.
- TPS objetivo, peak y SLO todavia no cuantificados.
- Estrategia de liquidacion y conciliacion pendiente.
- Jenkins Shared Library aun no implementada.

## Plantilla de entrada

```text
## YYYY-MM-DD - Titulo

### Decisiones
### Trabajo completado
### Validaciones
### Riesgos abiertos
### Siguiente paso
```
