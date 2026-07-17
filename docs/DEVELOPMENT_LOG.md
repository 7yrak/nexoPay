# Bitacora de desarrollo

Esta bitacora conserva contexto entre sesiones. Cada entrada debe indicar
decisiones, trabajo terminado, riesgos conocidos y siguiente paso verificable.

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
