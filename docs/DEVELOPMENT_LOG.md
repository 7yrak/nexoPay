# Bitacora de desarrollo

Esta bitacora conserva contexto entre sesiones. Cada entrada debe indicar
decisiones, trabajo terminado, riesgos conocidos y siguiente paso verificable.

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
