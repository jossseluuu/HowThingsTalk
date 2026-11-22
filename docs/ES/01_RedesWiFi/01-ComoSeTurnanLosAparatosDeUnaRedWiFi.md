---
title: "¿Cómo se turnan los aparatos de una red WiFi para no transmitir a la vez?"
section: "01_RedesWiFi"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "En WiFi no hay nadie que reparta turnos para cada trama. Cada aparato escucha el canal, espera a que quede libre un tiempo fijo y después cuenta un número aleatorio de ranuras de 9 µs. Si su trama no se confirma, supone que chocó y dobla el sorteo. Este método, CSMA/CA, explica por qué el WiFi pierde rendimiento con muchos aparatos o con uno lento."
prerequisites:
  - "Relación señal-ruido y por qué la velocidad de un enlace depende de ella (artículo 00·02)"
  - "Qué es un paquete y qué es un acuse de recibo (glosario)"
translation: "../../EN/01_WiFiNetworks/01-HowWiFiDevicesTakeTurnsToTransmit.md"
---

# ¿Cómo se turnan los aparatos de una red WiFi para no transmitir a la vez?

## Pregunta y encuadre

En una casa, el router, dos portátiles, tres móviles y una televisión comparten el mismo canal de radio. Si dos de ellos emiten a la vez en ese canal, sus señales se mezclan y el receptor no entiende ninguna. Nadie les dice a esos aparatos cuándo hablar trama a trama. Aun así, la red funciona. ¿Cómo deciden cuándo transmitir sin pisarse?

Este artículo explica el mecanismo de acceso al medio de la familia IEEE 802.11: la función de coordinación distribuida (DCF) y su versión con prioridades (EDCA). Usa los valores de la capa física OFDM en la banda de 5 GHz, que comparten 802.11a, n y ac.

- **Profundidad:** intermedia. Hay cuentas sencillas con microsegundos y probabilidades.
- **Conocimientos previos:** ayuda saber que la velocidad de un enlace depende de la relación señal-ruido, explicado en [¿Qué decide cuántos bits por segundo caben en una señal de radio?](../00_Fundamentos/02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md).
- **Fuera de alcance:** cómo se conecta un aparato a la red (asociación y autenticación), el cifrado, la elección de canal y el reparto por subportadoras de 802.11ax, que solo se menciona en los límites.

## Respuesta corta

Cada aparato escucha antes de hablar, espera a que el canal lleve un rato libre y después deja pasar un número aleatorio de ranuras; si su trama no recibe confirmación, supone que ha chocado y vuelve a intentarlo con un sorteo el doble de amplio.

### Resumen de 60 segundos

El método se llama CSMA/CA: acceso múltiple por detección de portadora, evitando colisiones. Un aparato que quiere enviar una trama mira primero si el canal está ocupado. Si está libre durante un intervalo fijo (34 µs en 5 GHz), no transmite enseguida. Elige al azar un número entre 0 y 15 y cuenta esas ranuras de 9 µs. Si alguien empieza a emitir mientras cuenta, congela la cuenta y la retoma cuando el canal vuelve a quedar libre.

Cuando llega a cero, transmite. El receptor responde con un acuse (ACK) 16 µs después. Si el ACK no llega, el emisor no sabe por qué, pero supone una colisión. Entonces dobla el rango del sorteo (0-31, 0-63…) hasta un máximo de 1023 y lo intenta de nuevo.

Como cada aparato sortea por su cuenta, es raro que dos elijan el mismo número. Raro no es imposible: con más aparatos compitiendo, las colisiones crecen. Además, el turno se reparte por oportunidades de transmitir, no por tiempo. Un aparato lento ocupa el canal mucho más rato en cada turno y frena a todos los demás.

## Modelo mental

Piensa en una cena con amigos sin moderador. Nadie habla mientras otro habla. Cuando alguien termina, todos esperan un momento. Si dos personas quieren hablar, cada una deja pasar un instante distinto antes de abrir la boca. Quien arranca antes se queda la palabra; el otro se calla y espera al siguiente silencio. Si dos empiezan a la vez, se paran y la próxima vez esperan algo más antes de intentarlo.

El silencio es el canal libre. El momento de espera común es el intervalo fijo. El instante distinto de cada persona es el sorteo aleatorio. Pararse y esperar más tras chocar es el doblado del rango.

**Dónde se rompe la analogía.** En la mesa, quien habla oye si otro empieza a la vez. Una radio WiFi no: mientras emite, su propia señal llega a su antena con muchísima más potencia que la de un vecino, y no puede escuchar a la vez. Solo descubre el choque porque no llega el ACK. Además, en la mesa todos se oyen entre sí. En WiFi, dos aparatos en extremos opuestos de la casa pueden no oírse, aunque ambos oigan al router. Y la «palabra» se reparte por intervenciones, no por minutos: quien habla muy despacio acapara la conversación.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

Secuencia de un envío con acceso básico (IEEE 802.11-2024, función de coordinación distribuida; valores de la capa física OFDM en 5 GHz):

1. **Llega una trama a la cola.** El aparato comprueba el canal con dos mecanismos. El físico (CCA, evaluación de canal libre) mide si hay señal 802.11 o energía en la radio. El virtual (NAV) consulta un temporizador interno que dice si otro aparato ha reservado el canal.
2. **Espera DIFS.** El canal tiene que estar libre durante un intervalo DIFS. DIFS = SIFS + 2 × ranura = 16 + 2 × 9 = 34 µs (Jain, diapositiva 5-17; Fairhurst).
3. **Cuenta atrás aleatoria.** Si no tenía una cuenta pendiente, sortea un número entero entre 0 y CW (ventana de contienda). Al principio CW = CWmin = 15. Resta uno por cada ranura de 9 µs en que el canal sigue libre (Jain, diapositivas 5-15 y 5-16).
4. **Congelación.** Si el canal se ocupa durante la cuenta, el contador se detiene sin perder su valor. Cuando el canal vuelve a estar libre durante un DIFS, continúa desde donde estaba (Jain, diapositiva 5-20).
5. **Transmisión.** Con el contador a cero, emite la trama. En su cabecera va un campo Duración, en microsegundos, con el tiempo que aún ocupará el canal hasta el final del ACK (Jain, diapositivas 5-18 y 5-24).
6. **Acuse.** El receptor comprueba la trama y responde con un ACK tras un intervalo SIFS de 16 µs. SIFS es más corto que DIFS. Así ningún otro aparato puede colarse entre la trama y su ACK.
7. **Éxito o reintento.** Si llega el ACK, CW vuelve a CWmin. Si no llega, CW pasa a min(2·CW + 1, CWmax), es decir 15 → 31 → 63 … → 1023, y se repite desde el paso 2 (Jain, diapositiva 5-16). Tras un número máximo de intentos, la trama se descarta y el problema pasa a capas superiores, como TCP.

### 2. Qué hace cada componente

**La detección física de portadora (CCA).** La radio decide si el canal está ocupado. Distingue dos casos. Si detecta el preámbulo de una trama 802.11, lo considera ocupado incluso con señales muy débiles. Si solo hay energía sin formato 802.11 (un microondas, otra tecnología), hace falta un nivel unos 20 dB más alto para darlo por ocupado (Coleman, Extreme Networks). El estándar fija un umbral de −82 dBm para el preámbulo en canales de 20 MHz; esa cifra es **no verificado** en las fuentes leídas para este artículo, que solo confirman la diferencia de unos 20 dB entre ambos umbrales.

**La detección virtual (NAV).** Todo aparato que decodifica una trama lee su campo Duración y carga ese valor en su NAV. Mientras el NAV no llegue a cero, trata el canal como ocupado aunque no oiga nada (Jain, diapositivas 5-15 y 5-18). Sirve para proteger el ACK y los intercambios de varias tramas.

**Los intervalos entre tramas.** Son esperas de longitud distinta que ordenan prioridades sin necesidad de mensajes:

- SIFS (16 µs): entre una trama y su respuesta inmediata, como el ACK.
- PIFS (SIFS + 1 ranura = 25 µs): para funciones de coordinación con prioridad (Jain, diapositiva 5-17).
- DIFS (SIFS + 2 ranuras = 34 µs): para el acceso normal sin prioridades.
- AIFS (SIFS + AIFSN × ranura): la versión con prioridades de EDCA, descrita abajo (IEEE 802.11-2024).

Quien tiene que esperar menos gana siempre al que espera más. Esa es toda la jerarquía.

**La ranura.** Es la unidad de la cuenta atrás. Su duración cubre el tiempo que tarda una radio en detectar que otra ha empezado a emitir, más el retardo de propagación. Vale 9 µs en OFDM a 5 GHz y 20 µs en el 802.11b original (Jain, diapositiva 5-17; Fairhurst). Si dos aparatos terminan su cuenta en la misma ranura, ninguno ve al otro a tiempo y chocan.

**La ventana de contienda (CW).** Es el rango del sorteo. Empieza pequeña para no perder tiempo cuando hay poca competencia. Crece tras cada fallo para repartir a los aparatos en más ranuras cuando hay mucha. Este crecimiento se llama retroceso exponencial binario.

**El ACK.** Es la única forma de saber que la trama llegó. Una radio semidúplex no puede escuchar mientras transmite. Además, la colisión ocurre en el receptor, que puede estar lejos del emisor. Por eso 802.11 no detecta colisiones como hacía el Ethernet con cable: confirma cada trama y deduce el fallo por ausencia (Jain, diapositivas 5-10 y 5-12).

**RTS/CTS.** Es un intercambio opcional previo a una trama larga. El emisor envía una trama corta RTS con la duración de todo el intercambio. El receptor responde con una CTS que repite la reserva. Quien oiga cualquiera de las dos carga su NAV y se calla (Jain, diapositiva 5-12). Si chocan dos RTS, se pierde muy poco tiempo, porque son tramas cortas.

**EDCA: cuatro colas con reglas distintas.** Las extensiones de calidad de servicio dividen el tráfico en cuatro categorías de acceso (AC). Cada una compite con su propio AIFSN, su propia ventana y un límite de ocupación (TXOP). Valores por defecto de un chip comercial, el TI CC33xx, que los usa tanto como punto de acceso como cliente:

| Categoría | AIFSN | AIFS a 5 GHz (calculado) | CWmin | CWmax | TXOP máximo |
|---|---|---|---|---|---|
| Voz (AC_VO) | 2 | 34 µs | 3 | 7 | 1,504 ms |
| Vídeo (AC_VI) | 2 | 34 µs | 7 | 15 | 3,008 ms |
| Normal (AC_BE) | 3 | 43 µs | 15 | 1023 | una trama |
| Fondo (AC_BK) | 7 | 79 µs | 15 | 1023 | una trama |

Fuente: TI SWRU616B, tabla 2-3. Los AIFSN coinciden con los valores por defecto del estándar para aparatos convencionales (IEEE 802.11-2024). Un TXOP de 0 en la tabla significa que la categoría transmite una sola trama por cada acceso ganado. El punto de acceso anuncia en sus balizas los valores que rigen en su red.

### 3. Parámetros que importan

**Tiempos básicos.** Capa física OFDM en 5 GHz frente a 802.11b en 2,4 GHz:

| Parámetro | OFDM 5 GHz (802.11a/n/ac) | 802.11b |
|---|---|---|
| Ranura | 9 µs | 20 µs |
| SIFS | 16 µs | 10 µs |
| DIFS (calculado) | 34 µs | 50 µs |
| CWmin | 15 | 31 |
| CWmax | 1023 | 1023 |

Fuentes: Jain, diapositiva 5-17; Fairhurst; Heusse *et al.* para 802.11b.

**Cuánto cuesta esperar.** Con CW = 15, el sorteo da de media 7,5 ranuras: 7,5 × 9 µs = 67,5 µs. Sumando DIFS y SIFS, cada trama paga unos 117,5 µs de espera antes de contar el preámbulo y el ACK (calculado). A 600 Mbit/s, una trama de 1500 bytes (12 000 bits) ocupa solo 20 µs de datos. La espera supera con mucho a los datos. Por eso 802.11n y posteriores agregan varias tramas en una sola transmisión: reparten el mismo coste fijo entre más bytes.

**Probabilidad de chocar en la primera ronda.** Supuesto ilustrativo: n aparatos empiezan a la vez una cuenta nueva con CW = 15. Hay colisión si el número más bajo lo sacan dos o más. Calculado:

| Aparatos compitiendo | Probabilidad de colisión |
|---|---|
| 2 | 6,3 % |
| 3 | 9,2 % |
| 5 | 15,0 % |
| 10 | 28,3 % |
| 20 | 50,4 % |

En la práctica no todos compiten a la vez, y tras cada choque la ventana crece. Bianchi modeló la red completa en saturación, con todos los aparatos siempre con datos. Concluyó que con acceso básico el rendimiento cae a medida que aumenta el número de aparatos. Con RTS/CTS se mantiene prácticamente constante, porque lo que choca son tramas cortas (Bianchi, IEEE JSAC 2000).

**El aparato lento.** CSMA/CA da a cada aparato la misma probabilidad de ganar el canal a largo plazo, no el mismo tiempo. Heusse *et al.* lo midieron en 802.11b. Con dos aparatos a 11 Mbit/s, cada uno obtuvo entre 3,1 y 3,4 Mbit/s de UDP. Con uno a 11 Mbit/s y otro a 1 Mbit/s, los dos bajaron a unos 0,76 Mbit/s. El rápido quedó por debajo de 1 Mbit/s, porque en cada turno el lento ocupa el canal unas once veces más (Heusse *et al.*, INFOCOM 2003, tabla II).

### 4. Diagrama o flujo

```text
ACCESO BÁSICO (OFDM 5 GHz: ranura 9 µs, SIFS 16 µs, DIFS 34 µs)

A ─[ trama de A ]─16µs─────────[ACK de B]
                                         ├─ DIFS 34µs ─┤ cuenta ┤[ trama de C ]...
C (quiere enviar, sorteó 5)
   canal ocupado → contador congelado en 5        5→4→3→2→1→0 ┘
D (quiere enviar, sorteó 9)
   canal ocupado → congelado en 9                 9→8→7→6→5→4 ║ C emite: D congela en 4
                                                              ║ y retoma en el siguiente hueco

TRAS UN FALLO (sin ACK)
  CW: 15 → 31 → 63 → 127 → 255 → 511 → 1023 → 1023 …   (tras éxito: vuelve a 15)

NODO OCULTO Y RTS/CTS
  A ))) ((( router ))) ((( C        A y C oyen al router, pero no se oyen entre sí
  A ──RTS (Duración = todo el intercambio)──→ router
  A ←─CTS (Duración restante)──── router ────→ C   C oye la CTS, carga su NAV y calla
  A ──DATOS──→ router
  A ←─ACK──── router
```

## Límites, excepciones y casos reales

- **El nodo oculto.** Si A y C no se oyen, ambos ven el canal libre y pueden emitir a la vez hacia el router (Jain, diapositiva 5-10). Sus tramas chocan en el router y ninguno lo sabe hasta que falta el ACK. RTS/CTS lo mitiga, pero añade dos tramas y dos SIFS a cada envío. Muchos equipos solo lo usan para tramas largas, por encima de un umbral configurable.
- **Redes vecinas en el mismo canal.** CSMA/CA no distingue entre tu red y la del vecino. Todos los aparatos que se oyen en ese canal se turnan como si fueran una sola red. Un canal compartido con varios pisos se reparte entre todos.
- **El ACK perdido no siempre es una colisión.** Una trama puede fallar por ruido o por señal débil. El emisor dobla igualmente la ventana, aunque no hubiera nadie más. Esperar más no arregla un enlace con mala relación señal-ruido; para eso está bajar la tasa de modulación.
- **Interferencias que no son WiFi.** Una señal ajena sin preámbulo 802.11 solo bloquea el canal si supera el umbral de energía, unos 20 dB más alto que el de detección de preámbulo (Coleman). Por debajo, el aparato transmite encima y la trama puede perderse.
- **802.11ax añade un reparto central, pero no sustituye a la contienda.** Desde 802.11ax, el punto de acceso puede repartir subportadoras entre varios clientes en una misma transmisión, parecido a lo que hace una antena de telefonía móvil (ver [cómo reparte una antena su capacidad](../02_RedesMoviles/01-ComoReparteUnaAntenaSuCapacidad.md)). Para empezar esa transmisión, el punto de acceso tiene que ganar el canal con las mismas reglas de este artículo (IEEE 802.11-2024).
- **Los valores de EDCA los decide el punto de acceso.** La tabla anterior son valores por defecto. El punto de acceso puede anunciar otros en sus balizas, y los aparatos de su red deben usarlos (TI SWRU616B).

## Consecuencias prácticas

- **La velocidad se reparte, y con pérdidas.** Si tres aparatos descargan a la vez, cada uno obtiene algo menos de un tercio. Parte del tiempo se va en esperas, ACK y colisiones.
- **Un aparato viejo o lejano frena a todos.** Como el turno es por trama y no por tiempo, un cliente con poca tasa ocupa el canal mucho más rato en cada turno (Heusse *et al.*). Algunos puntos de acceso compensan repartiendo por tiempo de emisión, pero eso es una decisión del fabricante, no de la norma.
- **La voz y las videollamadas pasan antes.** Con EDCA, una trama de voz espera 34 µs y sortea entre 0 y 3. Una de fondo espera 79 µs y sortea entre 0 y 15. Casi siempre gana la voz.
- **Más aparatos en el mismo canal es más contienda, no solo más reparto.** La probabilidad de choque crece con el número de aparatos que compiten. Por eso un canal saturado de redes vecinas rinde mal aunque la señal sea buena.
- **Para quien diseña:** la ranura y los intervalos son el coste fijo de no tener coordinador. A tasas altas ese coste domina, y la eficiencia depende de agregar tramas y de reducir el número de accesos al canal.

## Preguntas frecuentes

### ¿Es lo mismo que el CSMA/CD del Ethernet con cable?

No. En el Ethernet clásico con cable compartido, el emisor escuchaba el cable mientras transmitía y cortaba la trama al detectar una colisión (CD, *collision detection*). En radio eso no es posible, por la diferencia de potencia entre la propia señal y la ajena y porque el choque ocurre en el receptor. WiFi intenta evitar las colisiones antes de emitir (CA, *collision avoidance*) y las deduce después por la falta de ACK.

### ¿El router manda sobre quién habla?

En el acceso básico, no. El router compite por el canal con las mismas reglas que cualquier cliente, aunque es quien más tráfico envía. Lo que sí decide son los parámetros de EDCA de su red y, desde 802.11ax, puede repartir subportadoras en las transmisiones que inicia.

### ¿Por qué el número aleatorio y no un orden fijo?

Un orden fijo exigiría que todos supieran cuántos aparatos hay y quién va después. Eso requiere un coordinador y mensajes extra. El sorteo funciona sin conocer a los demás, y aparatos que llegan o se van no rompen nada.

### ¿Tener más barras de señal evita las esperas?

No. Las barras indican la potencia que recibe tu aparato. Las esperas dependen de cuántos aparatos compiten en el canal y de cuánto tiempo ocupa cada uno.

## Fuentes

- IEEE. *IEEE Std 802.11-2024: Wireless LAN Medium Access Control (MAC) and Physical Layer (PHY) Specifications*. Aprobado el 2024-09-26, publicado el 2025-04-28. <https://standards.ieee.org/standard/802_11-2024.html>
- R. Jain. *IEEE 802.11 Wireless LANs, Part I: Basics*. Washington University in St. Louis, curso CSE574, 2022. Diapositivas 5-10 a 5-24. <https://research.engineering.wustl.edu/~jain/cse574-22/ftp/j_05lan.pdf>
- G. Fairhurst. *Carrier Sense Multiple Access with Collision Avoidance (CSMA/CA)*. University of Aberdeen, Electronics Research Group. <https://erg.abdn.ac.uk/users/Gorry/course/lan-pages/csma-ca.html>
- Texas Instruments. *CC33xx WLAN Features Guide* (SWRU616B), apartado 2.7, tabla 2-3. <https://www.ti.com/document-viewer/lit/html/SWRU616B/GUID-8D9EA799-1FFC-40DC-A124-0FC66CCD82D4>
- D. Coleman. *What is a Clear Channel Assessment (CCA)?* Extreme Networks, 2017. <https://www.extremenetworks.com/resources/blogs/what-is-a-clear-channel-assessment-cca>
- G. Bianchi. «Performance Analysis of the IEEE 802.11 Distributed Coordination Function». *IEEE Journal on Selected Areas in Communications*, vol. 18, n.º 3, pp. 535-547, marzo de 2000. <https://s2.smu.edu/~camp/courses/ee8378/materials/bianchi2000.pdf>
- M. Heusse, F. Rousseau, G. Berger-Sabbatel, A. Duda. «Performance anomaly of 802.11b». *IEEE INFOCOM 2003*. <https://ieeexplore.ieee.org/document/1208921> · copia abierta: <https://hal.archives-ouvertes.fr/hal-01199120>
