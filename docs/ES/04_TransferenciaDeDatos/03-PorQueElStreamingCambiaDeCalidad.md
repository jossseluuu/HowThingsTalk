---
title: "¿Por qué la calidad de un vídeo en streaming cambia sola según tu conexión?"
section: "04_TransferenciaDeDatos"
language: "es"
status: "published"
order: 3
depth: "intermediate"
summary: "El vídeo está codificado varias veces, a distintas tasas de bits, y troceado en segmentos de pocos segundos. Antes de pedir cada segmento, el reproductor del móvil elige qué versión descargar según lo que ha medido de la red y lo lleno que tiene su búfer."
prerequisites:
  - "Cómo entrega TCP los datos en orden (segmentos, ACK y retransmisión)"
  - "Cómo sabe TCP a qué velocidad puede enviar (ventana de congestión y RTT)"
  - "Tasa de bits y rendimiento efectivo"
translation: "../../EN/04_DataTransfer/03-WhyStreamingVideoChangesQuality.md"
---

# ¿Por qué la calidad de un vídeo en streaming cambia sola según tu conexión?

## Pregunta y encuadre

Empiezas a ver un vídeo en el móvil y los primeros segundos se ven borrosos. Poco después la imagen se vuelve nítida. Al entrar en el ascensor vuelve a emborronarse, pero el vídeo no se detiene. Nadie ha tocado el ajuste de calidad.

La pregunta es quién decide ese cambio y con qué información. El artículo explica el streaming adaptativo sobre HTTP (ABR, *adaptive bitrate*): cómo se prepara el vídeo en el servidor, qué describe el manifiesto y cómo elige el reproductor la calidad de cada trozo. Usa como referencia HLS (RFC 8216) y menciona MPEG-DASH, que sigue el mismo principio. No cubre cómo comprime el códec cada imagen, ni el cifrado y la gestión de derechos, ni las variantes de baja latencia para directos. Tampoco describe el algoritmo concreto de ningún servicio comercial: casi ninguno lo publica. La profundidad es intermedia. Conviene haber leído [¿Cómo entrega TCP los datos en orden si los paquetes llegan desordenados?](01-ComoEntregaTCPLosDatosEnOrden.md) y [¿Cómo sabe TCP a qué velocidad puede enviar sin saturar la red?](02-ComoSabeTCPAQueVelocidadEnviar.md).

## Respuesta corta

El vídeo existe en el servidor en varias calidades, cortado en segmentos de pocos segundos, y el reproductor del móvil decide antes de cada segmento cuál pedir según la velocidad que ha medido y los segundos de vídeo que le quedan guardados.

### Resumen de 60 segundos

Quien publica el vídeo lo codifica varias veces. Cada versión tiene una resolución y una tasa de bits distintas: por ejemplo, de 416×234 a 145 kbit/s hasta 1920×1080 a 7800 kbit/s. Cada versión se corta en segmentos de unos 6 s. Los cortes caen en los mismos instantes en todas las versiones, así que se puede saltar de una a otra en cualquier frontera.

Un archivo índice, el manifiesto, lista las versiones y la tasa de bits de cada una. El reproductor lo descarga primero. Después pide los segmentos uno a uno con peticiones HTTP normales y los guarda en un búfer antes de mostrarlos.

Antes de cada petición, el reproductor elige la versión. Hay dos pistas. Una es la velocidad a la que bajaron los últimos segmentos. La otra es cuántos segundos de vídeo tiene en el búfer. Si el búfer se vacía, la imagen se congela: es el temido «cargando». Por eso el reproductor prefiere bajar de calidad a tiempo antes que quedarse sin vídeo.

El estándar no fija cómo se toma esa decisión. Cada reproductor usa su propio algoritmo. Por eso dos aplicaciones en la misma red pueden comportarse de forma distinta.

## Modelo mental

Piensa en un depósito de agua que alimenta un grifo siempre abierto. El grifo es la reproducción: consume exactamente un segundo de vídeo por cada segundo de reloj. El depósito es el búfer. Tú lo rellenas con cubos que traes de un pozo cuyo caudal cambia sin avisar. Puedes elegir cubos pequeños de agua turbia o cubos grandes de agua limpia. Si el depósito está casi lleno, te arriesgas con los cubos grandes. Si está bajando, traes cubos pequeños, que llegan antes, para que el grifo no se seque.

La analogía recoge la idea central: la decisión depende del nivel del depósito y de lo rápido que sale el pozo, y el peor resultado es quedarse sin agua. Se rompe en dos puntos. Primero, aquí el «cubo» no cambia de tamaño a voluntad: cada segmento dura siempre lo mismo en segundos de vídeo, y lo que cambia es cuántos bytes pesa. Segundo, el pozo no tiene un caudal propio que se pueda medir sin más. La velocidad que ve el reproductor depende de cómo TCP gestiona su ventana, y esa ventana se ve afectada por las propias pausas del reproductor.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

1. **Preparación.** El vídeo se codifica en varias versiones, llamadas variantes (HLS) o representaciones (DASH). El conjunto se conoce como escalera de tasas de bits. Cada variante se corta en segmentos de duración casi fija.
2. **Manifiesto.** Se publica una lista maestra con las variantes. En HLS, cada una lleva su atributo `BANDWIDTH` y una lista de medios que enumera sus segmentos. En DASH el papel lo cumple un único archivo MPD.
3. **Arranque.** El reproductor descarga el manifiesto. Elige una variante inicial y pide los primeros segmentos. Al principio el búfer está vacío y no aporta información, así que la primera elección suele ser prudente o basarse en la velocidad de las primeras descargas.
4. **Bucle de descarga.** Cada segmento se pide con una petición HTTP `GET` normal, que viaja sobre TCP o QUIC. Al terminar, el reproductor anota cuántos bytes ha recibido y cuánto ha tardado. El segmento entra en el búfer.
5. **Decisión.** Antes del siguiente `GET`, el algoritmo de adaptación (ABR) elige la variante a partir de la velocidad estimada, el nivel del búfer o ambas cosas.
6. **Pausa.** Si el búfer alcanza su tamaño máximo, el reproductor deja de descargar y espera a que la reproducción consuma vídeo. La descarga avanza a ráfagas: periodos de actividad separados por silencios.
7. **Cambio de variante.** Como las fronteras de los segmentos coinciden en todas las variantes, el siguiente segmento puede venir de otra calidad. El decodificador empieza con él una imagen completa y la transición es limpia.

### 2. Qué hace cada componente

**Escalera de tasas de bits.** Es el menú de calidades. La especificación de Apple para HLS propone como ejemplo nueve escalones en H.264. Van de 416×234 a 145 kbit/s hasta 1920×1080 a 7800 kbit/s, pasando por 1280×720 a 3000 y a 4500 kbit/s. La escalera no es universal: cada servicio diseña la suya.

**Segmento.** La especificación de Apple pide segmentos de 6 s nominales (regla 7.5) que empiecen por una imagen IDR (regla 7.4). Una IDR es una imagen que se decodifica sin depender de las anteriores. También recomienda que las fronteras de segmento caigan en los mismos instantes en todas las variantes (regla 8.22). Sin esa alineación, cambiar de calidad obligaría a descartar vídeo o a esperar.

**Manifiesto (HLS).** La etiqueta `EXT-X-TARGETDURATION` fija la duración máxima de segmento: cada duración, redondeada al entero más próximo, debe ser menor o igual que ese valor (RFC 8216, sección 4.3.3.1). `EXT-X-STREAM-INF` describe cada variante. Su atributo `BANDWIDTH` es la tasa de bits pico de los segmentos, en bits por segundo. `AVERAGE-BANDWIDTH`, opcional, es la media (sección 4.3.4.2). Apple exige que en vídeo bajo demanda los valores medidos queden dentro de un 10 % de los declarados (reglas 1.26 y 1.27).

**Reproductor y búfer.** El reproductor es software del móvil: la aplicación o el navegador. El búfer guarda segmentos descargados y aún no mostrados, medidos en segundos de vídeo. Su nivel evoluciona con una regla sencilla. Mientras descarga a una velocidad `C` una variante de tasa `R`, el búfer gana `C/R` segundos de vídeo por segundo de reloj y pierde 1 por la reproducción. Si `C` es menor que `R`, el búfer baja.

**Algoritmo de adaptación (ABR).** RFC 8216 dice expresamente que los algoritmos para cambiar de variante quedan fuera de su alcance (sección 6.3.1). Hay tres familias:

- **Basados en velocidad.** Estiman la capacidad a partir de las descargas recientes y eligen la variante más alta que quepa con margen. Un ejemplo de estimador es la media armónica de las últimas cinco descargas, que usa el algoritmo RobustMPC según la comparación de Spiteri y colaboradores.
- **Basados en búfer.** Eligen la variante solo a partir del nivel del búfer. En el experimento de Netflix de 2013 (algoritmo BBA), por debajo de 90 s de búfer se pedía siempre la calidad mínima. Entre 90 y 216 s, la calidad subía de forma escalonada con el nivel. Por encima, se pedía la máxima. BOLA formaliza esta idea como un problema de optimización: maximiza la calidad media más un término que penaliza las congelaciones, con una utilidad logarítmica de la tasa de bits.
- **Híbridos.** Usan la velocidad medida durante el arranque, cuando el búfer aún no dice nada, y el nivel del búfer después. Es el caso del algoritmo DYNAMIC del reproductor de referencia de DASH, dash.js.

### 3. Parámetros que importan

- **Duración de segmento:** 6 s nominales en la especificación de Apple, con imágenes IDR cada 2 s (regla 1.13). El experimento de Netflix usaba segmentos de 4 s; el de BOLA, de 3 s.
- **Peso de un segmento (ejemplo de cálculo):** 6 s a 7800 kbit/s son 46 800 kbit, unos 5,85 MB. El mismo segmento a 145 kbit/s pesa unos 109 kB. La relación es de más de 50 a 1.
- **Evolución del búfer (ejemplo de cálculo):** con 20 Mbit/s disponibles, un segmento de 6 s a 7800 kbit/s baja en 2,34 s, y el búfer gana 3,66 s netos por segmento. Si la capacidad cae a 4 Mbit/s, ese mismo segmento tarda 11,7 s y el búfer pierde 5,7 s por segmento. Con 30 s guardados, la imagen se congelaría en aproximadamente un minuto. Bajando a la variante de 3000 kbit/s, cada segmento tarda 4,5 s y el búfer vuelve a crecer 1,5 s por segmento.
- **Tamaño del búfer:** el reproductor de Netflix del experimento de 2013 admitía 240 s. El estudio de BOLA indica que dash.js usaba entonces 12 s por defecto. Un búfer grande da margen; uno pequeño reacciona antes y gasta menos datos si el usuario abandona.
- **Directo:** en una lista sin final, el cliente no debe empezar por un segmento situado a menos de tres duraciones objetivo del final de la lista (RFC 8216, sección 6.3.3). Con segmentos de 6 s, eso deja al espectador al menos 18 s por detrás del último segmento publicado. El cliente vuelve a pedir la lista como pronto tras una duración objetivo si ha cambiado, o tras la mitad si no ha cambiado (sección 6.3.4).
- **Consumo de datos (ejemplo de cálculo):** una hora a 7800 kbit/s son unos 3,5 GB. Una hora a 145 kbit/s, unos 65 MB.

### 4. Diagrama o flujo

```text
 SERVIDOR (HTTP)                                   MÓVIL (reproductor)
 ---------------                                   -------------------
 lista maestra ------------- GET ---------------->  lee variantes y BANDWIDTH
   variante 1080p 7800 kbit/s: seg1 seg2 seg3 ...
   variante  720p 3000 kbit/s: seg1 seg2 seg3 ...   (fronteras alineadas)
   variante  234p  145 kbit/s: seg1 seg2 seg3 ...
                                                    +---------------------+
                     +----------------------------> | ALGORITMO ABR       |
                     |                              | entradas:           |
 segmento elegido <--+--- GET segN (variante X) --- |  velocidad medida   |
        |                                           |  nivel del búfer    |
        +---- bytes por TCP/QUIC ------------------> +---------------------+
                                                              |
                                                              v
                                    BÚFER [seg seg seg seg ....] --> pantalla
                                    gana C/R s por segundo           consume
                                    si se llena: pausa la descarga   1 s/s

 nivel del búfer
  ^   máximo ......____......____......     (descarga a ráfagas)
  |            ___/    \____/    \___
  |       ___/                         \    capacidad cae: el búfer baja
  |   ___/                              \__ -> ABR elige variante menor
  |  /  <- arranque                        \_/ -> el búfer se recupera
  +-----------------------------------------------> tiempo
```

## Límites, excepciones y casos reales

**La velocidad medida es una estimación ruidosa.** En el estudio de Netflix, la velocidad media por segmento de una sola sesión osciló entre 17 Mbit/s y 500 kbit/s. En una muestra de 300 000 sesiones, alrededor del 10 % tuvo una mediana de velocidad inferior a la mitad de su percentil 95. Entre las causas citan interferencias WiFi, congestión en la red, servidores sobrecargados y procesos del propio aparato. Los autores estimaron que entre el 20 % y el 30 % de las congelaciones observadas podían deberse a una mala elección de calidad, no a falta real de capacidad.

**Las pausas del reproductor confunden a TCP.** Cuando el búfer está lleno, el reproductor deja de pedir segmentos. Si la conexión queda inactiva más que el temporizador de retransmisión, RFC 5681 indica que el emisor debe reducir su ventana como mucho a la inicial antes de volver a enviar. El siguiente segmento empieza entonces con una ventana pequeña y baja más despacio de lo que permitiría la línea. Un algoritmo que solo mira la velocidad puede leerlo como una caída de capacidad.

**El arranque es el momento más difícil.** Con el búfer vacío no hay información sobre la red. Netflix comprobó que su primera versión basada solo en el búfer pasaba unos 90 s de vídeo en la calidad mínima al empezar. Eso le costaba unos 100 kbit/s de calidad media en horas punta frente al algoritmo anterior. Las versiones posteriores usaron la velocidad medida para subir antes. Por eso muchos vídeos empiezan borrosos y mejoran en pocos segundos.

**Los segmentos no pesan lo mismo.** La codificación es de tasa variable: una escena con mucho movimiento ocupa más que un plano fijo. `BANDWIDTH` declara el pico y `AVERAGE-BANDWIDTH` la media. Apple recomienda que en vídeo bajo demanda el pico no supere el 200 % de la media (regla 1.30). Un algoritmo que solo usa la media nominal puede quedarse corto en una escena exigente.

**Los resultados dependen del banco de pruebas.** El estudio de Netflix es un experimento real con más de medio millón de usuarios por prueba, en 2013. Los autores advierten de que el algoritmo de producción ha cambiado desde entonces. Las cifras de BOLA proceden de simulaciones con trazas de red: entre el 84 % y el 95 % de la utilidad máxima alcanzable conociendo la red de antemano. No son medidas de un servicio en producción.

**Abandonar a medias.** Si la capacidad se hunde en mitad de una descarga grande, esperar a que termine puede vaciar el búfer. Algunos reproductores cancelan el segmento en curso y lo piden de nuevo en una calidad menor. La versión práctica de BOLA incluye esta posibilidad.

## Consecuencias prácticas

La velocidad de un test no es la que ve el reproductor. El test mide una descarga larga y continua. El reproductor mide descargas cortas, separadas por pausas, con una ventana TCP que a veces vuelve a empezar desde abajo.

Bajar de calidad es la defensa contra la congelación, no un fallo. Una imagen más borrosa durante unos segundos suele molestar menos que una pausa. Por eso los algoritmos bajan pronto y suben con cautela.

Elegir a mano una calidad fija desactiva la adaptación. Si la red no la sostiene, el búfer se vacía y la imagen se congela. La opción automática existe precisamente para evitarlo.

La calidad máxima tiene un coste en datos muy superior al de la mínima: unos 3,5 GB por hora frente a unos 65 MB con la escalera de ejemplo de Apple. En una tarifa móvil limitada, la diferencia importa.

Un directo por internet va por detrás de la emisión. Las reglas de HLS obligan a mantener varios segmentos de margen, y con segmentos de 6 s eso supone decenas de segundos. Existen extensiones de baja latencia que este artículo no cubre.

Para quien diseña un servicio, la escalera, la duración de segmento, el tamaño del búfer y el algoritmo son decisiones que se compensan entre sí. Segmentos cortos permiten reaccionar antes, pero cada uno empieza con una imagen IDR que ocupa más. Un búfer grande evita congelaciones, pero en directo aumenta el retraso.

## Preguntas frecuentes

### ¿Decide el servidor la calidad que me envía?

En HLS y DASH, no. El servidor solo publica archivos y responde peticiones HTTP. Es el reproductor del móvil quien elige qué segmento pedir. Por eso el mismo servidor puede atender a la vez a un móvil en 234p y a un televisor en 1080p.

### ¿Es lo mismo que el control de congestión de TCP?

No, aunque los dos se adaptan a la red. TCP decide a qué ritmo enviar los bytes de una descarga concreta. El algoritmo ABR decide qué archivo descargar. Trabajan en capas distintas y se influyen: la ventana de TCP condiciona la velocidad que mide el reproductor.

### ¿Es lo mismo que la compresión del códec?

No. El códec decide cómo representar cada imagen con pocos bits y se aplica al preparar cada variante. El streaming adaptativo elige entre variantes ya comprimidas. Un códec más eficiente permite la misma calidad con menos tasa de bits, pero no adapta nada por sí solo.

### ¿Por qué baja deprisa y sube despacio?

Porque los dos errores no cuestan lo mismo. Subir demasiado pronto puede vaciar el búfer y congelar la imagen. Subir tarde solo significa unos segundos de peor calidad. Los algoritmos basados en búfer reflejan esa asimetría: solo suben cuando hay margen acumulado.

## Fuentes

- [RFC 8216 — HTTP Live Streaming, R. Pantos y W. May, IETF, informativo (2017)](https://www.rfc-editor.org/rfc/rfc8216.html). Listas maestra y de medios, variantes, `EXT-X-TARGETDURATION`, `BANDWIDTH` y `AVERAGE-BANDWIDTH`, algoritmos de cambio fuera de alcance (6.3.1), regla de las tres duraciones objetivo (6.3.3) y recarga de listas (6.3.4).
- [HLS Authoring Specification for Apple Devices, Apple](https://developer.apple.com/streaming/hls-authoring-specification-for-apple-devices.html). Escalera de ejemplo en H.264 (regla 1.25), IDR cada 2 s (1.13), precisión de `BANDWIDTH` (1.26 a 1.30), segmentos de 6 s que empiezan por IDR (7.4 a 7.7) y alineación entre variantes (8.15 y 8.22).
- [ISO/IEC 23009-1 — Dynamic adaptive streaming over HTTP (DASH), Part 1: Media presentation description and segment formats, ISO/IEC](https://www.iso.org/standard/23009-1). Norma de MPEG-DASH, con el manifiesto MPD y sus representaciones.
- [T.-Y. Huang, R. Johari, N. McKeown, M. Trunnell y M. Watson. «A Buffer-Based Approach to Rate Adaptation: Evidence from a Large Video Streaming Service». *ACM SIGCOMM* (2014)](https://web.stanford.edu/class/cs244/papers/sigcomm2014-video.pdf). Variabilidad de la velocidad medida, algoritmo BBA, reserva de 90 s, búfer de 240 s, segmentos de 4 s y reducción de congelaciones en Netflix.
- [K. Spiteri, R. Urgaonkar y R. K. Sitaraman. «BOLA: Near-Optimal Bitrate Adaptation for Online Videos». *IEEE/ACM Transactions on Networking* (2020), versión arXiv](https://www.arxiv.org/pdf/1601.06748). Formulación con optimización de Lyapunov, decisión basada solo en el búfer, implementación en dash.js, algoritmo DYNAMIC y comparación con otros algoritmos.
- [RFC 5681 — TCP Congestion Control, IETF (2009)](https://www.rfc-editor.org/rfc/rfc5681.html). Reinicio de la ventana de congestión tras un periodo de inactividad (sección 4.1).
