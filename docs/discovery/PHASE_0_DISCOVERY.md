# Etapa 0 - Expediente de descubrimiento

## Estado

- Estado: `IN_PROGRESS`.
- Inicio: 2026-07-16.
- Responsable de aprobacion: pendiente.
- Objetivo: cerrar decisiones de producto, regulacion y capacidad antes de
  implementar contratos transaccionales.

Este documento contiene el estado vigente de las decisiones. La bitacora
conserva su evolucion cronologica y los ADR formalizan decisiones aprobadas.

## Evidencia de apoyo

- [Evaluacion del entorno local](LOCAL_ENVIRONMENT_ASSESSMENT.md)
- [Notas regulatorias iniciales de Chile](CHILE_REGULATORY_NOTES.md)
- [Notas del piloto ESVAL](ESVAL_PILOT_NOTES.md)
- [Registros de decisiones](../adr/README.md)

## Regla de decision

Una decision pasa por `OPEN`, `PROPOSED`, `VALIDATING` y `APPROVED`. Si falta
evidencia externa o aprobacion competente, no puede marcarse como aprobada.

## Tablero

| ID | Decision | Estado | Propuesta inicial | Evidencia requerida |
| --- | --- | --- | --- | --- |
| D0.1 | Pais y moneda inicial | APPROVED | Chile y CLP; sin multimoneda en MVP | Confirmacion del propietario de producto |
| D0.2 | Rol operativo/regulatorio | VALIDATING | Recaudador tecnologico sin custodia ni liquidacion | Opinion legal y contratos con PSP/comercio |
| D0.3 | PSP/adquirente inicial | VALIDATING | Evaluar Webpay Plus y Khipu; mantener fake PSP | APIs, certificacion, costos, SLA y modelo multiempresa |
| D0.4 | Facturador piloto | VALIDATING | ESVAL con adaptador simulado hasta obtener convenio | Sponsor, API/archivos, contrato y datos de prueba |
| D0.5 | Capacidad objetivo | APPROVED | 10M tx/dia, 2.000 TPS peak, prueba a 2x | Confirmacion del propietario de producto |
| D0.6 | SLO y recuperacion | PROPOSED | Payments 99,95%; gestion 99,9% | Analisis de impacto y costo |
| D0.7 | Plataforma de ejecucion | VALIDATING | Maquina local solo para desarrollo; produccion pendiente | Topologia productiva, costo y recuperacion |
| D0.8 | Identidad B2B | APPROVED | Keycloak para desarrollo; produccion se reevalua | ADR, MFA y prueba de organizaciones |
| D0.9 | PCI y tokenizacion | VALIDATING | Captura alojada y tokenizacion PSP | Revision PCI/QSA o especialista |
| D0.10 | Retencion y residencia | OPEN | Minimizar datos y separar auditoria | Requisitos legales y operacionales |

`tx/dia` representa transacciones de pago, no todas las llamadas HTTP ni los
eventos derivados. Esa distincion debe conservarse en capacidad y costos.

## D0.1 - Pais y moneda

### Decision

El MVP operara en Chile y exclusivamente en pesos chilenos (`CLP`). Pagador,
facturador, PSP y liquidacion seran domesticos. Multimoneda y pagos
transfronterizos quedan fuera de alcance.

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

NexoPay opera como recaudador tecnologico y orquestador: administra deuda e
inicia la experiencia de pago, pero el adquirente o PSP regulado recibe y
liquida los fondos directamente al comercio o facturador. NexoPay no mantiene
saldos, cuentas de dinero ni fondos de clientes.

Esta descripcion es funcional, no una conclusion regulatoria. La clasificacion
final depende de contratos y actividades efectivas, especialmente afiliacion,
iniciacion y liquidacion.

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

Los primeros candidatos son Webpay Plus para tarjetas y Khipu para transferencias
bancarias. ESVAL publica actualmente ambos medios en su canal para otros
servicios. Cada candidato se comparara por:

- Medios de pago, tokenizacion, 3DS y recurring payments.
- Idempotencia y consulta posterior a timeout.
- Webhooks firmados y modelo de estados.
- Refund, void, partial capture y conciliacion.
- Sandbox, certificacion, SLA, soporte, costos y limites.
- Alcance PCI que permite reducir.

Se implementara siempre un fake PSP determinista antes del proveedor real.

Transbank asigna codigo de comercio al cliente y exige integracion y validacion.
Debe confirmarse si cada facturador usara sus propias credenciales y liquidacion,
y bajo que acuerdo NexoPay operara como integrador multiempresa.

## D0.4 - Facturador piloto

ESVAL es el facturador objetivo. Su sitio publico permite consultar y pagar por
numero de cliente o RUT. Para otros servicios publica seleccion de una o varias
facturas, pago mediante Khipu o Webpay y envio de comprobante.

No se encontro documentacion publica de una API B2B de deuda o confirmacion. La
seleccion del piloto no autoriza usar endpoints internos ni automatizar el sitio
publico. Hasta obtener convenio, especificaciones y datos de prueba se usara un
`fake-esval` basado en contratos sinteticos.

### Estrategia de deuda propuesta

- Online: consultar la fuente ESVAL cuando exista API con SLA adecuado.
- Carga: aceptar archivos completos o incrementales, versionados y conciliables.
- Hibrida: consulta online primaria y carga como contingencia solo si el
  contrato define precedencia y vigencia.
- En cualquier modo, el payment intent guarda un snapshot inmutable de deuda,
  monto y version para auditoria.
- Antes de pagar se valida vigencia; despues se confirma de manera idempotente.

Capacidades a confirmar:

- Identificacion de cliente y consulta de deuda.
- Reserva opcional de documentos.
- Confirmacion idempotente y respuesta a duplicados.
- Reversa, anulacion o mecanismo compensatorio.
- Archivo/API de conciliacion y ventanas operacionales.

## D0.5 - Capacidad

### Decision

El objetivo a tres anos es 10 millones de pagos diarios, peak de diseno de
2.000 TPS y pruebas de plataforma a 4.000 TPS. Esto no implica que la primera
version deba operar con infraestructura productiva de ese tamano.

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

El desarrollo inicial se ejecutara on-premise en la maquina Linux actual. La
evaluacion tecnica esta en `LOCAL_ENVIRONMENT_ASSESSMENT.md` y concluye que es
apta para desarrollo y demos controladas, no para produccion ni pruebas finales
de 2.000 TPS.

La topologia productiva permanece abierta. Se compararan cloud y on-premise con
alta disponibilidad, PostgreSQL multi-zona o equivalente, Kafka, secret manager,
KMS/HSM, WAF, proteccion DDoS y evidencia de cumplimiento.

La decision se registrara mediante ADR e incluira costo inicial, costo al peak,
operacion, lock-in aceptado y plan de recuperacion.

## D0.8 - Identidad B2B

### Decision para desarrollo

Se usara Keycloak para desarrollo local y Alpha interna. Soporta OIDC, OAuth 2.0,
MFA, WebAuthn, identity brokering y Organizations para casos B2B. Se ejecutara
con PostgreSQL y configuracion exportable, sin acoplar permisos de dominio a su
base de datos.

Fuente tecnica:

- [Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/index.html)

La decision productiva se revisara en la etapa 7. Autohospedar identidad exige
alta disponibilidad, parches urgentes, backups, monitoreo y runbooks; si el
equipo operativo no puede sostenerlos se migrara a un servicio administrado.

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
- [x] Jurisdiccion y moneda documentadas.
- [x] Flujo de fondos funcional documentado.
- [ ] Rol regulatorio y estrategia PCI revisados externamente.
- [ ] PSP y facturador iniciales identificados.
- [x] Capacidad objetivo cuantificada.
- [ ] SLO, RTO y RPO aprobados.
- [ ] Cloud, identidad, tenancy y retencion decididos o con ADR calendarizado.
- [ ] Threat model y mapa de datos revisados.
- [ ] Riesgos residuales tienen responsable y fecha de revision.

## Primer bloque de respuestas requerido

1. Confirmar si existe contacto o convenio con ESVAL.
2. Confirmar si ESVAL entregara deuda por archivo, API o ambos.
3. Definir quien contrata Webpay/Khipu y es titular de cada credencial.
4. Aprobar o ajustar la propuesta de SLO, RTO y RPO.
5. Definir volumen de lanzamiento, separado del objetivo a tres anos.
