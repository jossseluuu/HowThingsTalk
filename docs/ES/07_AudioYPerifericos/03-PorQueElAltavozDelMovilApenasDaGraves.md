---
title: "¿Por qué el altavoz del móvil apenas da graves?"
section: "07_AudioYPerifericos"
language: "es"
status: "published"
order: 3
depth: "intermediate"
summary: "El sonido que llega a tu oído depende de cuánto aire empuja el altavoz y de lo deprisa que cambia ese empuje. Cada octava hacia los graves exige mover la membrana cuatro veces más para sonar igual de fuerte. Una membrana de un centímetro cuadrado que solo puede desplazarse unas décimas de milímetro se queda sin recorrido mucho antes de llegar a los graves. Además, su resonancia ronda 1 kHz y por debajo de ella la respuesta cae en picado."
prerequisites:
  - "Qué es una onda de sonido: una variación de presión del aire"
  - "Qué son la frecuencia y la octava (doblar o dividir por dos la frecuencia)"
  - "Decibelios como relación entre dos magnitudes"
translation: "../../EN/07_AudioAndPeripherals/03-WhyPhoneSpeakersHaveLittleBass.md"
---

# ¿Por qué el altavoz del móvil apenas da graves?

## Pregunta y encuadre

Pones música en el altavoz del móvil y suena «a lata». Las voces se entienden bien. El bombo y el bajo casi desaparecen. ¿Es un problema de calidad o hay algo físico que lo impide?

Este artículo explica por qué un altavoz del tamaño de una uña no puede reproducir graves con fuerza. Cubre tres cosas: cómo convierte la corriente en movimiento, cuánto aire hay que mover para producir un sonido grave y qué limita ese movimiento. Termina con lo que hacen los amplificadores «inteligentes» para sacar un poco más.

- **Profundidad:** intermedia. Hay dos fórmulas sencillas y cálculos propios con números, sin derivaciones largas.
- **Conocimientos previos:** saber que el sonido es una variación de presión, qué es una octava y manejar decibelios. El artículo [¿Cómo convierte el micrófono del móvil el sonido en números?](02-ComoConvierteElMicrofonoElSonidoEnNumeros.md) recorre el camino inverso y ayuda, pero no es imprescindible.
- **Fuera de alcance:** los convertidores digital-analógicos y la etapa de amplificación en detalle, el audio por Bluetooth, los trucos psicoacústicos para simular graves, los auriculares y la comparación entre modelos comerciales.

## Respuesta corta

Para producir la misma presión sonora, cada vez que bajas una octava la membrana tiene que desplazarse cuatro veces más, y una membrana de un centímetro cuadrado que solo admite unas décimas de milímetro de recorrido se queda sin margen por debajo de 1 kHz, que es justo donde tiene su resonancia.

### Resumen de 60 segundos

Un altavoz es un pistón. Una bobina dentro de un imán recibe la corriente y empuja una membrana. La membrana empuja el aire.

Lo que oyes a cierta distancia no depende de cuánto se mueve la membrana, sino de la aceleración del volumen de aire que desplaza. Si la frecuencia baja a la mitad, el mismo recorrido produce una cuarta parte de aceleración. Para compensarlo hay que multiplicar el recorrido por cuatro en cada octava.

Un altavoz de móvil tiene una membrana de alrededor de 1 cm². Su recorrido máximo es de unas décimas de milímetro: 0,4 mm en los modelos de un estudio de 2020. Con números redondos, un tono de 1 kHz a 80 dB a medio metro pide unos 0,4 mm. A 250 Hz pediría 6 mm. A 100 Hz, casi 4 cm.

Hay un segundo freno. La membrana y su suspensión forman un muelle con masa, con una resonancia cerca de 1 kHz. Por debajo de ella, el desplazamiento deja de crecer al bajar la frecuencia y la presión cae unos 12 dB por octava.

Los amplificadores inteligentes vigilan la temperatura de la bobina y el recorrido de la membrana en tiempo real. Así pueden empujar el altavoz hasta su límite real sin romperlo. Ganan algunos decibelios. No cambian la física.

## Modelo mental

Imagina que quieres hacer olas en una piscina con una tabla de la mano. Si mueves la tabla adelante y atrás muy deprisa, unos centímetros bastan para crear ondas pequeñas y rápidas. Si quieres olas lentas y largas, tienes que empujar mucha más agua en cada vaivén. Con una tabla pequeña, eso significa recorrer mucha más distancia.

Ahora imagina que la tabla está atada con un elástico corto. Puede vibrar rápido. Pero en cuanto intentas un vaivén lento y amplio, el elástico se tensa y no te deja.

El altavoz del móvil es esa tabla pequeña con elástico corto. Para los agudos le sobra. Para los graves necesitaría un recorrido que no tiene.

**Dónde se rompe la analogía.** En la piscina, el agua apenas se comprime y las olas viajan por la superficie. El sonido es una onda de presión que se propaga en las tres dimensiones del aire. Lo que cuenta no es el volumen de agua movido, sino lo deprisa que cambia el caudal de aire: su aceleración. Por eso la penalización en graves es tan dura, cuatro veces más recorrido por octava, y no solo el doble. El elástico tampoco explica la resonancia: la masa de la membrana y la rigidez de su suspensión forman un sistema que vibra con más facilidad a una frecuencia concreta.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

El camino de la señal hasta el aire tiene cinco pasos:

1. **El procesador entrega muestras de audio.** Un convertidor digital-analógico las transforma en una tensión.
2. **El amplificador aplica esa tensión al altavoz.** En los móviles suele ser un amplificador de clase D, con un elevador de tensión porque la batería da pocos voltios (TI, SSZT848).
3. **La corriente crea una fuerza.** La bobina está dentro del campo de un imán. La corriente que la recorre produce una fuerza F = Bl · i (TI, SLAA857).
4. **La membrana se mueve.** La bobina está pegada a la membrana. Su movimiento depende de la fuerza, de su masa, del rozamiento y de la rigidez de la suspensión.
5. **La membrana empuja el aire.** El caudal de aire que desplaza genera la onda de presión que llega al oído.

### 2. Qué hace cada componente

**El motor: imán y bobina.** El altavoz del estudio de Park, Jiang y Hwang (2020) mide 16,0 × 10,0 × 2,11 mm y tiene una bobina de 6,0 Ω. Lleva tres imanes de neodimio, una membrana central y otra lateral que hace de suspensión. El producto Bl, llamado factor de fuerza, mide cuántos newtons da el motor por cada amperio. En un altavoz de este tamaño es pequeño. La guía de Klippel sobre microaltavoces lo sitúa por debajo de 1 N/A. El altavoz de un iPhone 7 que TI usa como ejemplo tiene Bl = 0,841 T·m (SLAA857). Park *et al.* lo explican así: reducir el tamaño obliga a reducir el circuito magnético y eso baja la fuerza en todas las frecuencias.

**La membrana y la suspensión.** Son la parte móvil. En un microaltavoz no hay araña, la pieza ondulada que centra la bobina en un altavoz grande. El borde de la propia membrana hace de suspensión (Klippel). Klippel describe la superficie que radia como de alrededor de 1 cm de diámetro.

**El sistema masa-muelle.** Park *et al.* escriben la ecuación del movimiento de la membrana:

M · z'' + C · z' + K · z = F(t)

Aquí z es el desplazamiento, M la masa móvil, C el rozamiento mecánico y K la rigidez de la suspensión. Es el mismo sistema que un peso colgado de un muelle. Tiene una frecuencia de resonancia:

f₀ = 1 / (2π · √(M_ms · C_ms))

donde C_ms es la elasticidad, la inversa de la rigidez (SLAA857). Con los valores del altavoz del iPhone 7, M_ms = 0,116 g y C_ms = 0,21 mm/N, sale f₀ = 1020 Hz. Los microaltavoces de un estudio de OPPO y Realme (Peng *et al.*, 2020) tienen su resonancia en 700 ± 100 Hz. Klippel da un rango general de 200 Hz a 4 kHz.

**La caja trasera.** El altavoz se monta en una pequeña cavidad cerrada. Sin ella, la onda que sale por detrás de la membrana tendría signo contrario y anularía a la de delante en graves. La nota de Analog Devices lo menciona como la razón de encerrar el altavoz. El precio es que el aire de la cavidad actúa como un muelle más. Según Klippel, el aire añade masa a la membrana y, dentro de una caja, añade rigidez. Más rigidez significa una resonancia más alta. En un móvil, esa cavidad es de unas décimas de centímetro cúbico: Park *et al.* miden sus prototipos con 0,6 cm³.

**De la membrana al oído.** Aquí está la clave. Garrett (*Understanding Acoustics*, 2020) muestra que la presión radiada por una fuente pequeña depende de su caudal de aire, que llama velocidad de volumen U. En campo lejano, la amplitud de la presión a una distancia r vale (ec. 12.21):

|p| = ρ · c · k · |U| / (4π · r)

Como k = ω / c, eso es ρ · ω · |U| / (4π · r). Y el caudal es la superficie de la membrana por su velocidad: U = S_d · ω · X, donde X es la amplitud del desplazamiento. Sustituyendo (cálculo propio):

|p| = ρ · ω² · S_d · X / (4π · r)

La presión sube con el **cuadrado** de la frecuencia. A la mitad de frecuencia, el mismo desplazamiento da una cuarta parte de presión: 12 dB menos. Para mantener el nivel, X tiene que multiplicarse por cuatro en cada octava. Garrett resume la consecuencia en su capítulo: es más difícil radiar frecuencias bajas.

**Por qué la respuesta cae por debajo de la resonancia.** Juntando las dos piezas sale la forma de la respuesta (razonamiento propio a partir de las ecuaciones de Park *et al.* y Garrett):

- **Por encima de f₀** manda la masa. El desplazamiento cae con 1/f² al subir la frecuencia, y la presión, que sube con f², queda plana. Es la zona útil.
- **Por debajo de f₀** manda la rigidez. El desplazamiento por voltio se vuelve constante. La presión cae entonces con f²: unos 12 dB por octava.

En un altavoz con f₀ = 1020 Hz, un tono de 250 Hz queda dos octavas por debajo. Con la misma tensión, sonaría del orden de 24 dB más bajo que en la zona plana (cálculo propio, ignorando el efecto del factor de calidad cerca de la resonancia).

### 3. Parámetros que importan

**El recorrido que haría falta.** Cálculo propio con la fórmula anterior. Supuestos: membrana de 1 cm², tono puro de 80 dB SPL (0,2 Pa eficaces) medido a 0,5 m, en campo libre y aire a 1,2 kg/m³:

| Frecuencia | Desplazamiento de pico necesario |
|---|---|
| 4000 Hz | 0,023 mm |
| 2000 Hz | 0,094 mm |
| 1000 Hz | 0,38 mm |
| 500 Hz | 1,5 mm |
| 250 Hz | 6,0 mm |
| 100 Hz | 37,5 mm |

**El recorrido disponible.** Los microaltavoces de Peng *et al.* admiten X_max = 0,4 mm. Con ese límite, el ejemplo de la tabla llega a 1 kHz con lo justo y no puede ir mucho más abajo.

**La comparación con un altavoz grande.** Un altavoz de graves doméstico con una membrana de unos 130 cm² (valor ilustrativo) necesitaría 0,29 mm a 100 Hz para el mismo nivel (cálculo propio). Es 130 veces menos recorrido porque mueve 130 veces más superficie. Lo que importa es el producto S_d · X: el volumen de aire desplazado.

**Un altavoz concreto.** Las fichas de dos casos publicados:

| Parámetro | Altavoz del iPhone 7 (TI, SLAA857) | Microaltavoz de Peng *et al.* (2020) |
|---|---|---|
| Resistencia de la bobina | 7,35 Ω | 7 Ω ±15 % a 2 kHz |
| Factor de fuerza Bl | 0,841 T·m | no publicado |
| Masa móvil M_ms | 0,116 g | no publicada |
| Elasticidad C_ms | 0,21 mm/N | no publicada |
| Resonancia | 1020 Hz | 700 ± 100 Hz |
| Recorrido máximo | no publicado | 0,4 mm |
| Potencia nominal | no publicada | 1,13 W (1,7 W en picos breves) |
| Temperatura máxima | no publicada | 100 °C |
| Sensibilidad | no publicada | 93 dB a 2 kHz y > 84 dB a 500 Hz (2,83 V, 10 cm, con pantalla) |

La sensibilidad de Peng *et al.* ya muestra la caída: al menos 9 dB menos a 500 Hz que a 2 kHz con la misma tensión.

**El límite térmico.** Casi toda la potencia eléctrica acaba en calor en la bobina: P = i² · R_e (SLAA857). TI modela la temperatura con dos nodos. La bobina tiene una constante de tiempo de unos 4,2 s y el imán de unos 412 s. La bobina se calienta en segundos; el imán, en minutos. Peng *et al.* midieron que, a su potencia nominal, la bobina pasaba de 100 °C en un plazo de entre 20 y 70 segundos.

### 4. Diagrama o flujo

```text
 MUESTRAS DE AUDIO ─► DAC ─► AMPLIFICADOR CLASE D (con elevador)
                                   │ tensión u      ▲ medida de tensión y corriente
                                   ▼                │ (amplificador inteligente)
 ┌──────────────── MICROALTAVOZ ≈ 16 × 10 × 2 mm ───┴─────────────────┐
 │  BOBINA en el campo del IMÁN:  F = Bl · i     (Bl < 1 N/A)         │
 │        │                                                           │
 │        ▼                                                           │
 │  MEMBRANA ~1 cm² + SUSPENSIÓN:  M·z'' + C·z' + K·z = F             │
 │        │                       resonancia f₀ ≈ 0,7-1 kHz           │
 │        │  recorrido máximo X_max ≈ 0,4 mm                          │
 │  CAVIDAD TRASERA ~0,6 cm³ (más rigidez → f₀ más alta)              │
 └────────┬───────────────────────────────────────────────────────────┘
          │ caudal de aire U = S_d · ω · X
          ▼
  PRESIÓN EN EL OÍDO:  |p| = ρ · ω² · S_d · X / (4π · r)
          │
          ├─ por encima de f₀: presión plana   (zona útil)
          └─ por debajo de f₀: cae ~12 dB por octava
             y cada octava pediría ×4 de recorrido para compensar
```

## Límites, excepciones y casos reales

- **La resonancia no es exacta.** Peng *et al.* dan una tolerancia de ±100 Hz sobre 700 Hz entre unidades del mismo modelo. Dos móviles iguales no suenan exactamente igual en graves.
- **El modelo lineal se queda corto.** Klippel señala que la mayoría de los microaltavoces muestran fluencia (*creep*): la suspensión cede más a frecuencias bajas de lo que predice el modelo simple, y el desplazamiento real es mayor. Bl, la rigidez y la inductancia cambian además con la posición de la membrana. Cerca del recorrido máximo aparecen distorsiones.
- **Pasarse de recorrido rompe el altavoz.** La nota de Analog Devices advierte que forzar graves por debajo de la resonancia puede llevar la membrana más allá de su límite y destruirla.
- **El calor limita antes que la potencia nominal.** La potencia que haría falta para sonar fuerte suele superar la potencia continua que aguanta un altavoz pequeño (TI, SSZT848). El margen existe solo durante unos segundos, por la constante de tiempo térmica de la bobina.
- **La temperatura ambiente cuenta.** Un móvil al sol ya parte de una bobina caliente. Peng *et al.* calibraron su estimación de temperatura con el ambiente entre 25 y 95 °C.
- **La caja manda tanto como el altavoz.** El mismo altavoz dará otra respuesta en otra cavidad. El volumen trasero, los conductos y la rejilla de salida del móvil forman parte del sistema acústico.

## Consecuencias prácticas

- **Los amplificadores inteligentes exprimen el margen.** En lugar de limitar la tensión con mucho colchón, miden la tensión y la corriente en la salida (*IV sense*). Con eso calculan la resistencia de la bobina y, de ella, su temperatura, porque la resistencia del cobre sube al calentarse (SSZT848, SLAA857). TI usa un tono piloto inaudible de baja frecuencia, por ejemplo 60 Hz, porque ahí la fuerza contraelectromotriz casi desaparece y la medida da la resistencia limpia. Un modelo del altavoz, ajustado en tiempo real, predice el desplazamiento antes de que ocurra. Si va a superar X_max, la señal se atenúa a tiempo gracias a un pequeño retardo (SLAA857).
- **Ganan decibelios, no milagros.** Analog Devices afirma que su gestión dinámica de altavoz sube el nivel entre 6 y 8 dB y extiende los graves hasta una cuarta parte de la frecuencia de resonancia. Son cifras del fabricante, sin datos de medida publicados en la nota.
- **Por eso el volumen máximo depende de la música.** Si una canción tiene mucho bajo, el protector tiene que atenuar antes para no pasarse de recorrido (SLAA857). Una voz sola puede sonar más fuerte que un tema con mucho bombo.
- **Dos altavoces ayudan poco en graves.** Sumar superficie suma caudal: lo que cuenta es el producto S_d · X total. Pero dos membranas de 1 cm² siguen siendo unas 65 veces menos superficie que la de un altavoz doméstico de 130 cm² (cálculo propio).
- **Para quien diseña.** Cada décima de centímetro cúbico de cavidad y cada décima de milímetro de recorrido compiten con la batería y la cámara por el espacio. Park *et al.* proponen rediseñar el circuito magnético para ganar fuerza en graves sin crecer.

## Preguntas frecuentes

### ¿Si subo el volumen, aparecen los graves?

No en la misma proporción. Subir el volumen sube todas las frecuencias por igual hasta que algo llega a su límite. En graves el límite llega antes: el recorrido de la membrana. A partir de ahí, el protector recorta los graves o el sonido se distorsiona.

### ¿Un ecualizador puede compensar la falta de graves?

Solo un poco. Un ecualizador sube la tensión en graves, y eso pide más recorrido y más calor. Por debajo de la resonancia, cada 12 dB de refuerzo cuadruplican el desplazamiento. El límite de 0,4 mm se alcanza enseguida. Es exactamente lo que vigila el amplificador inteligente.

### ¿Es lo mismo la resonancia que la frecuencia más grave que se oye?

No. La resonancia es donde el sistema masa-muelle se mueve con más facilidad. Por debajo de ella el altavoz sigue sonando, pero cada vez menos: unos 12 dB por octava. Los microaltavoces de Peng *et al.* resuenan en torno a 700 Hz y aun así dan más de 84 dB a 500 Hz en su prueba de referencia.

### ¿Por qué entonces se oye el bajo de una canción, aunque sea flojo?

Porque las notas graves de un instrumento real tienen armónicos: componentes a dos, tres o más veces su frecuencia. El altavoz reproduce bien esos armónicos, que caen en su zona útil. Lo que se pierde es sobre todo la fundamental y la sensación de cuerpo. Los procesados que refuerzan esos armónicos para simular graves quedan fuera del alcance de este artículo.

## Fuentes

- S. L. Garrett. *Understanding Acoustics: An Experimentalist's View of Sound and Vibration*, cap. 12 «Radiation and Scattering». Graduate Texts in Physics, Springer, 2020. DOI: [10.1007/978-3-030-44787-8_12](https://doi.org/10.1007/978-3-030-44787-8_12). <https://link.springer.com/chapter/10.1007/978-3-030-44787-8_12>
- S. Xue, S. Palit, P. Wei y A. Du. *SLAA857: SmartPA Speaker Protection Algorithm*. Texas Instruments, agosto de 2018. <https://www.ti.com/lit/pdf/slaa857>
- M. Kucic. *SSZT848: Don't Limit Your Audio: How to Achieve Loud Sound from a Small Speaker*. Texas Instruments, diciembre de 2017. <https://www.ti.com/lit/pdf/sszt848>
- K.-H. Park, Z.-X. Jiang y S.-M. Hwang. «Design and Analysis of a Novel Microspeaker with Enhanced Low-Frequency SPL and Size Reduction». *Applied Sciences*, vol. 10, n.º 24, 8902, 2020. DOI: [10.3390/app10248902](https://doi.org/10.3390/app10248902). <https://www.mdpi.com/2076-3417/10/24/8902>
- C. Peng, Y. Shi, B. Yan, L. Wu, Z. Chen, C. D. Salvador y D. Liu. «Power-based thermal limits for micro-speaker protection algorithms». Audio Engineering Society, 148.ª Convención, e-Brief 594, junio de 2020. <https://cesardsalvador.github.io/doc/Peng2020PowerBasedThermalLimitsForMicrospeakers.pdf>
- Klippel GmbH. *Hands-On Training 7: Microspeaker*. <https://www.klippel.de/training/attachments/training7/Training_7_Microspeaker_en.pdf>
- Analog Devices (Maxim Integrated). «Unleash the Full Potential of Your Micro Speakers». Nota técnica, 24 de septiembre de 2019. <https://www.analog.com/en/resources/technical-articles/unleash-the-full-potential-of-your-micro-speakers.html>
