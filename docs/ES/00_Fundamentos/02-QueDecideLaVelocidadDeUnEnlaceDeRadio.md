---
title: "¿Qué decide cuántos bits por segundo caben en una señal de radio?"
section: "00_Fundamentos"
language: "es"
status: "published"
order: 2
depth: "intermediate"
summary: "La velocidad máxima de un enlace de radio la fijan dos cosas: el ancho de banda del canal y la relación entre la señal y el ruido. Shannon lo resumió en C = B·log2(1 + S/N). Los aparatos reales eligen modulación y codificación según la SNR que miden y se quedan algo por debajo de ese límite."
prerequisites:
  - "Decibelios y dBm (repasados en el artículo 00·01)"
  - "Qué es el ancho de banda de un canal, medido en hercios"
translation: "../../EN/00_Foundations/02-WhatSetsTheSpeedOfARadioLink.md"
---

# ¿Qué decide cuántos bits por segundo caben en una señal de radio?

## Pregunta y encuadre

Con cobertura «a tope» el móvil descarga rápido. Con una sola barra, la misma red y el mismo plan van mucho más lentos. La pregunta es qué fija ese techo. ¿Por qué un canal de radio no admite cualquier cantidad de bits por segundo?

Este artículo explica el límite físico y matemático que comparten el WiFi, la telefonía móvil, el Bluetooth y cualquier otro enlace de radio. También muestra cómo los aparatos reales se acercan a ese límite. Usa como ejemplo las tablas de la norma 5G NR, porque son públicas.

- **Profundidad:** intermedia. Aparece una fórmula con logaritmos, explicada paso a paso.
- **Conocimientos previos:** decibelios y dBm, que se repasan en [¿Por qué una señal de radio de más frecuencia llega menos lejos?](01-PorQueMasFrecuenciaLlegaMenosLejos.md). Saber que un canal ocupa un ancho de banda medido en hercios (por ejemplo, 20 MHz).
- **Fuera de alcance:** el detalle de cada modulación, la teoría de códigos correctores, los protocolos de acceso al medio (quién habla cuándo) y la interferencia entre redes vecinas.

## Respuesta corta

El número máximo de bits por segundo lo fijan el ancho de banda del canal y la relación entre la potencia de la señal y la del ruido; ni más señal sola ni más canal solo bastan.

### Resumen de 60 segundos

Todo receptor oye ruido. Parte de ese ruido es térmico: lo producen los electrones agitados por la temperatura y está siempre presente. Su potencia crece con el ancho de banda que se escucha.

Claude Shannon demostró en 1948 cuál es la tasa máxima que un canal con ruido de este tipo puede transportar sin errores. La fórmula es C = B·log2(1 + S/N). B es el ancho de banda en hercios y S/N es la relación señal-ruido (SNR). Por debajo de C se puede transmitir con tan pocos errores como se quiera. Por encima, no.

Dos consecuencias se ven en la fórmula. Duplicar el ancho de banda casi duplica la capacidad cuando la señal es fuerte. Pero si la señal es débil, el ruido extra del canal ancho se come buena parte de la ganancia. Y con señal fuerte, cada 3 dB de SNR adicionales solo añaden 1 bit por segundo y por hercio.

Los aparatos no alcanzan el límite. Miden la SNR y eligen una modulación y un código corrector que funcionen con ella. En 5G NR la norma fija 15 combinaciones, desde 0,15 hasta 7,41 bits por segundo y por hercio.

## Modelo mental

Imagina que alguien te dicta números en una sala con ruido de fondo. Puede dictar más deprisa si la sala es más silenciosa o si habla más alto. También puede usar varias voces a la vez, cada una en un tono distinto. Si el ruido sube, tiene que hablar más despacio o repetir cosas para que no te equivoques.

Hablar más alto es subir la potencia de la señal. El ruido de la sala es el ruido del receptor. Usar más tonos es ocupar más ancho de banda. Repetir y comprobar es lo que hacen los códigos correctores.

**Dónde se rompe la analogía.** En la sala, un ruido de fondo fijo no cambia por usar más tonos. En radio sí cambia: el ruido térmico es proporcional al ancho de banda que escucha el receptor, así que cada hercio nuevo trae su propio ruido. Además, la persona que dicta no tiene un límite claro de velocidad. El canal de radio sí lo tiene, y se puede calcular con precisión.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

El camino de los bits tiene cuatro etapas:

1. **El receptor mide la calidad del canal.** Estima la potencia de la señal útil y la del ruido más la interferencia.
2. **El receptor informa al emisor.** En 5G NR lo hace con un número llamado CQI (indicador de calidad del canal), de 0 a 15 (3GPP TS 38.214, apartado 5.2.2.1).
3. **El emisor elige cómo transmitir.** Escoge una modulación (cuántos bits lleva cada símbolo) y una tasa de código (qué parte de los bits enviados es información y qué parte es redundancia para corregir errores).
4. **El receptor decodifica.** Si la SNR real era suficiente para esa combinación, recupera los bits. Si no, pide una retransmisión y el emisor baja de nivel.

Este ciclo se repite continuamente. Por eso la velocidad cambia al moverse, aunque nada más cambie.

### 2. Qué hace cada componente

**El ruido térmico.** Cualquier resistencia a una temperatura por encima del cero absoluto genera ruido. La potencia disponible es p = k·T·b, donde k es la constante de Boltzmann (1,38 × 10⁻²³ J/K), T es la temperatura en kelvin y b el ancho de banda en hercios. La Rec. ITU-R P.372-16 toma como referencia T0 = 290 K. Con ese valor, 10·log10(k·T0) = −204 dBW/Hz, es decir, −174 dBm/Hz (P.372-16, apartado 1.2.1 y ec. 6). Con 20 MHz, el ruido térmico vale −174 + 73 = −101 dBm.

**La figura de ruido del receptor.** El amplificador y el resto de la electrónica añaden su propio ruido. Se resume en una cifra en dB que se suma al ruido térmico. Su valor depende de cada diseño. En los ejemplos de este artículo se usan 7 dB como supuesto ilustrativo, no como dato de ningún aparato concreto.

**La relación señal-ruido (SNR).** Es la potencia de la señal dividida por la del ruido. En dB, es una resta: SNR = S (dBm) − N (dBm). Una señal de −70 dBm sobre un suelo de ruido de −94 dBm da 24 dB de SNR.

**El límite de Shannon.** Shannon trató un canal de ancho de banda limitado con ruido blanco gaussiano, que es el modelo del ruido térmico (Shannon, 1948, parte IV). Con potencia media de emisión limitada, la capacidad es:

```text
C = B · log2(1 + S/N)     [bit/s]
B en Hz · S/N en veces (no en dB)
```

El teorema tiene dos caras. Por debajo de C existe una forma de codificar que deja la tasa de errores tan baja como se quiera. Por encima de C, ningún código lo consigue. La fórmula no dice cómo construir ese código: solo dice que existe.

**La modulación.** Cada símbolo transmitido elige un punto de una «constelación». QPSK tiene 4 puntos y lleva 2 bits por símbolo. 16QAM lleva 4 bits, 64QAM lleva 6 y 256QAM lleva 8. Más puntos significan puntos más juntos. Con los puntos más juntos, menos ruido basta para confundir uno con otro.

**El código corrector.** Añade redundancia calculada. Una tasa de código de 948/1024 significa que el 92,6 % de los bits enviados es información. Una tasa de 78/1024 significa que solo lo es el 7,6 %; el resto sirve para corregir errores.

**La eficiencia espectral.** Es el producto de las dos cosas: bits por símbolo × tasa de código. Se mide en bit/s/Hz. Para 64QAM con tasa 948/1024 sale 6 × 948/1024 = 5,55 bit/s/Hz. Es la cifra que 3GPP publica en sus tablas.

### 3. Parámetros que importan

**Lo que dice la fórmula con números.** Escenario: suelo de ruido = ruido térmico a 290 K + 7 dB de figura de ruido supuesta. Capacidad de Shannon calculada con C = B·log2(1 + S/N).

| Señal recibida | Canal de 20 MHz (ruido −94 dBm) | Canal de 40 MHz (ruido −91 dBm) |
|---|---|---|
| −50 dBm | SNR 44 dB · 292 Mbit/s | SNR 41 dB · 545 Mbit/s |
| −70 dBm | SNR 24 dB · 160 Mbit/s | SNR 21 dB · 279 Mbit/s |
| −85 dBm | SNR 9 dB · 63 Mbit/s | SNR 6 dB · 92 Mbit/s |
| −95 dBm | SNR −1 dB · 17 Mbit/s | SNR −4 dB · 19 Mbit/s |

Tres lecturas de la tabla:

- Con señal fuerte (−50 dBm), duplicar el canal multiplica la capacidad por 1,86.
- Con señal débil (−95 dBm), duplicar el canal solo la multiplica por 1,15. El canal ancho recoge el doble de ruido y la misma señal.
- Con SNR negativa (la señal está por debajo del ruido) la capacidad no es cero. Baja, pero el enlace todavía puede transportar datos con mucha redundancia.

**La regla de los 3 dB.** Con SNR alta, log2(1 + S/N) ≈ log2(S/N). Duplicar S/N (+3 dB) suma 1 a ese logaritmo. Cada 3 dB extra aporta 1 bit/s/Hz. Por eso pasar de 20 a 23 dB de SNR mejora poco en proporción, y pasar de 0 a 3 dB mejora mucho.

**Lo que usa un sistema real: tablas CQI de 5G NR.** La norma 3GPP TS 38.214 (Rel-19, tabla 5.2.2.1-3) fija estas combinaciones. La última columna es la SNR mínima que exigiría el límite de Shannon para esa eficiencia, calculada como 2^η − 1. No es la SNR que necesita un receptor real, que es mayor.

| CQI | Modulación | Tasa de código (×1024) | Eficiencia (bit/s/Hz) | SNR mínima de Shannon |
|---|---|---|---|---|
| 1 | QPSK | 78 | 0,1523 | −9,5 dB |
| 3 | QPSK | 449 | 0,8770 | −0,8 dB |
| 6 | 16QAM | 616 | 2,4063 | 6,3 dB |
| 9 | 64QAM | 666 | 3,9023 | 11,4 dB |
| 12 | 256QAM | 711 | 5,5547 | 16,6 dB |
| 15 | 256QAM | 948 | 7,4063 | 22,3 dB |

El terminal elige el CQI más alto con el que un bloque de datos se recibiría con una probabilidad de error no mayor de 0,1 (TS 38.214, apartado 5.2.2.1). Ese 10 % de bloques fallidos no se pierde: se recupera con retransmisiones.

La norma no fija qué SNR necesita cada CQI. Eso depende del receptor de cada fabricante y del canal real. Por eso este artículo no da esa cifra.

### 4. Diagrama o flujo

```text
RUIDO TÉRMICO: −174 dBm/Hz × ancho de banda B   (+ figura de ruido del receptor)
        │
        ▼
SNR = señal recibida (dBm) − ruido (dBm)
        │
        ├──► LÍMITE TEÓRICO:  C = B · log2(1 + S/N)
        │
        ▼
RECEPTOR mide la SNR ──► informa CQI (0…15) ──► EMISOR
                                                   │
                                                   ▼
                       elige modulación (2…8 bits/símbolo)
                       y tasa de código (78…948 /1024)
                                                   │
                                                   ▼
               tasa real ≈ B × eficiencia × (fracción de recursos para datos)
                                                   │
                                                   ▼
               ¿error en el bloque? ── sí ──► retransmisión y CQI más bajo
                                     └─ no ──► bits entregados
```

## Límites, excepciones y casos reales

- **La fórmula supone ruido blanco gaussiano.** La interferencia de otras redes no siempre se comporta así. Llega a ráfagas y se concentra en partes del canal. En la práctica, los sistemas miden señal frente a ruido más interferencia (SINR) y la tratan como si fuera SNR.
- **No todo el canal transporta datos de usuario.** Una parte de los recursos se dedica a señales de referencia, control y sincronización. La tasa útil es menor que ancho de banda × eficiencia. La fracción exacta depende de la configuración de cada red.
- **La SNR cambia en milisegundos.** Al moverse, las reflexiones se suman y se cancelan (multitrayecto). Por eso el CQI se informa una y otra vez, y la velocidad oscila aunque el usuario no lo note.
- **El límite es por canal, no por aparato.** Varias antenas pueden crear varios canales en paralelo sobre la misma banda (MIMO). Telatar demostró en 1999 que, con un entorno con muchas reflexiones, la capacidad crece aproximadamente en proporción al menor número de antenas en los dos extremos. MIMO no viola el límite de Shannon: lo aplica a cada canal.
- **El tiempo compartido divide la capacidad.** Si diez aparatos comparten el mismo canal WiFi o la misma celda, se reparten la capacidad. Una señal excelente no garantiza velocidad si el canal está ocupado.

## Consecuencias prácticas

- **Las barras no son velocidad.** El icono de cobertura suele reflejar la potencia recibida. La velocidad depende de la SNR o SINR y de cuántos comparten el canal. Con buena potencia y mucha interferencia, la velocidad puede ser baja.
- **Un canal más ancho no siempre compensa.** Lejos del punto de acceso, un canal de 80 MHz recibe cuatro veces más ruido que uno de 20 MHz. Con −90 dBm y los supuestos del ejemplo, la capacidad pasa de 36 a 56 Mbit/s: un factor 1,55, no 4.
- **Acercarse al router ayuda más cuando la señal es débil.** De −95 a −85 dBm, la capacidad del ejemplo de 20 MHz se multiplica casi por cuatro. De −60 a −50 dBm, solo sube un 29 %.
- **Para quien diseña aparatos.** Cada dB de figura de ruido que se ahorra en el receptor es un dB de SNR ganado sin tocar la potencia del emisor. En la zona de señal débil, ese dB vale mucho.

## Preguntas frecuentes

### ¿Ancho de banda y velocidad son lo mismo?

No. El ancho de banda es la anchura del canal en hercios. La velocidad es una tasa de bits en bit/s. La relación entre los dos es la eficiencia espectral, que depende de la SNR. En este proyecto se reserva «ancho de banda» para los hercios, como indica el [glosario](../../../guides/TaxonomyAndGlossary.md).

### ¿Se puede superar el límite de Shannon?

No en el modelo para el que se demostró. Los códigos modernos se quedan cerca, pero por debajo. Lo que sí se puede es cambiar el problema: más ancho de banda, más antenas (más canales en paralelo) o más SNR.

### ¿Por qué la velocidad anunciada nunca se alcanza?

La cifra comercial suele corresponder a la mejor combinación de modulación y código, con el canal más ancho y todas las antenas, y sin nadie más compartiendo el canal. Exige una SNR alta que solo se da cerca del emisor y sin interferencias.

### ¿Hay un mínimo de energía por bit?

Sí, y sale de la misma fórmula. Si se reparte una potencia fija sobre un ancho de banda cada vez mayor, la SNR cae y la capacidad deja de crecer. El resultado es que cada bit necesita al menos ln 2 veces la densidad de ruido, unos −1,6 dB. Ningún sistema transmite datos fiables con menos energía por bit.

## Fuentes

- C. E. Shannon. «A Mathematical Theory of Communication». *The Bell System Technical Journal*, vol. 27, pp. 379-423 (julio) y 623-656 (octubre), 1948. Parte IV: canal continuo con ruido blanco. Copia de la reimpresión con correcciones: <https://people.math.harvard.edu/~ctm/home/text/others/shannon/entropy/entropy.pdf>
- ITU-R. *Recommendation ITU-R P.372-16: Radio noise* (08/2022). Apartado 1.2.1, ecuaciones 2 y 6. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.372-16-202208-I!!PDF-E.pdf>. Versión vigente: P.372-18 (09/2026), <https://www.itu.int/rec/R-REC-P.372/en>
- 3GPP. *TS 38.214: NR; Physical layer procedures for data* (Rel-19). Apartado 5.2.2.1, tablas 5.2.2.1-2 y 5.2.2.1-3. <https://www.etsi.org/deliver/etsi_ts/138200_138299/138214/>
- I. E. Telatar. «Capacity of Multi-antenna Gaussian Channels». *European Transactions on Telecommunications*, vol. 10, n.º 6, pp. 585-595, 1999. DOI: [10.1002/ett.4460100604](https://doi.org/10.1002/ett.4460100604)
