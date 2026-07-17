# Threat model inicial de NexoPay

## Alcance

Revision inicial para la Alpha interna con datos sinteticos. Cubre Payment
Portal, Checkout Web/SDK, Payments Platform, Management Plane, Billing
Connectors, workers, Keycloak, PostgreSQL, Kafka, Redis y Jenkins local.

- Fecha de revision: 2026-07-16.
- Proxima revision: antes de implementar el vertical slice de etapa 3 o el
  2026-08-15, lo que ocurra primero.
- Responsable temporal: Architecture/Security owner.
- Aprobacion externa: no realizada; requerida por PG-04, PG-05 y PG-08.

## Activos criticos

- Estado canonico e idempotencia de pagos.
- Ledger append-only y trazabilidad de movimientos.
- Snapshot de deuda, monto, version y confirmacion al facturador.
- Credenciales PSP/facturador, claves de webhook y tokens de sesion.
- Identidad, tenant, roles y auditoria administrativa.
- Datos personales y financieros que puedan incorporarse en etapas posteriores.
- Contratos, artefactos de CI y configuracion de infraestructura.

PAN y CVV no son activos administrados por NexoPay: el diseno debe impedir que
entren en APIs, eventos, logs o persistencia de la plataforma.

## Limites de confianza

1. Navegador del pagador hacia Payment Portal/Checkout Web.
2. Pagina del comercio hacia iframe y Checkout SDK mediante `postMessage`.
3. Canales publicos hacia Payments Platform.
4. Management Portal hacia Management API e IdP.
5. Payment Core hacia PSP y Billing Connectors.
6. Servicios sincronos hacia Kafka y workers asincronos.
7. Aplicaciones hacia PostgreSQL, Redis y secret manager futuro.
8. Repositorios Git hacia Jenkins y artefactos desplegables.
9. Entorno local hacia Internet y futura plataforma cloud.

## Reglas invariables

- Tenant y permisos provienen de identidad autenticada, nunca solo del body.
- Toda mutacion financiera exige idempotencia persistida y scope definido.
- PostgreSQL es fuente de verdad; Redis solo acelera datos reconstruibles.
- Evento y cambio de dominio se acoplan mediante transactional outbox.
- Consumidores y webhooks deduplican; entrega es al menos una vez.
- Un timeout de proveedor produce estado `UNKNOWN`/reconciliable, no rechazo
  inventado.
- Secretos, referencias de cliente y tokens no se registran en telemetria.
- El parent del iframe no recibe secretos ni datos financieros restringidos.

## Registro de amenazas

| ID | Amenaza | Impacto | Control inicial | Riesgo residual | Responsable/Hito |
| --- | --- | --- | --- | --- | --- |
| T-01 | Referencia cruzada entre tenants | Exposicion o pago indebido | `tenant_id` autenticado, constraints, filtros y pruebas negativas | Alto hasta implementar servicios | Backend lead / etapas 2 y 5 |
| T-02 | Replay o doble submit | Pago/ledger duplicado | `Idempotency-Key`, unique constraints, maquina de estados | Medio | Payments lead / etapa 2 |
| T-03 | Falsificacion de estado PSP | Confirmacion financiera falsa | Firma/autenticacion, consulta posterior y estados canonicos | Alto hasta PSP real | Payments/Security / etapa 7 |
| T-04 | Timeout interpretado como rechazo | Cobro real con estado local fallido | Estado `UNKNOWN`, polling y conciliacion | Medio | Payments lead / etapas 2 y 6 |
| T-05 | `postMessage` desde origen no autorizado | Secuestro de checkout o fuga | Origins exactos, nonce, source window y payload cerrado | Alto hasta pruebas browser | Frontend/Security / etapa 3 |
| T-06 | Robo/replay de checkout session | Pago iniciado por tercero | Token opaco corto, expiracion, binding y uso unico | Medio | Payments/Frontend / etapas 2 y 3 |
| T-07 | Inyeccion o mass assignment en APIs | Cambio de tenant/monto/estado | Schemas cerrados, monto derivado de deuda, validacion server-side | Medio | Backend lead / etapa 2 |
| T-08 | Datos sensibles en logs/trazas | Brecha de privacidad/PCI | Allowlist de campos, redaccion y pruebas de telemetria | Medio | Todos / desde etapa 2 |
| T-09 | Evento duplicado, tardio o poison | Efecto duplicado o backlog | Inbox, aggregate version, DLQ y replay controlado | Medio | Event lead / etapa 6 |
| T-10 | Compromiso de usuario administrativo | Cambio de secretos o refund | MFA, RBAC backend, step-up y auditoria | Alto hasta etapa 5/7 | Identity/Security |
| T-11 | Credencial en Git o imagen | Acceso a proveedor/cliente | Secret scanning, referencias a secret manager, rotacion | Medio | Platform lead / etapa 7 |
| T-12 | Dependencia o artefacto manipulado | Ejecucion de codigo malicioso | Lockfiles, versiones fijadas, audit, SBOM y firma futura | Medio | Platform lead / etapas 1 y 7 |
| T-13 | Jenkinsfile o checkout no confiable | Ejecucion en controlador | Repos privados, job local aislado, mount read-only | Medio local | Platform lead / etapa 1 |
| T-14 | Servicios locales expuestos | Acceso no autorizado | Puertos en `127.0.0.1`, datos sinteticos y passwords locales | Bajo en Alpha | Developer / continuo |
| T-15 | Perdida/corrupcion de base | Estado irrecuperable | Migraciones, backups/restore y reconciliacion | Alto hasta plataforma real | Platform/Payments / etapa 8 |
| T-16 | Abuso de consultas de deuda | Enumeracion de clientes | POST body, rate limit, respuesta uniforme y monitoreo | Medio | Billing/Security / etapas 4 y 7 |

## Datos prohibidos en Alpha

- PAN, CVV, track data o tokens productivos.
- RUT, nombre, correo, telefono o numero de cliente real.
- Deuda, comprobantes o conciliaciones reales.
- Credenciales de ESVAL, Transbank, Khipu u otro tercero.
- Dumps o copias de bases productivas.

## Criterio de revision

Cada etapa actualiza las amenazas que introduce. Un riesgo `Alto` no impide la
Alpha sintetica si su limite de confianza aun no existe, pero bloquea habilitar
esa capacidad con datos o dinero real. El pentest y la aprobacion profesional
permanecen como gates de produccion, no como tareas declaradas completas aqui.
