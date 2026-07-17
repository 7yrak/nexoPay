# ADR 0006 - Multi-tenancy con schema compartido y defensa en profundidad

- Estado: Accepted para Alpha y primera implementacion.
- Fecha: 2026-07-16.

## Contexto

La plataforma debe soportar muchas empresas sin multiplicar infraestructura
antes de validar el producto. El mayor riesgo es una referencia cruzada entre
tenants en pagos, deuda o gestion.

## Decision

Usar PostgreSQL compartido por dominio con `tenant_id` obligatorio en toda
entidad tenant-scoped. El tenant se deriva de identidad autenticada y se aplica
en repositorios, unique constraints, foreign keys compuestas y pruebas
negativas. Row Level Security se evaluara como defensa adicional, no como
reemplazo de autorizacion de dominio.

- Payments Platform y Management API no comparten tablas ni acceso directo.
- Credenciales reales se aislan por tenant como referencias a secret manager.
- Eventos incluyen tenant opaco y se particionan por aggregate ID estable.
- Operaciones globales requieren rol explicito y auditoria reforzada.
- Exports y analytics operan sobre proyecciones, no sobre Payment Core.

## Consecuencias

- Menor costo y complejidad inicial que database-per-tenant.
- Cada query y constraint requiere disciplina tenant-aware.
- Pruebas de referencias cruzadas son obligatorias desde etapas 2 y 5.
- Sharding/cells se introduce por capacidad o aislamiento regulatorio sin
  cambiar IDs ni contratos externos.
