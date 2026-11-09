---
title: "¿Cómo convierte el micrófono del móvil el sonido en números?"
section: "07_AudioYPerifericos"
language: "es"
status: "published"
order: 2
depth: "intermediate"
summary: "El micrófono de un móvil es un condensador diminuto: una membrana de medio milímetro que se acerca y se aleja de una placa fija con perforaciones. Ese cambio de capacidad se convierte en tensión y un modulador la traduce en un chorro de unos y ceros a unos 3 MHz, cuya densidad sigue a la onda. Un filtro digital promedia ese chorro y entrega muestras de audio normales, por ejemplo 48 000 por segundo."
prerequisites:
  - "Qué es una onda de sonido: una variación de presión del aire"
  - "Decibelios como relación entre dos magnitudes"
translation: "../../EN/07_AudioAndPeripherals/02-HowAPhoneMicrophoneTurnsSoundIntoNumbers.md"
---

# ¿Cómo convierte el micrófono del móvil el sonido en números?

## Pregunta y encuadre

Cuando hablas por teléfono, tu voz es aire que empuja y tira. Al otro lado llega una lista de números. En medio hay una pieza más pequeña que un grano de arroz. ¿Qué ocurre dentro de ella? ¿Cómo se pasa de una presión que cambia de forma continua a bits?

Este artículo explica el micrófono MEMS (*microelectromechanical system*), el tipo que llevan los móviles, los portátiles y los auriculares. Cubre dos etapas: cómo la membrana convierte la presión en una señal eléctrica y cómo esa señal se convierte en un flujo de bits. Se centra en los micrófonos digitales con salida PDM, los más habituales en estos aparatos.

- **Profundidad:** intermedia. Hay alguna fórmula sencilla y cálculos con números, sin derivaciones.
- **Conocimientos previos:** saber que el sonido es una variación de presión y manejar decibelios como relación entre dos magnitudes. Si conoces [¿Cómo puede un auricular quitar ruido emitiendo más sonido?](01-ComoQuitaRuidoUnAuricular.md), ya tienes la idea de presión acústica; no es imprescindible.
- **Fuera de alcance:** la combinación de varios micrófonos para enfocar una voz (*beamforming*), la reducción de ruido en llamadas, los códecs que comprimen el audio después y la comparación entre modelos comerciales.

## Respuesta corta

La presión del sonido mueve una membrana que forma un condensador con una placa fija; el cambio de capacidad se vuelve tensión, un modulador sigma-delta la traduce en un chorro de bits sueltos cuya proporción de unos sigue a la onda, y un filtro digital lo promedia para obtener muestras de audio.

### Resumen de 60 segundos

Un sonido de 94 dB SPL, muy fuerte, equivale a una presión de solo 1 Pa. Es una cienmilésima parte de la presión atmosférica. El micrófono tiene que notar eso y mucho menos.

Para lograrlo usa un condensador. Una membrana de silicio de alrededor de medio milímetro está separada de una placa fija por unas pocas micras de aire. La placa tiene agujeros para que el aire pase. Cuando la presión empuja la membrana, la distancia cambia y cambia la capacidad. Con una tensión de polarización aplicada, eso produce una tensión de salida proporcional al movimiento.

Un chip pegado a la membrana amplifica esa tensión y la digitaliza de una forma peculiar. No produce números de 16 bits, sino un solo bit cada vez, unos 3 millones de veces por segundo. Si la presión es alta, salen más unos. Si es baja, más ceros. Es la modulación por densidad de pulsos (PDM).

El procesador del móvil promedia ese chorro y se queda con una muestra de cada 64. Con un reloj de 3,072 MHz salen 48 000 muestras por segundo, ya en el formato de audio habitual.

## Modelo mental

Piensa en una foto impresa en un periódico. Solo hay tinta negra y papel blanco. Aun así ves grises. En las zonas oscuras hay muchos puntos; en las claras, pocos. Tu ojo no distingue cada punto: promedia una zona pequeña y percibe un tono.

El chorro PDM hace lo mismo con el tiempo. Cada bit es un punto negro o blanco. La densidad de unos en un intervalo corto marca el nivel de la onda en ese momento. El filtro digital hace el papel del ojo: promedia y reconstruye el «gris», que aquí es el valor de la presión.

**Dónde se rompe la analogía.** En el periódico, la densidad de puntos se decide mirando el tono de cada zona por separado. En el micrófono, cada bit se decide mirando el error acumulado de los bits anteriores. Ese lazo de realimentación es lo que empuja el error de cuantificación hacia frecuencias muy altas, lejos del audio. La analogía tampoco dice nada de la primera etapa: cómo la presión se convierte en tensión. Eso lo hace la membrana.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

El camino del sonido dentro del micrófono tiene cinco pasos:

1. **El sonido entra por un orificio.** El encapsulado tiene un puerto acústico que lleva la presión hasta la membrana.
2. **La membrana se mueve.** La presión la desplaza respecto a una placa fija. Las dos forman un condensador.
3. **El cambio de capacidad se vuelve tensión.** Una tensión de polarización mantiene carga en el condensador. Al variar la distancia, varía la tensión entre las placas.
4. **El chip lo amplifica y lo modula.** Un circuito integrado (ASIC) en el mismo encapsulado amplifica la señal y la convierte en un flujo de 1 bit.
5. **El receptor lo filtra y lo diezma.** El procesador del aparato promedia el flujo con un filtro paso bajo y se queda con una muestra de cada N. El resultado es audio PCM de 16 bits o más.

### 2. Qué hace cada componente

**La membrana y la placa.** Según la revisión de Zawawi y coautores (2020), un micrófono capacitivo MEMS tiene una membrana móvil y una placa trasera fija, separadas por un hueco de aire. Un aislante hace de separador. La capacidad sigue la fórmula de un condensador plano:

C = ε₀ · A / g

Aquí A es el área de la membrana y g la distancia entre placas. Si la membrana se acerca, g baja y C sube. Los tamaños son pequeños. La misma revisión cita membranas de unos 0,5 mm de diámetro en los micrófonos Knowles del iPhone 4, de 1 mm en un modelo de Infineon y de 0,59 mm en uno de Analog Devices. Los diseños de investigación que recopila tienen huecos de aire de entre 2 y 3,6 µm.

Un cálculo propio, con valores ilustrativos: una membrana circular de 0,5 mm de diámetro a 2 µm de la placa forma un condensador de unos 0,87 pF. Es una cantidad minúscula. Unos centímetros de pista o de cable añadirían una capacidad parásita comparable. El amplificador va dentro del mismo encapsulado, a milímetros de la membrana.

**La polarización y la señal eléctrica.** El condensador necesita carga para producir una señal. Zawawi *et al.* expresan la sensibilidad como el producto de dos factores:

S = (V_b / g) · (Δg / ΔP)

El primer factor es eléctrico: cuánta tensión sale por cada micra de movimiento. Depende de la tensión de polarización V_b y del hueco g. El segundo es mecánico: cuánto se mueve la membrana por cada pascal. Depende de su tamaño, su tensión mecánica y su grosor. En un micrófono digital moderno, la tensión de polarización la genera una bomba de carga dentro del ASIC. El diagrama de bloques del Infineon IM68D128BV01 la muestra como un bloque propio. Los micrófonos *electret* clásicos, en cambio, guardan una carga permanente en un material especial (Zawawi *et al.*).

**Los agujeros de la placa.** Entre la membrana y la placa queda una lámina de aire muy fina. Si no pudiera escapar, actuaría como un amortiguador y frenaría a la membrana, sobre todo en agudos. La sensibilidad caería mucho. La placa está perforada para que el aire entre y salga del hueco. Más agujeros y más hueco reducen ese amortiguamiento (Zawawi *et al.*).

**El modulador de 1 bit.** El ASIC no usa un convertidor que mida la tensión con 16 bits de una vez. Usa un modulador de un solo bit que trabaja muy deprisa. MT-022, el tutorial de Analog Devices sobre convertidores sigma-delta, describe su estructura: un integrador, un comparador y un convertidor digital-analógico de 1 bit en un lazo. En cada ciclo, el lazo compara la entrada con lo que ya ha emitido. Si se ha quedado corto, emite un uno. Si se ha pasado, un cero. La tensión media de la salida la fija la proporción de unos del flujo. En el datasheet de Infineon, una presión positiva aumenta la densidad de unos, y una alternancia regular de unos y ceros representa el cero.

**La conformación del ruido.** Un solo bit es una cuantificación muy tosca. Un convertidor ideal de N bits tiene una relación señal-ruido de 6,02·N + 1,76 dB (MT-022). Con N = 1, son unos 7,8 dB. La solución tiene dos partes:

- **Sobremuestreo.** Si se muestrea mucho más deprisa de lo necesario, el error de cuantificación se reparte en una banda más ancha. Cada vez que se dobla la frecuencia, la parte que cae en la banda de audio baja unos 3 dB. Es una consecuencia del modelo de ruido de MT-022, no una cifra que el tutorial dé de forma explícita.
- **Conformación.** El lazo con integrador empuja el error hacia frecuencias altas, fuera de la banda útil. Según MT-022, la relación señal-ruido mejora 9 dB por cada duplicación del sobremuestreo con un lazo de primer orden y 15 dB con uno de segundo orden. Su ejemplo: con un sobremuestreo de 64, un sistema ideal de segundo orden alcanza unos 80 dB, el equivalente a unos 13 bits efectivos.

**El filtro de diezmado.** El receptor recibe millones de bits por segundo con mucho ruido fuera de la banda de audio. Un filtro digital paso bajo lo elimina y promedia los bits en números de más resolución. Después se descartan las muestras sobrantes: se queda una de cada M. MT-022 llama a esto diezmado. M puede ser cualquier entero, siempre que la frecuencia de salida siga siendo más del doble del ancho de banda de la señal.

**Dos micrófonos en un solo cable.** La nota AN5027 de STMicroelectronics explica cómo dos micrófonos comparten la misma línea de datos. Cada uno tiene un pin de selección de canal. Uno emite su bit cuando el reloj está alto y deja la línea en alta impedancia cuando está bajo. El otro hace lo contrario. El receptor separa después los bits intercalados. Así, dos micrófonos comparten el mismo reloj y una sola línea de datos.

### 3. Parámetros que importan

**La escala de presiones.** La referencia para medir micrófonos es un tono de 1 kHz a 94 dB SPL, que equivale a 1 Pa (Lewis, 2012; AN-1112). Algunos valores, por cálculo propio a partir de la referencia de 20 µPa que implica esa equivalencia:

| Nivel | Presión | Fracción de la presión atmosférica (101 325 Pa) |
|---|---|---|
| 29 dB SPL | 0,56 mPa | 6 × 10⁻⁹ |
| 60 dB SPL | 0,02 Pa | 2 × 10⁻⁷ |
| 94 dB SPL | 1 Pa | 1 × 10⁻⁵ |
| 120 dB SPL | 20 Pa | 2 × 10⁻⁴ |

**Sensibilidad.** En un micrófono analógico se da en dBV respecto a 1 V/Pa. Según Lewis, la mayoría de micrófonos analógicos *electret* y MEMS están entre −46 y −35 dBV, es decir, entre 5,0 y 17,8 mV/Pa. Un ejemplo típico es −38 dBV, unos 12,6 mV/Pa (cálculo propio). En un micrófono digital se da en dBFS: qué fracción del fondo de escala digital produce un tono de 94 dB SPL. En el ejemplo de Lewis, el fondo de escala corresponde a 120 dB SPL y la sensibilidad es 120 − 94 = −26 dBFS. Por eso, en un micrófono digital, cambiar la ganancia interna cambia también el nivel máximo.

**Ruido, saturación y rango dinámico.** La nota AN-1112 de InvenSense define estas magnitudes:

- **SNR:** diferencia entre la referencia de 94 dB SPL y el ruido propio del micrófono, ponderado A en una banda de 20 kHz. Incluye el ruido de la membrana y el del ASIC.
- **Ruido equivalente de entrada (EIN):** el ruido expresado como un sonido. Vale 94 dB − SNR. Con una SNR de 65 dB, el EIN es de 29 dB SPL.
- **Punto de sobrecarga acústica (AOP):** el nivel en el que la distorsión armónica total llega al 10 %. A partir de ahí la señal se recorta.
- **Rango dinámico:** AOP − EIN. En el ejemplo de AN-1112, 120 − 29 = 91 dB.

**Un micrófono concreto.** El datasheet del Infineon IM68D128BV01 (V1.1, abril de 2025) da estas cifras:

| Parámetro | Valor |
|---|---|
| Tamaño del encapsulado | 3,5 × 2,65 × 0,98 mm |
| Sensibilidad | −37 dBFS típica (1 kHz, 94 dB SPL) |
| SNR, modo normal | 67,5 dB(A), 20 Hz-20 kHz, reloj de 3,072 MHz |
| AOP | 128 dB SPL al 10 % de distorsión |
| Caída en graves (−3 dB) | 20 Hz |
| Resonancia | 36 kHz |
| Consumo, modo normal | 580 µA a 1,8 V, unos 1,0 mW (cálculo propio) |
| Consumo, bajo consumo | 190 µA a 768 kHz, unos 0,34 mW, con banda de 20 Hz-8 kHz |

**Reloj y diezmado.** AN5027 da la relación básica: frecuencia PDM = frecuencia de audio × factor de diezmado. El factor suele estar entre 48 y 128. Sus ejemplos sobremuestrean por 64 para obtener audio a 16 o 48 kHz. Con un factor de 64, un reloj de 3,072 MHz da 48 kHz (cálculo propio). La biblioteca de ST entrega PCM de 16 bits. El flujo de datos baja así de 3,072 Mbit/s a 768 kbit/s, una cuarta parte (cálculo propio).

### 4. Diagrama o flujo

```text
  PRESIÓN SONORA (ej. 1 Pa a 94 dB SPL)
        │  puerto acústico
        ▼
 ┌──────────────────────────────────────── ENCAPSULADO ≈ 3,5 × 2,65 mm ─┐
 │                                                                      │
 │  MEMBRANA ~0,5-1 mm ─┐                                               │
 │     ↕ hueco ~2-4 µm  ├─ condensador C = ε₀·A/g  (≈ 1 pF)             │
 │  PLACA PERFORADA ────┘        │                                      │
 │        ▲                      ▼                                      │
 │  bomba de carga (V_b)    AMPLIFICADOR                                │
 │                               │                                      │
 │                               ▼                                      │
 │                  MODULADOR SIGMA-DELTA DE 1 BIT                      │
 │                  entrada ─► (−) ─► integrador ─► comparador ─┬─► bit │
 │                              ▲                               │       │
 │                              └──── DAC de 1 bit ◄────────────┘       │
 └──────────────────────────────────────────────────┬───────────────────┘
                                                    │ DATOS PDM: 1011011101101...
                         RELOJ 3,072 MHz ──────────►│ (densidad de unos ≈ presión)
                                                    ▼
                        PROCESADOR DEL APARATO: filtro paso bajo (p. ej. sinc)
                                                    │  quedarse con 1 de cada 64
                                                    ▼
                                    PCM 16 bits · 48 000 muestras/s
```

## Límites, excepciones y casos reales

- **El ruido del propio micrófono pone el suelo.** El modulador ideal del ejemplo de MT-022 llegaría a unos 80 dB de SNR. El micrófono completo de Infineon se queda en 67,5 dB(A). Según AN-1112, la SNR incluye el ruido de la membrana y del ASIC, no solo el de la conversión. Un sonido por debajo del EIN, unos 29 dB SPL en el ejemplo de AN-1112, queda enterrado en ese ruido.
- **Los sonidos fuertes se recortan.** Por encima del AOP, la distorsión pasa del 10 %. En el modelo de Infineon, la distorsión es del 0,3 % a 94 dB SPL, del 1 % a 122 dB SPL y del 10 % a 128 dB SPL. Un sonido muy fuerte y muy cercano puede pasar de ese límite.
- **La respuesta no es plana en los extremos.** Por debajo de la caída de graves (20 Hz en el modelo citado), la sensibilidad baja. AN-1112 describe esa caída como de primer orden, 6 dB por octava, y la de agudos como de segundo orden, 12 dB por octava. La resonancia de 36 kHz queda fuera de la banda de audio.
- **El modulador no es lineal por encima del segundo orden.** MT-022 advierte que los lazos de orden superior a dos no siguen el modelo lineal simple. Los diseñadores tienen que vigilar su estabilidad.
- **La sensibilidad tiene tolerancia.** AN-1112 cita tolerancias de ±1 a ±3 dB entre unidades. El modelo de Infineon especifica de −38 a −36 dBFS. En un sistema con varios micrófonos, esa diferencia se nota si no se calibra.
- **Arrancar lleva tiempo.** El micrófono de Infineon emite un patrón de cero (unos y ceros alternos) a los 3 ms de recibir alimentación y reloj. Su sensibilidad no está dentro de ±1 dB hasta 25 ms después con relojes de 768 kHz o más.

## Consecuencias prácticas

- **Un móvil puede llevar varios micrófonos.** Son baratos de integrar: un encapsulado de pocos milímetros y una línea compartida para cada par. Zawawi *et al.*, citando a IHS Markit, indican que un teléfono puede usar hasta ocho.
- **El modo de bajo consumo existe por algo.** Bajar el reloj de 3,072 MHz a 768 kHz reduce el consumo de unos 1,0 mW a unos 0,34 mW en el modelo de Infineon. A cambio, la banda útil baja a 8 kHz. Basta para la voz, no para la música.
- **Las grabaciones de ambientes silenciosos suenan a soplido.** El ruido propio del micrófono equivale a un sonido de unos 29 a 33 dB SPL en los ejemplos de Lewis y AN-1112. En una habitación muy silenciosa, ese ruido se vuelve audible cuando se sube la ganancia.
- **Para quien diseña.** En un micrófono digital, la sensibilidad y el nivel máximo van atados: el fondo de escala ya está fijado dentro del chip (Lewis). No se puede ganar margen ante sonidos fuertes sin perder nivel en los débiles. Y el procesador tiene que reservar recursos para el filtro de diezmado de cada canal.

## Preguntas frecuentes

### ¿Es lo mismo un micrófono MEMS que un micrófono *electret*?

Los dos son condensadores: una membrana frente a una placa. La diferencia está en la carga. El *electret* guarda una carga permanente en un material especial (Zawawi *et al.*). Un MEMS de silicio como el de Infineon recibe la tensión de polarización de una bomba de carga en su chip. Además, el MEMS se fabrica con técnicas de micromecanizado de semiconductores (Zawawi *et al.*).

### Si el reloj va a 3 MHz, ¿graba ultrasonidos?

No en la salida final. La frecuencia alta del reloj sirve para empujar el ruido de cuantificación fuera de la banda de audio, no para captar más banda. El filtro de diezmado elimina todo lo que queda por encima de la banda útil. Con una salida de 48 kHz, la frecuencia más alta que se puede representar es 24 kHz.

### ¿Por qué no usar directamente un convertidor de 16 bits?

Existen micrófonos con otras salidas digitales: Lewis cita uno con salida I²S. El enfoque de 1 bit mantiene simple la parte analógica del micrófono: un integrador, un comparador y un convertidor de un solo bit. Además, el sobremuestreo relaja el filtro antisolapamiento analógico (MT-022). La parte compleja, el filtro digital, se traslada al procesador del aparato.

### ¿Qué significa «−26 dBFS» en la ficha de un micrófono?

Que un tono de 94 dB SPL produce una salida digital 26 dB por debajo del máximo que el micrófono puede representar. En el ejemplo de Lewis, ese máximo corresponde a unos 120 dB SPL. Es una medida relativa al fondo de escala digital, no una tensión.

## Fuentes

- S. A. Zawawi, A. A. Hamzah, B. Y. Majlis y F. Mohd-Yasin. «A Review of MEMS Capacitive Microphones». *Micromachines*, vol. 11, n.º 5, 484, 2020. DOI: [10.3390/mi11050484](https://doi.org/10.3390/mi11050484). <https://www.mdpi.com/2072-666X/11/5/484>
- J. Lewis. «Understanding Microphone Sensitivity». *Analog Dialogue*, vol. 46, n.º 2, Analog Devices, mayo de 2012. <https://www.analog.com/en/resources/analog-dialogue/articles/understanding-microphone-sensitivity.html>
- InvenSense. *AN-1112: Microphone Specifications Explained*. Revisión 1.1, junio de 2016. <https://www.mouser.lt/pdfDocs/AN-1112-v11-3.pdf>
- W. Kester. *MT-022: ADC Architectures III: Sigma-Delta ADC Basics*. Analog Devices, Rev. A, octubre de 2008. <https://www.analog.com/media/en/training-seminars/tutorials/MT-022.pdf>
- STMicroelectronics. *AN5027: Interfacing PDM digital microphones using STM32 MCUs and MPUs*. Rev. 2, julio de 2019. <https://www.st.com/resource/en/application_note/an5027-interfacing-pdm-digital-microphones-using-stm32-mcus-and-mpus-stmicroelectronics.pdf>
- Infineon Technologies. *IM68D128BV01: XENSIV MEMS microphone, datasheet*. V1.1, 14 de abril de 2025. <https://www.mouser.com/catalog/specsheets/Infineon_04-30-2025_DS_IM68D128BV01_V1.1.pdf>
