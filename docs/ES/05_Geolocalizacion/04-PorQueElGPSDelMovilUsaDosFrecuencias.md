---
title: "¿Por qué los móviles recientes escuchan el GPS en dos frecuencias?"
section: "05_Geolocalizacion"
language: "es"
status: "published"
order: 4
depth: "intermediate"
summary: "La ionosfera retrasa la señal de los satélites, y el retraso depende de la frecuencia. Un móvil que recibe el mismo satélite en L1 (1575,42 MHz) y en L5 (1176,45 MHz) puede medir ese retraso en lugar de estimarlo con un modelo. Además, el código de L5 es diez veces más rápido que el de L1, y eso hace que los rebotes en edificios falseen menos la distancia."
prerequisites:
  - "Frecuencia, longitud de onda y decibelios (artículo 00·01)"
  - "Multitrayecto (artículo 00·03)"
  - "Cómo calcula el GPS una posición (artículo 05·01)"
translation: "../../EN/05_Positioning/04-WhyPhoneGPSUsesTwoFrequencies.md"
---

# ¿Por qué los móviles recientes escuchan el GPS en dos frecuencias?

## Pregunta y encuadre

Durante años, el GPS de un móvil escuchaba una sola señal civil: la L1, a 1575,42 MHz. Hoy muchos móviles reciben además una segunda frecuencia, la L5, a 1176,45 MHz. La pregunta es qué gana un receptor por escuchar al mismo satélite dos veces.

El artículo explica dos mecanismos. El primero es cómo dos frecuencias permiten medir el retraso que introduce la ionosfera. El segundo es por qué la señal L5 resiste mejor los rebotes. No cubre las técnicas de precisión centimétrica con fase de portadora (RTK, PPP), aunque también se apoyan en varias frecuencias. Tampoco cubre la ayuda de la red para arrancar, que está en el [artículo 05·03](03-ComoAyudaLaRedMovilAlGPSAArrancar.md).

Hace falta el principio del [artículo 05·01](01-ComoCalculaElGPSTuPosicion.md): código C/A, correlación y pseudodistancias. También ayudan los decibelios ([artículo 00·01](../00_Fundamentos/01-PorQueMasFrecuenciaLlegaMenosLejos.md)) y el multitrayecto ([artículo 00·03](../00_Fundamentos/03-PorQueLaCoberturaCambiaEnPocosCentimetros.md)). La profundidad es intermedia. Hay una fórmula sencilla, explicada paso a paso. Las cuentas propias se marcan como «calculado».

## Respuesta corta

Como la ionosfera retrasa más la frecuencia baja que la alta, comparar las dos medidas de un mismo satélite revela ese retraso y permite quitarlo, y el código diez veces más rápido de L5 hace que los rebotes deformen menos la medida.

### Resumen de 60 segundos

El receptor calcula la distancia a cada satélite a partir del tiempo de viaje de la señal. Pero la señal no viaja siempre a la velocidad de la luz. Al cruzar la ionosfera, una capa de gas ionizado a cientos de kilómetros de altura, se retrasa. Ese retraso cambia con la hora, el lugar y la actividad del Sol. Con una sola frecuencia, el receptor solo puede estimarlo con un modelo que emiten los satélites. Ese modelo corrige, según la especificación, al menos la mitad del error.

El retraso ionosférico es inversamente proporcional al cuadrado de la frecuencia. L5 sufre unas 1,79 veces más retraso que L1. Si el receptor mide la pseudodistancia en las dos, la diferencia entre ambas solo puede venir de la ionosfera. A partir de ella calcula el retraso y lo resta.

La segunda ventaja es el código. En L5 cada chip dura una décima parte que en L1. El pico de correlación es diez veces más estrecho. Un rebote en una fachada que llega con retraso se separa antes del pico bueno y lo deforma menos. Un estudio con el primer móvil de doble frecuencia midió un error por multitrayecto unas tres veces menor en L5 que en L1.

No todo son ventajas. Combinar dos medidas suma el ruido de las dos. Y no todos los satélites GPS emiten L5 todavía.

## Modelo mental

Imagina que mides una mesa en un taller con dos reglas, una de acero y otra de aluminio. El taller está a una temperatura que no conoces. Las dos reglas se han dilatado, así que las dos dan una medida errónea. Pero sabes que el aluminio se dilata casi el doble que el acero. La diferencia entre las dos lecturas te dice cuánto ha influido la temperatura. Con eso puedes calcular la longitud real sin termómetro.

Las dos frecuencias hacen lo mismo. La ionosfera es la temperatura desconocida. L1 y L5 son las dos reglas, y la física dice exactamente cuánto más se «dilata» L5: un factor de 1,79.

La analogía se rompe en tres puntos. Primero, la ionosfera no es la misma para todos los satélites: cada señal cruza una zona distinta del cielo, así que el cálculo se hace satélite a satélite. Segundo, cada lectura tiene su propio ruido, y al restar dos lecturas parecidas ese ruido crece; con reglas bien grabadas apenas importaría. Tercero, la analogía no explica la otra mitad de la historia: L5 no solo es otra frecuencia, es una señal mejor diseñada, con un código más rápido.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

Un satélite GPS moderno emite a la vez en varias frecuencias. Todas salen del mismo reloj y llevan códigos sincronizados. Un móvil de doble frecuencia sigue este proceso para cada satélite que emite L5:

1. **Seguimiento en dos bandas.** El receptor engancha el código C/A en L1 y el código de L5 del mismo satélite. Lo hace con dos cadenas de radio, porque las frecuencias están separadas casi 400 MHz.
2. **Dos pseudodistancias.** Obtiene una pseudodistancia en cada frecuencia. Las dos contienen la misma distancia geométrica, el mismo error de reloj y el mismo retraso troposférico. Solo se diferencian en el retraso ionosférico y en los retrasos internos del equipo.
3. **Corrección de retrasos internos.** Aplica las correcciones de grupo que el satélite emite en su mensaje (TGD e ISC). Compensan que cada señal tarda un tiempo distinto en salir de la electrónica del satélite.
4. **Eliminación de la ionosfera.** Combina las dos pseudodistancias de forma que el término ionosférico se cancela.
5. **Solución.** Usa esas medidas corregidas en el cálculo de posición de siempre, con cuatro o más satélites.

Galileo emite su señal E5a en la misma frecuencia que L5, 1176,45 MHz, y su E1 en la misma que L1. Por eso el mismo hardware de doble banda sirve para las dos constelaciones.

### 2. Qué hace cada componente

**La ionosfera.** Es la parte de la atmósfera donde la radiación solar arranca electrones a los átomos. Según la ESA (Navipedia), la densidad máxima ronda entre 10¹¹ y 10¹² electrones por metro cúbico, entre 300 y 500 km de altura. Lo que importa para el GPS es el contenido total de electrones a lo largo del camino de la señal (STEC). Se mide en TECU: 1 TECU son 10¹⁶ electrones por metro cuadrado.

**El retraso.** Según la misma fuente, el retraso del código en metros es:

```text
I(f) = 40,3 × STEC / f²
```

- I(f) es el retraso del código, en metros.
- STEC es el contenido de electrones del camino, en electrones/m².
- f es la frecuencia de la señal, en hercios.

La fórmula captura más del 99,9 % del efecto. Los términos de orden superior suponen típicamente menos del 0,1 % del retraso total. Una curiosidad: el código se retrasa, pero la fase de la portadora se adelanta en la misma cantidad.

**La señal L5.** Según la especificación IS-GPS-705, su portadora está en 1176,45 MHz, que es 115 veces la frecuencia fundamental de 10,23 MHz del sistema. Ocupa una banda de 24 MHz. Tiene dos componentes en cuadratura. La I5 lleva datos de navegación a 50 bit/s, con codificación de errores. La Q5 no lleva datos: es una señal piloto. Cada componente usa un código de 10 230 chips que se repite cada milisegundo, a 10,23 millones de chips por segundo. El servicio GPS la describe como una señal con más potencia y más ancho de banda, emitida en una banda reservada a servicios de seguridad aeronáutica.

**La componente piloto.** Sin datos encima, el receptor no tiene que adivinar dónde cambia el signo de un bit. Puede acumular la señal durante más tiempo y seguirla con más estabilidad. La especificación no da una ganancia concreta para esto, así que aquí no se cuantifica.

**Las correcciones de grupo.** La electrónica del satélite no retrasa igual cada señal. El mensaje de navegación moderno (CNAV) emite el TGD y las correcciones entre señales ISC_L5I5 e ISC_L5Q5. La IS-GPS-705 indica que se usan tanto para los usuarios de una sola frecuencia en L5 como para los de doble frecuencia L1/L5. La misma especificación admite una diferencia media de retraso de grupo de hasta 30 ns. Eso son unos 9 m (calculado). Sin corregirla, la combinación arrastraría ese error.

**La combinación sin ionosfera.** Llamemos P1 y P5 a las pseudodistancias en L1 y L5, ya corregidas de retrasos internos. Si el retraso en L1 es I1, en L5 es γ·I1, con:

```text
γ = (f_L1 / f_L5)² = (1575,42 / 1176,45)² = (154/115)² ≈ 1,793
```

Entonces:

```text
P1 = ρ + I1
P5 = ρ + γ·I1

Retraso ionosférico en L1:   I1   = (P5 − P1) / (γ − 1)
Medida sin ionosfera:        P_IF = (γ·P1 − P5) / (γ − 1) ≈ 2,261·P1 − 1,261·P5
```

Aquí ρ agrupa todo lo que es igual en las dos frecuencias: distancia, reloj y troposfera. El valor de γ aparece tal cual en la IS-GPS-705. La ESA describe esta misma combinación para pares de frecuencias y señala que elimina el efecto de primer orden.

**El correlador de L5.** Funciona igual que el de L1 del [artículo 05·01](01-ComoCalculaElGPSTuPosicion.md), pero con chips diez veces más cortos. Un chip de L5 dura unos 97,8 ns, que equivalen a unos 29,3 m de recorrido. En L1 eran unos 293 m (calculado).

### 3. Parámetros que importan

**Las dos señales, frente a frente.**

| Parámetro | L1 C/A | L5 |
|---|---|---|
| Frecuencia | 1575,42 MHz | 1176,45 MHz |
| Longitud de onda | 19,0 cm (calculado) | 25,5 cm (calculado) |
| Ritmo del código | 1,023 Mchip/s | 10,23 Mchip/s |
| Longitud del código | 1023 chips, 1 ms | 10 230 chips, 1 ms |
| Recorrido de un chip | unos 293 m (calculado) | unos 29,3 m (calculado) |
| Banda ocupada | unos 2 MHz | 24 MHz |
| Potencia mínima recibida | −158,5 dBW | −157,9 dBW por componente (bloque IIF); −157,0 dBW (GPS III) |
| Componente sin datos | no | sí (Q5) |
| Retraso ionosférico por TECU | 0,162 m (calculado) | 0,291 m (calculado) |

Las dos componentes de L5 juntas suman 3 dB más que una sola. Eso da unos −154,9 dBW para el bloque IIF y −154,0 dBW para GPS III (calculado). Es decir, entre 3,6 y 4,5 dB más que el C/A de L1 en el mínimo especificado.

**Un ejemplo numérico.** Supón un camino con 50 TECU, un valor elegido solo para ilustrar. El retraso es de unos 8,1 m en L1 y de unos 14,6 m en L5. La diferencia entre las dos pseudodistancias es de unos 6,4 m. Dividida por γ − 1 ≈ 0,793, devuelve los 8,1 m de L1 (calculado). Según la ESA, el retraso en una sola frecuencia puede llegar a varias decenas de metros. El modelo que emiten los satélites corrige al menos un 50 % del error en valor eficaz, según la IS-GPS-705. Lo que queda tras la combinación son los términos de orden superior: en este ejemplo, menos de 1 cm (calculado con el 0,1 % de la ESA).

**El precio: el ruido.** La combinación multiplica P1 por 2,261 y P5 por 1,261. Si las dos medidas tienen el mismo ruido aleatorio σ, el resultado tiene un ruido de unos 2,6·σ (calculado). El receptor cambia un error sistemático grande por un error aleatorio mayor. Compensa cuando el error ionosférico supera a ese ruido, o cuando el ruido se reduce promediando en el tiempo.

**Multitrayecto.** Una regla aproximada de la teoría del correlador: un rebote que llega con más de un chip de retraso apenas deforma el pico principal. En L1, eso exige un camino unos 293 m más largo. En L5 basta con unos 29 m (calculado). En una calle, la mayoría de los rebotes de fachadas recorren decenas de metros de más, no cientos. El estudio de Robustelli y colaboradores (2019) lo midió en un móvil estático durante una hora, en una zona con poco multitrayecto:

| Señal | Desviación típica del multitrayecto de código |
|---|---|
| GPS L1 | 6,11 m |
| GPS L5 | 2,09 m |
| Galileo E1 | 3,77 m |
| Galileo E5a | 1,21 m |

En el mismo ensayo, la posición con solo Galileo tuvo un error horizontal de 13,50 m con E1 y de 4,57 m con E5a.

### 4. Diagrama o flujo

```text
 Satélite (un solo reloj)
   ├── L1 = 1575,42 MHz   código C/A, 1,023 Mchip/s
   └── L5 = 1176,45 MHz   código L5, 10,23 Mchip/s (I5 datos + Q5 piloto)
          │
          ▼
 Ionosfera: retraso = 40,3·STEC/f²
   L1 se retrasa I1        L5 se retrasa 1,793·I1
          │
          ▼
 Móvil: dos cadenas de radio, dos correladores
   P1 = ρ + I1             P5 = ρ + 1,793·I1
          │
          ├─ resta TGD/ISC (retrasos internos del satélite)
          ▼
 I1   = (P5 − P1) / 0,793           ─► retraso medido, no modelado
 P_IF = 2,261·P1 − 1,261·P5         ─► medida sin ionosfera (ruido ×2,6)
          │
          ▼
 Cálculo de posición con 4 o más satélites (artículo 05·01)
```

## Límites, excepciones y casos reales

**L5 aún no está completa.** El servicio GPS indica que L5 sigue siendo preoperativa y que se usa bajo responsabilidad del usuario hasta que se declare operativa. También advierte que las señales nuevas tienen uso limitado hasta que las emiten entre 18 y 24 satélites. Los satélites antiguos solo emiten L1 C/A para uso civil. En la práctica, el móvil mezcla satélites con una y con dos frecuencias. Galileo E5a ayuda a cubrir el hueco.

**La troposfera no se va.** La capa baja de la atmósfera también retrasa la señal, pero casi igual en las dos frecuencias. La combinación no la elimina; hay que seguir modelándola.

**El multitrayecto no se cancela, se amplifica.** La combinación sin ionosfera arrastra el multitrayecto de las dos medidas, multiplicado por sus coeficientes. La ventaja de L5 frente a los rebotes se aprovecha mejor usando L5 directamente en la medida que mezclándola con una L1 más contaminada.

**El móvil no es un receptor geodésico.** En el estudio del Xiaomi Mi 8, la antena tenía polarización lineal, más sensible a los rebotes. Las señales L1/E1 llegaban bastante más fuertes que L5/E5a, a pesar de que L5 se emite con más potencia. El chip, además, escuchaba 200 ms y descansaba 800 ms para ahorrar batería. Las cifras de ese estudio son de un móvil concreto de 2018, en dos ubicaciones y un día cada una.

**Qué hace exactamente cada chip no es público.** Un fabricante puede usar L5 para medir la ionosfera, para reducir el multitrayecto o para las dos cosas, y combinarlo con otros sensores. Los algoritmos concretos de los chips comerciales no se han podido verificar para este artículo.

**Dos frecuencias no son precisión centimétrica.** La medida de código sigue teniendo errores de metros por ruido, rebotes y geometría. Los centímetros exigen medir la fase de la portadora, resolver su ambigüedad y usar correcciones externas. Eso queda fuera de este artículo.

## Consecuencias prácticas

En una ciudad, el beneficio más visible suele venir del multitrayecto. Un rebote que en L1 desplaza la medida decenas de metros se nota mucho menos en L5. La posición tiende a saltar menos de acera.

La ionosfera importa más donde y cuando está más cargada: cerca del ecuador, a mediodía y en los años de máximo solar. Ahí un receptor de una frecuencia depende de un modelo que solo corrige una parte. Uno de dos frecuencias lo mide.

Desde Android 10, los móviles deben ofrecer a las aplicaciones las medidas GNSS en bruto. El soporte de varias frecuencias es opcional y depende del chip. Según la documentación de Android, la mayoría de los móviles de gama alta de los principales fabricantes ya admiten L1 y L5. Esas medidas permiten a aplicaciones científicas y de topografía procesar las dos frecuencias por su cuenta.

Para quien diseña el aparato, la doble frecuencia tiene un coste. La antena tiene que funcionar bien en dos bandas separadas casi 400 MHz. Hace falta una segunda cadena de radio y procesar una señal de 24 MHz en lugar de unos 2 MHz. El consumo adicional depende del chip y no se cuantifica aquí.

## Preguntas frecuentes

### ¿Es lo mismo que usar varias constelaciones?

No. Usar GPS, Galileo, GLONASS y BeiDou a la vez da más satélites y mejor geometría, pero cada uno puede seguir siendo de una sola frecuencia. La doble frecuencia significa recibir el mismo satélite en dos bandas. Un móvil puede tener una cosa sin la otra, aunque los recientes suelen tener las dos.

### ¿Por qué L5 y no L2?

La señal civil L2C, en 1227 MHz, también permite corregir la ionosfera junto con L1, según el servicio GPS. L5 tiene un código más rápido, más ancho de banda y una banda protegida para aviación. Además comparte frecuencia con Galileo E5a. Los motivos concretos de cada fabricante para elegir L5 no están documentados en las fuentes consultadas.

### ¿L5 llega mejor dentro de los edificios?

No se puede afirmar en general. L5 se emite con algo más de potencia y su frecuencia es algo menor. Pero en el estudio del Mi 8 el móvil recibía L5 más débil que L1, probablemente por la antena. Dentro de un edificio, las dos siguen siendo señales muy por debajo del ruido.

### ¿Si el móvil mide la ionosfera, ya no necesita el modelo que emite el satélite?

Para los satélites que recibe en dos frecuencias, no lo necesita para el retraso ionosférico. Para los que solo recibe en L1, sigue usándolo. Además, con señal débil o mucho ruido, el modelo puede dar mejor resultado que una combinación ruidosa.

## Fuentes

- [IS-GPS-705J — Interface Specification, NAVSTAR GPS Space Segment / User Segment L5 Interfaces (2022)](https://www.gps.gov/technical/icwg/IS-GPS-705J.pdf). Portadora de 1176,45 MHz y banda de 24 MHz (3.3.1.1); código de 10 230 chips a 10,23 Mchip/s (3.2.1.1, 3.3.2.2); componentes I5 con datos a 50 bit/s y Q5 sin datos (3.2.2); potencia mínima recibida (tabla 3-III); retraso diferencial de grupo (3.3.1.7.2); TGD, ISC y corrección ionosférica L1/L5 con γ = (154/115)² (20.3.3.3.1); modelo de una frecuencia con al menos un 50 % de reducción del error (20.3.3.3.1.3).
- [IS-GPS-200N — Interface Specification, NAVSTAR GPS Space Segment / Navigation User Segment Interfaces (2022)](https://www.gps.gov/technical/icwg/IS-GPS-200N.pdf). Código C/A de 1023 chips a 1,023 Mchip/s (3.2.1.3).
- [GPS Standard Positioning Service (SPS) Performance Standard, 5.ª edición (2020)](https://www.gps.gov/technical/ps/2020-SPS-performance-standard.pdf). Frecuencia L1 y potencia mínima recibida de −158,5 dBW para el C/A (apartado 2.2).
- [GPS.gov — New Civil Signals](https://gps.gov/systems/gps/modernization/civilsignals). L5 en banda reservada a seguridad aeronáutica, con más potencia y ancho de banda; L5 y L2C preoperativas; corrección ionosférica con doble frecuencia; uso limitado hasta disponer de 18 a 24 satélites.
- [European GNSS Service Centre — Galileo Open Service Signal-In-Space ICD, versión 2.1 (2023)](https://www.gsc-europa.eu/sites/default/files/sites/all/files/Galileo_OS_SIS_ICD_v2.1.pdf). Portadoras E1 (1575,420 MHz) y E5a (1176,450 MHz) (tabla 2) y ritmo de código de E5a de 10,230 Mchip/s (tabla 5).
- [ESA Navipedia — Ionospheric Delay](https://gssc.esa.int/navipedia/index.php/Ionospheric_Delay). Fórmula 40,3·STEC/f², definición de TECU, retraso del código y adelanto de la fase, términos de orden superior por debajo del 0,1 % y retrasos de hasta varias decenas de metros.
- [ESA Navipedia — Ionosphere-free Combination for Dual Frequency Receivers](https://gssc.esa.int/navipedia/index.php/Ionosphere-free_Combination_for_Dual_Frequency_Receivers). Combinación de código y fase que elimina el efecto ionosférico de primer orden.
- [ESA Navipedia — Klobuchar Ionospheric Model](https://gssc.esa.int/navipedia/index.php/Klobuchar_Ionospheric_Model). Modelo de una frecuencia emitido por GPS; reducción de en torno al 50 % del error eficaz.
- [U. Robustelli, V. Baiocchi y G. Pugliano, «Assessment of Dual Frequency GNSS Observations from a Xiaomi Mi 8 Android Smartphone and Positioning Performance Analysis», Electronics 8(1), 91 (2019)](https://www.mdpi.com/2079-9292/8/1/91). Chip Broadcom BCM47755; multitrayecto de código en L1/L5 y E1/E5a; posición con E1 frente a E5a; ciclo de 200/800 ms y antena de polarización lineal.
- [Android Developers — Raw GNSS measurements](https://developer.android.com/develop/sensors-and-location/sensors/gnss). Medidas en bruto obligatorias desde Android 10; soporte de varias frecuencias opcional; mayoría de móviles de gama alta con L1 y L5.
