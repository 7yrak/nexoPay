# ADR 0009 - Limite seguro del checkout alojado

- Estado: Accepted para Alpha sintetica.
- Fecha: 2026-07-17.

## Contexto

Payment Portal y sitios de comercios deben iniciar pagos sin recibir secretos,
tokens de sesion ni autoridad para declarar un resultado financiero. Checkout
Web necesita consultar y confirmar el pago, pero exponer su bearer al JavaScript
del navegador ampliaria el riesgo de exfiltracion, replay y manipulacion.

El SDK debe funcionar sin dependencia de framework y permitir iframe, modal o
redireccion. Un iframe por si solo no autentica al parent: origen, ventana,
version y forma del mensaje deben comprobarse explicitamente.

## Decision

- Payment Portal crea payment intent y checkout session mediante una Route
  Handler server-side con identidad autorizada. Su respuesta publica contiene
  solo `sessionId`.
- Checkout Web se divide en una aplicacion React sin logica financiera y un BFF
  Node. Solo el BFF conoce `CHECKOUT_CLIENT_SECRET`, canjea el bearer efimero y
  llama a Payments Platform.
- El bearer no se inserta en HTML, URLs, storage, respuestas al browser,
  `postMessage` ni logs.
- Payments Platform permite reemitir de forma determinista la credencial de una
  sesion vigente solo a un cliente interno autenticado. Sesiones expiradas o
  cerradas no aceptan nuevos comandos.
- `allowedOrigins` se persiste al crear la sesion. Checkout Web compara
  `parentOrigin` exactamente y genera CSP `frame-ancestors` desde esa lista.
  `localhost` y `127.0.0.1` son origenes distintos.
- El SDK es un Web Component con Shadow DOM cerrado e iframe `sandbox`. Valida
  `event.origin`, `event.source`, `version`, `sessionId` y un schema cerrado
  antes de despachar un evento al comercio.
- Los mensajes del browser solo pueden contener estado visual, altura o codigo
  de error. Se prohiben token, monto, payment intent, provider token y cualquier
  dato suficiente para completar o acreditar un pago.
- Cerrar el modal solo descarta la vista y permite reingresar; cancelar dentro
  de Checkout Web ejecuta un comando idempotente y cierra la sesion canonica.
- Exito y comprobante siempre se recuperan desde Payments Platform y ledger,
  nunca desde query params, retorno del proveedor o un mensaje del parent.

## Consecuencias

- Un XSS en el portal parent no obtiene el bearer de checkout ni secretos de
  comercio, aunque podria abrir una sesion ya creada hasta que expire.
- Checkout Web agrega una frontera BFF desplegable que debe escalar con el
  trafico del checkout, pero permanece stateless y sin base propia.
- La allowlist debe configurarse antes de crear una sesion; cambios posteriores
  no alteran sesiones existentes.
- Redireccion conserva `parentOrigin` en Alpha para reutilizar una URL, aunque
  la comunicacion `postMessage` solo tiene efecto cuando existe parent distinto.
- La autenticacion por client secret compartido se limita a loopback. En cloud
  debe reemplazarse por identidad de workload y secret manager.

## Alternativas descartadas

- Entregar el bearer al SDK o al parent: permite exfiltracion y aumenta replay.
- Llamar Payments Platform directamente desde React: obliga a CORS y expone la
  credencial de sesion al runtime del browser.
- Usar `postMessage('*')`: permite fuga de eventos y confusion entre ventanas.
- Crear sesiones desde JavaScript publico: expone identidad de comercio y
  permite alterar deuda o monto.
- Declarar exito por URL de retorno: el navegador no es fuente de verdad.
