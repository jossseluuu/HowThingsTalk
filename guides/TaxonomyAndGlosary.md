# Taxonomía y glosario / Taxonomy and glossary

<div align="center">

**Español** · [English](#english)

</div>

---

## 1. Taxonomía del repositorio / Repository taxonomy

El contenido del repositorio se organiza por tema y por profundidad. La estructura esencial es:

The content of the repository is organized by topic and depth. The core structure is:

| Nº | Sección / Section | Carpeta ES / ES folder | Carpeta EN / EN folder |
|---|---|---|---|
| `00` | Fundamentos / Foundations | `00_Fundamentos` | `00_Foundations` |
| `01` | Redes WiFi / WiFi Networks | `01_RedesWiFi` | `01_WiFiNetworks` |
| `02` | Redes Móviles / Cellular Networks | `02_RedesMoviles` | `02_CellularNetworks` |
| `03` | Corto Alcance / Short-Range Radio | `03_CortoAlcance` | `03_ShortRangeRadio` |
| `04` | Transferencia de Datos / Data Transfer | `04_TransferenciaDeDatos` | `04_DataTransfer` |
| `05` | Geolocalización / Positioning | `05_Geolocalizacion` | `05_Positioning` |
| `06` | Almacenamiento / Storage | `06_Almacenamiento` | `06_Storage` |
| `07` | Audio y Periféricos / Audio and Peripherals | `07_AudioYPerifericos` | `07_AudioAndPeripherals` |
| `08` | Energía y Baterías / Power and Batteries | `08_EnergiaYBaterias` | `08_PowerAndBatteries` |
| `09` | IoT y Smart Home / IoT and Smart Home | `09_IOT` | `09_IOT` |
| `10` | Seguridad y Criptografía / Security and Cryptography | `10_SeguridadYCriptografia` | `10_SecurityAndCryptography` |
| `11` | Privacidad y Huella / Privacy and Footprint | `11_PrivacidadYHuella` | `11_PrivacyAndFootprint` |
| `99` | Referencia / Reference | `99_Referencia` | `99_Reference` |

Cada sección tiene una función distinta:

Each section serves a different purpose:

- `00` explora los conceptos físicos y matemáticos básicos.
- `01` a `04` cubren redes, protocolos y transferencia de información.
- `05` a `08` cubren localización, almacenamiento, audio y energía.
- `09` a `11` cubren sistemas conectados, seguridad y privacidad.
- `99` guarda referencias rápidas, términos y documentos útiles.

- `00` explores the basic physical and mathematical concepts.
- `01` to `04` cover networks, protocols, and information transfer.
- `05` to `08` cover positioning, storage, audio, and power.
- `09` to `11` cover connected systems, security, and privacy.
- `99` holds quick references, terms, and useful documents.

---

## 2. Tipos de artículo / Article types

Los artículos del proyecto se pueden clasificar en estas categorías:

Articles in the project can be classified into these categories:

### A. Fundamentos / Foundations

Explican la física o la idea base que hace posible un mecanismo, por ejemplo:

- propagación de radio;
- potencia y decibelios;
- latencia y throughput;
- señal, ruido y canal.

They explain the underlying physics or concept that makes a mechanism possible, for example:

- radio propagation;
- power and decibels;
- latency and throughput;
- signal, noise, and channel.

### B. Mecanismo / Mechanism

Explican cómo funciona un sistema concreto: WiFi, Bluetooth, GPS, celular, SSD, batería, etc.

They explain how a specific system works: WiFi, Bluetooth, GPS, cellular, SSD, battery, etc.

### C. Consecuencia / Consequence

Explican el impacto en la vida real: por qué falla la cobertura, por qué se corta el audio, por qué un dispositivo en frío se apaga.

They explain the real-world impact: why coverage fails, why audio cuts out, why a device shuts down in the cold.

### D. Seguridad y privacidad / Security and privacy

Interpretan cómo se protege la información, qué datos se filtran y qué significa la privacidad en sistemas conectados.

They interpret how information is protected, what data leaks, and what privacy means in connected systems.

### E. Referencia / Reference

Suelen ser tablas, listas, cronologías, mitos, comparativas o glosarios.

These are usually tables, lists, timelines, myth-busting entries, comparisons, or glossaries.

---

## 3. Glosario principal / Main glossary

### Ancho de banda / Bandwidth
Anchura del rango de frecuencias que ocupa una señal o un canal, medida en hercios (por ejemplo, 20 MHz en un canal WiFi). En el lenguaje común se usa también como sinónimo de capacidad de datos; en este proyecto, cuando se habla de bits por segundo, se dice tasa de bits o throughput.

Width of the frequency range occupied by a signal or channel, measured in hertz (for example, 20 MHz for a WiFi channel). In everyday language it is also used to mean data capacity; in this project, when we mean bits per second, we say bitrate or throughput.

### Antena / Antenna
Una estructura que convierte corriente eléctrica en ondas electromagnéticas y viceversa. Su forma, tamaño y orientación afectan la dirección, la ganancia y la eficiencia del enlace.

A structure that converts electrical current into electromagnetic waves and vice versa. Its shape, size, and orientation affect directionality, gain, and efficiency.

### Banda / Band
Un rango de frecuencias usado por una tecnología. Ejemplos: 2,4 GHz, 5 GHz, banda baja, banda de microondas.

A range of frequencies used by a technology. Examples: 2.4 GHz, 5 GHz, low band, microwave band.

### BSSID
Identificador de un punto de acceso WiFi concreto, normalmente su dirección MAC (48 bits). Una misma red (SSID) puede tener muchos BSSID, uno por punto de acceso y banda.

Identifier of a specific WiFi access point, usually its MAC address (48 bits). A single network (SSID) can have many BSSIDs, one per access point and band.

### Canal / Channel
Un segmento concreto de frecuencia dentro de una banda. En WiFi y Bluetooth, el canal determina qué grupos de frecuencia usa el enlace.

A specific segment of frequency within a band. In WiFi and Bluetooth, the channel determines which frequency group the link uses.

### Capa / Layer
Parte de una pila de protocolos: física, enlace, red, transporte, aplicación. Cada capa resuelve un problema concreto.

Part of a protocol stack: physical, link, network, transport, application. Each layer solves a different problem.

### Códec / Codec
Algoritmo que comprime y descomprime audio o vídeo. Cambia la calidad, el ancho de banda y la latencia.

Algorithm that compresses and decompresses audio or video. It changes quality, bandwidth, and latency.

### Decibelio / Decibel
Unidad logarítmica que expresa la relación entre dos potencias: +3 dB es aproximadamente el doble y +10 dB, diez veces más. El dBm es la potencia referida a 1 mW, de modo que 0 dBm = 1 mW y −70 dBm = 0,1 nW.

Logarithmic unit expressing the ratio between two powers: +3 dB is roughly double and +10 dB is ten times more. dBm is power referenced to 1 mW, so 0 dBm = 1 mW and −70 dBm = 0.1 nW.

### Enlace / Link
Conexión lógica o física entre dos puntos: radio, cable, Bluetooth, WiFi, etc.

Logical or physical connection between two points: radio, cable, Bluetooth, WiFi, etc.

### Frecuencia / Frequency
Número de ciclos por segundo de una onda. Se mide en hercios (Hz), y determina cómo interactúa la onda con el entorno.

Number of cycles per second of a wave. Measured in hertz, it determines how the wave interacts with the environment.

### GPS
Sistema estadounidense de navegación por satélite, uno de los GNSS junto a Galileo, GLONASS y BeiDou. Se usa para posicionamiento, navegación y sincronización horaria.

The US satellite navigation system, one of the GNSS alongside Galileo, GLONASS, and BeiDou. Used for positioning, navigation, and time synchronization.

### Latencia / Latency
Tiempo que tarda un dato en recorrer un sistema o una red; normalmente se expresa en milisegundos.

Time required for data to traverse a system or network; typically expressed in milliseconds.

### MIMO
Multiple Input Multiple Output: uso de varias antenas para transmitir o recibir más información simultáneamente.

Multiple Input Multiple Output: the use of multiple antennas to transmit or receive more information simultaneously.

### Paquete / Packet
Unidad de datos de red. Una aplicación puede dividir un archivo en muchos paquetes que viajan por la red y se reensamblan al llegar.

Data unit of a network. An application can split a file into many packets that travel over the network and are reassembled when they arrive.

### Potencia / Power
Cantidad de energía transmitida por unidad de tiempo. En radio, hay diferencias cruciales entre potencia, intensidad y nivel medido.

Amount of energy transmitted per unit of time. In radio, there are crucial differences between power, intensity, and measured level.

### Protocolo / Protocol
Conjunto de reglas para que dos sistemas puedan comunicarse sin ambigüedad.

Set of rules that allows two systems to communicate without ambiguity.

### Rendimiento efectivo / Throughput
Cantidad efectiva de datos que pasa por un sistema o canal por unidad de tiempo, teniendo en cuenta pérdidas y overhead.

Actual amount of data that passes through a system or channel over time, taking into account losses and overhead.

### Ruido / Noise
Energía no deseada que se suma a la señal y dificulta recuperar el mensaje. Más que la potencia de la señal, lo que decide si un enlace funciona es la relación señal-ruido (SNR).

Unwanted energy added to the signal that makes the message harder to recover. More than signal strength, what decides whether a link works is the signal-to-noise ratio (SNR).

### Señal / Signal
Magnitud física que varía para transportar información. La calidad real depende de la fuerza, el ruido, el canal y la interferencia.

Physical quantity that varies to carry information. Real quality depends on signal strength, noise, the channel, and interference.

### Tasa de bits / Bitrate
Velocidad de transferencia digital, expresada en bits por segundo (bit/s, Mbit/s).

Rate of digital transfer, expressed in bits per second (bit/s, Mbit/s).

### UWB
Ultra Wide Band: técnica de radio con ancho de banda muy grande, útil para localización de alta precisión y sistemas de ranging.

Ultra Wide Band: radio technique with a very large bandwidth, useful for high-precision localization and ranging systems.

### WiFi
Tecnología de redes inalámbricas de área local (familia IEEE 802.11) que opera en 2,4 GHz, 5 GHz y, en los equipos recientes, 6 GHz. Permite conectar dispositivos a una red local y a internet.

Wireless local area networking technology (IEEE 802.11 family) operating in 2.4 GHz, 5 GHz, and, on recent equipment, 6 GHz. It allows devices to connect to a local network and to the internet.

---

## 4. Términos recurrentes / Recurring terms

Algunos conceptos aparecen en muchas secciones y conviene usarlos de forma consistente:

Some concepts appear across many sections and should be used consistently:

- `latencia` / `latency`
- `ancho de banda` / `bandwidth`
- `potencia` / `power`
- `señal` / `signal`
- `ruido` / `noise`
- `protocolo` / `protocol`
- `capa` / `layer`
- `enlace` / `link`
- `batería` / `battery`
- `sensor` / `sensor`
- `telemetría` / `telemetry`
- `privacidad` / `privacy`
- `seguridad` / `security`

---

## 5. Criterio para definir un término / Criteria for defining a term

Un término debe definirse si:

- aparece en más de una sección;
- es necesario para entender un mecanismo central;
- tiene un uso ambiguo en la literatura o en la vida cotidiana;
- se necesita para explicar un concepto técnico sin caer en metáforas vagas.

A term should be defined if:

- it appears in more than one section;
- it is necessary to understand a central mechanism;
- it has ambiguous usage in literature or everyday life;
- it is needed to explain a technical concept without falling into vague metaphors.

---

## English

### Taxonomy summary

The library is structured by topic and depth. Each section corresponds to a technology family, and each article answers one concrete question within that family.

The glossary is intentionally small but precise: terms are defined once, then reused consistently across the entire project so that readers do not have to relearn the same concept in every chapter.

[Volver al español · Back to Spanish](#taxonomía-y-glosario--taxonomy-and-glossary)

[← Volver al índice general / Back to the master index](../docs/Index.md)
