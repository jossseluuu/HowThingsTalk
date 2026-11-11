---
title: "¿Cómo sabe el móvil dónde está con el GPS apagado y sin conectarse a ninguna red WiFi?"
section: "05_Geolocalizacion"
language: "es"
status: "published"
order: 2
depth: "intermediate"
summary: "Los puntos de acceso WiFi y las antenas de telefonía anuncian constantemente su identificador, y cualquiera puede leerlo sin conectarse. El móvil anota cuáles oye y con qué intensidad, y los compara con una base de datos que guarda dónde está cada uno. Esa base la construyen otros móviles que sí tenían GPS cuando pasaron por allí."
prerequisites:
  - "Frecuencia, longitud de onda y decibelios (artículo 00·01)"
  - "Multitrayecto (artículo 00·03)"
  - "Cómo calcula el GPS una posición (artículo 05·01)"
translation: "../../EN/05_Positioning/02-HowYourPhoneFindsItselfWithoutGPS.md"
---

# ¿Cómo sabe el móvil dónde está con el GPS apagado y sin conectarse a ninguna red WiFi?

## Pregunta y encuadre

Dentro de un centro comercial o de una estación, el mapa del móvil sitúa el punto azul en el edificio correcto. A veces el GPS no recibe ningún satélite. Y el móvil no está conectado a la WiFi del local. La pregunta es de dónde sale esa posición.

El artículo explica el posicionamiento por WiFi y por celdas de telefonía: qué señales lee el móvil, qué base de datos consulta, cómo se construye esa base y cómo se pasa de una lista de identificadores a unas coordenadas. No cubre la ayuda que la red da al GPS para arrancar antes, ni la localización por Bluetooth o por UWB, ni los métodos de posicionamiento que calcula el propio operador de telefonía. Tampoco explica cómo consultar estas bases de datos sobre aparatos ajenos: la frontera está en la [guía de estilo](../../../guides/StyleGuide.md).

Conviene conocer los decibelios ([artículo 00·01](../00_Fundamentos/01-PorQueMasFrecuenciaLlegaMenosLejos.md)), el multitrayecto ([artículo 00·03](../00_Fundamentos/03-PorQueLaCoberturaCambiaEnPocosCentimetros.md)) y el principio del GPS ([artículo 05·01](01-ComoCalculaElGPSTuPosicion.md)). La profundidad es intermedia y no hay ecuaciones más allá de una media ponderada.

## Respuesta corta

El móvil no mide su posición: reconoce qué puntos de acceso y qué antenas tiene alrededor, y busca en una base de datos dónde se vio cada uno.

### Resumen de 60 segundos

Un punto de acceso WiFi emite varias veces por segundo una trama de anuncio, la baliza (*beacon*). Lleva el nombre de la red y un identificador único del aparato, el BSSID. No va cifrada, porque sirve precisamente para que otros la encuentren. Cualquier móvil puede leerla sin conectarse. Con la telefonía pasa algo parecido: el móvil siempre sabe qué celda le da servicio, porque necesita su identificador para poder funcionar.

Con esa lista de identificadores y la intensidad con que llega cada uno, el móvil pregunta a un servicio de localización. Ese servicio tiene una base de datos enorme con la posición aproximada de cientos de millones de puntos de acceso y antenas. La han construido otros móviles: cuando tienen GPS, envían en segundo plano qué redes oyen y dónde están.

Con las posiciones de los puntos de acceso cercanos, una media ponderada por la intensidad da la posición del móvil. Según Google, con dos o más puntos de acceso el error típico ronda los 20 m. Con solo la celda de telefonía, cientos de metros.

## Modelo mental

Imagina que te dejan con los ojos vendados en una ciudad que conoces bien. No sabes dónde estás. Pero oyes la campana de una iglesia muy fuerte, otra más lejana y el pitido de un paso de peatones concreto. Sabes dónde está cada cosa. Si la primera campana suena muy fuerte, estás cerca de ella. Sin medir nada, te sitúas a unas calles de distancia.

Ahora supón que el plano de dónde está cada campana no lo hiciste tú. Lo dibujaron miles de paseantes que sí veían, anotando qué sonidos oían en cada esquina.

La analogía se rompe en tres puntos. Las campanas no se mueven, pero un punto de acceso sí puede cambiar de casa o ser un router portátil que viaja en un tren. La intensidad de la radio no dice la distancia con fiabilidad: una pared o un cuerpo restan más decibelios que muchos metros de aire. Y el plano no lo consulta tu cabeza: en muchos casos hay que pedirlo por internet a un servidor, que así recibe la lista de lo que oyes.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

Hay dos fases separadas en el tiempo.

**Fase de recogida (construcción de la base).** Millones de móviles con posición conocida, normalmente por GNSS, registran qué puntos de acceso y qué celdas detectan, con qué intensidad y dónde estaban. Lo envían al proveedor de su sistema operativo. Apple describe que el iPhone envía periódicamente «las ubicaciones geoetiquetadas» de los puntos de acceso y antenas cercanos, de forma anónima y cifrada. Rye y Levin (2024) describen el mismo esquema para Apple y Google. Cuando suficientes aparatos ven un BSSID durante suficiente tiempo, el servicio calcula una posición estimada para él y lo añade a la base.

**Fase de consulta (localizarte a ti).**

1. **Escucha.** El móvil hace un escaneo WiFi: pasa por los canales y recoge las balizas, o envía una solicitud de sondeo (*probe request*) y recoge las respuestas. Anota el BSSID y la intensidad de cada punto de acceso.
2. **Celda.** Añade el identificador de la celda que le da servicio y, si las tiene, las de las celdas vecinas.
3. **Consulta.** Envía la lista al servicio de localización.
4. **Cálculo.** Según el proveedor, el servidor devuelve una posición ya calculada o las posiciones de los puntos de acceso, y el móvil hace la cuenta.
5. **Fusión.** El sistema operativo combina el resultado con el GPS (si lo hay), los sensores de movimiento y la posición anterior.

### 2. Qué hace cada componente

**La baliza WiFi.** La define la norma IEEE 802.11. Es una trama de gestión que el punto de acceso emite de forma periódica. Lleva el BSSID en la cabecera, el nombre de la red (SSID), los canales y las capacidades. Se transmite sin cifrar incluso en redes con WPA3, porque el cifrado se negocia después, al conectarse. El intervalo entre balizas es configurable. El valor habitual en los equipos es de 100 unidades de tiempo de 1024 µs, unos 102,4 ms (valor habitual, no fijado por la norma; no verificado en el texto del estándar).

**El BSSID.** Es un identificador de 48 bits, normalmente la dirección MAC de la radio del punto de acceso. Un mismo router con dos bandas suele tener dos BSSID. Para el sistema de localización, el BSSID es la clave de búsqueda. El nombre de la red no sirve, porque miles de redes se llaman igual.

**La dirección del propio móvil.** Un escaneo activo revela que hay un móvil, pero no hace falta que revele cuál. Los sistemas actuales usan direcciones aleatorias al buscar redes ([artículo 11·01](../11_PrivacidadYHuella/01-PorQueElMovilCambiaSuDireccionMAC.md)). Lo que el móvil necesita para localizarse son los identificadores de los demás, no el suyo.

**El identificador de celda.** Toda celda de telefonía emite su identidad en la información de sistema que difunde. El móvil la necesita para registrarse en la red. Se compone del código de país (MCC), el de operador (MNC), el código de área (LAC o TAC) y el identificador de celda. En LTE ese último ocupa 28 bits y en 5G NR, 36 bits, según la documentación de la API de Google. Leerlo no requiere ninguna conexión de datos.

**El avance temporal (*timing advance*).** La estación base ordena a cada móvil que adelante su transmisión según lo lejos que esté, para que todas las señales lleguen alineadas. Ese valor es, en la práctica, una medida del retardo de ida y vuelta. Convierte «estoy en esta celda» en «estoy en un anillo a cierta distancia de la antena». La API de Google acepta este dato como entrada opcional.

**La base de datos.** Guarda, para cada BSSID o celda, una posición estimada. Rye y Levin obtuvieron de la base de Apple unos 490 millones de BSSID con posición en su estudio de 2024. Un punto de acceso nuevo no aparece al instante: necesita que varios aparatos lo vean durante un tiempo.

**El servicio de consulta.** Hay dos diseños documentados.

- **Google** recibe la lista de BSSID con su intensidad y devuelve una posición y un radio de precisión. Exige al menos dos puntos de acceso «físicamente distintos y estacionarios». Con menos, responde que no ha encontrado nada.
- **Apple**, según Rye y Levin, recibe los BSSID y devuelve la posición de cada uno y la de hasta 400 BSSID cercanos más. El cálculo final lo hace el propio aparato.

**El cálculo de posición.** El método más sencillo es el centroide: la media de las posiciones de los puntos de acceso oídos. El centroide ponderado da más peso a los que llegan con más intensidad. Hay métodos más finos, como comparar la huella de intensidades con mapas medidos (*fingerprinting*) o filtros de partículas. En el estudio de Cheng y otros (2005), en zonas urbanas densas el centroide simple fue casi tan preciso como los métodos complejos.

### 3. Parámetros que importan

**Precisión esperable.** La documentación de Google da estos radios típicos para su servicio:

| Datos disponibles | Radio de precisión típico |
|---|---|
| Dos o más puntos de acceso WiFi | unos 20 m |
| Celdas pequeñas (*small cells*) | 10 a 30 m |
| Celdas macro | cientos de metros; raro bajar de 100 m |
| Solo la dirección IP | miles de metros |

La documentación define la precisión como el radio de un círculo alrededor de la posición, pero no indica el nivel de confianza de ese círculo.

**Densidad de puntos de acceso.** Cheng y otros midieron en 2005 tres zonas del área de Seattle con tarjetas 802.11b, usando GPS como referencia:

| Zona | Puntos de acceso por km² | Puntos de acceso por escaneo (media) | Error mediano (centroide simple) |
|---|---|---|---|
| Centro urbano | 1030 | 2,66 | 24,4 m |
| Barrio residencial de densidad media | 1000 | 2,56 | 14,8 m |
| Barrio residencial disperso | 130 | 1,41 | 37,0 m |

El rango global fue de 13 a 40 m de error mediano. Son datos de 2005: hoy la densidad de puntos de acceso es mayor y los móviles oyen más redes por escaneo, pero no hay una cifra equivalente actual con la misma metodología. La tendencia sí se mantiene: cuantos más puntos de acceso se oyen, menor es el error.

**Intensidad recibida.** Android entrega cada resultado de escaneo con el BSSID, la frecuencia del canal en MHz y el nivel en dBm. La API de Google espera valores entre −128 y −10 dBm. La intensidad sirve para ponderar, no para medir distancias exactas. Como explica el [artículo 00·03](../00_Fundamentos/03-PorQueLaCoberturaCambiaEnPocosCentimetros.md), unos centímetros de movimiento pueden cambiarla en decenas de decibelios.

**Frecuencia de escaneo.** El sistema limita cuánto escanean las aplicaciones. En Android 9 y posteriores, cada aplicación en primer plano puede pedir cuatro escaneos cada 2 minutos. Todas las aplicaciones en segundo plano juntas, uno cada 30 minutos. Además, leer los resultados exige permiso de ubicación y tener la ubicación activada: el sistema trata la lista de redes cercanas como un dato de posición.

### 4. Diagrama o flujo

```text
 FASE DE RECOGIDA (otros móviles, antes)
 Móvil con GPS ─► "en (lat, lon) oigo BSSID A −55 dBm, B −72 dBm, celda X"
               ─► envío anónimo y cifrado al proveedor
               ─► base de datos: A ≈ (lat_A, lon_A), B ≈ (lat_B, lon_B), X ≈ ...

 FASE DE CONSULTA (tu móvil, ahora, sin GPS ni conexión WiFi)
   Balizas WiFi (sin cifrar, cada ~100 ms)        Celda que da servicio
   BSSID A  −55 dBm                               MCC-MNC-TAC-ID celda
   BSSID B  −72 dBm                               (+ avance temporal)
   BSSID C  −80 dBm
          │                                              │
          └──────────────┬───────────────────────────────┘
                         ▼
            Consulta al servicio (por datos móviles)
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
   Modelo Google:                 Modelo Apple:
   servidor calcula               servidor devuelve posiciones de A, B, C
   y devuelve posición            y de hasta 400 BSSID cercanos;
   + radio de precisión           el aparato calcula
          │                             │
          └──────────────┬──────────────┘
                         ▼
     Centroide ponderado: más peso a A (la más intensa)
                         ▼
     Fusión con sensores y con la última posición conocida
                         ▼
                Punto azul (error típico ~20 m con WiFi)
```

## Límites, excepciones y casos reales

**Puntos de acceso que se mueven.** Un router que se muda conserva su BSSID. Durante un tiempo, la base lo seguirá situando en la dirección antigua. Si el móvil oye solo ese, puede aparecer en otra ciudad. Google indica que su servicio ignora los puntos de acceso en movimiento, como los de aviones o trenes, y exige al menos dos distintos. Las consultas con varios puntos permiten descartar el que no encaja con los demás.

**BSSID aleatorios.** Algunos puntos de acceso cambian su BSSID. Rye y Levin señalan que el modo de punto de acceso personal de iOS 17 y Android 14 usa BSSID aleatorios. Esos puntos de acceso no sirven como referencia, y es intencionado: así no se puede seguir su rastro.

**La exclusión «_nomap».** Google y Apple permiten que el dueño de una red la excluya de su base. Basta con que el nombre de la red termine en «_nomap». Apple advierte que no funciona con redes ocultas, porque los móviles no pueden ver ese nombre. Google indica que cada proveedor de localización decide por su cuenta si respeta la marca.

**Sin conexión de datos.** En el modelo de consulta al servidor, sin datos móviles ni WiFi conectada no hay respuesta. Lo único disponible es lo que el aparato tuviera guardado de consultas anteriores. Cómo gestiona cada sistema esa información guardada no está documentado públicamente (no verificado).

**Interiores y plantas.** El centroide da una posición en el plano. Distinguir la planta de un edificio es mucho más difícil con WiFi, porque un punto de acceso se oye a través de los forjados. En el estudio de Cheng y otros, el error medio en interiores osciló entre 9 y 98 m según el lugar.

**Zonas sin redes.** En carretera o en el campo puede no haber ningún punto de acceso. En el barrio disperso del estudio de 2005, más de la mitad de los escaneos no oyó ninguno. Entonces solo queda la celda, con errores de cientos de metros o más.

**Diferencias entre aparatos.** La antena y el receptor de cada móvil miden la intensidad de forma algo distinta. Un mismo punto de acceso puede llegar con unos decibelios de diferencia en dos móviles junto a junto. Los métodos de huella son más sensibles a esto que el centroide.

## Consecuencias prácticas

Desactivar el GPS no apaga la localización del móvil. Mientras la radio WiFi pueda escanear y haya una celda, el sistema puede situarse a unas decenas de metros en ciudad. Por eso los sistemas operativos tratan los resultados de escaneo WiFi como un dato de ubicación protegido por permisos.

La posición por WiFi es rápida y funciona dentro de edificios, justo donde el GPS falla. El GPS es preciso a cielo abierto, justo donde suele haber pocas redes. Por eso los móviles combinan los dos: cada uno cubre el hueco del otro.

En el modelo de consulta al servidor, el proveedor recibe la lista de redes que te rodean. Apple y Google declaran que esos datos se tratan sin identificar a la persona. La privacidad depende de esa política, no del mecanismo, que por diseño revela el entorno.

Para el dueño de un punto de acceso, su BSSID acaba en bases de datos públicas aunque nunca se conecte nadie de fuera. Es una consecuencia directa de que la baliza deba ser legible por todos. La marca «_nomap» y los BSSID aleatorios son las dos formas documentadas de reducirlo.

## Preguntas frecuentes

### ¿Es lo mismo que la triangulación por antenas de telefonía?

No. Con la celda, el móvil sabe a qué antena está conectado, no a qué distancia exacta de tres antenas. El resultado típico es un área de cientos de metros. La WiFi da más precisión porque los puntos de acceso tienen menos alcance y son muchos más.

### ¿Tengo que conectarme a la red WiFi para que esto funcione?

No. Las balizas se emiten sin cifrar para que cualquiera pueda descubrir la red. El móvil solo las escucha. Conectarse no aporta nada a la posición.

### ¿Tener la WiFi conectada a mi casa ayuda a localizarme?

Solo como un punto de acceso más. Lo que cuenta es cuántos puntos de acceso oye el móvil y si la base de datos los conoce, no a cuál está conectado.

### ¿La localización por IP es lo mismo?

No. La IP identifica, como mucho, la zona del proveedor de internet. Google da precisiones de miles de metros para la IP. La WiFi identifica aparatos físicos concretos, con alcance de unas decenas de metros.

## Fuentes

- [Google — Geolocation API: Geolocation requests](https://developers.google.com/maps/documentation/geolocation/requests-geolocation). Campos de la consulta (BSSID, intensidad en dBm entre −128 y −10, celdas con MCC, MNC, LAC/TAC, identificador de 28 bits en LTE y 36 bits en NR, avance temporal), mínimo de dos puntos de acceso estacionarios y definición de la precisión como radio en metros.
- [Google — Geolocation API overview](https://developers.google.com/maps/documentation/geolocation/overview). Radios de precisión típicos: unos 20 m con dos o más puntos de acceso, 10–30 m con celdas pequeñas, cientos de metros con celdas macro y miles de metros con la IP.
- [Apple — Location Services & Privacy](https://www.apple.com/legal/privacy/data/en/location-services/). Uso de GPS, Bluetooth y ubicaciones colaborativas de puntos de acceso y antenas; envío periódico, anónimo y cifrado de las ubicaciones geoetiquetadas de redes cercanas.
- [Apple — About privacy and Location Services in iOS, iPadOS, and watchOS](https://support.apple.com/en-us/102515). Exclusión de un punto de acceso con el sufijo «_nomap» y su limitación con redes ocultas.
- [Google — Control access point inclusion in Google's Location services](https://support.google.com/maps/answer/1725632). Exclusión con el sufijo «_nomap»; cada proveedor decide si la respeta.
- [Android Developers — Wi-Fi scanning overview](https://developer.android.com/develop/connectivity/wifi/wifi-scan). Límites de escaneo (4 cada 2 min en primer plano, 1 cada 30 min en segundo plano en total) y permiso de ubicación obligatorio.
- [Android Developers — ScanResult](https://developer.android.com/reference/android/net/wifi/ScanResult). Contenido de un resultado de escaneo: BSSID, SSID, nivel en dBm y frecuencia del canal.
- [E. Rye y D. Levin, «Surveilling the Masses with Wi-Fi-Based Positioning Systems», arXiv:2405.14975 (2024)](https://arxiv.org/abs/2405.14975). Construcción colaborativa de las bases, diferencias entre los servicios de Apple y Google, hasta 400 BSSID cercanos devueltos por Apple, cálculo en el aparato y BSSID aleatorios en iOS 17 y Android 14.
- [Y.-C. Cheng, Y. Chawathe, A. LaMarca y J. Krumm, «Accuracy Characterization for Metropolitan-scale Wi-Fi Localization», MobiSys 2005](https://usenix.org/legacy/events/mobisys05/tech/full_papers/cheng/cheng_html/index.html). Error mediano de 13 a 40 m, densidades de puntos de acceso, comparación entre centroide, huella y filtro de partículas, y error en interiores.
- [IEEE 802.11 Working Group — IEEE Std 802.11, Wireless LAN Medium Access Control (MAC) and Physical Layer (PHY) Specifications](https://www.ieee802.org/11/). Definición de las tramas de baliza y de sondeo, del BSSID y de la unidad de tiempo de 1024 µs.
