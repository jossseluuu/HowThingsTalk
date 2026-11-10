---
title: "¿Cómo sabe TCP a qué velocidad puede enviar sin saturar la red?"
section: "04_TransferenciaDeDatos"
language: "es"
status: "published"
order: 2
depth: "intermediate"
summary: "Ningún aparato conoce de antemano la capacidad libre del camino hasta el otro extremo. TCP la descubre tanteando: aumenta la cantidad de datos en vuelo mientras llegan confirmaciones y la reduce cuando detecta pérdidas o marcas de congestión."
prerequisites:
  - "Cómo entrega TCP los datos en orden (números de secuencia, ACK y retransmisión)"
  - "Latencia, RTT y rendimiento efectivo"
translation: "../../EN/04_DataTransfer/02-HowTCPDecidesHowFastToSend.md"
---

# ¿Cómo sabe TCP a qué velocidad puede enviar sin saturar la red?

## Pregunta y encuadre

Cuando el móvil descarga un archivo, entre el servidor y la pantalla hay decenas de enlaces: fibra, routers, el WiFi de casa. Cada uno tiene su capacidad y la comparte con otros usuarios que entran y salen sin avisar. El servidor no ve nada de eso. Solo ve los datos que envía y las confirmaciones que vuelven.

La pregunta es cómo decide TCP cuántos datos puede tener «en el aire» en cada momento sin desbordar la red ni desperdiciarla. El artículo explica el control de congestión del emisor: arranque lento, evitación de congestión, reacción a pérdidas, CUBIC, ECN y la idea que hay detrás de BBR. No cubre cómo TCP ordena y retransmite los bytes; eso está en [¿Cómo entrega TCP los datos en orden si los paquetes llegan desordenados?](01-ComoEntregaTCPLosDatosEnOrden.md), que conviene leer antes. Tampoco cubre la gestión de colas dentro de los routers ni otros transportes como QUIC. La profundidad es intermedia: hay alguna fórmula, siempre con un ejemplo numérico.

## Respuesta corta

TCP no conoce la velocidad del camino: la descubre a base de ensayo y error, ampliando una ventana de datos en vuelo mientras las confirmaciones llegan bien y recortándola en cuanto aparece una señal de congestión.

### Resumen de 60 segundos

El emisor mantiene una variable llamada ventana de congestión (`cwnd`). Indica cuántos bytes puede tener enviados y aún sin confirmar. Como cada ventana tarda un tiempo de ida y vuelta (RTT) en confirmarse, la velocidad media sale aproximadamente de dividir `cwnd` entre el RTT.

Al empezar, la ventana es pequeña y se duplica más o menos en cada RTT. Es el «arranque lento», que de lento solo tiene el nombre. Cuando aparece una pérdida, el emisor lo interpreta como que algún enlace se ha llenado. Entonces recorta la ventana y pasa a crecer con más cautela. El resultado es un diente de sierra: subir, tocar el límite, bajar, volver a subir.

La regla clásica (Reno) añade un segmento por RTT y divide la ventana a la mitad tras una pérdida. CUBIC, el algoritmo por defecto en Linux, Windows y los sistemas de Apple, recupera la ventana con una curva cúbica y recorta solo un 30 %. BBR, todavía en fase experimental en el IETF, cambia de enfoque: en lugar de esperar a las pérdidas, mide el ancho de banda y el RTT mínimo del camino y envía a ese ritmo.

## Modelo mental

Imagina que llenas de agua una manguera larga que no ves, que acaba en un depósito con un desagüe de caudal desconocido. Solo sabes que el agua llega porque alguien al otro lado te avisa cada vez que entra un litro. Abres poco el grifo y lo vas abriendo más mientras los avisos llegan con regularidad. Si el depósito rebosa, te enteras porque se pierde agua y faltan avisos. Entonces cierras bastante el grifo y empiezas a abrirlo otra vez, ahora más despacio.

La analogía recoge la idea central: el emisor regula sin ver la red y aprende de lo que vuelve. Se rompe en tres puntos. Primero, TCP no regula un caudal continuo, sino una cantidad de datos pendientes de confirmar; el ritmo sale de cuánto tardan en volver las confirmaciones. Segundo, el «depósito» no es uno: es el enlace más lento del camino en ese momento, y puede cambiar a mitad de una descarga. Tercero, el agua que rebosa no siempre significa que el depósito esté lleno: en un enlace de radio un paquete puede perderse por un error de transmisión, y TCP clásico lo confunde con congestión.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

1. **Arranque.** Tras el saludo inicial de tres pasos, el emisor fija la ventana inicial (IW). Con la regla experimental de RFC 6928, que usan muchos sistemas, puede llegar a 10 segmentos, con un máximo de 14 600 bytes.
2. **Arranque lento (slow start).** Por cada ACK que confirma datos nuevos, `cwnd` crece como mucho un segmento máximo (SMSS). Si se confirma un segmento por ACK, una ventana entera de ACK suma una ventana entera de segmentos: la ventana se duplica en cada RTT. Esta fase dura mientras `cwnd` sea menor que el umbral `ssthresh`.
3. **Evitación de congestión.** Cuando `cwnd` supera `ssthresh`, el crecimiento pasa a ser de aproximadamente un segmento por RTT en el algoritmo clásico. CUBIC usa otra curva (ver más abajo).
4. **Señal de congestión.** El emisor detecta que algo va mal por una de tres vías: llegan tres ACK duplicados (un hueco en el flujo), vence el temporizador de retransmisión o el receptor devuelve una marca ECN.
5. **Reacción.** Ante tres ACK duplicados, la regla clásica retransmite el segmento perdido, baja `ssthresh` a la mitad de los datos en vuelo y sigue en evitación de congestión desde ahí. Ante un vencimiento del temporizador, la reacción es más dura: `cwnd` cae a un solo segmento y se repite el arranque lento hasta el nuevo `ssthresh`.
6. **Vuelta a empezar.** La ventana vuelve a crecer hasta la siguiente señal. Si la conexión queda inactiva más de un tiempo de retransmisión, la ventana se reduce como mucho a la inicial antes de volver a enviar.

### 2. Qué hace cada componente

**Ventana de congestión (`cwnd`).** Variable interna del emisor. Limita los datos enviados y no confirmados por motivos de red. Nunca viaja en ningún paquete: el receptor no la conoce.

**Ventana de recepción (`rwnd`).** La anuncia el receptor en cada segmento y protege su búfer. RFC 5681 fija que el emisor no puede tener en vuelo más que el mínimo de `cwnd` y `rwnd`. Una protege la red; la otra, al receptor.

**Umbral de arranque lento (`ssthresh`).** Marca la frontera entre crecer de forma exponencial y crecer con cautela. Al principio puede ser muy alto. Tras una congestión pasa a valer, según RFC 5681, el mayor entre la mitad de los datos en vuelo y dos segmentos.

**Temporizador de retransmisión (RTO).** RFC 6298 lo calcula a partir de una media suavizada del RTT (`SRTT`) y de su variación (`RTTVAR`): `RTO = SRTT + 4·RTTVAR`. Recomienda un mínimo de 1 s y que el valor se duplique en cada vencimiento consecutivo. Por eso un vencimiento es caro: el emisor ha estado parado al menos un segundo.

**Algoritmo de crecimiento.** Decide cómo se recupera `cwnd` tras una congestión:

- **Reno (RFC 5681):** suma un segmento por RTT y multiplica por 0,5 tras una pérdida. A esto se le llama «aumento aditivo, reducción multiplicativa».
- **CUBIC (RFC 9438, estándar desde 2023):** la ventana sigue la función `W(t) = C·(t − K)³ + W_max`. `W_max` es la ventana justo antes de la última reducción, `t` son los segundos transcurridos desde entonces y `C = 0,4` segmentos/s³. `K` es el tiempo que tarda la curva en volver a `W_max`. La curva sube rápido al principio, se aplana cerca del punto donde hubo congestión y, si no pasa nada, vuelve a acelerar para buscar capacidad nueva. Tras una pérdida multiplica la ventana por 0,7, no por 0,5.
- **BBR (borrador del IETF, versión 3):** no usa la pérdida como señal principal. Estima dos cosas: el ancho de banda máximo entregado recientemente y el RTT mínimo reciente. Su producto es el producto ancho de banda-retardo (BDP): los datos justos para llenar el camino sin crear cola. BBR espacia el envío de paquetes (pacing) a ese ritmo y limita los datos en vuelo alrededor del BDP.

**ECN (RFC 3168).** Dos bits de la cabecera IP permiten a un router marcar un paquete como «congestión experimentada» (CE) en vez de descartarlo. El receptor devuelve el aviso con el indicador ECE en sus ACK y el emisor responde con CWR. La reacción del emisor debe ser esencialmente la misma que ante una pérdida: reducir la ventana, como mucho una vez por ventana de datos. La diferencia es que no hace falta tirar ningún paquete ni retransmitirlo.

### 3. Parámetros que importan

- **Ventana inicial:** de 2 a 4 segmentos según RFC 5681; hasta 10 segmentos (máximo 14 600 bytes) con la extensión experimental de RFC 6928.
- **Relación ventana-velocidad:** velocidad media ≈ `cwnd / RTT`. Con `cwnd` = 64 KB y RTT = 40 ms salen unos 13 Mbit/s, tenga el enlace la capacidad que tenga.
- **BDP del camino:** capacidad del cuello de botella × RTT. Un enlace de 100 Mbit/s con 40 ms de RTT tiene un BDP de 500 000 bytes, unos 342 segmentos de 1460 bytes. Con menos datos en vuelo, el enlace queda infrautilizado. Con más, el exceso se acumula en colas.
- **Tiempo de arranque (ejemplo de cálculo):** desde 10 segmentos, duplicando en cada RTT, se superan los 342 segmentos tras 6 RTT, unos 240 ms con RTT de 40 ms. Es el caso ideal de un ACK por segmento. Si el receptor confirma uno de cada dos segmentos (ACK retardados), el crecimiento por RTT es menor.
- **Tiempo de recuperación tras una pérdida (ejemplo de cálculo):** en un camino de 1 Gbit/s con 100 ms de RTT, el BDP ronda los 8560 segmentos. Reno parte la ventana a la mitad y la recupera a un segmento por RTT: unos 4280 RTT, más de 7 minutos. Con CUBIC, `K = ∛(W_max·0,3/C)` ≈ 18,6 s, y ese tiempo no depende del RTT. Este es el problema que CUBIC se diseñó para resolver: caminos rápidos y largos.
- **Parámetros de CUBIC:** `C = 0,4` y factor de reducción 0,7 (RFC 9438, secciones 5.1 y 4.6).
- **RTO:** mínimo recomendado de 1 s, valor inicial de 1 s antes de medir el RTT y duplicación en cada vencimiento (RFC 6298).

### 4. Diagrama o flujo

```text
                 cwnd < ssthresh                 cwnd ≥ ssthresh
              +-------------------+          +-----------------------+
 conexión --> |  ARRANQUE LENTO   | -------> | EVITACIÓN DE          |
 (cwnd = IW)  |  +1 SMSS por ACK  |          | CONGESTIÓN            |
              |  ≈ ×2 por RTT     |          | Reno: +1 SMSS por RTT |
              +-------------------+          | CUBIC: curva cúbica   |
                  ^                          +-----------------------+
                  |                             |               |
                  |              3 ACK duplicados o marca ECN   |
                  |                             v               |
                  |                  ssthresh = vuelo/2 (Reno)  |
                  |                  ventana ×0,7 (CUBIC)       |
                  |                  sigue en evitación         |
                  |                                             |
                  |               vence el temporizador (RTO)   |
                  +---------------------------------------------+
                         ssthresh = max(vuelo/2, 2·SMSS)
                         cwnd = 1 segmento, RTO ×2

 cwnd
  ^        /|        /|        /|
  |       / |       / |       / |      diente de sierra (Reno)
  |      /  |      /  |      /  |
  |     /   |/    /   |/    /   |/
  |    /
  |  _/   <- arranque lento
  +----------------------------------------> tiempo
```

## Límites, excepciones y casos reales

**Una pérdida no siempre es congestión.** Reno y CUBIC tratan cualquier pérdida como señal de que la red está llena. En enlaces de radio las capas inferiores retransmiten muchos errores, pero no todos. El artículo de Cardwell y colaboradores (2016) midió en un enlace de 100 Mbit/s y 100 ms de RTT que el rendimiento de CUBIC caía unas 10 veces con un 0,1 % de pérdida aleatoria. BBRv1 mantenía el rendimiento cerca del máximo hasta un 5 % de pérdida. Son cifras de BBRv1 en un banco de pruebas, no de la versión 3 que describe el borrador actual.

**Colas demasiado grandes (bufferbloat).** Un algoritmo basado en pérdidas sigue aumentando la ventana hasta que algo se desborda. Si el router tiene un búfer grande, primero lo llena entero. Los paquetes esperan en cola y el RTT sube. Como ejemplo de cálculo, 1 MB en cola ante un enlace de subida de 10 Mbit/s añade 0,8 s de espera a cada paquete. En las pruebas de Google descritas en ese mismo artículo, cambiar CUBIC por BBRv1 en servidores de YouTube redujo la mediana del RTT un 53 % de media global.

**La reacción depende de quién envía.** El control de congestión vive en el emisor. En una descarga manda el algoritmo del servidor, no el del móvil. Dos descargas iguales pueden comportarse distinto según el servidor.

**Convivencia de algoritmos.** CUBIC incluye una región «amistosa con Reno» para no quitar capacidad a los flujos Reno en caminos con BDP pequeño (RFC 9438, sección 3.2). El borrador de BBR sigue en discusión en el IETF y su estado previsto es experimental. Cómo reparte la capacidad cuando comparte camino con flujos CUBIC no se trata en este artículo.

**Conexiones cortas.** Una página web pequeña puede terminar antes de salir del arranque lento. En ese caso la ventana inicial y el RTT pesan más que la capacidad del enlace. Esa fue la motivación de RFC 6928 para subir la ventana inicial a 10 segmentos.

**ECN necesita que todos colaboren.** La marca solo funciona si emisor, receptor y routers intermedios la soportan y la respetan. Si un equipo intermedio borra los bits, el emisor vuelve a depender de las pérdidas.

## Consecuencias prácticas

Una descarga no arranca a su velocidad máxima. Durante los primeros RTT la velocidad sube a saltos, y con RTT largos ese arranque se nota más. Por eso un servidor lejano tarda más en «coger velocidad» que uno cercano aunque la línea sea la misma.

La velocidad de una conexión TCP no es una propiedad fija de la línea. Depende de la ventana, y la ventana depende del RTT y de las pérdidas. Con la misma fibra, una conexión a 10 ms de RTT recupera su ventana mucho antes que otra a 150 ms.

Cuando una descarga grande satura el enlace de casa, una videollamada simultánea puede empeorar aunque «quede ancho de banda». La descarga llena las colas y todo lo que pasa por ellas espera más. Es el efecto bufferbloat descrito arriba.

Para quien diseña servicios, elegir el algoritmo del servidor es una decisión de ingeniería con consecuencias medibles. Afecta a la latencia, al rendimiento en redes con pérdidas y a cómo se reparte la capacidad con otros flujos.

## Preguntas frecuentes

### ¿Es lo mismo control de congestión que control de flujo?

No. El control de flujo protege al receptor: este anuncia en `rwnd` cuánto espacio libre tiene en su búfer. El control de congestión protege la red: el emisor estima `cwnd` sin que nadie se lo diga. El límite real es el menor de los dos.

### ¿TCP mide la velocidad de mi conexión?

Los algoritmos clásicos, no. Reno y CUBIC no estiman la capacidad: amplían la ventana hasta que algo falla. BBR sí construye una estimación explícita del ancho de banda y del RTT mínimo, y la actualiza mientras envía.

### Si el ACK confirma los datos, ¿por qué hace falta un temporizador?

Porque si se pierden todos los segmentos de una ventana, o sus ACK, no llega nada que avise. El temporizador es la red de seguridad. Como recomienda un mínimo de 1 s y se duplica en cada vencimiento, es mucho más lento que la retransmisión rápida por ACK duplicados.

### ¿El arranque lento es lento?

No. El nombre se refiere a que empieza con una ventana pequeña en vez de enviar de golpe todo lo que permite el receptor. En la práctica la ventana se duplica en cada RTT, que es un crecimiento exponencial.

## Fuentes

- [RFC 5681 — TCP Congestion Control, IETF (2009)](https://www.rfc-editor.org/rfc/rfc5681.html). Definiciones de `cwnd`, `rwnd` y `ssthresh`; arranque lento, evitación de congestión, retransmisión rápida y reacción al vencimiento del temporizador.
- [RFC 6928 — Increasing TCP's Initial Window, IETF (2013, experimental)](https://www.rfc-editor.org/rfc/rfc6928.html). Ventana inicial de hasta 10 segmentos y 14 600 bytes.
- [RFC 6298 — Computing TCP's Retransmission Timer, IETF (2011)](https://www.rfc-editor.org/rfc/rfc6298.html). Cálculo del RTO, mínimo de 1 s y duplicación en cada vencimiento.
- [RFC 9438 — CUBIC for Fast and Long-Distance Networks, IETF (2023)](https://www.rfc-editor.org/rfc/rfc9438.html). Función de ventana cúbica, `C = 0,4`, factor 0,7 y adopción por defecto en Linux, Windows y Apple.
- [RFC 3168 — The Addition of Explicit Congestion Notification (ECN) to IP, IETF (2001)](https://www.rfc-editor.org/rfc/rfc3168.html). Bits ECN de la cabecera IP, indicadores ECE y CWR y reacción del emisor.
- [draft-ietf-ccwg-bbr — BBR Congestion Control, IETF (borrador -07, noviembre de 2026)](https://datatracker.ietf.org/doc/draft-ietf-ccwg-bbr/). Especificación de BBRv3: modelo de ancho de banda, RTT mínimo, BDP y pacing.
- [N. Cardwell, Y. Cheng, C. S. Gunn, S. H. Yeganeh y V. Jacobson. «BBR: Congestion-Based Congestion Control». *ACM Queue*, vol. 14, n.º 5 (2016)](https://queue.acm.org/detail.cfm?id=3022184). Motivación de BBRv1 y medidas en la red de Google (pérdida aleatoria, RTT en YouTube).
