# Etapa 0 - Expediente de descubrimiento

## Estado

- Estado: `IN_PROGRESS`.
- Inicio: 2026-07-16.
- Responsable de aprobacion: pendiente.
- Objetivo: cerrar decisiones de producto, regulacion y capacidad antes de
  implementar contratos transaccionales.

Este documento contiene el estado vigente de las decisiones. La bitacora
conserva su evolucion cronologica y los ADR formalizan decisiones aprobadas.

## Regla de decision

Una decision pasa por `OPEN`, `PROPOSED`, `VALIDATING` y `APPROVED`. Si falta
evidencia externa o aprobacion competente, no puede marcarse como aprobada.

## Tablero

| ID | Decision | Estado | Propuesta inicial | Evidencia requerida |
| --- | --- | --- | --- | --- |
| D0.1 | Pais y moneda inicial | OPEN | Una jurisdiccion y una moneda en MVP | Confirmacion de negocio y legal |
| D0.2 | Rol operativo/regulatorio | PROPOSED | Gateway/orquestador sin custodia de fondos | Opinion legal en el pais inicial |
| D0.3 | PSP/adquirente inicial | OPEN | Un proveedor primario y fake PSP | APIs, certificacion, costos y SLA |
| D0.4 | Facturador piloto | OPEN | Una empresa o sandbox representativo | Sponsor, protocolo y datos de prueba |
| D0.5 | Capacidad objetivo | PROPOSED | 10M tx/dia, 2.000 TPS peak, prueba a 2x | Proyeccion comercial y carga medida |
| D0.6 | SLO y recuperacion | PROPOSED | Payments 99,95%; gestion 99,9% | Analisis de impacto y costo |
| D0.7 | Proveedor cloud | OPEN | Servicio administrado y multi-zona | Comparacion costo, region y capacidades |
| D0.8 | Identidad B2B | OPEN | OIDC administrado con MFA | Requisitos de tenants y operacion |
| D0.9 | PCI y tokenizacion | VALIDATING | Captura alojada y tokenizacion PSP | Revision PCI/QSA o especialista |
| D0.10 | Retencion y residencia | OPEN | Minimizar datos y separar auditoria | Requisitos legales y operacionales |

`tx/dia` representa transacciones de pago, no todas las llamadas HTTP ni los
eventos derivados. Esa distincion debe conservarse en capacidad y costos.

## D0.1 - Pais y moneda

### Preguntas

- En que pais se constituira y operara inicialmente NexoPay.
- Cual sera la primera moneda.
- Si el pagador, comercio, facturador y cuenta de liquidacion estaran en el
  mismo pais durante el MVP.
- Si el MVP requiere impuestos o documentos tributarios propios de NexoPay.

### Recomendacion

Limitar el MVP a un pais, una moneda y liquidacion domestica. Operacion
multimoneda o transfronteriza requiere una etapa posterior.

## D0.2 - Rol operativo y flujo de fondos

### Opcion recomendada para MVP

NexoPay opera como gateway/orquestador tecnologico: inicia y coordina el pago,
pero el adquirente o PSP regulado recibe y liquida los fondos al comercio o
facturador. NexoPay no mantiene saldos de clientes ni cuentas de dinero.

### Alternativas de mayor alcance

- Agregador: NexoPay agrupa comercios y participa en liquidacion.
- Subadquirente/payfac: NexoPay incorpora comercios bajo un adquirente sponsor.
- Custodia/wallet: NexoPay mantiene saldo o fondos de usuarios.

Las alternativas cambian contratos, compliance, conciliacion, riesgo, capital y
licencias. No se implementaran sin decision legal y comercial explicita.

### Preguntas

- Quien firma el contrato de aceptacion de pagos con cada comercio.
- Quien recibe el dinero primero y quien liquida al facturador.
- Quien define comisiones y emite sus documentos asociados.
- Quien responde por fraude, contracargos y disputas.
- Si NexoPay necesita dividir un pago entre varias partes.

## D0.3 - PSP/adquirente

La evaluacion comenzara despues de confirmar jurisdiccion. Cada candidato se
comparara por:

- Medios de pago, tokenizacion, 3DS y recurring payments.
- Idempotencia y consulta posterior a timeout.
- Webhooks firmados y modelo de estados.
- Refund, void, partial capture y conciliacion.
- Sandbox, certificacion, SLA, soporte, costos y limites.
- Alcance PCI que permite reducir.

Se implementara siempre un fake PSP determinista antes del proveedor real.

## D0.4 - Facturador piloto

El primer piloto debe representar el flujo real sin ser el proveedor mas
complejo. Debe ofrecer sponsor de negocio, documentacion, sandbox o datos
sinteticos y una forma confiable de consultar y confirmar pagos.

Capacidades a confirmar:

- Identificacion de cliente y consulta de deuda.
- Reserva opcional de documentos.
- Confirmacion idempotente y respuesta a duplicados.
- Reversa, anulacion o mecanismo compensatorio.
- Archivo/API de conciliacion y ventanas operacionales.

## D0.5 - Capacidad

### Escenarios de referencia

| Escenario | Pagos diarios | Promedio | Peak de diseno |
| --- | ---: | ---: | ---: |
| Inicial | 1.000.000 | 12 TPS | 250 TPS |
| Objetivo propuesto | 10.000.000 | 116 TPS | 2.000 TPS |
| Expansion | 50.000.000 | 579 TPS | 10.000 TPS |

El peak no se deduce solo del promedio: debe modelar vencimientos, campanas,
horarios de pago y reintentos. Para cada pago se estimaran consultas, comandos,
eventos, webhooks y escrituras de ledger.

### Informacion requerida

- Pagos diarios al inicio, a 12 meses y a 36 meses.
- Concentracion esperada en la hora y minuto de mayor carga.
- Numero de empresas/comercios y de usuarios administrativos.
- Retencion de pagos, ledger, auditoria, logs y comprobantes.
- Tamanos aproximados de payload y numero de eventos por pago.

## D0.6 - SLO, RTO y RPO

### Propuesta para evaluacion

- Checkout API y Payment Core: 99,95% mensual, excluyendo proveedores externos.
- Checkout Web: 99,95% mensual.
- Management API/Portal: 99,9% mensual.
- Latencia interna de creacion/consulta: p95 menor a 500 ms sin contar PSP o
  facturador.
- RTO inicial: 60 minutos para desastre regional.
- RPO inicial: 5 minutos como maximo para datos recuperables de desastre, con
  reconciliacion obligatoria de cualquier ventana.
- Cero tolerancia a ledger duplicado o inconsistente.

Estos valores son propuestas, no compromisos. Se aprobaran despues de comparar
impacto comercial, arquitectura y costo.

## D0.7 - Cloud y regiones

Se compararan proveedores con region adecuada para residencia y latencia,
PostgreSQL administrado multi-zona, Kafka administrado o compatible, Kubernetes,
secret manager, KMS, WAF, proteccion DDoS y evidencia de cumplimiento.

La decision se registrara mediante ADR e incluira costo inicial, costo al peak,
operacion, lock-in aceptado y plan de recuperacion.

## D0.8 - Identidad B2B

Requisitos minimos:

- OIDC/OAuth 2.0, MFA y recuperacion segura.
- Organizaciones/tenants y roles por comercio.
- Service accounts y rotacion de credenciales.
- Auditoria de login y acciones privilegiadas.
- Step-up authentication para devoluciones y secretos.

La identidad de pagadores del portal publico se evaluara por separado; no debe
forzarse una cuenta si el negocio permite pago de deuda con identificador.

## D0.9 - PCI, privacidad y datos

### Propuesta

- Checkout Web se aloja en dominio NexoPay.
- El comercio integra redirect o iframe mediante Checkout SDK.
- La captura usa campos/tokenizacion del PSP cuando sea posible.
- NexoPay no almacena CVV y evita almacenar PAN.
- Logs, trazas, analytics y soporte excluyen datos sensibles.

El alcance final debe ser revisado por un profesional PCI competente y por
asesoria legal de privacidad en la jurisdiccion elegida.

## Mapa inicial de datos

| Dato | Sistema propietario | Sensibilidad | Regla inicial |
| --- | --- | --- | --- |
| Payment ID y estado | Payments Platform | Interno | Retencion por definir |
| Monto y moneda | Payments Platform/Ledger | Financiero | Inmutable y auditable |
| PAN/CVV | PSP o campos alojados | Restringido | No persistir en NexoPay |
| Token de pago | Payments Platform | Restringido | Cifrar y limitar acceso |
| Identificador de deuda | Billing Connectors/Payments | Personal potencial | Minimizar y cifrar |
| Usuarios y roles | Management API/IdP | Personal | MFA y auditoria |
| Secretos | Secret manager | Critico | Nunca Git, DB o logs |
| Logs y trazas | Plataforma | Interno/personal potencial | Redaccion y retencion corta |

## Criterios de cierre

- [ ] Alcance MVP y fuera de alcance aprobados.
- [ ] Jurisdiccion, moneda y flujo de fondos documentados.
- [ ] Rol regulatorio y estrategia PCI revisados externamente.
- [ ] PSP y facturador iniciales identificados.
- [ ] Capacidad, SLO, RTO y RPO cuantificados.
- [ ] Cloud, identidad, tenancy y retencion decididos o con ADR calendarizado.
- [ ] Threat model y mapa de datos revisados.
- [ ] Riesgos residuales tienen responsable y fecha de revision.

## Primer bloque de respuestas requerido

1. Pais y moneda inicial.
2. Si NexoPay recibira/custodiara fondos o solo orquestara al PSP.
3. Tipo de primera empresa piloto: servicio basico, retail u otro.
4. Volumen esperado al inicio y en tres anos, si existe una estimacion.
5. Preferencia o restriccion de cloud e identidad, si ya existe.
