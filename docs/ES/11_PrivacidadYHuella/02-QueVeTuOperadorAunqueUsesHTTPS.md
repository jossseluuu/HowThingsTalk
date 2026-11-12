---
title: "¿Qué puede ver tu operador de internet de lo que haces aunque la web use HTTPS?"
section: "11_PrivacidadYHuella"
language: "es"
status: "published"
order: 2
depth: "intermediate"
summary: "HTTPS cifra el contenido: la ruta de la página, los formularios, las cookies y lo que descargas. No oculta con quién hablas ni cuánto y cuándo hablas. Las direcciones IP, las consultas DNS clásicas y el nombre del sitio en el saludo TLS viajan en claro, y el tamaño y el ritmo de los paquetes también dicen cosas. DNS cifrado y Encrypted Client Hello tapan parte de eso, pero no todo."
prerequisites:
  - "Cómo entrega TCP los datos en orden (artículo 04·01)"
  - "Cómo acuerdan dos aparatos una clave secreta (artículo 10·01)"
translation: "../../EN/11_PrivacyAndFootprint/02-WhatYourISPSeesDespiteHTTPS.md"
---

# ¿Qué puede ver tu operador de internet de lo que haces aunque la web use HTTPS?

## Pregunta y encuadre

El candado del navegador indica que la conexión va cifrada con TLS. Mucha gente lo entiende como «nadie ve lo que hago». La pregunta es más precisa: ¿qué datos de una visita a una web siguen siendo visibles para quien está en medio del camino, como el operador de internet o el dueño de una WiFi pública?

El artículo separa lo que HTTPS protege de lo que deja a la vista. Recorre los cuatro rastros principales: las direcciones IP, la consulta DNS, el nombre del sitio en el saludo TLS y el tamaño y el ritmo del tráfico. Explica también qué cambian el DNS cifrado y Encrypted Client Hello (ECH). No es una guía para configurar VPN ni navegadores, y no compara servicios. Las técnicas de análisis de tráfico se describen a nivel de concepto, para entender la propia exposición.

La profundidad es intermedia. Ayuda saber qué son un paquete, una dirección IP y un puerto ([artículo 04·01](../04_TransferenciaDeDatos/01-ComoEntregaTCPLosDatosEnOrden.md)). También ayuda saber que TLS empieza con un intercambio de claves ([artículo 10·01](../10_SeguridadYCriptografia/01-ComoAcuerdanDosAparatosUnaClaveSecreta.md)).

## Respuesta corta

HTTPS oculta qué dices, pero no a quién se lo dices ni cuánto hablas: la IP del servidor, la consulta DNS clásica, el nombre del sitio en el saludo TLS y el patrón de tamaños y tiempos siguen a la vista de quien está en el camino.

### Resumen de 60 segundos

Cuando visitas `https://ejemplo.org/salud/consulta?id=7`, todo lo que va después del nombre del sitio viaja cifrado. Eso incluye la ruta `/salud/consulta`, el parámetro `id=7`, las cookies, los formularios y el contenido de la página. El operador no puede leerlo.

Pero para llegar hasta ahí el móvil hace cosas que sí se ven:

1. **Pregunta por el nombre.** Una consulta DNS clásica pregunta «¿qué IP tiene `ejemplo.org`?» sin cifrar.
2. **Abre la conexión.** Cada paquete lleva en claro la IP de origen y la de destino, porque los routers la necesitan para encaminarlo.
3. **Saluda al servidor.** El primer mensaje de TLS lleva el nombre del sitio en claro, en un campo llamado SNI. El servidor lo necesita para saber qué certificado presentar.
4. **Transfiere datos.** El cifrado no oculta cuántos bytes van y vienen ni cuándo.

El DNS cifrado tapa el primer rastro. ECH tapa el tercero si el servidor lo admite. Los otros dos quedan, aunque con matices.

## Modelo mental

Piensa en una carta dentro de un sobre opaco y bien cerrado. Nadie en Correos puede leerla. Pero el sobre lleva la dirección del destinatario, porque si no, no llega. El cartero también ve el peso del sobre, el día en que lo envías y cuántos envías a la semana.

HTTPS es el sobre opaco. La IP de destino es la dirección del sobre. El tamaño y el ritmo del tráfico son el peso y la frecuencia.

La analogía se rompe en dos puntos. Primero, en internet una dirección IP no siempre identifica a un destinatario: muchos sitios distintos comparten la misma IP en un mismo proveedor. Por eso el protocolo añade una segunda etiqueta con el nombre del sitio, el SNI, y esa etiqueta iba en claro por diseño. Segundo, una carta no tiene que preguntar antes la dirección a una centralita que lo anota todo. En la web, la consulta DNS hace justo eso.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

Una visita típica a una web por HTTPS sigue esta secuencia:

1. **Resolución DNS.** El sistema pregunta a un servidor DNS recursivo la IP del nombre. Sin cifrar, la consulta viaja por UDP al puerto 53. Según RFC 9076 (apartados 1 y 5.1), cualquiera en el camino ve el nombre consultado y la IP de quien pregunta. El resolutor recursivo, que suele ser del propio operador, ve todas las consultas de sus clientes.
2. **Conexión de transporte.** El móvil abre una conexión TCP, o QUIC sobre UDP, hacia la IP del servidor. Las cabeceras IP y TCP van en claro. RFC 7624 (apartado 3) recuerda que el cifrado protege poco frente a inferencias basadas en esas partes no cifradas.
3. **Saludo TLS.** El cliente envía un mensaje `ClientHello`. Lleva la extensión `server_name` (SNI) con el nombre del sitio. Según RFC 8744 (apartado 2), ese nombre viaja en claro. RFC 9849 (apartado 1) lo describe como lo más sensible que TLS 1.3 deja sin cifrar.
4. **Respuesta del servidor.** En TLS 1.3, todo lo que sigue al `ServerHello` va cifrado, incluido el certificado del servidor (RFC 8446, apartado 1.2 y apartado 4.4.2). En TLS 1.2, el certificado viajaba en claro y repetía el nombre del sitio.
5. **Datos de aplicación.** Las peticiones y respuestas HTTP van en registros TLS cifrados. Aquí están la ruta, los parámetros, las cabeceras, las cookies y el contenido.

### 2. Qué hace cada componente

**La dirección IP.** Identifica la máquina a la que se entrega cada paquete. No se puede cifrar extremo a extremo, porque cada router del camino la lee. Pero no siempre señala un sitio concreto. RFC 8744 (apartados 1 y 2.2) explica que el alojamiento virtual y las redes de distribución de contenidos (CDN) sirven muchos sitios desde pocas direcciones. Una IP de un proveedor grande puede corresponder a miles de dominios.

**El DNS.** Traduce nombres en direcciones. El DNS clásico no cifra nada. Las variantes cifradas son dos principales:

- **DNS sobre TLS (DoT, RFC 7858, mayo de 2016):** usa TCP y el puerto 853, reservado para este fin.
- **DNS sobre HTTPS (DoH, RFC 8484, octubre de 2018):** mete cada consulta en una petición HTTPS al puerto 443. Según el RFC (apartado 8.1), compartir puerto y conexión con otro tráfico HTTPS hace más difícil distinguirlo.

El cifrado oculta las consultas a quien está en el camino. No a quien las responde. RFC 9076 (apartado 6.1) insiste en que el resolutor ve las consultas igual. Elegir un resolutor desplaza la confianza hacia quien lo gestiona; no la elimina.

**El SNI.** Existe porque una misma IP puede alojar muchos sitios. Al recibir el `ClientHello`, el servidor mira el nombre y elige el certificado y la configuración correctos. RFC 8744 (apartado 2.1) enumera quién más lo lee: cortafuegos de empresa, sistemas de bloqueo de sitios, gestión de calidad de servicio de los operadores y vigilancia masiva.

**Encrypted Client Hello (ECH, RFC 9849, marzo de 2026).** Divide el saludo en dos:

- un `ClientHelloInner`, el saludo real con el nombre verdadero, cifrado con una clave pública del proveedor;
- un `ClientHelloOuter`, un sobre exterior visible cuyo SNI lleva un nombre público genérico del proveedor, el `public_name`.

La clave pública se publica en el DNS, en registros HTTPS (RFC 9460), o viene preconfigurada. Quien está en el camino ve el nombre genérico y la IP del proveedor. El sitio real queda escondido entre todos los que comparten ese proveedor. Para que no destaque quién usa ECH, los clientes sin configuración pueden enviar una extensión ECH falsa, llamada GREASE (RFC 9849, apartado 6). ECH solo funciona si el servidor lo admite. La proporción de sitios que lo admiten no está medida en este artículo (no verificado).

**QUIC y HTTP/3.** QUIC cifra casi todo el paquete. Pero sus primeros paquetes, los *Initial*, se protegen con claves que se derivan de un identificador de conexión visible y de un valor público fijo (RFC 9001, apartado 5.2). Cualquiera puede derivarlas. El `ClientHello` va dentro (RFC 9001, apartado 4.3), así que el SNI es legible igual que en TLS sobre TCP (inferencia a partir de esos dos apartados). ECH lo protege también aquí.

**El tamaño y el ritmo.** TLS no oculta la longitud de lo que transmite (RFC 8446, apartado 1). Permite rellenar registros para disimularla (apartado 5.4), pero el relleno es opcional y la política queda fuera de la norma. El apéndice E.3 del mismo RFC reconoce que TLS es vulnerable al análisis de longitudes y tiempos.

### 3. Parámetros que importan

| Rastro | DNS clásico + TLS 1.2 | DNS cifrado + TLS 1.3 | DNS cifrado + TLS 1.3 + ECH |
|---|---|---|---|
| IP del servidor | Visible | Visible | Visible |
| Nombre en DNS | Visible (UDP 53) | Oculto al operador; lo ve el resolutor | Oculto al operador; lo ve el resolutor |
| Nombre en SNI | Visible | Visible | Solo el nombre público del proveedor |
| Nombre en el certificado | Visible | Cifrado | Cifrado |
| Ruta, parámetros, cookies, contenido | Cifrados | Cifrados | Cifrados |
| Tamaño, dirección y tiempos | Visibles | Visibles | Visibles |

Algunas cifras de referencia sobre lo que revela el tamaño y el ritmo:

- **HTTPS sin relleno, páginas de un mismo sitio.** Miller y colaboradores recogieron 463 125 cargas de páginas de 10 sitios web conocidos entre diciembre de 2013 y enero de 2014. Con solo el tamaño de las ráfagas de paquetes, identificaron la página concreta visitada dentro de un mismo sitio con un 90 % de acierto, sobre 500 páginas por sitio. Un relleno adaptado bajó el acierto al 25 % a cambio de un 9 % más de tráfico.
- **Escala real.** Panchenko y colaboradores probaron métodos de este tipo con más de 300 000 páginas, sobre tráfico de Tor. Concluyeron que ningún método, incluido el suyo, escalaba a un escenario realista con un fondo enorme de páginas posibles. Reconocer una página cualquiera entre todas se parecía a buscar una aguja en un pajar.

Las dos cifras miden escenarios distintos. La primera supone que ya se sabe qué sitio visitas y solo hay que adivinar la página. La segunda intenta adivinar entre cualquier página de internet.

### 4. Diagrama o flujo

```text
 Móvil                     Operador / WiFi                    Servidor
   │                       (quien está en medio)                  │
   │── DNS clásico: «¿IP de ejemplo.org?» ──► VISIBLE: nombre     │
   │   (DoT/DoH: cifrado → solo ve que hay consultas a un resolutor)
   │                                                              │
   │── Paquete IP: origen 10.0.0.5 → destino 203.0.113.7 ────────►│
   │                         VISIBLE: las dos IP, puertos, hora   │
   │                                                              │
   │── ClientHello con SNI = «ejemplo.org» ──────────────────────►│
   │                         VISIBLE (con ECH: «cdn-proveedor.net»)│
   │◄─ ServerHello ── [certificado y resto, cifrados en TLS 1.3] ─│
   │                                                              │
   │══ GET /salud/consulta?id=7 + cookies ══ CIFRADO ════════════►│
   │◄═ página HTML, imágenes ═══════════════ CIFRADO ═════════════│
   │                         VISIBLE: cuántos bytes, en qué ráfagas,│
   │                         en qué momento y en qué dirección    │
```

## Límites, excepciones y casos reales

**La IP sola es ambigua, pero no siempre.** En un sitio alojado en un gran CDN, la IP comparte espacio con muchos otros dominios. En un sitio con servidor propio, la IP lo identifica casi por completo. ECH no cambia esto: si el proveedor solo aloja un sitio, el conjunto donde esconderse tiene tamaño uno. RFC 9849 (apartado 1) advierte que ECH no basta por sí solo para proteger la identidad del servidor.

**ECH sin DNS cifrado sirve de poco.** Si la consulta DNS sale en claro, el nombre ya se ha visto antes del saludo. RFC 9849 (apartado 1) lo señala y remite a DoH, DoT y DNS sobre QUIC.

**El DNS cifrado traslada la confianza.** Si el resolutor cifrado es el del propio operador, este sigue viendo las consultas, solo que ya no las ve un tercero en la WiFi del café. Si es otro servicio, ese servicio pasa a verlas.

**Otras fugas fuera del navegador.** RFC 9849 (apartado 10.7) menciona conexiones previas sin cifrar y consultas de validez de certificados como posibles fuentes de información. Las aplicaciones del móvil pueden usar sus propios resolutores y conexiones, con reglas distintas a las del navegador (no verificado de forma general).

**El análisis de tráfico depende mucho del escenario.** Las cifras de Miller son de un mundo cerrado de 10 sitios, sin relleno y con datos de 2013-2014. Las de Panchenko muestran que el problema se complica mucho a escala real. Ninguna de las dos es una medida de lo que un operador concreto hace hoy. Lo que sí establecen los RFC es que TLS no promete ocultar longitudes ni tiempos.

**Una VPN no elimina el problema, lo mueve.** Con una VPN, el operador ve una sola conexión cifrada hacia el servidor VPN, con su tamaño y su ritmo. El proveedor de la VPN pasa a ocupar el lugar del operador: ve las IP de destino y, según su configuración, el DNS y el SNI.

## Consecuencias prácticas

Para quien usa internet, HTTPS protege lo más delicado del contenido: contraseñas, mensajes, números de tarjeta, la página exacta y lo que escribes en ella. Pero no oculta la lista de sitios que visitas ni cuándo. En una WiFi pública, el dueño de la red puede ver esa lista igual que el operador.

El DNS cifrado y ECH reducen esa lista a «IP de proveedores y nombres públicos genéricos». No la hacen desaparecer. Y siempre queda alguien que ve más: el resolutor DNS, el proveedor que aloja el sitio o la VPN.

Para quien diseña servicios, alojar el sitio detrás de un proveedor grande con ECH aumenta el conjunto entre el que se esconde cada visita. Rellenar las respuestas sensibles hasta tamaños fijos dificulta el análisis por tamaño, a cambio de más tráfico.

Para quien gestiona redes, filtrar o priorizar por SNI deja de funcionar con ECH. Lo que se verá será el nombre público del proveedor, no el sitio.

## Preguntas frecuentes

### ¿El modo incógnito del navegador oculta algo al operador?

No. El modo incógnito evita guardar historial, cookies y datos en el propio aparato. El tráfico que sale a la red es el mismo, con las mismas IP, consultas DNS y SNI.

### Si la web usa HTTPS, ¿el operador sabe qué vídeo veo o qué artículo leo?

No puede leer la ruta ni el contenido. Sí ve el sitio, por la IP o el SNI, y el volumen de datos. Los estudios de análisis de tráfico muestran que, en condiciones controladas, el patrón de tamaños puede delatar páginas concretas de un sitio conocido. En condiciones reales es bastante más difícil.

### ¿Es lo mismo cifrar el DNS que usar ECH?

No. Son dos rastros distintos. El DNS cifrado oculta la pregunta «¿qué IP tiene este nombre?». ECH oculta el nombre dentro del saludo TLS. Hacen falta los dos para que el nombre no aparezca en claro en ningún momento.

### ¿Y la dirección MAC del móvil?

La dirección MAC no sale de la red local: el operador de internet no la ve. Su papel en la privacidad se explica en el [artículo 11·01](01-PorQueElMovilCambiaSuDireccionMAC.md).

## Fuentes

- [RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3 (IETF, agosto de 2018)](https://www.rfc-editor.org/rfc/rfc8446.html). Cifrado de todo lo posterior al `ServerHello` (apartado 1.2), certificado cifrado (4.4.2), TLS no oculta longitudes (1), relleno opcional de registros (5.4) y vulnerabilidad al análisis de longitudes y tiempos (apéndice E.3).
- [RFC 8744 — Issues and Requirements for Server Name Identification (SNI) Encryption in TLS (IETF, julio de 2020)](https://www.rfc-editor.org/rfc/rfc8744.html). SNI en claro, alojamiento compartido y CDN (apartados 1 y 2.2) y usos del SNI por terceros (2.1).
- [RFC 9849 — TLS Encrypted Client Hello (IETF, marzo de 2026)](https://www.rfc-editor.org/rfc/rfc9849.html). Saludos interior y exterior y `public_name` (apartados 3, 5 y 6.1), publicación de la configuración en DNS (3.2), GREASE (6), límites de ECH y necesidad de DNS cifrado (1) y otras fugas (10.7).
- [RFC 9076 — DNS Privacy Considerations (IETF, julio de 2021)](https://www.rfc-editor.org/rfc/rfc9076.html). Qué revela una consulta DNS en claro y a quién (apartados 1 y 5.1) y el resolutor como punto que lo ve todo (6.1).
- [RFC 7858 — Specification for DNS over Transport Layer Security (TLS) (IETF, mayo de 2016)](https://www.rfc-editor.org/rfc/rfc7858.html). DNS sobre TLS en el puerto 853 (apartado 3.1).
- [RFC 8484 — DNS Queries over HTTPS (DoH) (IETF, octubre de 2018)](https://www.rfc-editor.org/rfc/rfc8484.html). Consultas DNS dentro de HTTPS y efecto de compartir el puerto 443 (apartado 8.1).
- [RFC 9460 — Service Binding and Parameter Specification via the DNS (SVCB and HTTPS Resource Records) (IETF, noviembre de 2023)](https://www.rfc-editor.org/rfc/rfc9460.html). Registros HTTPS que transportan la configuración de ECH.
- [RFC 9001 — Using TLS to Secure QUIC (IETF, mayo de 2021)](https://www.rfc-editor.org/rfc/rfc9001.html). `ClientHello` en los paquetes *Initial* (apartado 4.3) y claves derivadas de datos públicos (5.2).
- [RFC 7624 — Confidentiality in the Face of Pervasive Surveillance: A Threat Model and Problem Statement (IAB, agosto de 2015)](https://www.rfc-editor.org/rfc/rfc7624.html). Inferencia a partir de cabeceras no cifradas y tamaños de flujos (apartados 2 y 3).
- [Miller, B., Huang, L., Joseph, A. D., Tygar, J. D. — I Know Why You Went to the Clinic: Risks and Realization of HTTPS Traffic Analysis. Privacy Enhancing Technologies Symposium (PETS), 2014](https://petsymposium.org/2014/papers/Miller.pdf). 463 125 cargas de 10 sitios, 90 % de acierto por página dentro de un sitio y defensa que lo baja al 25 % con un 9 % más de tráfico.
- [Panchenko, A. et al. — Website Fingerprinting at Internet Scale. NDSS Symposium, 2016](https://www.ndss-symposium.org/wp-content/uploads/2017/09/website-fingerprinting-internet-scale.pdf). Más de 300 000 páginas sobre Tor y conclusión de que ningún método evaluado escala a un escenario realista.
