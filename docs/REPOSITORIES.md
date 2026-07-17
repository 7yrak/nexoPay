# Catalogo de repositorios

## Regla de dependencias

Las dependencias apuntan hacia contratos estables. Ningun repositorio importa
fuentes internas de otro repositorio ni comparte tablas por conveniencia.

| Repositorio | Puede depender de | No puede depender de |
| --- | --- | --- |
| payment-portal | contracts, Checkout API | bases de datos, Management API |
| management-portal | contracts, Management API | Payment Core, bases de datos |
| checkout-web | contracts, Checkout API | Management API, secretos PSP |
| checkout-sdk | API publica del checkout | implementacion de portales |
| payments-platform | contracts, PSP clients | Management API, frontends |
| management-api | contracts, proyecciones de lectura | tablas internas de pagos |
| billing-connectors | contracts, SDK de proveedores | frontends, ledger |
| event-workers | contracts, eventos y APIs autorizadas | escritura directa al ledger |
| platform-infrastructure | artefactos publicados | codigo de dominio |
| contracts | herramientas de especificacion | cualquier implementacion |

## Propiedad de datos

- Payments Platform: pagos, intentos, ledger y outbox transaccional.
- Management API: tenants, usuarios, configuracion, branding y credenciales.
- Billing Connectors: estado tecnico de adaptadores, no la verdad financiera.
- Event Workers: inbox, entregas, conciliaciones y proyecciones derivadas.

Cada servicio tiene usuario y esquema de base de datos propios. El acceso de
solo lectura entre esquemas tambien requiere una decision de arquitectura.

## Versionamiento

- APIs HTTP: version mayor en ruta, por ejemplo `/v1`.
- Eventos: nombre estable y version explicita en el envelope.
- SDK: SemVer y CDN con version mayor inmutable.
- Imagenes: commit SHA y version SemVer cuando exista release.
- Infraestructura: modulos con tags; providers y charts con versiones fijadas.

## Jenkins

Cada repositorio contiene un `Jenkinsfile`. Los pipelines base detectan si el
repositorio sigue en fase documental y omiten builds inexistentes. Al agregar
codigo, los manifiestos y wrappers pasan a ser obligatorios y los stages se
activan sin redisenar el pipeline.

La logica comun terminara en una Jenkins Shared Library mantenida desde
`nexopay-platform-infrastructure`; los Jenkinsfiles conservaran declaracion de
aplicacion, stack y politicas, sin copiar logica compleja.
