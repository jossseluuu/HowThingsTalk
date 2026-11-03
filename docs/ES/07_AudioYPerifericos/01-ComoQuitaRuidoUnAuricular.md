---
title: "¿Cómo puede un auricular quitar ruido emitiendo más sonido?"
section: "07_AudioYPerifericos"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "Un auricular con cancelación activa de ruido mide el ruido con micrófonos y emite por su altavoz una onda de la misma forma y signo contrario, de modo que las dos se restan dentro del oído. Solo funciona bien en graves: allí las ondas son largas y lentas, y el error de tiempo y de posición pesa poco. Los agudos los frenan las almohadillas, no la electrónica."
prerequisites:
  - "Qué son la frecuencia y la longitud de onda (repasadas en el artículo 00·01)"
  - "Decibelios como relación entre dos potencias"
translation: "../../EN/07_AudioAndPeripherals/01-HowHeadphonesCancelNoise.md"
---

# ¿Cómo puede un auricular quitar ruido emitiendo más sonido?

## Pregunta y encuadre

Al activar la cancelación de ruido, el zumbido del avión o del autobús casi desaparece. Pero las voces de alrededor se siguen oyendo. Parece una contradicción: el auricular no tapa más el oído, sino que añade sonido por su altavoz. ¿Cómo puede sumar sonido y que el resultado sea más silencio? ¿Y por qué funciona con unos ruidos y con otros no?

Este artículo explica el principio físico de la cancelación activa de ruido (ANC, *active noise control*) y las tres arquitecturas que usan los auriculares: prealimentada, realimentada e híbrida. También explica por qué el efecto se concentra en las frecuencias graves.

- **Profundidad:** intermedia. Aparece algo de trigonometría con números, sin derivaciones.
- **Conocimientos previos:** frecuencia y longitud de onda, que se repasan en [¿Por qué una señal de radio de más frecuencia llega menos lejos?](../00_Fundamentos/01-PorQueMasFrecuenciaLlegaMenosLejos.md). La relación λ = c/f vale igual para el sonido, cambiando la velocidad de la luz por la del sonido. Decibelios como relación entre dos potencias.
- **Fuera de alcance:** la reducción de ruido del micrófono en llamadas, el modo transparencia, la cancelación en coches y salas, y la comparación entre modelos comerciales.

## Respuesta corta

El auricular mide el ruido y emite una copia invertida que se resta de él dentro del oído; la resta solo sale bien cuando la onda es lo bastante larga y lenta como para que la copia llegue a tiempo y en el sitio correcto, y eso pasa en los graves.

### Resumen de 60 segundos

El sonido es una variación de presión del aire. Dos variaciones que coinciden en el mismo punto se suman. Si una empuja cuando la otra tira, con la misma fuerza, el resultado es casi cero. Esto es la interferencia destructiva, y Paul Lueg la patentó para silenciar ruido en 1936.

Un auricular con ANC lleva micrófonos y un procesador. Un micrófono exterior oye el ruido antes de que entre. Otro, interior, oye lo que queda junto al tímpano. El procesador calcula la onda que el altavoz debe emitir para anular ese resto y la ajusta sin parar.

La resta necesita precisión. Un desfase de solo 18° deja la atenuación en 10 dB. A 100 Hz, 18° son medio milisegundo de margen. A 1 kHz, solo 50 microsegundos. Por eso la electrónica gana en graves y pierde en agudos. Los agudos los frena el material de las almohadillas, que aísla mejor cuanto más alta es la frecuencia.

## Modelo mental

Piensa en un columpio que alguien empuja a ritmo constante. Para frenarlo, tú empujas en sentido contrario y con la misma fuerza, justo en cada vaivén. Si el columpio va lento, es fácil acertar el momento. Si oscilara diez veces por segundo, llegarías tarde a cada empujón. Acabarías empujando a destiempo y lo moverías todavía más.

El columpio es la presión del aire junto al tímpano. Quien lo empuja es el ruido. Tú eres el altavoz del auricular. Llegar tarde es el retardo de los micrófonos, el procesador y el propio altavoz.

**Dónde se rompe la analogía.** El columpio está en un solo sitio. El sonido no: la presión es distinta en cada punto del espacio. La onda contraria solo anula el ruido en una zona pequeña alrededor del punto donde se ajusta. Fuera de esa zona, las dos ondas pueden incluso sumarse. Además, el ruido real no es un vaivén regular: mezcla muchas frecuencias a la vez, y hay que acertar con todas.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

El ciclo de un auricular con ANC tiene cinco pasos, que se repiten miles de veces por segundo:

1. **El ruido llega al auricular.** Una parte atraviesa la carcasa y las almohadillas. Por el camino se atenúa y se deforma. Ese recorrido, desde fuera hasta el oído, se llama *camino primario*.
2. **Los micrófonos lo miden.** Un micrófono de referencia, en el exterior, capta el ruido antes de que entre. Un micrófono de error, dentro de la cavidad, capta lo que de verdad llega cerca del oído.
3. **El procesador calcula la antionda.** Un filtro digital transforma la señal de los micrófonos en la señal que debe ir al altavoz.
4. **El altavoz la emite.** La onda emitida recorre su propio camino hasta el micrófono de error. Ese recorrido se llama *camino secundario*.
5. **El resto se mide y el filtro se corrige.** El micrófono de error oye la suma del ruido y la antionda. Un algoritmo adaptativo ajusta el filtro para que esa suma sea lo más pequeña posible.

### 2. Qué hace cada componente

**La superposición.** En el aire, a niveles normales, las presiones de dos ondas en un punto se suman. Si el ruido vale p y la antionda vale −p, el resultado es cero. En la práctica, la antionda nunca es exacta. Si tiene la amplitud correcta pero llega con un desfase φ, el resto vale 2·sen(φ/2) veces el ruido original. Con amplitud perfecta y fase perfecta, la cancelación sería total. Cualquier error deja un resto.

**El camino secundario.** Entre la señal que calcula el procesador y lo que oye el micrófono de error hay mucho: el amplificador, el altavoz, el aire de la cavidad y el propio micrófono. Cada uno retrasa y colorea la señal. Rivera Benois, Nowak y Zölzer definen este camino como las características del altavoz y del micrófono de error, más el trayecto acústico entre ellos. El procesador necesita un modelo de él para saber qué sonará realmente cuando mande una señal.

**El algoritmo FxLMS.** Es el algoritmo adaptativo más usado en ANC, según la revisión de Lu y coautores (2021). Ajusta los coeficientes del filtro paso a paso, en la dirección que reduce el error. Su particularidad es que filtra la señal de referencia con el modelo del camino secundario antes de usarla. De ahí la «x» (*filtered-x*). Sin ese paso, el algoritmo corregiría pensando que su salida llega al oído al instante, y se equivocaría de dirección. Si el modelo del camino secundario difiere del real, la convergencia se frena y el sistema puede volverse inestable.

**Las tres arquitecturas.**

- **Prealimentada (*feedforward*).** Usa el micrófono exterior. Tiene una ventaja: oye el ruido antes que el oído. Tiene una condición estricta: la causalidad. El retardo del procesador más el del camino secundario debe ser menor o igual que el retardo del camino primario (Rivera Benois *et al.*). Si la antionda llega después que el ruido, ya no puede anularlo. Además, solo cancela el ruido que el micrófono exterior capta igual que el oído.
- **Realimentada (*feedback*).** Usa solo el micrófono interior. No necesita conocer el ruido de antemano: corrige lo que oye. Su límite es el retardo del lazo. Según Rivera Benois *et al.*, ese retardo limita el ancho de banda en el que atenúa. Por eso las primeras versiones eran analógicas: así evitaban el retardo de las conversiones analógico-digital y digital-analógica.
- **Híbrida.** Suma las salidas de las dos anteriores. La prealimentada se encarga del ruido que puede anticipar. La realimentada recoge lo que queda.

**El efecto cama de agua.** Un sistema realimentado no puede atenuar en todas las frecuencias a la vez. Si reduce el ruido en una zona del espectro, lo aumenta en otra (Lu *et al.*, 2021). El diseño elige dónde concentrar la atenuación y dónde aceptar un pequeño aumento.

**El aislamiento pasivo.** La carcasa, la espuma y el ajuste al oído frenan el sonido sin electrónica. Según Rivera Benois, Papantoni y Zölzer (2018), esta atenuación pasiva es eficaz en frecuencias medias y altas. La baja frecuencia se trata con ANC. Las dos cosas se complementan.

### 3. Parámetros que importan

**Precisión necesaria.** Cálculo propio a partir de la fórmula del resto, 2·sen(φ/2), con amplitud exacta y un solo tono:

| Atenuación buscada | Desfase máximo | Margen de tiempo a 100 Hz | Margen de tiempo a 1 kHz |
|---|---|---|---|
| 10 dB | 18,2° | 505 µs | 51 µs |
| 20 dB | 5,7° | 159 µs | 16 µs |
| 30 dB | 1,8° | 50 µs | 5 µs |

Un mismo error de tiempo es un desfase mayor cuanto más alta es la frecuencia. Ejemplo: con una frecuencia de muestreo de 48 kHz, una muestra dura 20,8 µs. Un error de una sola muestra es un desfase de 0,75° a 100 Hz y de 7,5° a 1 kHz. Con solo ese error, el techo de atenuación sería de unos 38 dB a 100 Hz y de unos 18 dB a 1 kHz.

**Tamaño de la zona de silencio.** Imagina un campo sonoro difuso en el que se anula la presión en un punto. La zona donde el ruido baja al menos 10 dB mide alrededor de una décima de longitud de onda de diámetro (Elliott *et al.*, 1988). Rafaely lo precisa en 0,088λ para un tono de 300 Hz. Con una velocidad del sonido de unos 343 m/s en aire a 20 °C, eso da:

| Frecuencia | Longitud de onda | Zona de −10 dB (≈ λ/10) |
|---|---|---|
| 100 Hz | 3,43 m | 34 cm |
| 300 Hz | 1,14 m | 11 cm |
| 1 kHz | 34 cm | 3,4 cm |
| 3 kHz | 11 cm | 1,1 cm |

Estas cifras son de un campo libre y difuso, no de una cavidad cerrada. Sirven para la idea: a 100 Hz la zona abarca toda la oreja con margen. A 3 kHz es más pequeña que el conducto auditivo, y la posición del micrófono de error respecto al tímpano ya importa.

**Un ejemplo medido.** Rivera Benois, Papantoni y Zölzer (2018) midieron un prototipo de auricular sobre una cabeza artificial. Usaron ruido blanco de banda ancha modulado en amplitud a 70 Hz y muestrearon a 48 kHz. Los niveles totales en el oído fueron:

| Situación | Nivel en el oído |
|---|---|
| Sin auricular | 82,3 dB SPL |
| Auricular puesto, ANC apagada (solo aislamiento pasivo) | 65,3 dB SPL |
| ANC prealimentada clásica | 61,6 dB SPL |
| ANC híbrida (con ponderación psicoacústica) | 60,1 dB SPL |

En ese ensayo, la parte pasiva aporta unos 17 dB y la activa unos 5 dB más en nivel total. Las cifras totales esconden el reparto por frecuencias. El controlador prealimentado atenuaba aproximadamente entre 60 Hz y 7,5 kHz, y amplificaba por debajo de 60 Hz. Los tres controladores mostraban un pico de amplificación hacia 6,3 kHz. El controlador realimentado del sistema híbrido atenúa con eficacia hasta 1 kHz. Son datos de un prototipo de investigación, no de un producto comercial.

### 4. Diagrama o flujo

```text
                     CAMINO PRIMARIO (carcasa + almohadilla)
RUIDO EXTERIOR ──────────────────────────────────────────────┐
     │                                                       ▼
     ▼                                               ┌───────────────┐
 MIC. DE REFERENCIA ──► FILTRO W_ff ──┐              │  CAVIDAD DEL  │
 (fuera)                              │              │     OÍDO      │
                                      ▼              │               │
                                    ( + ) ──► ALTAVOZ ──► antionda   │
                                      ▲    CAMINO SECUNDARIO         │
                                      │              │  ruido + anti │
                         FILTRO W_fb ◄┴── MIC. DE ERROR ◄────────────┘
                              ▲            (dentro)        │
                              │                            │
                     ALGORITMO FxLMS ◄── usa el modelo Ŝ ◄─┘
                     (ajusta W_ff y W_fb para minimizar el resto)

Condición prealimentada: retardo(W_ff) + retardo(secundario) ≤ retardo(primario)
```

## Límites, excepciones y casos reales

- **El ajuste al oído cambia los dos caminos.** Si la almohadilla no sella bien, entra más ruido por fugas y el camino secundario cambia. El modelo Ŝ deja de coincidir con la realidad, y la cancelación empeora. Por eso los sistemas adaptativos vuelven a ajustarse con el uso.
- **El ruido que no se puede anticipar.** La parte prealimentada solo cancela lo que su micrófono exterior capta de forma parecida a como llega al oído. El ruido de un golpe seco o el del viento sobre el propio micrófono no cumplen esa condición.
- **La causalidad pone un límite duro.** El sonido recorre 1 cm en unos 29 µs. Si el micrófono exterior está, por ejemplo, a 2 cm del altavoz, el ruido tarda unos 58 µs en cruzar esa distancia. Eso son menos de tres muestras a 48 kHz para medir, calcular y emitir. La distancia es un supuesto ilustrativo, no un dato de ningún modelo.
- **Algo de amplificación es inevitable.** El efecto cama de agua obliga a pagar en alguna frecuencia lo que se gana en otra. En el prototipo medido, el pago estaba por debajo de 60 Hz y hacia 6,3 kHz.
- **La zona de silencio es local.** La cancelación se optimiza en el micrófono de error, no en el tímpano. En graves la diferencia apenas importa. En frecuencias medias y altas, los pocos centímetros entre los dos puntos ya son una fracción apreciable de la longitud de onda.

## Consecuencias prácticas

- **Zumbidos sí, voces menos.** El ruido de motores, aviones y aire acondicionado concentra mucha energía en graves y es bastante estable. Es justo lo que la ANC cancela mejor. La voz cambia deprisa y gran parte de lo que la hace inteligible está en frecuencias medias, por encima de la zona donde la ANC rinde mejor. Lo que se reduce de la voz se debe sobre todo al aislamiento pasivo.
- **El aislamiento pasivo sigue siendo la mitad del trabajo.** En el ensayo citado, la carcasa y las almohadillas aportaban más reducción total que la electrónica. Un auricular mal ajustado pierde las dos cosas a la vez.
- **Para quien diseña.** Cada microsegundo de retardo que se ahorra en conversores, filtros y altavoz amplía el rango de frecuencias que se puede cancelar. Por eso los primeros controladores realimentados eran analógicos: evitaban el retardo de las conversiones (Rivera Benois *et al.*).

## Preguntas frecuentes

### ¿Es lo mismo que la reducción de ruido del micrófono en llamadas?

No. La reducción de ruido en llamadas limpia la señal del micrófono antes de enviarla a la otra persona. No cambia lo que oyes tú. La ANC actúa sobre el sonido que llega a tu oído y emite una onda real por el altavoz.

### ¿El auricular «absorbe» el ruido?

No. El ruido sigue llegando. Lo que hace el altavoz es crear una segunda presión que se le resta dentro de la cavidad. La energía del ruido no se destruye: en la zona de silencio las dos ondas se compensan, y fuera de ella se reparten de otra forma.

### ¿Por qué no cancelan también los agudos con un procesador más rápido?

La velocidad del procesador es solo una parte del retardo. También cuentan el altavoz, los micrófonos y el recorrido del sonido. Y aunque el retardo fuera cero, la zona de silencio a varios kilohercios mide pocos milímetros. Tendría que coincidir con el tímpano, que se mueve con cada oreja y cada colocación. Para los agudos, el aislamiento pasivo es más eficaz y más barato.

### ¿Es una idea nueva?

No. Paul Lueg patentó en 1936 la idea de captar un sonido con un micrófono y reproducirlo con fase opuesta por un altavoz. Lo que ha cambiado es el procesado digital adaptativo, que permite ajustar la antionda en tiempo real.

## Fuentes

- P. Lueg. *Process of silencing sound oscillations*. Patente US 2.043.416, solicitada el 8 de marzo de 1934 y concedida el 9 de junio de 1936. <https://patents.google.com/patent/US2043416A/en>
- S. M. Kuo y D. R. Morgan. «Active Noise Control: A Tutorial Review». *Proceedings of the IEEE*, vol. 87, n.º 6, 1999. DOI: [10.1109/5.763310](https://doi.org/10.1109/5.763310). Ficha: <https://nokia.com/bell-labs/publications-and-media/publications/active-noise-control-a-tutorial-review>
- L. Lu, K.-L. Yin, R. C. de Lamare, Z. Zheng, Y. Yu, X. Yang y B. Chen. «A survey on active noise control techniques, Part I: Linear systems». Preprint arXiv:2110.00531, 2021. <https://arxiv.org/abs/2110.00531>
- P. Rivera Benois, P. Nowak y U. Zölzer. «Hybrid Active Noise Control Structures: A Short Overview». Helmut-Schmidt-Universität, Hamburgo. <https://www.hsu-hh.de/ant/wp-content/uploads/sites/699/2019/01/Hybrid-Active-Noise-Control-Structures_-A-Short-Overview-Rivera-Nowak-Zoelzer.pdf>
- P. Rivera Benois, V. Papantoni y U. Zölzer. «Psychoacoustic Hybrid Active Noise Control Structure for Application in Headphones». *25th International Congress on Sound and Vibration (ICSV25)*, Hiroshima, 2018. <https://www.hsu-hh.de/ant/wp-content/uploads/sites/699/2018/08/ICSV25_full_paper_Rivera_Papantoni_Zoelzer.pdf>
- S. J. Elliott, P. Joseph y P. A. Nelson. «Active control in diffuse sound fields». *Proceedings of the Institute of Acoustics*, vol. 10, parte 2, pp. 605-614, 1988. <https://www.ioa.org.uk/system/files/publications/SJ%20ELLIOTT%20P%20JOSEPH%20PA%20NELSON%20ACTIVE%20CONTROL%20IN%20DIFFUSE%20SOUND%20FIELDS.pdf>
- B. Rafaely. «Zones of quiet in a broadband diffuse sound field». arXiv:2310.04191, 2023. <https://arxiv.org/abs/2310.04191>
