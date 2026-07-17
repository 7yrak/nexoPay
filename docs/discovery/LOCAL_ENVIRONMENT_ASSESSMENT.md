# Evaluacion del entorno local

Evaluacion realizada el 2026-07-16 sobre la maquina de desarrollo actual.

## Inventario

| Recurso | Resultado |
| --- | --- |
| Sistema | Ubuntu 26.04 LTS, kernel 7.0 x86_64 |
| CPU | AMD Ryzen 7 3750H, 8 CPU logicos, AMD-V |
| Memoria | 13 GiB totales, aproximadamente 6 GiB disponibles al medir |
| Swap | 4 GiB totales, 2 GiB en uso al medir |
| Disco principal | NVMe 477 GB, 381 GB libres |
| Disco secundario | HDD 1 TB |
| Contenedores | Docker 29.1.3 operativo, 8 CPU y 13 GiB visibles |
| Java | OpenJDK 21 instalado |
| Node.js | Node 24 instalado |
| Faltantes | pnpm, kubectl, kind/minikube/k3d, Terraform y Helm |

## Clasificacion

### Apto

- Desarrollo de un servicio Kotlin o frontend individual.
- Docker Compose reducido con PostgreSQL, Kafka single-node o compatible,
  Keycloak y un subconjunto de servicios.
- Demos de Alpha interna con datos sinteticos.
- Pruebas unitarias, integracion y contratos con limites de memoria.

### Apto con restricciones

- Ejecutar simultaneamente todos los servicios JVM, observabilidad completa,
  Kafka, PostgreSQL y Keycloak. Requiere perfiles y limites de memoria.
- Kubernetes local. Puede usarse `kind` o `k3d`, pero Docker Compose sera mas
  estable al comienzo con 13 GiB.
- Pruebas de carga funcionales pequenas, no benchmarks de capacidad.

### No apto

- Produccion o manejo de dinero real.
- Alta disponibilidad, recuperacion regional o tolerancia a falla de nodo.
- Certificar el objetivo de 2.000 TPS o ejecutar carga distribuida realista.
- Almacenar secretos productivos o datos personales reales de ESVAL.

## Riesgos

- Un solo host es un unico punto de falla.
- La memoria disponible es limitada y ya existe uso de swap.
- El procesador movil no representa hardware servidor.
- El HDD secundario no debe usarse para benchmarks transaccionales.
- Desarrollo, Jenkins y servicios compitiendo en el mismo host alteraran
  cualquier medicion de rendimiento.

## Configuracion local recomendada

- Docker Compose antes de Kubernetes.
- PostgreSQL con memoria limitada y volumen en NVMe.
- Kafka en modo KRaft single-node con retencion corta para desarrollo.
- Keycloak con limite aproximado de 768 MiB a 1 GiB.
- Iniciar solo servicios del vertical slice activo.
- Datos sinteticos; secretos locales mediante archivos fuera de Git.
- Observabilidad basica primero y perfil completo opcional.

## Herramientas por instalar en etapa 1

1. pnpm mediante Corepack.
2. Terraform y Helm cuando comience infraestructura reproducible.
3. kubectl y kind/k3d solo despues de estabilizar Docker Compose.
4. Herramientas de seguridad y generacion de SBOM en Jenkins o contenedores.

## Decision

Esta maquina sera el entorno de desarrollo inicial. La infraestructura
productiva se decidira por separado y requerira al menos redundancia de compute,
datos, red, energia, backups y observabilidad.
