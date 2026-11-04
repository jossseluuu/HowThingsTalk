---
title: "¿Por qué la señal cambia tanto al mover el móvil unos centímetros?"
section: "00_Fundamentos"
language: "es"
status: "published"
order: 3
depth: "intermediate"
summary: "La señal llega al móvil por muchos caminos a la vez: directa y rebotada en paredes, muebles y personas. Cada copia llega con un retraso distinto y se suman o se anulan según su fase. Basta moverse media longitud de onda (6 cm a 2,4 GHz) para pasar de un refuerzo a un hueco de 20 o 30 dB."
prerequisites:
  - "Decibelios y longitud de onda (repasados en el artículo 00·01)"
  - "Qué es la relación señal-ruido (artículo 00·02)"
translation: "../../EN/00_Foundations/03-WhySignalChangesOverAFewCentimetres.md"
---

# ¿Por qué la señal cambia tanto al mover el móvil unos centímetros?

## Pregunta y encuadre

Es fácil comprobarlo. El portátil va lento en una esquina de la mesa y bien a un palmo de distancia. El móvil pierde una barra al girarlo en la mano. El router no se ha movido y las paredes son las mismas. La pregunta es por qué la señal cambia tanto en distancias tan cortas.

Este artículo explica el desvanecimiento por multitrayecto. Es el mecanismo que hace que la potencia recibida suba y baje en pocos centímetros y en pocos milisegundos. Afecta al WiFi, a la telefonía móvil, al Bluetooth y a cualquier enlace de radio que no tenga un camino único y limpio.

- **Profundidad:** intermedia. Aparecen una suma de vectores y una distribución de probabilidad, explicadas paso a paso.
- **Conocimientos previos:** decibelios y la relación longitud de onda = velocidad de la luz / frecuencia, repasados en [¿Por qué una señal de radio de más frecuencia llega menos lejos?](01-PorQueMasFrecuenciaLlegaMenosLejos.md). La relación señal-ruido (SNR), explicada en [¿Qué decide cuántos bits por segundo caben en una señal de radio?](02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md).
- **Fuera de alcance:** el diseño de los ecualizadores, la teoría completa de MIMO, los modelos de canal de cada norma y la propagación en ondas milimétricas.

## Respuesta corta

La señal llega por muchos caminos a la vez, y sus copias se suman o se anulan según la distancia exacta que ha recorrido cada una; moverse media longitud de onda basta para cambiar esa suma.

### Resumen de 60 segundos

En una casa o en una calle, la onda del emisor rebota en paredes, suelos, muebles, coches y personas. Al receptor llegan muchas copias de la misma señal. Cada una ha recorrido un camino distinto y llega con un retraso distinto.

Lo que importa no es solo cuánta energía trae cada copia. Importa en qué punto de su ciclo llega. Si dos copias llegan en fase, se refuerzan. Si llegan desfasadas media onda, se restan. A 2,4 GHz la onda mide unos 12,5 cm. Un cambio de 6 cm en la longitud de un camino basta para pasar de sumar a restar.

Con muchas copias sin una dominante, la suma se comporta como un valor aleatorio. Sigue una distribución llamada de Rayleigh. En ella, la potencia cae 10 dB por debajo de la media durante un 9,5 % del recorrido, y 20 dB durante un 1 %. Los huecos más profundos están separados unos pocos centímetros.

Los aparatos se defienden de tres formas: varias antenas separadas unos centímetros, muchas subportadoras repartidas por el canal y códigos correctores que reparten cada dato por varias de ellas.

## Modelo mental

Piensa en la superficie de una piscina con varias personas dando palmadas en el agua al mismo ritmo, desde puntos distintos. Las ondas se cruzan. En algunos puntos dos crestas coinciden y el agua sube mucho. En otros, una cresta coincide con un valle y el agua casi no se mueve. Un corcho que se desplace un palmo pasa de un punto agitado a uno tranquilo.

Las personas son los caminos de la señal. El corcho es la antena del móvil. El dibujo de zonas agitadas y tranquilas es el patrón de interferencia que el receptor atraviesa al moverse.

**Dónde se rompe la analogía.** En la piscina ves cada fuente y puedes predecir el dibujo. En una habitación hay decenas de caminos, y cada uno cambia si alguien abre una puerta o cruza el pasillo. Por eso no se predice el valor exacto en cada punto. Se describe su estadística. Además, la radio no solo varía en el espacio: dentro de un mismo canal, unas frecuencias caen en un hueco y otras no. El corcho no tiene equivalente para eso.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

1. **El emisor radia.** La onda sale en muchas direcciones a la vez.
2. **El entorno la reparte.** Parte llega en línea recta, si hay visión directa. El resto se refleja, se difracta en bordes o se dispersa en objetos pequeños.
3. **Las copias llegan con retrasos distintos.** Cada camino tiene su longitud. Una diferencia de 3 m equivale a unos 10 ns de retraso.
4. **La antena suma todo.** La antena no distingue copias. Recibe la suma de todos los campos en ese punto y en ese instante.
5. **El resultado depende de las fases.** Cada copia llega con una amplitud y una fase. La suma puede ser mayor o mucho menor que cualquiera de ellas.
6. **Moverse cambia las fases.** Al desplazar la antena una fracción de longitud de onda, cambian las longitudes relativas de los caminos. La suma cambia con ellas.

### 2. Qué hace cada componente

**Los caminos y su fase.** Una copia que recorre un camino d llega con una fase proporcional a d/λ, donde λ es la longitud de onda. Cada λ de recorrido extra es una vuelta completa. Media λ extra es media vuelta: la copia llega invertida respecto a otra.

**La suma de vectores.** Cada copia se puede dibujar como una flecha. Su longitud es la amplitud y su ángulo es la fase. La señal recibida es la suma de todas las flechas. La Rec. ITU-R P.1057-7 describe así la distribución de Rayleigh: la longitud de la suma de muchos vectores de amplitud parecida y fase uniformemente aleatoria (apartado 5). Cuando ninguna copia domina, el módulo de la suma sigue esa distribución.

**La distribución de Rayleigh.** Su función de distribución acumulada es F(x) = 1 − exp(−x²/b²), donde b es el valor cuadrático medio de la amplitud (P.1057-7, apartado 5, ec. 10). Pasada a potencia, la probabilidad de estar más de X dB por debajo de la potencia media es:

```text
P(potencia < media − X dB) = 1 − exp(−10^(−X/10))
```

Para niveles bajos esta curva tiene una pendiente fija. Cada década de probabilidad equivale a 10 dB de potencia (P.1057-7, apartado 5, ec. 11).

**La componente directa.** Si hay visión directa, una copia es mucho más fuerte que las demás. Entonces la suma sigue una distribución de Nakagami-Rice: un vector fijo más un vector de Rayleigh (P.1057-7, apartado 7). Se resume con el factor K, la relación entre la potencia de la componente fija y la de la parte aleatoria. Con K = 0 la distribución vuelve a ser de Rayleigh (Rec. ITU-R P.1406-2, apartado 3.2.2). Cuanto mayor es K, menos profundos son los huecos.

**La correlación en el espacio.** Clarke (1968) calculó cómo se parece el campo en dos puntos separados una distancia δ. Supuso que las ondas llegan desde todas las direcciones horizontales con la misma probabilidad. El resultado es la función de Bessel J0(2πδ/λ) (Clarke, ec. 9). La correlación de la envolvente es aproximadamente su cuadrado (ec. 16). J0 se anula por primera vez en 2πδ/λ = 2,405, es decir, en δ ≈ 0,38 λ. A esa distancia, dos antenas ven desvanecimientos casi independientes.

**El efecto Doppler.** Si el receptor se mueve a velocidad v, cada copia sufre un desplazamiento de frecuencia distinto, según su dirección de llegada. El máximo es f_m = v/λ (Clarke, ec. 21). Recorrer el patrón espacial a velocidad v lo convierte en una variación temporal. La correlación en el tiempo es J0(2π·f_m·τ) (Clarke, ec. 18).

**La dispersión de retardos.** Las copias no solo tienen fases distintas. Llegan repartidas en el tiempo. El valor eficaz de ese reparto se llama dispersión de retardos (r.m.s. delay spread). Si es grande frente a la duración de un símbolo, un símbolo se mezcla con el siguiente (P.1406-2, apartado 4.1). En frecuencia, el mismo fenómeno hace que unas partes del canal caigan en un hueco y otras no. Clarke relaciona el ancho de banda en el que la señal se mantiene coherente con la inversa de esa dispersión.

### 3. Parámetros que importan

**Escalas de longitud.** Longitud de onda λ = c/f. Distancia a la primera decorrelación según el modelo de Clarke: 0,38 λ.

| Frecuencia | λ | λ/2 | 0,38 λ |
|---|---|---|---|
| 700 MHz | 42,8 cm | 21,4 cm | 16,4 cm |
| 2,4 GHz | 12,5 cm | 6,2 cm | 4,8 cm |
| 5,5 GHz | 5,5 cm | 2,7 cm | 2,1 cm |
| 6 GHz | 5,0 cm | 2,5 cm | 1,9 cm |

La P.1406-2 indica que los mínimos de un patrón de interferencia están separados al menos media longitud de onda (apartado 3.3). En entornos urbanos o con árboles, el campo medido a lo largo de unas decenas de longitudes de onda sigue aproximadamente una distribución de Rayleigh. Los desvanecimientos de 30 dB o más por debajo de la media «son comunes» (P.1406-2, apartado 3.3).

**Profundidad y frecuencia de los huecos (Rayleigh, sin componente directa).** Calculado con la fórmula anterior.

| Nivel respecto a la potencia media | Fracción del recorrido por debajo |
|---|---|
| −3 dB | 39 % |
| −10 dB | 9,5 % |
| −20 dB | 1,0 % |
| −30 dB | 0,1 % |

La mediana de la potencia queda 1,6 dB por debajo de la media. Un único punto de medida no dice casi nada de la cobertura real.

**Escalas de tiempo.** Supuestos ilustrativos: una persona que camina a 1,4 m/s y un coche a 120 km/h. El tiempo de decorrelación es el primer cero de J0(2π·f_m·τ), en τ ≈ 0,38/f_m.

| Escenario | f_m = v/λ | Tiempo hasta la primera decorrelación |
|---|---|---|
| Caminando, 2,4 GHz | 11 Hz | 34 ms |
| Caminando, 5,5 GHz | 26 Hz | 15 ms |
| Coche a 120 km/h, 2 GHz | 222 Hz | 1,7 ms |

**Dispersión de retardos en interiores.** Rec. ITU-R P.1238-13, tabla 7. Antenas omnidireccionales, resolución de 10 ns. Valores del 10 %, la mediana y el 90 %, para las mayores salas típicas de cada entorno.

| Entorno | 3,7 GHz | 5,2 GHz |
|---|---|---|
| Residencial | 15 · 22 · 27 ns | 17 · 23 · 30 ns |
| Oficina | 30 · 38 · 45 ns | 38 · 60 · 110 ns |
| Comercial | 105 · 145 · 170 ns | 135 · 190 · 205 ns |

Con 60 ns de dispersión, la inversa es de unos 17 MHz. Es solo un orden de magnitud: el factor exacto depende de cómo se defina la coherencia. Pero basta para ver que un canal WiFi de 20 MHz o más es más ancho que la zona coherente. Dentro del canal habrá frecuencias en hueco y frecuencias reforzadas.

**Dos escalas que no hay que confundir.** La P.1406-2 separa la variación rápida, por multitrayecto, que ocurre en unas pocas longitudes de onda, de la variación lenta, debida a cambios en el sombreado de edificios y obstáculos (apartado 3.2). La lenta se describe con una distribución log-normal. En zona urbana amplia, su desviación típica a 2 GHz es de unos 7,5 dB (P.1406-2, ec. 2, válida de 100 a 3000 MHz). Para separar las dos, la norma recomienda medir a lo largo de unas 40 longitudes de onda. Hacen falta unas 36 muestras separadas al menos 0,8 λ para obtener la mediana con 1 dB de precisión y un 90 % de probabilidad (apartado 3.2).

### 4. Diagrama o flujo

```text
              pared                           mueble
                │  camino 2 (más largo)         │
   EMISOR ──────┼──────────────────────────────┼────► ANTENA
     │ camino 1 (directo)                       │  camino 3
     └──────────────────────────────────────────┘
                                │
                                ▼
       cada copia llega con amplitud a_i y fase φ_i = 2π·d_i/λ
                                │
                                ▼
              señal recibida = suma de vectores  Σ a_i·e^(jφ_i)
                                │
          ┌─────────────────────┴─────────────────────┐
          ▼                                           ▼
   fases parecidas: refuerzo                fases opuestas: hueco
                                                 (−20 a −30 dB)
                                │
                                ▼
   mover la antena ~0,38 λ ──► otra suma casi independiente
   moverse a velocidad v   ──► el patrón se recorre en el tiempo (f_m = v/λ)
   copias repartidas en τ  ──► unas frecuencias del canal en hueco, otras no
```

## Límites, excepciones y casos reales

- **El modelo de Clarke supone ondas desde todas las direcciones.** Si la mayoría de las copias llega de un sector estrecho, la correlación baja mucho más despacio con la distancia. Por eso una estación base elevada, con los objetos dispersores lejos y en una dirección, necesita separar sus antenas mucho más. La P.1406-2 da unas 8 longitudes de onda en horizontal, con orientación óptima, y unas 17 en vertical para bajar la correlación a 0,7 (apartado 5.3).
- **Con visión directa, los huecos son menos profundos.** La distribución pasa de Rayleigh a Nakagami-Rice. Cuanto más domina el camino directo, más se parece la señal a un valor fijo con pequeñas oscilaciones.
- **No hace falta moverse para sufrirlo.** Si el móvil está quieto, el entorno sigue cambiando. Las personas que cruzan, las puertas y los vehículos cambian los caminos. Las reflexiones en vehículos en movimiento pueden causar desvanecimiento con el receptor parado (P.1406-2, apartado 3.3).
- **El cuerpo cuenta.** La mano y la cabeza absorben y tapan parte de la señal. Ese efecto se suma al multitrayecto, pero es otro mecanismo. Su valor depende del aparato y de cómo se sujete. No se da una cifra general aquí.
- **La estadística es local.** La distribución de Rayleigh describe la variación dentro de una zona de unas decenas de longitudes de onda. La media de esa zona cambia a su vez con la distancia y el sombreado. Mezclar las dos escalas lleva a conclusiones erróneas.

## Consecuencias prácticas

- **Mover el aparato unos centímetros puede cambiar mucho la velocidad.** Si la señal media es débil, un hueco de 10 o 20 dB baja la SNR lo suficiente para que el enlace elija una modulación más lenta. Lo explica el [artículo 00·02](02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md).
- **Por eso los routers y los móviles tienen varias antenas.** Con dos antenas a más de 0,38 λ, en el modelo de Clarke, la probabilidad de que ambas estén a la vez 10 dB por debajo de la media cae del 9,5 % a cerca del 0,9 %. A 2,4 GHz eso son menos de 5 cm de separación. Cabe en un móvil.
- **Por eso el WiFi y el 5G reparten los datos en muchas subportadoras.** Si unas frecuencias del canal caen en un hueco, las demás siguen llevando datos. Los códigos correctores reparten cada dato entre varias subportadoras y recuperan lo que se pierde.
- **Medir la cobertura exige promediar.** Una sola lectura de dBm puede estar 20 dB por debajo de la media de esa zona. Las mediciones serias recorren decenas de longitudes de onda y toman la mediana.
- **Para quien diseña aparatos.** La separación entre antenas, su polarización y su posición frente a la mano deciden cuánto se aprovecha la diversidad. Es una de las razones por las que la colocación de las antenas se estudia con tanto cuidado.

## Preguntas frecuentes

### ¿Es lo mismo que la atenuación por distancia?

No. La atenuación por distancia y por paredes, explicada en el [artículo 00·01](01-PorQueMasFrecuenciaLlegaMenosLejos.md), fija el nivel medio de la señal en una zona. El multitrayecto hace oscilar la señal alrededor de esa media en distancias de centímetros. Los dos efectos se suman.

### ¿Los huecos son energía que se pierde?

No. La energía no desaparece: se redistribuye. Donde las copias se restan hay un hueco, y a pocos centímetros, donde se suman, hay un refuerzo. En promedio sobre la zona, la potencia es la que corresponde a la distancia y los obstáculos.

### ¿Por qué el móvil marca las mismas barras aunque la velocidad cambie?

El indicador de cobertura suele promediar la potencia recibida durante un tiempo. Los huecos rápidos duran milisegundos y quedan ocultos en ese promedio. El enlace, en cambio, reacciona a ellos al elegir modulación y código.

### ¿El multitrayecto siempre es malo?

No. Los sistemas con varias antenas en los dos extremos (MIMO) aprovechan que los caminos sean distintos. Con muchas reflexiones, pueden enviar varios flujos de datos a la vez sobre el mismo canal. Sin multitrayecto, esa ganancia desaparece en gran parte. El límite de capacidad de MIMO se trata en el [artículo 00·02](02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md).

## Fuentes

- R. H. Clarke. «A Statistical Theory of Mobile-Radio Reception». *The Bell System Technical Journal*, vol. 47, n.º 6, pp. 957-1000, julio-agosto de 1968. Ecuaciones 9, 16, 18 y 21. Texto completo escaneado: <https://archive.org/details/bstj47-6-957>
- ITU-R. *Recommendation ITU-R P.1057-7: Probability distributions relevant to radiowave propagation modelling* (08/2022). Apartados 5 (Rayleigh) y 7 (Nakagami-Rice). <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.1057-7-202208-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.1406-2: Propagation effects relating to terrestrial land mobile and broadcasting services in the VHF and UHF bands* (07/2015). Apartados 3.2, 3.3, 4.1 y 5.3. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.1406-2-201507-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.1238-13: Propagation data and prediction methods for the planning of indoor radiocommunication systems and radio local area networks in the frequency range from 300 MHz to 450 GHz* (09/2025). Apartado 4 y tabla 7. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.1238-13-202509-I!!PDF-E.pdf>
