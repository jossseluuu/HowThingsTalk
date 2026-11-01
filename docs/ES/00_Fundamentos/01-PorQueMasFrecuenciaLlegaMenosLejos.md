---
title: "¿Por qué una señal de radio de más frecuencia llega menos lejos?"
section: "00_Fundamentos"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "Una onda de 5 GHz no se gasta más por el aire que una de 2,4 GHz. Llega peor porque las antenas pequeñas de los aparatos recogen menos energía cuando la onda es más corta, porque los materiales absorben más a más frecuencia y porque las ondas cortas rodean peor los obstáculos."
prerequisites:
  - "Qué es una onda electromagnética y qué son su frecuencia y su longitud de onda"
  - "Decibelios y dBm (se repasan dentro del artículo)"
translation: "../../EN/00_Foundations/01-WhyHigherFrequenciesReachLess.md"
---

# ¿Por qué una señal de radio de más frecuencia llega menos lejos?

## Pregunta y encuadre

Es una experiencia común: el WiFi de 2,4 GHz llega a la habitación del fondo y el de 5 GHz no. La pregunta es por qué. ¿Qué tiene una frecuencia más alta que la hace llegar menos lejos?

La respuesta habitual dice que «las frecuencias altas se atenúan más por el camino». Este artículo explica por qué esa frase es engañosa y qué tres mecanismos físicos producen de verdad el efecto. Usa el WiFi como ejemplo, pero la idea vale para cualquier enlace de radio: telefonía móvil, Bluetooth o GPS.

- **Profundidad:** intermedia. Aparecen fórmulas sencillas, siempre explicadas antes.
- **Conocimientos previos:** saber que una onda tiene una frecuencia (ciclos por segundo) y una longitud de onda (la distancia entre dos crestas). Las dos están ligadas: longitud de onda = velocidad de la luz / frecuencia. Los decibelios se repasan abajo.
- **Fuera de alcance:** el diseño de antenas, los protocolos WiFi, la normativa de potencias de cada país y la absorción de los gases de la atmósfera en ondas milimétricas.

**Repaso rápido de decibelios.** El decibelio (dB) expresa una relación entre dos potencias en escala logarítmica. +3 dB es aproximadamente el doble de potencia. +10 dB es diez veces más. Una pérdida de 20 dB deja una centésima parte de la potencia. Las pérdidas en dB se suman en lugar de multiplicarse, y por eso se usan en radio.

## Respuesta corta

La onda de más frecuencia no se gasta más en el vacío: llega peor porque la antena pequeña que la recibe captura menos energía, porque las paredes la absorben más y porque rodea peor los obstáculos.

### Resumen de 60 segundos

Un emisor reparte su potencia sobre una esfera que crece con la distancia. Esa dispersión es igual para todas las frecuencias. A 10 m, cada metro cuadrado recibe la misma potencia a 2,4 GHz que a 5,5 GHz.

La diferencia está en el receptor. Una antena que oye por igual en todas direcciones solo recoge la energía que cae sobre una superficie efectiva proporcional al cuadrado de la longitud de onda. A 5,5 GHz la onda mide menos de la mitad que a 2,4 GHz. Por tanto, esa superficie se reduce unas cinco veces. Eso son unos 7 dB menos de señal en espacio libre.

Al entrar en un edificio se añaden dos efectos. Los materiales comunes (hormigón, yeso, madera, vidrio) conducen mejor la electricidad a más frecuencia, y por eso absorben más energía por cada centímetro de grosor. Además, una onda más corta rodea peor los obstáculos que tapan el camino directo.

## Modelo mental

Imagina un aspersor que riega un jardín. El agua se reparte en todas direcciones. Cuanto más lejos estás, menos agua cae en cada metro cuadrado. Para medir el riego colocas un vaso en el suelo. Lo que recoges depende de dos cosas: cuánta agua cae por metro cuadrado y el tamaño de la boca del vaso.

En radio, el «agua por metro cuadrado» no depende de la frecuencia. Lo que cambia es el vaso. Una antena pequeña que oye en todas direcciones tiene una boca efectiva que encoge con la longitud de onda. Subir la frecuencia es como regar con el mismo aspersor y medir con un vaso más estrecho.

**Dónde se rompe la analogía.** La boca de una antena no es un agujero físico: es un área equivalente que sale de la física de la antena. Si pudieras mantener fijo el tamaño real del «vaso» (por ejemplo, con una antena parabólica de diámetro fijo), la frecuencia alta ganaría en lugar de perder. Además, el agua no atraviesa paredes ni rodea esquinas. Las ondas sí lo hacen, y en eso la frecuencia importa por razones que el aspersor no recoge.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

El camino de la señal tiene tres tramos donde puede perder potencia:

1. **Dispersión geométrica.** El emisor reparte su potencia sobre una esfera de superficie 4πd², donde d es la distancia. La densidad de potencia es S = P / (4πd²), medida en W/m² (Rec. ITU-R P.525-5, ec. 3). En esta fórmula no aparece la frecuencia.
2. **Captura en la antena receptora.** La potencia recibida es la densidad de potencia por el área efectiva de la antena: Pr = S · Ae (P.525-5, ec. 4). Para una antena isótropa, la referencia teórica que oye igual en todas direcciones, Ae = λ² / (4π), siendo λ la longitud de onda. Aquí entra la frecuencia.
3. **Obstáculos en el camino.** En un edificio, la onda atraviesa materiales que absorben energía y rodea objetos por difracción. Los dos fenómenos dependen de la frecuencia (Rec. ITU-R P.2040-3 y P.526-16).

Al combinar los tramos 1 y 2 sale la **pérdida básica de transmisión en espacio libre** entre antenas isótropas (P.525-5, ec. 5 y 6):

```text
Lbf = 20·log10(4π·d / λ)   [dB]
Lbf = 32,4 + 20·log10(f) + 20·log10(d)   [dB]   con f en MHz y d en km
```

El término 20·log10(f) suele leerse como «el aire atenúa más las frecuencias altas». No es así. Ese término procede por completo del área efectiva λ²/(4π) de la antena receptora isótropa. El espacio libre no absorbe nada: la energía solo se reparte.

### 2. Qué hace cada componente

**El emisor y su antena.** Fijan la potencia radiada y cómo se reparte por direcciones (la ganancia). Un router o un móvil usan antenas de baja ganancia porque deben cubrir todas las direcciones. Su comportamiento se parece al de la antena isótropa de referencia.

**El espacio entre ambos.** En vacío o aire limpio, a estas frecuencias, solo dispersa. Doblar la distancia reduce la densidad de potencia a la cuarta parte (−6 dB) a cualquier frecuencia.

**La antena receptora.** Su área efectiva es Ae = G · λ² / (4π), donde G es su ganancia. Si G se mantiene fija, el área cae con el cuadrado de la frecuencia. A 2437 MHz (canal 6 de WiFi), λ = 12,3 cm y Ae = 12,0 cm² para G = 1. A 5500 MHz (canal 100), λ = 5,45 cm y Ae = 2,4 cm². Es cinco veces menos superficie de captura y la diferencia es de 7,1 dB.

Friis mostró en 1946 la otra cara del mismo hecho. Si las dos antenas conservan un **área física fija**, la relación es Pr/Pt = Ar · At / (d² · λ²). En ese caso, subir la frecuencia *mejora* el enlace. Por eso los radioenlaces con parábolas usan frecuencias altas. Que la frecuencia alta «llegue menos» depende de qué se mantiene constante: la ganancia o el tamaño de la antena.

**Los materiales.** La Rec. ITU-R P.2040-3 modela cada material con dos números: la permitividad relativa η′ y la conductividad σ (en S/m). En casi todos los materiales de construcción, η′ apenas cambia con la frecuencia, pero σ crece con ella (σ = c · f^d, con f en GHz). Para materiales de pocas pérdidas, la atenuación dentro del material es A ≈ 1636 · σ / √η′ dB/m (P.2040-3, ec. 27a). Más conductividad equivale a más energía convertida en calor por cada metro de material.

**Los obstáculos que tapan el camino.** La difracción permite a la onda rodear un borde, pero su eficacia depende del tamaño del obstáculo comparado con la longitud de onda. La Rec. ITU-R P.526-16 resume la geometría en un parámetro adimensional ν. Este parámetro crece cuando λ disminuye (ν ∝ 1/√λ para la misma geometría), y a mayor ν, mayor pérdida por difracción. La zona que debe estar despejada para considerar libre el camino es el primer elipsoide de Fresnel. Su radio es R1 = 550 · √(d1·d2 / ((d1+d2)·f)) metros, con d1 y d2 en km y f en MHz (P.526-16, ec. 3).

### 3. Parámetros que importan

Todos los valores siguientes están calculados con las fórmulas citadas. El escenario son 2437 MHz frente a 5500 MHz.

| Magnitud | 2437 MHz | 5500 MHz | Fuente de la fórmula |
|---|---|---|---|
| Longitud de onda λ | 12,3 cm | 5,45 cm | λ = c / f |
| Área efectiva de una antena isótropa | 12,0 cm² | 2,4 cm² | P.525-5, ec. 4 |
| Pérdida en espacio libre a 10 m | 60,1 dB | 67,2 dB | P.525-5, ec. 6 |
| Radio del primer elipsoide de Fresnel a mitad de un enlace de 20 m | 0,79 m | 0,52 m | P.526-16, ec. 3 |

Atenuación interna de algunos materiales según el modelo de la Rec. ITU-R P.2040-3 (tabla 3 y ec. 27a). Las cifras solo cuentan la absorción dentro del material. No incluyen la reflexión en sus caras, que añade pérdida adicional.

| Material | 2437 MHz | 5500 MHz | Ejemplo: grosor y pérdida (2437 → 5500 MHz) |
|---|---|---|---|
| Hormigón | 66 dB/m | 125 dB/m | muro de 15 cm: 9,9 dB → 18,8 dB |
| Ladrillo | 23 dB/m | 26 dB/m | tabique de 10 cm: 2,3 dB → 2,6 dB |
| Madera | 14 dB/m | 34 dB/m | puerta maciza de 4 cm: 0,6 dB → 1,4 dB |
| Placa de yeso | 19 dB/m | 42 dB/m | placa de 1,25 cm: 0,2 dB → 0,5 dB |
| Vidrio | 8 dB/m | 23 dB/m | luna de 6 mm: 0,05 dB → 0,14 dB |

Dos lecturas de la tabla:

- El hormigón casi duplica su atenuación por metro entre una banda y otra. Es el material donde más se nota.
- El ladrillo apenas cambia, porque en el modelo de la ITU su conductividad casi no depende de la frecuencia (exponente d = 0,16).

**Qué significan 7 dB en distancia.** En espacio libre, la pérdida sube 20 dB por cada factor 10 de distancia, igual en las dos bandas. Para recibir la misma potencia con antenas de igual ganancia, el enlace a 5500 MHz debe ser 2437/5500 ≈ 0,44 veces más corto. En interiores la proporción real cambia, porque se suman paredes y reflexiones.

### 4. Diagrama o flujo

```text
EMISOR (potencia P, ganancia Gt)
   │
   ▼
Dispersión esférica: S = P·Gt / (4π·d²)       ← no depende de f
   │
   ▼
Obstáculos en el camino:
   ├─ absorción en materiales: A ≈ 1636·σ(f)/√η′ dB/m   ← σ crece con f
   └─ difracción en bordes: pérdida crece con ν ∝ 1/√λ  ← peor a más f
   │
   ▼
ANTENA RECEPTORA: Pr = S · Gr·λ²/(4π)         ← cae con 1/f² si Gr es fija
   │
   ▼
RECEPTOR: compara Pr con el ruido → ¿se puede decodificar?
```

## Límites, excepciones y casos reales

- **El modelo de espacio libre es un caso ideal.** Supone un camino despejado y ninguna reflexión. En una casa, la señal llega por muchos caminos a la vez (multitrayecto). Las sumas y cancelaciones entre ellos producen variaciones de varios dB al mover el aparato unos centímetros. El orden de magnitud de esas variaciones depende del entorno y no se cuantifica aquí.
- **La atenuación de la tabla no es la pérdida total de una pared.** Las cifras de la Rec. ITU-R P.2040-3 describen la absorción dentro de un material homogéneo. Una pared real tiene capas, cámaras de aire, armaduras de acero y tuberías. Además, parte de la energía se refleja en cada cara. La pérdida medida de un muro concreto puede alejarse mucho del modelo.
- **El metal bloquea las dos bandas.** Con una conductividad de 10⁷ S/m (P.2040-3, tabla 3), una lámina metálica es prácticamente opaca a ambas frecuencias. La señal pasa por huecos, puertas y ventanas, y ahí vuelve a contar la difracción.
- **Los rangos de validez importan.** Los parámetros de la tabla 3 de P.2040-3 proceden de medidas entre 1 y 100 GHz para la mayoría de materiales. La propia recomendación avisa de que los rangos son orientativos.
- **La ganancia de antena puede compensar.** Si el aparato mete varias antenas en el mismo espacio y combina sus señales (formación de haz), recupera área efectiva. Esto es más fácil a más frecuencia, porque cada antena es más pequeña. Es la razón de que los sistemas en ondas milimétricas usen matrices de muchas antenas.
- **No todo es propagación.** Los canales más anchos que suelen usarse en 5 GHz recogen más ruido térmico, que es proporcional al ancho de banda. Esto reduce el margen del receptor por un motivo que nada tiene que ver con la distancia.

## Consecuencias prácticas

- **Para quien usa el aparato.** Una banda baja cruza mejor paredes y forjados. Una banda alta llega menos lejos dentro de casa, pero sus canales suelen estar menos ocupados y ofrecen más capacidad cerca del punto de acceso. Ver que un móvil cambia de banda al alejarse del router es una consecuencia directa de este artículo.
- **Para quien planifica una red.** Cubrir una vivienda con una banda alta suele requerir más puntos de acceso. Cada punto cubre menos superficie, y a cambio las celdas vecinas se interfieren menos entre sí.
- **Para la telefonía móvil.** Por la misma física, las bandas bajas se usan para dar cobertura en zonas amplias y dentro de edificios. Las bandas altas se usan donde hace falta capacidad y la distancia es corta.
- **Para quien diseña aparatos.** A más frecuencia, la antena es más pequeña. Pero una antena pequeña de baja ganancia recoge menos energía. Ganar alcance exige más ganancia, lo que obliga a apuntar el haz o a usar varias antenas.

## Preguntas frecuentes

### ¿El aire absorbe más las frecuencias altas?

En las bandas de WiFi y de telefonía habituales, la absorción del aire es despreciable frente a los efectos de este artículo. Los gases atmosféricos (oxígeno y vapor de agua) sí absorben de forma notable en ondas milimétricas, a decenas de GHz. Ese fenómeno lo describe la Rec. ITU-R P.676 y queda fuera de este artículo.

### ¿La pérdida en espacio libre es energía que se pierde?

No. Es energía que se reparte. La potencia total sigue en la esfera; lo que baja es la parte que cae sobre la antena receptora. Por eso la ITU la llama pérdida «básica» de transmisión y la define entre antenas isótropas: es una referencia geométrica, no una absorción.

### ¿Una onda de más frecuencia viaja más despacio?

No. En el aire, todas las ondas de radio viajan prácticamente a la velocidad de la luz, unos 300 000 km/s. La frecuencia cambia la longitud de onda, no la velocidad.

### Entonces, ¿por qué las parabólicas de satélite usan frecuencias altas?

Porque mantienen fijo el tamaño físico de la antena, no su ganancia. Con un área física fija, la ganancia crece con el cuadrado de la frecuencia. Según la fórmula de Friis, el enlace mejora al subir la frecuencia. Es el caso contrario al de un móvil, cuya antena debe oír en todas direcciones.

## Fuentes

- ITU-R. *Recommendation ITU-R P.525-5: Calculation of free-space attenuation* (11/2024). Ecuaciones 3 a 6. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.525-5-202411-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.2040-3: Effects of building materials and structures on radiowave propagation above about 100 MHz* (08/2023). Tabla 3 y ecuación 27a. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.2040-3-202308-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.526-16: Propagation by diffraction* (11/2025). Ecuaciones 2, 3 y parámetro ν. <https://www.itu.int/dms_pubrec/itu-r/rec/p/R-REC-P.526-16-202511-I!!PDF-E.pdf>
- ITU-R. *Recommendation ITU-R P.676: Attenuation by atmospheric gases and related effects*. Citada solo como referencia de lo que queda fuera de alcance. <https://www.itu.int/rec/R-REC-P.676/en>
- H. T. Friis. «A Note on a Simple Transmission Formula». *Proceedings of the IRE*, vol. 34, n.º 5, pp. 254-256, mayo de 1946. DOI: [10.1109/JRPROC.1946.234568](https://doi.org/10.1109/JRPROC.1946.234568)
