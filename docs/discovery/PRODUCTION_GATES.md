# Gates obligatorios antes de produccion

## Proposito

La etapa 0 cierra las decisiones necesarias para una Alpha interna sintetica.
No constituye aprobacion regulatoria, PCI, contractual ni productiva. Este
registro concentra las validaciones externas que NexoPay no puede completar sin
cliente, proveedor, asesoria competente y una plataforma productiva.

Ningun componente puede procesar dinero, tarjetas, RUT, deuda o credenciales
reales mientras exista un gate aplicable en estado distinto de `PASSED`.

## Estados

- `OPEN`: falta trabajo o evidencia.
- `IN_REVIEW`: evidencia entregada al aprobador competente.
- `PASSED`: aprobador y evidencia registrados.
- `REJECTED`: el diseno debe cambiar antes de continuar.

## Registro

| ID | Gate | Estado | Responsable | Aprobador requerido | Limite |
| --- | --- | --- | --- | --- | --- |
| PG-01 | Clasificacion legal y contratos del modelo sin custodia | OPEN | Product owner | Abogado chileno competente | Antes de integrar PSP real en etapa 7 |
| PG-02 | Seleccion, contrato, credenciales y certificacion PSP/adquirente | OPEN | Payments lead | PSP y responsable comercial | Antes de transacciones reales |
| PG-03 | Convenio ESVAL, interfaz autorizada y datos de prueba | OPEN | Product owner | ESVAL y responsable de integracion | Antes de crear adaptador `esval` |
| PG-04 | Alcance PCI, tokenizacion y responsabilidades | OPEN | Security lead | QSA o profesional PCI competente | Antes de aceptar datos de pago reales |
| PG-05 | Privacidad, bases legales, residencia y retencion | OPEN | Privacy owner | Asesoria legal de privacidad | Antes de datos personales reales |
| PG-06 | Cloud, identidad y arquitectura productiva | OPEN | Platform lead | Architecture/Security review | Antes de staging representativo |
| PG-07 | SLO, capacidad, backups, restore, RTO y RPO medidos | OPEN | Platform lead | Product owner y Operations | Antes del candidato a piloto |
| PG-08 | Threat model final, pentest y hallazgos altos/criticos | OPEN | Security lead | Pentester independiente/Security | Antes del piloto productivo |
| PG-09 | Modelo operativo, conciliacion, soporte e incidentes | OPEN | Operations lead | Product owner y proveedor | Antes del piloto productivo |

Los nombres de personas se asignaran al conformar el equipo. Mientras un rol no
tenga titular, el gate permanece `OPEN` y la responsabilidad final recae en el
product owner.

## Evidencia minima

### PG-01

- Diagrama contractual y de flujo de fondos real.
- Identidad de quien afilia, recibe, liquida, cobra y responde por disputas.
- Opinion legal fechada y alcance de esa opinion.
- Contratos consistentes con la arquitectura sin custodia.

### PG-02 y PG-03

- Contrato y ambiente sandbox autorizados.
- Credenciales con titular, rotacion y scope definidos.
- Especificacion versionada, SLA, limites y proceso de certificacion.
- Datos de prueba autorizados y plan de conciliacion.

### PG-04 y PG-05

- Diagrama de flujo de datos y matriz de responsabilidades.
- Evidencia de que PAN/CVV no se persisten en NexoPay.
- Inventario, base legal, retencion, residencia y derechos de titulares.
- Controles y documentos exigidos por la revision profesional.

### PG-06 a PG-09

- ADR productivos aceptados y threat model actualizado.
- Resultados reproducibles de carga, fallos, backup y restore.
- SLO y error budgets aprobados con costo conocido.
- Pentest, remediaciones, runbooks, guardia y criterios de suspension.

## Regla de cambio

Marcar un gate `PASSED` exige agregar fecha, persona/entidad aprobadora y enlace
a evidencia no sensible. Una conversacion, demo local o implementacion tecnica
no reemplaza una aprobacion externa. Si cambia el flujo de fondos, proveedor,
captura de tarjeta o tratamiento de datos, los gates afectados vuelven a
`OPEN`.
