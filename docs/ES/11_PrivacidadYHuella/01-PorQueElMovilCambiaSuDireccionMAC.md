---
title: "¿Por qué el móvil usa una dirección MAC distinta en cada red WiFi?"
section: "11_PrivacidadYHuella"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "La dirección MAC de fábrica es un número fijo que el móvil emite sin cifrar en cada trama WiFi, así que sirve para reconocerlo en cualquier sitio. Los sistemas actuales la sustituyen por direcciones aleatorias: una al buscar redes y otra propia para cada red. Eso corta el rastreo más fácil, pero no todos los demás."
prerequisites:
  - "Qué es una red WiFi y un punto de acceso, a nivel general"
  - "Capas de una pila de protocolos (glosario)"
translation: "../../EN/11_PrivacyAndFootprint/01-WhyPhonesRandomizeTheirMACAddress.md"
---

# ¿Por qué el móvil usa una dirección MAC distinta en cada red WiFi?

## Pregunta y encuadre

Si abres los ajustes WiFi de un móvil reciente, verás algo como «dirección privada» o «MAC aleatoria». La dirección que el móvil usa en la red de casa no es la misma que usa en la del trabajo. Y ninguna de las dos es la que vino grabada de fábrica. La pregunta es por qué los sistemas operativos hacen esto y qué consiguen de verdad.

El artículo explica qué es una dirección MAC, por qué la de fábrica sirve para seguir a un aparato y cómo la sustituyen iOS y Android. También explica qué rastros quedan aunque la dirección cambie. No es una guía de ajustes. Tampoco describe técnicas para rastrear aparatos ajenos: los puntos débiles se nombran a nivel de concepto, para entender la propia exposición. No cubre Bluetooth, que tiene su propio mecanismo de direcciones aleatorias.

La profundidad es intermedia. Basta saber qué es una red WiFi y tener una idea de las capas de una pila de protocolos, que se definen en el [glosario](../../../guides/TaxonomyAndGlossary.md).

## Respuesta corta

La MAC de fábrica es un identificador fijo que viaja sin cifrar en cada trama WiFi; al cambiarla por direcciones aleatorias, distintas al buscar redes y distintas en cada red, el móvil deja de ser el mismo número en todas partes.

### Resumen de 60 segundos

Cada tarjeta WiFi trae una dirección de 48 bits asignada por el fabricante. Esa dirección va en la cabecera de cada trama que emite el aparato. El cifrado WiFi protege el contenido, pero no esa cabecera. Además, un móvil con el WiFi encendido pregunta cada poco «¿hay alguna red por aquí?», incluso sin estar conectado. Cada pregunta lleva su dirección.

Con una dirección fija, cualquier receptor en una tienda, una estación o un estadio puede anotar «este aparato pasó por aquí a las 9:14». Con varios receptores, eso se convierte en un recorrido.

La solución es inventar direcciones. El estándar reserva un bit para marcar una dirección como «administrada localmente», es decir, no asignada por un fabricante. El móvil pone ese bit a 1 y rellena al azar casi todo lo demás. Usa una dirección aleatoria para buscar redes. Al conectarse, usa otra dirección propia de esa red, distinta en cada red. Así, la red de casa y la cafetería no ven el mismo número.

El cambio no hace invisible al móvil. Otros datos de las tramas, el contenido de la conexión y el inicio de sesión pueden seguir identificándolo.

## Modelo mental

Piensa en un abrigo con tu nombre bordado en la espalda. Vayas donde vayas, cualquiera que mire puede apuntar que pasaste. La dirección MAC de fábrica es ese bordado.

La aleatorización equivale a quitarte el bordado y llevar una pegatina distinta en cada sitio. En el gimnasio llevas siempre la misma, para que te reconozcan como socio. En el trabajo llevas otra. Cuando solo vas por la calle, llevas una que cambias a menudo.

La analogía se rompe en un punto importante. Una pegatina nueva no cambia tu forma de andar, tu ropa ni tu voz. En WiFi pasa lo mismo: el móvil sigue emitiendo una combinación de capacidades, una forma de buscar redes y un tráfico propio. Esos detalles pueden delatarlo aunque la dirección cambie. Además, en cuanto inicias sesión en una red, la pegatina da igual: la red ya sabe quién eres.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

Una dirección MAC identifica una interfaz de red en el nivel de enlace. En WiFi, cada trama lleva la dirección de origen y la de destino. Los puntos de acceso las necesitan para saber a quién responder.

Un móvil tiene tres momentos en los que emite su dirección:

1. **Búsqueda de redes.** Para encontrar redes conocidas, el móvil envía tramas *probe request*. RFC 9724 recuerda que estas tramas llevan la dirección MAC de origen. Se emiten aunque el móvil no esté conectado a nada.
2. **Asociación.** Al conectarse a una red, el móvil se presenta al punto de acceso con una dirección y negocia la conexión.
3. **Conexión activa.** Cada trama de datos lleva la dirección. El contenido va cifrado con WPA2 o WPA3, pero la cabecera con las direcciones no.

La aleatorización actúa en los tres momentos, con reglas distintas en cada uno.

### 2. Qué hace cada componente

**La dirección de fábrica.** Tiene 48 bits. Según RFC 9724 (apartado 2.1), los tres primeros octetos son el OUI, que identifica a la organización que la emitió. Los otros tres los asigna el fabricante a cada tarjeta. El resultado es único en el mundo y no cambia nunca.

**Los dos bits de control.** En el primer octeto hay dos bits especiales (RFC 9724, figura 1):

- el bit I/G, el menos significativo, indica si la dirección es de un solo equipo (0) o de grupo (1);
- el bit U/L, el siguiente, indica si es universal, asignada por fabricante (0), o administrada localmente (1).

Una dirección aleatoria de un móvil lleva U/L = 1 e I/G = 0. La documentación de Android lo confirma: esos dos bits quedan fijados y los otros 46 son aleatorios. Una consecuencia práctica (calculada): el segundo carácter hexadecimal de una dirección aleatoria siempre es 2, 6, A o E.

**El espacio de direcciones.** Con 46 bits libres hay 2⁴⁶ ≈ 7,0 × 10¹³ direcciones posibles (calculado). Dos móviles de la misma red casi nunca coinciden. RFC 9724 (apartado 2.3) recoge experimentos que encontraron duplicados en proporción despreciable.

**Las direcciones para buscar redes.** Los sistemas usan una dirección aleatoria en las búsquedas cuando no están conectados. Según la guía de seguridad de Apple, la dirección cambia al desconectarse. Apple aleatoriza también los números de secuencia de las tramas. Lo hace cada vez que cambia la dirección y en cada nueva búsqueda sin conexión. Esto existe desde iOS 16.1 y macOS 13.1. Antes, un contador continuo podía enlazar dos direcciones aleatorias seguidas.

**La dirección propia de cada red.** Al conectarse, el móvil no usa la dirección de búsqueda. Usa una dirección ligada a esa red concreta. La red ve siempre el mismo número, lo que mantiene funcionando la reconexión y las reservas de IP. Otra red ve otro número distinto.

**El sistema operativo.** Decide cuándo se genera cada dirección y cuánto dura. Las políticas de iOS y Android difieren, y se detallan a continuación.

### 3. Parámetros que importan

**Android (según la documentación de Android Open Source Project).**

- Desde Android 10, las conexiones usan por defecto una dirección aleatoria *persistente*.
- Se genera a partir de los datos del perfil de red: el SSID, el tipo de seguridad o, en redes Passpoint, el FQDN.
- Se mantiene hasta un restablecimiento de fábrica. Olvidar la red y volver a añadirla no la cambia, porque los datos del perfil son los mismos.
- Desde Android 12 existe una variante *no persistente* para algunas redes. Se activa si una app que sugiere redes lo pide, o en redes abiertas sin portal cautivo si el fabricante habilita una opción que viene desactivada.
- La variante no persistente solo cambia al empezar una conexión nueva. Cambia si la concesión DHCP ha caducado y han pasado más de 4 horas desde la última desconexión, o si la dirección tiene más de 24 horas.

**Apple (según el artículo de soporte 102509 y la guía de seguridad de plataforma).**

- Las direcciones privadas por red existen desde iOS 14, iPadOS 14 y watchOS 7.
- Desde iOS 18, iPadOS 18 y visionOS 2 hay tres opciones: desactivada, fija y rotativa. También existen en watchOS 11 y macOS Sequoia 15.
- **Fija:** no cambia con el tiempo. Es la opción por defecto en redes nuevas con WPA2 o una seguridad superior.
- **Rotativa:** cambia cada 2 semanas. Es la opción por defecto en redes nuevas con seguridad débil o sin seguridad.
- En esas versiones, olvidar una red borra también su dirección privada, salvo que se haya olvidado esa red hace menos de 24 horas. En versiones anteriores el plazo era de 2 semanas.
- Borrar el contenido del aparato o restablecer los ajustes de red da direcciones nuevas en la siguiente conexión.

**Otros sistemas.** RFC 9724 resume pruebas hechas en 2021. Windows 10, Linux, Android 10 e iOS 14 usaban direcciones por red y direcciones aleatorias para buscar redes. Windows 10 era el único que además ofrecía rotación diaria. Solo Android 10 e iOS 14 tenían activada por defecto la aleatorización de las búsquedas. El RFC indica que no detectó cambios importantes hasta su publicación, en marzo de 2025.

**La recomendación de IEEE.** Según RFC 9724, la norma IEEE 802E-2020 recomienda identificadores temporales salvo que haya un motivo fuerte para que sean permanentes.

### 4. Diagrama o flujo

```text
 Dirección de fábrica (48 bits; no se usa con la aleatorización activa)
   OUI del fabricante (24 bits) │ número de la tarjeta (24 bits)

 Dirección aleatoria
   primer octeto: xxxxxx10   ← U/L = 1 (local), I/G = 0 (un solo equipo)
   resto: 46 bits al azar    → 2⁴⁶ ≈ 7,0 × 10¹³ combinaciones

 ┌───────────────────────────┐
 │ Sin conexión: busca redes │── probe request ──► dirección A₁, A₂, A₃…
 └───────────────────────────┘                     (cambian a menudo)
              │
              ▼ se conecta a «Casa»
 ┌───────────────────────────┐
 │ Red «Casa»                │── tramas de datos ─► dirección C
 └───────────────────────────┘   (la misma cada vez que vuelve a «Casa»)
              │
              ▼ más tarde, en «Oficina»
 ┌───────────────────────────┐
 │ Red «Oficina»             │── tramas de datos ─► dirección O ≠ C
 └───────────────────────────┘

 Resultado: «Casa» y «Oficina» no comparten identificador de enlace,
 y quien solo escucha búsquedas ve direcciones que no se repiten.
```

## Límites, excepciones y casos reales

**La dirección no es el único rastro.** Martin y colaboradores analizaron unos 66 millones de *probe requests* capturados entre enero de 2015 y diciembre de 2016. Contenían unos 2,6 millones de direcciones distintas. Cerca del 53 % de esas direcciones eran aleatorias. Pero los autores advierten que la cifra cuenta direcciones, no aparatos: un solo móvil que aleatoriza genera muchas. En su estudio identificaron varias vías para enlazar direcciones aleatorias con el mismo aparato:

- la combinación de capacidades que cada modelo anuncia en sus búsquedas, que funciona como una huella;
- campos heredados que se derivaban de la dirección de fábrica;
- fallos de implementación que hacían que el aparato emitiera su dirección real en ciertas tramas.

Con métodos pasivos, el estudio afirma que la aleatorización fallaba en torno al 96 % de los móviles Android analizados. Son datos de 2015 y 2016. Las versiones actuales corrigieron muchos de esos fallos, como muestra la aleatorización de números de secuencia de Apple. No hay una cifra pública y actual equivalente para los sistemas de hoy (no verificado).

**Otros identificadores de capas superiores.** RFC 9724 (apartados 2.3 y 5) señala que el identificador de cliente DHCP puede enlazar sesiones aunque la dirección MAC cambie. Por eso RFC 7844 pide que la dirección de enlace, la IP y el identificador DHCP cambien a la vez. Si uno se queda fijo, los demás pierden valor.

**Dentro de una red, la dirección es estable a propósito.** Con la dirección persistente de Android o la fija de Apple, el punto de acceso de casa ve siempre el mismo número. Eso es deseable: la red reconoce al aparato. Pero también significa que esa red concreta puede seguir sus visitas. La dirección aleatoria protege entre redes, no frente a la red a la que te conectas.

**Iniciar sesión anula el anonimato del enlace.** Si una red pide usuario y contraseña, un certificado o un formulario con tu nombre, ya sabe quién eres. RFC 9724 (apartado 6.5) recuerda que un cambio de dirección obliga a una nueva sesión 802.1X y a nuevas direcciones IP. No oculta la identidad de la cuenta.

**Redes que dependían de la dirección fija.** Muchos servicios usaban la MAC como identificador: filtros de acceso, reservas de IP, controles por aparato o portales que recuerdan a un cliente. RFC 9724 explica que la aleatorización por defecto afectó a esos servicios. En 2020 se abrieron en IEEE dos proyectos para tratarlo: 802.11bh, sobre los efectos en las redes y sus servicios, y 802.11bi, sobre nuevos mecanismos de privacidad en la capa de enlace. Según el RFC, en 2024 seguían en marcha.

**Buscar redes sin estar conectado.** Un móvil no conectado a ninguna red sigue buscando redes, por ejemplo para encontrar las conocidas o para servicios de ubicación. Apple indica que esas búsquedas usan direcciones aleatorias. No todos los fabricantes documentan este comportamiento (no verificado para el conjunto de Android).

## Consecuencias prácticas

Para quien usa el móvil, la aleatorización elimina el rastreo más barato: anotar una dirección fija en muchos lugares. No elimina el rastreo de una red en la que inicias sesión, ni el que se hace por contenido, cuenta o aplicación.

Que una red te vea con la misma dirección cada vez es normal y no es un fallo. Es el compromiso entre privacidad y una red que funcione bien.

Si un router o un control parental «pierde» un aparato, la causa puede ser que su dirección cambió. Pasa, por ejemplo, al olvidar la red en iOS o con la dirección rotativa en redes abiertas.

Para quien diseña redes, la MAC ya no sirve como identidad de un aparato. La autenticación tiene que apoyarse en credenciales, certificados o cuentas. RFC 9724 (apartado 9) pide que las futuras especificaciones den a los operadores formas de autenticar aparatos aunque las direcciones cambien.

Para quien diseña aparatos, cambiar la dirección no basta. El resto de identificadores tiene que cambiar a la vez: números de secuencia, identificadores DHCP y cualquier campo derivado de la dirección de fábrica.

## Preguntas frecuentes

### ¿La dirección MAC es lo mismo que la dirección IP?

No. La MAC identifica la interfaz en el enlace local, entre el móvil y el punto de acceso. No pasa del router. La IP identifica al aparato en la red y es la que ven los servidores de internet. La aleatorización de la MAC no cambia la IP pública que ve una web.

### ¿El cifrado WPA2 o WPA3 no oculta ya la dirección?

No. Cifra el contenido de las tramas de datos, pero las direcciones de la cabecera viajan en claro. El punto de acceso las necesita para entregar cada trama. Además, las búsquedas de redes se emiten antes de cualquier cifrado.

### ¿Por qué no cambia la dirección cada pocos minutos también dentro de una red?

Porque cada cambio cuenta como un aparato nuevo. Según RFC 9724, provoca una desconexión, una nueva sesión de autenticación y una nueva dirección IP. Las conexiones abiertas se cortarían. Por eso las políticas por defecto mantienen una dirección por red y reservan los cambios frecuentes para las búsquedas o para redes abiertas.

### ¿Es lo mismo que las direcciones aleatorias de Bluetooth?

Persiguen el mismo objetivo, pero son otro mecanismo, definido por el Bluetooth SIG y con reglas propias. Quedan fuera de este artículo.

## Fuentes

- [RFC 9724 — State of Affairs for Randomized and Changing Media Access Control (MAC) Addresses (IETF, marzo de 2025)](https://www.rfc-editor.org/rfc/rfc9724.html). Estructura de la dirección y bits I/G y U/L (apartado 2.1, figura 1), dirección en las *probe requests* (2.2), duplicados y correlación por identificador DHCP (2.3), IEEE 802E-2020, 802.11bh y 802.11bi (3), RFC 7844 (5), efectos de un cambio de dirección (6.5), pruebas de 2021 en varios sistemas (7, tablas 1 y 2) y requisitos futuros (9).
- [Android Open Source Project — MAC randomization behavior](https://source.android.com/docs/core/connect/wifi-mac-randomization-behavior). Bits fijados y 46 bits aleatorios, aleatorización persistente desde Android 10, generación a partir del perfil de red, variante no persistente desde Android 12 y sus reglas de 4 y 24 horas.
- [Apple — Use private Wi-Fi addresses on Apple devices (artículo 102509)](https://support.apple.com/en-us/102509). Versiones que admiten direcciones privadas, opciones desactivada, fija y rotativa, rotación cada 2 semanas, valores por defecto según la seguridad de la red y efecto de olvidar la red.
- [Apple Platform Security — Privacy features when connecting to wireless networks](https://support.apple.com/guide/security/privacy-features-connecting-wireless-networks-secb9cb3140c/web). Dirección aleatoria en las búsquedas sin conexión, cambio al desconectarse y aleatorización de números de secuencia desde iOS 16.1 y macOS 13.1.
- [Martin, J. et al. — A Study of MAC Address Randomization in Mobile Devices and When it Fails. Proceedings on Privacy Enhancing Technologies, 2017 (4), 365-383](https://petsymposium.org/popets/2017/popets-2017-0054.pdf). Conjunto de unos 66 millones de *probe requests* y 2,6 millones de direcciones (2015-2016), proporción de direcciones aleatorias y vías por las que la aleatorización fallaba.
