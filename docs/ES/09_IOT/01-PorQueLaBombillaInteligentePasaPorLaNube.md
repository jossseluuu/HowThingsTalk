---
title: "¿Por qué una bombilla inteligente pasa por un servidor lejano para encenderse desde la misma habitación?"
section: "09_IOT"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "El router de casa no deja entrar conexiones que nadie ha pedido desde dentro. Por eso la bombilla abre ella misma una conexión permanente hacia el servidor de su fabricante y la mantiene viva. La app no habla con la bombilla: habla con ese servidor, que reenvía la orden por la conexión ya abierta. Es una decisión de diseño, no una necesidad física: existen caminos locales, como el que estandariza Matter."
prerequisites:
  - "Qué es una conexión TCP y cómo entrega los datos en orden (artículo 04·01)"
  - "Qué es una dirección IP y una red local (lo necesario se explica en el artículo)"
translation: "../../EN/09_IOT/01-WhySmartBulbsGoThroughTheCloud.md"
---

# ¿Por qué una bombilla inteligente pasa por un servidor lejano para encenderse desde la misma habitación?

## Pregunta y encuadre

Estás en el salón. El móvil y la bombilla están conectados al mismo router WiFi, a pocos metros. Tocas el botón de la app y la luz se enciende. Pero la orden no ha ido del móvil a la bombilla. Ha salido de casa, ha llegado a un centro de datos del fabricante, quizá a cientos o miles de kilómetros, y ha vuelto. La pregunta es por qué se diseña así algo que parece tan sencillo.

Este artículo explica el caso más común: una bombilla WiFi que se controla con la app de su fabricante. También compara ese diseño con el control local que define Matter.

- **Profundidad:** intermedia. Hay cuentas de retardo sencillas y un diagrama de mensajes.
- **Conocimientos previos:** saber qué es una conexión TCP ayuda. Está explicado en [¿Cómo entrega TCP los datos en orden si los paquetes llegan desordenados?](../04_TransferenciaDeDatos/01-ComoEntregaTCPLosDatosEnOrden.md). Lo necesario sobre direcciones IP y routers se explica aquí.
- **Fuera de alcance:** cómo configurar un router o una bombilla, la comparación entre marcas, el cifrado TLS por dentro y las bombillas que solo usan Bluetooth.

## Respuesta corta

Desde fuera de casa nadie puede abrir una conexión hacia la bombilla, así que es la bombilla la que abre y mantiene una conexión hacia el servidor del fabricante, y la app siempre usa ese camino, también cuando estás al lado, porque es el único que funciona igual en todas partes.

### Resumen de 60 segundos

Dentro de casa, cada aparato tiene una dirección privada. El router comparte una única dirección pública entre todos ellos. Para hacerlo apunta qué aparato de dentro ha hablado con qué servidor de fuera. Cuando llega una respuesta, sabe a quién entregarla. Cuando llega algo que nadie de dentro ha pedido, no tiene ninguna entrada que le diga a quién dárselo.

Por eso la bombilla, al encenderse, se conecta ella misma al servidor de su fabricante. Esa conexión sale de casa, así que el router la deja pasar y apunta el camino de vuelta. La bombilla la mantiene abierta enviando de vez en cuando un mensaje mínimo.

Cuando tocas la app, el móvil envía la orden al servidor. El servidor la reenvía a la bombilla por la conexión que ella abrió. Funciona igual desde el sofá que desde otro país. Además, el servidor guarda el estado, las rutinas y la cuenta del usuario.

El precio es depender de internet y de la empresa. Hay alternativas locales, pero exigen resolver en casa cosas que el servidor resuelve solo: encontrar la bombilla y demostrar quién tiene permiso para mandarle órdenes.

## Modelo mental

Piensa en un edificio de oficinas con una recepción muy estricta. La recepcionista deja salir cualquier llamada. Si alguien de fuera devuelve una llamada, se la pasa a quien la hizo. Pero si llama un desconocido preguntando por «el de la lámpara», cuelga: no sabe quién es ni si alguien lo espera.

La bombilla trabaja en una de esas oficinas. Al llegar por la mañana llama a la central de su empresa y deja la línea abierta. Tú, aunque estés en el despacho de al lado, no vas a verla: llamas a la central y la central le pasa el recado por esa línea.

La recepcionista es el router con NAT. La central es el servidor del fabricante. La línea abierta es la conexión persistente.

**Dónde se rompe la analogía.** Una línea telefónica abierta ocupa algo todo el rato. La conexión de la bombilla no: entre mensajes no viaja nada, salvo algún paquete pequeño para que el router no la olvide. Además, en el edificio no hay forma de ir al despacho de al lado. En una red local sí la hay: el móvil y la bombilla pueden hablarse directamente. Que no lo hagan es una decisión de diseño, no una imposibilidad física.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

**La puesta en marcha.** La bombilla se une a la red WiFi de casa. El router le da una dirección IP privada. Durante la configuración, la app la asocia a la cuenta del usuario en el servidor del fabricante.

**La conexión hacia fuera.** La bombilla abre una conexión cifrada con un servidor de mensajería del fabricante. Un protocolo muy usado para esto es MQTT. Su norma registra en IANA el puerto TCP 8883 para MQTT sobre TLS y el 1883 sin cifrar (MQTT 5.0, apartado 4.2). Una plataforma documentada, AWS IoT Core, acepta MQTT sobre TLS en el puerto 8883, y también en el 443 con una extensión de TLS llamada ALPN (AWS, *Protocols*). El 443 es el mismo puerto que usa HTTPS.

**La suscripción.** En MQTT, el cliente es quien abre la conexión de red con el servidor (MQTT 5.0, apartado 1.2). Por esa conexión, la bombilla se suscribe a un tema, un nombre que identifica sus mensajes. Después, el servidor usa un mensaje PUBLISH para entregar cada mensaje a cada cliente con una suscripción que coincida (MQTT 5.0, apartado 3.3.4). Lo transmite por la conexión de esa sesión cuando está activa (MQTT 5.0, apartado 4.8.1). Así, los mensajes van del servidor a la bombilla por un camino que abrió la bombilla.

**La orden.** Cuando tocas la app:

1. El móvil envía la orden a la API del fabricante, por internet.
2. El servidor apunta el nuevo estado deseado: «encendida».
3. El servidor publica un mensaje en el tema de la bombilla.
4. El mensaje baja por la conexión que la bombilla mantiene abierta.
5. La bombilla enciende el LED y publica su nuevo estado.
6. El servidor actualiza el estado y la app lo muestra.

AWS documenta este esquema con un documento llamado *sombra* del dispositivo (AWS, *Device Shadow*). La sombra guarda dos estados: el deseado, que escribe la app, y el comunicado, que escribe el aparato. La diferencia entre ambos es lo que el servidor envía al aparato. Si el aparato estaba desconectado, recibe el estado actual al volver. No todos los fabricantes usan AWS, pero la idea es habitual: el estado de la bombilla vive en la nube, y la bombilla lo sigue.

### 2. Qué hace cada componente

**El router con NAT.** En casa hay muchas direcciones privadas y normalmente una sola pública. El router traduce entre ellas (NAT). Cuando un aparato de dentro envía un paquete hacia fuera, el router crea una asignación: «este puerto público corresponde a este aparato y este puerto privado». Las normas del IETF describen cómo deben comportarse esas asignaciones (RFC 4787 para UDP y RFC 5382 para TCP).

Lo importante es el filtrado. Con el filtrado más permisivo, cualquier paquete que llega a un puerto público ya asignado se entrega al aparato de dentro. Con los más estrictos, solo pasa si viene de una dirección, o de una dirección y un puerto, a los que el aparato ya había enviado algo (RFC 4787, apartado 5). En todos los casos, la asignación nace de un paquete que sale. Un paquete que llega sin asignación previa no tiene destino conocido dentro de casa.

**Los temporizadores del router.** Las asignaciones caducan si no se usan.

- En UDP, una asignación no debe caducar antes de 2 minutos, y se recomienda un valor por defecto de 5 minutos o más (RFC 4787, REQ-5). El tráfico que sale debe renovarla (REQ-6).
- En TCP, una conexión establecida no debe borrarse antes de 2 horas y 4 minutos sin tráfico (RFC 5382, REQ-5).

La realidad no siempre sigue la norma. El IAB cita una prueba de 2010 con 34 routers domésticos: más de la mitad no respetaba los tiempos recomendados para UDP (RFC 7452, apartado 4).

**El mensaje para mantenerse vivo.** Por eso la bombilla envía algo periódicamente aunque no tenga nada que decir. En MQTT, el cliente fija un intervalo, el *Keep Alive*. Es un número de 16 bits en segundos: como máximo 65 535 s, unas 18 horas. Si no tiene otra cosa que enviar, el cliente debe mandar un PINGREQ. Si el servidor no recibe nada en 1,5 veces ese intervalo, debe cerrar la conexión (MQTT 5.0, apartado 3.1.2.10). El servidor puede imponer un intervalo menor en su respuesta (apartado 3.2.2.3.14). El valor concreto que usa cada bombilla lo elige el fabricante: queda **no verificado** aquí.

**El servidor del fabricante.** Hace mucho más que reenviar. Guarda la asociación entre cuenta y bombilla. Decide quién puede mandarle órdenes. Ejecuta rutinas y horarios. Conecta con asistentes de voz. El IAB llama a este esquema patrón «del dispositivo a la nube» (RFC 7452, apartado 2.2).

**La app.** Solo necesita saber hablar con la API del fabricante. No necesita saber dónde está la bombilla ni si está en la misma red.

### 3. Parámetros que importan

**El coste en distancia.** La UIT da un valor de planificación de 5 µs por kilómetro para un sistema de fibra óptica, marcado como provisional (UIT-T G.114, tabla A.1). Con un servidor a 2000 km:

```text
móvil → servidor:     2000 km × 5 µs/km = 10 ms
servidor → bombilla:  2000 km × 5 µs/km = 10 ms
orden completa (solo propagación):       ≈ 20 ms
confirmación de vuelta a la app:        +≈ 20 ms
```

Son mínimos. La ruta real por la fibra es más larga que la distancia en línea recta. A eso se suman el WiFi de casa, las colas de los routers y el proceso en el servidor. El retardo total típico de una bombilla comercial depende de cada caso y queda **no verificado** aquí. Una orden local, del móvil a la bombilla por el mismo router, recorre unos pocos metros: la propagación es despreciable.

**Lo que viaja.** Una orden de encendido es un mensaje de pocos bytes. El tráfico de mantenimiento son paquetes pequeños cada cierto tiempo. Lo que cuesta no es el ancho de banda, sino la distancia y los saltos.

**Lo que haría falta para un camino local.** Para hablar directamente con la bombilla, la app tendría que resolver dos problemas que el servidor resuelve solo:

- **Encontrarla.** La dirección privada de la bombilla puede cambiar. Un método estándar es el DNS multidifusión (mDNS): las preguntas van a la dirección 224.0.0.251, o FF02::FB en IPv6, en el puerto UDP 5353 (RFC 6762, apartados 2 y 3). Los nombres que acaban en `.local` solo tienen sentido en el enlace donde se originan (RFC 6762, apartado 3). Funciona dentro de casa, no desde fuera.
- **Confiar en ella, y ella en la app.** Hace falta que los dos compartan credenciales sin pasar por el servidor. Si no, cualquiera conectado al WiFi podría encender la luz.

### 4. Diagrama o flujo

```text
 AL ENCENDER LA BOMBILLA (una vez)
 Bombilla (192.168.1.23) ── TCP + TLS hacia broker:8883 ──► Router NAT ──► Internet ──► Broker MQTT
 Router: crea la asignación  puerto público 40001 ↔ 192.168.1.23:51515
 Bombilla ── SUBSCRIBE "casa/bombilla-23/orden" ─────────────────────────────────────► Broker

 MANTENIMIENTO (cada intervalo Keep Alive)
 Bombilla ── PINGREQ ──► Broker ── PINGRESP ──► Bombilla      (renueva la asignación del router)

 ORDEN DESDE EL SOFÁ (móvil en la misma red)
 Móvil ── HTTPS "encender" ──► Router ──► Internet ──► API del fabricante
 API: estado deseado = encendida ──► Broker
 Broker ── PUBLISH "casa/bombilla-23/orden" ──► Internet ──► Router (puerto 40001)
 Router: la asignación existe → entrega a 192.168.1.23:51515
 Bombilla: enciende ── PUBLISH estado = encendida ──► Broker ──► API ──► app

 LO QUE NO FUNCIONA (sin asignación previa)
 Servidor cualquiera ── conexión nueva hacia la IP pública ──► Router: ¿para quién es? → no se entrega
```

## Límites, excepciones y casos reales

- **Sin internet, sin app.** Si se cae la conexión de casa, la orden no llega al servidor y la bombilla no la recibe, aunque el móvil esté a un metro. Algunos fabricantes ofrecen además un modo local. Cuáles y cómo funciona cada uno queda fuera de este artículo.
- **Si la empresa cierra el servicio.** El IAB lo señala como un riesgo del patrón: un cambio de modelo de negocio, o una quiebra, puede dejar inservible el hardware (RFC 7452, apartado 2.2). Como mitigación menciona publicar el código o permitir otro software en el aparato.
- **Bombillas Zigbee o Thread.** No hablan IP con el router WiFi directamente: necesitan un puente. El IAB lo llama patrón «del dispositivo a la pasarela» (RFC 7452, apartado 2.3). La pasarela puede, a su vez, depender de la nube o funcionar en local. En Thread, ese puente es un router de borde, que encamina IPv6 entre la red Thread y la red de casa (Google, *Thread and IPv6*).
- **Matter estandariza el camino local.** El control y el resto de la comunicación operativa de Matter van sobre IPv6 (Google, *What is Matter?*). Los aparatos que comparten un dominio de seguridad forman un *fabric*, con un certificado raíz común (Google, *Fabric*). Al dar de alta un aparato, el controlador le entrega credenciales de ese fabric. Después, los dos abren una sesión autenticada con certificados, llamada CASE (Google, *Commissioning*). El controlador encuentra los aparatos ya configurados con DNS-SD, buscando el servicio `_matter._tcp` con nombres `.local` (Google, *Discovery*). Un aparato puede pertenecer a varios fabrics a la vez: es lo que se llama multiadministrador (Google, *Fabric*). Eso resuelve los dos problemas del camino local: encontrar el aparato y confiar en él. Si cada producto Matter sigue usando además la nube para el acceso remoto, depende del fabricante.
- **IPv6 no lo arregla por sí solo.** Con IPv6 cada aparato puede tener una dirección pública y no hace falta NAT. Pero los routers domésticos suelen filtrar las conexiones entrantes no solicitadas. Este artículo no lo ha verificado con una norma: queda **no verificado**.
- **El interruptor de la pared.** Si cortas la corriente con el interruptor clásico, la bombilla se apaga y deja de estar conectada. Al volver la corriente, tiene que reconectarse a la WiFi y al servidor antes de obedecer órdenes de la app. Su comportamiento al recibir corriente lo decide cada fabricante.

## Consecuencias prácticas

- **El retardo es pequeño pero variable.** La propagación añade al menos unas decenas de milisegundos con un servidor lejano. Las colas y el proceso pueden añadir más, y eso cambia de una vez a otra.
- **La luz depende de cosas que no están en casa.** La conexión a internet, el servidor del fabricante y la cuenta del usuario tienen que funcionar a la vez.
- **El fabricante ve cada orden.** Cada encendido pasa por su servidor, que además guarda el estado de la bombilla. Qué conserva y durante cuánto tiempo depende de su política, no del protocolo.
- **Para quien diseña el aparato, la nube es el camino fácil.** Una conexión saliente funciona detrás de casi cualquier router sin que el usuario configure nada. El mismo camino sirve dentro y fuera de casa. Y añadir funciones nuevas solo exige cambiar el servidor.
- **El control local exige más trabajo en casa.** Descubrir el aparato y repartir credenciales sin servidor es justo lo que estándares como Matter tienen que definir.

## Preguntas frecuentes

### ¿Pasa por la nube porque la bombilla es demasiado simple para otra cosa?

No. La bombilla ya ejecuta una pila IP y una conexión TLS completas para hablar con el servidor. Podría hablar con el móvil en local. Pasa por la nube porque ese camino funciona detrás del router sin configuración y es el mismo dentro y fuera de casa.

### ¿Es lo mismo que abrir un puerto en el router?

No. Abrir un puerto es pedir al router que entregue a un aparato concreto lo que llegue a un puerto público, aunque nadie de dentro lo haya pedido. La conexión de la bombilla no necesita eso: la abre ella hacia fuera. Configurar el router queda fuera de este artículo.

### ¿Matter elimina la nube?

Define un camino local estándar: descubrimiento en la red de casa, credenciales compartidas en un fabric y sesiones autenticadas. Con eso, un controlador de casa puede mandar órdenes sin pasar por un servidor externo. Si un producto concreto usa además la nube para el acceso remoto o los asistentes de voz es decisión de cada fabricante.

### ¿Y una bombilla Bluetooth?

Con Bluetooth, el móvil habla directamente con la bombilla mientras está cerca. No hay router ni servidor en medio, pero tampoco control desde fuera de casa si no hay un puente. Es otro diseño y queda fuera de este artículo.

## Fuentes

- IETF / IAB. Tschofenig, Arkko, Thaler y McPherson. *RFC 7452: Architectural Considerations in Smart Object Networking* (marzo de 2015). Apartados 2.2, 2.3 y 4. <https://www.rfc-editor.org/rfc/rfc7452>
- IETF. Audet y Jennings. *RFC 4787: Network Address Translation (NAT) Behavioral Requirements for Unicast UDP* (enero de 2007). Apartados 4.3 y 5. <https://www.rfc-editor.org/rfc/rfc4787>
- IETF. Guha y otros. *RFC 5382: NAT Behavioral Requirements for TCP* (octubre de 2008). Apartado 5. <https://www.rfc-editor.org/rfc/rfc5382>
- IETF. Cheshire y Krochmal. *RFC 6762: Multicast DNS* (febrero de 2013). Apartados 2 y 3. <https://www.rfc-editor.org/rfc/rfc6762>
- OASIS. *MQTT Version 5.0*, OASIS Standard (7 de marzo de 2019). Apartados 1.2, 3.1.2.10, 3.2.2.3.14, 3.3.4, 4.2 y 4.8.1. <https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html>
- Amazon Web Services. *AWS IoT Core Developer Guide*: «Device communication protocols» y «AWS IoT Device Shadow service». <https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html> · <https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html>
- Google Home Developers. *Matter Primer*: «What is Matter?», «Fabric», «Commissioning», «Commissionable and operational discovery» y «Thread and IPv6». <https://developers.home.google.com/matter/primer> · <https://developers.home.google.com/matter/primer/fabric> · <https://developers.home.google.com/matter/primer/commissioning> · <https://developers.home.google.com/matter/primer/commissionable-and-operational-discovery> · <https://developers.home.google.com/matter/primer/thread-and-ipv6>
- UIT-T. *Recomendación G.114: One-way transmission time* (05/2003). Tabla A.1. <https://www.itu.int/rec/T-REC-G.114>
