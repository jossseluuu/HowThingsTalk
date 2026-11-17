---
title: "¿Cómo sabe el móvil qué porcentaje de batería le queda?"
section: "08_EnergiaYBaterias"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "Una batería no tiene un sensor de carga: el porcentaje es una estimación. Un chip llamado indicador de carga cuenta la corriente que entra y sale, y corrige la deriva de esa cuenta con la tensión que la celda muestra en reposo. Después predice cuánta carga podrá sacar antes de que la tensión caiga al mínimo con la carga y la temperatura actuales, y lo divide entre la capacidad aprendida."
prerequisites:
  - "Corriente, tensión y la ley de Ohm (V = I · R)"
  - "Qué es un miliamperio-hora (mAh)"
translation: "../../EN/08_PowerAndBatteries/01-HowYourPhoneKnowsItsBatteryPercentage.md"
---

# ¿Cómo sabe el móvil qué porcentaje de batería le queda?

## Pregunta y encuadre

La barra de la batería dice 37 %. Parece una lectura directa, como la de un termómetro. No lo es. Dentro de una celda de ion-litio no hay nada que mida «cuánta carga queda». La pregunta de este artículo es concreta: ¿de dónde sale ese número y por qué a veces se comporta de forma extraña?

El recorrido va desde lo que se puede medir (tensión, corriente y temperatura) hasta el número que pinta la pantalla. Pasa por el chip que hace la estimación, que en inglés se llama *fuel gauge* o *gas gauge* y aquí llamaremos **indicador de carga**. Los detalles salen de dos familias de indicadores documentadas por sus fabricantes: Impedance Track, de Texas Instruments, y ModelGauge m5, de Maxim (hoy Analog Devices). No es una guía para «calibrar» la batería ni para alargar su vida, y no compara modelos. Por qué la batería se agota antes con frío o con los años aparece solo como consecuencia del mecanismo.

La profundidad es intermedia. Basta con saber qué son la corriente y la tensión, la ley de Ohm y qué significa un miliamperio-hora.

## Respuesta corta

El porcentaje es una predicción: el indicador de carga cuenta la carga que sale, corrige esa cuenta con la tensión que la celda muestra en reposo y calcula cuánta carga podrá entregar todavía antes de que la tensión caiga al mínimo con el consumo y la temperatura de ese momento.

### Resumen de 60 segundos

Hay dos formas sencillas de estimar la carga, y las dos fallan solas.

1. **Mirar la tensión.** En reposo, la tensión de una celda depende de su estado de carga. Pero en cuanto el móvil consume, la tensión cae por la resistencia interna de la celda. Además, en la zona media la curva es muy plana: pocos milivoltios separan estados de carga bastante distintos.
2. **Contar la carga.** Un contador de culombios suma la corriente que entra y sale, segundo a segundo. Es preciso a corto plazo. Pero cualquier pequeño error de medida se acumula sin límite, y la cuenta deriva.

El indicador de carga combina las dos. El contador sigue los cambios rápidos. La tensión en reposo corrige la deriva cada vez que hay ocasión. Con dos lecturas en reposo y la carga contada entre ellas, el chip aprende además la capacidad real de la celda, que baja con los años.

El último paso es una predicción. El porcentaje no mide la carga química que queda, sino la que se podrá sacar antes de que la tensión baje al mínimo de funcionamiento. Esa cantidad depende del consumo y de la temperatura. Por eso el número puede moverse sin que haya cambiado la carga de la celda.

## Modelo mental

Piensa en un barco sin GPS. El navegante calcula su posición por estima: rumbo, velocidad y tiempo. Es una cuenta precisa a corto plazo, pero cada pequeño error se suma, y tras horas de navegación la posición calculada se aleja de la real. Cuando aparece un faro conocido en la costa, el navegante corrige la posición de golpe y sigue con la estima desde ahí.

El contador de culombios es la estima. La tensión en reposo es el faro.

La analogía se rompe en dos puntos. Primero, el faro solo se ve cuando el barco está casi parado: la tensión solo dice la verdad cuando la celda ha reposado un buen rato con muy poco consumo. Segundo, en el mar la distancia que queda hasta el puerto no depende de lo rápido que vayas. En una batería sí: lo que queda «hasta el puerto» es la carga que se podrá sacar antes de que la tensión caiga al mínimo, y esa carga es menor con consumos altos o con frío. El indicador no solo calcula dónde está, sino también hasta dónde llegará al ritmo actual.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

El indicador de carga es un chip pequeño, junto a la batería o dentro del circuito de carga. Hace cinco cosas en bucle.

1. **Mide.** Lee la tensión de la celda, la corriente que pasa por una resistencia de medida y la temperatura.
2. **Cuenta la carga.** Integra la corriente en el tiempo. Cada segundo resta o suma la carga que ha salido o entrado.
3. **Se ancla en la tensión de reposo.** Cuando la celda lleva tiempo con muy poco consumo, su tensión se acerca a la tensión en circuito abierto (OCV, *open circuit voltage*). Una tabla propia de la química de la celda convierte esa OCV en estado de carga. Ese valor reemplaza o corrige a la cuenta acumulada.
4. **Aprende la celda.** Compara cuánto cambia el estado de carga según la OCV con cuánta carga ha contado. De ahí saca la capacidad real. Compara también la tensión bajo carga con la OCV esperada, y de ahí saca la resistencia interna.
5. **Predice y comunica.** Simula la descarga a partir de ahora, con el consumo y la temperatura actuales, hasta la tensión mínima. La carga que sale de esa simulación, dividida entre la capacidad útil, es el porcentaje. El sistema operativo lo lee y lo muestra.

### 2. Qué hace cada componente

**La resistencia de medida y el contador de culombios.** La corriente pasa por una resistencia muy pequeña. El chip mide la diminuta caída de tensión en ella y la convierte en corriente. En el diseño de referencia del MAX17055 la resistencia es de 10 mΩ (hoja de datos, figura 1). Con 1 A, eso deja una caída de 10 mV. El paso mínimo de medida es de 1,5625 µV, que con 10 mΩ equivale a 156,25 µA. El error de *offset* es de ±1,5 µV como media a largo plazo, y el de ganancia de ±1 % de la lectura (hoja de datos, p. 4). Las cifras de corriente en amperios son cálculo de este artículo con la ley de Ohm.

Ese *offset* parece despreciable, pero se integra. ±1,5 µV sobre 10 mΩ son ±150 µA. Durante 24 horas suman 3,6 mAh, y en 30 días unos 108 mAh. Sobre una celda de 4000 mAh, elegida como ejemplo, sería un error de unos 2,7 puntos al mes. Es cálculo propio con un caso desfavorable: supone que el error tiene siempre el mismo signo. La hoja de datos lo resume así: el error de *offset* «no se puede eliminar», y la cuenta necesita corrección periódica (p. 15).

**La tensión y su tabla.** El chip mide la tensión de la celda con un error de ±7,5 mV a 25 °C y de ±20 mV entre −40 y +85 °C (MAX17055, p. 4). La tabla que traduce OCV en estado de carga depende de la química de la celda. Texas Instruments guarda tablas distintas para cada química, por ejemplo óxido de litio y cobalto con ánodo de carbono, y las identifica con un código de química (SLUA364B, apartado 2.2). Analog Devices afirma que la relación entre OCV y estado de carga se mantiene en gran medida independiente de la edad de la celda (nota técnica de ModelGauge m5 EZ). Esa estabilidad es lo que la convierte en buen faro.

**La resistencia interna.** Bajo consumo, la tensión en bornes es menor que la OCV:

```text
V_bornes = OCV − I · R
```

Cada 100 mΩ de resistencia interna, con 1 A de consumo, restan 100 mV. Es la ley de Ohm, no un dato medido de ninguna celda. La resistencia no es fija. Texas Instruments la modela con una dependencia exponencial de la temperatura y guarda una tabla de valores para 15 puntos de profundidad de descarga (SLUA364B, apartados 2.4 y 2.6). Apple explica lo mismo de forma cualitativa: la impedancia sube con la edad química de la batería y sube temporalmente con poca carga y con frío (Apple, «Batería y rendimiento del iPhone»).

**El anclaje en reposo.** En Impedance Track, el chip entra en reposo cuando la corriente baja de un umbral. Se recomienda que no pase de C/20, es decir, la corriente que vaciaría la batería en 20 horas. Toma lecturas de OCV tras 30 minutos de reposo y luego cada 100 segundos. Considera la celda bien relajada cuando la tensión varía menos de 4 µV por segundo, o tras un máximo de 5 horas. Con la batería entre 0 % y 80 % de profundidad de descarga, eso tarda en torno a 1 hora. Por encima del 80 %, de 3 a 4 horas, y más con frío. Antes de usar la lectura, el chip resta la pequeña caída que aún provoca la corriente de reposo (SLUA364B, apartados 2.2 y 2.3).

**El aprendizaje de la capacidad.** Con dos lecturas de OCV en reposo, una antes y otra después de una descarga, y la carga contada entre ellas, la capacidad sale de una división:

```text
Qmax = carga contada / (DOD2 − DOD1)
```

DOD es la profundidad de descarga, el complemento del estado de carga. Un ejemplo con números redondos: si la OCV indica un 20 % de profundidad, luego un 70 %, y entre medias se han contado 1500 mAh, la capacidad es 1500 / 0,5 = 3000 mAh. El ejemplo es de este artículo. Texas Instruments solo acepta la medida si se cumplen tres condiciones. La temperatura debe estar entre 10 y 40 °C. La carga contada debe superar el 37 % de la capacidad de diseño. Y ninguna de las dos lecturas puede caer entre 3737 y 3800 mV, una zona que el documento califica de «plana» y que depende de la química (SLUA364B, apartado 2.3). ModelGauge hace algo parecido de forma continua: compara el cambio de porcentaje que indica la OCV con la carga contada (guía de usuario del MAX17055, apartado 1.2.6).

**La mezcla continua.** ModelGauge m5 no espera al reposo para corregir. Al conectar la batería por primera vez, confía sobre todo en la estimación por tensión, porque el contador no sabe de dónde parte. Con los ciclos, el peso pasa al contador. A partir de ahí aplica una corrección pequeña y de tamaño fijo, hacia arriba o hacia abajo, según el signo de la diferencia entre la cuenta y el modelo de tensión. Según Analog Devices, la hace más de 200 000 veces al día, con carga y en reposo. El resultado no acumula la deriva del *offset* (nota técnica de ModelGauge m5 EZ; guía de usuario, apartado 1.2.1).

### 3. Parámetros que importan

| Parámetro | Valor | Escenario y fuente |
|---|---|---|
| Resistencia de medida | 10 mΩ | diseño de referencia del MAX17055 (hoja de datos, figura 1) |
| Paso de medida de corriente | 1,5625 µV (156,25 µA con 10 mΩ) | MAX17055 (hoja de datos, p. 4) |
| *Offset* de corriente | ±1,5 µV de media a largo plazo | MAX17055 (hoja de datos, p. 4) |
| Error de tensión | ±7,5 mV a 25 °C · ±20 mV de −40 a +85 °C | MAX17055 (hoja de datos, p. 4) |
| Consumo del propio indicador | 7 µA típicos en hibernación · 18 µA activo | MAX17055 (hoja de datos, p. 4) |
| Reposo antes de leer la OCV | 30 min, luego cada 100 s | Impedance Track (SLUA364B, 2.2) |
| Criterio de celda relajada | dV/dt < 4 µV/s o 5 h como máximo | Impedance Track (SLUA364B, 2.3) |
| Zona demasiado plana para aprender | 3737 a 3800 mV | química por defecto de Impedance Track (SLUA364B, 2.3) |
| Paso de la simulación de descarga | 4 % de profundidad | Impedance Track (SLUA364B, 2.6) |
| Tensión de vacío por defecto | 3,3 V | registro VEmpty del MAX17055 (hoja de datos, p. 12) |
| Precisión publicada | 97 % de las pruebas con menos de un 3 % de error | MAX17055 tras el primer ciclo, a +20 y +40 °C, más de 300 baterías (hoja de datos, p. 12) |

La precisión de la última fila tiene condiciones. Son pruebas a temperatura ambiente o superior, con descargas de más de 3 horas. Analog Devices excluye además químicas cuya curva de OCV se aleja mucho de las habituales (nota técnica de ModelGauge m5 EZ).

### 4. Diagrama o flujo

```text
            ┌──────────────── celda de ion-litio ────────────────┐
            │                                                     │
   tensión V│              corriente I (por 10 mΩ)     temperatura T
            ▼                       ▼                         ▼
   ┌─────────────────┐    ┌──────────────────┐     ┌─────────────────────┐
   │ ¿en reposo?     │    │ contador de      │     │ resistencia interna │
   │ dV/dt pequeño   │    │ culombios        │     │ R(profundidad, T)   │
   │ → OCV → tabla   │    │ Q += I · Δt      │     │ = (OCV − V) / I     │
   └────────┬────────┘    └────────┬─────────┘     └──────────┬──────────┘
            │ corrige la deriva    │                          │
            └──────────►  estado de carga actual  ◄───────────┘
                                   │
                                   ▼
            simulación: V = OCV − I · R, paso a paso, hasta V_mínima
                                   │
                                   ▼
          carga útil restante / capacidad útil × 100  =  37 %
                                   │
                                   ▼
               registro del chip → núcleo → sistema → pantalla
```

Dónde queda el «0 %» dentro de la carga química:

```text
carga química total (Qmax)
├────────── ya usada ──────────┼───── útil a este ritmo ─────┼── no extraíble a este ritmo ──┤
                               ▲                             ▲
                             ahora              OCV − I·R toca la tensión mínima: 0 %

Con más consumo o con más frío, I·R crece y la segunda marca se desplaza a la izquierda.
```

La carga que queda a la derecha de la segunda marca existe, pero no se puede usar a ese ritmo sin que la tensión caiga por debajo del mínimo.

## Límites, excepciones y casos reales

**Lo que muestra la pantalla no es la capacidad química.** Texas Instruments distingue dos capacidades. Qmax es la capacidad química total. La capacidad útil (FCC) es la carga que sale hasta la tensión de corte, menos una reserva. La útil es menor con consumos altos y con frío, porque la caída I · R adelanta el momento de llegar al corte (SLUA364B, apartado 1). El porcentaje se calcula sobre la útil: carga útil restante × 100 / FCC (apartado 2.7). Así, el mismo estado químico puede dar porcentajes distintos según la carga y la temperatura.

**El número puede subir sin cargar.** Si baja el consumo o sube la temperatura, la caída I · R se reduce y la simulación encuentra más carga útil. La guía del MAX17055 lo dice de forma explícita: la capacidad disponible puede cambiar de golpe e incluso aumentar durante la descarga (apartado 1.2.2). Para que eso no se vea como saltos, el chip publica una versión filtrada que converge hacia el valor real poco a poco (apartado 1.2.2).

**El final se fuerza a cuadrar.** Cerca del vacío, ModelGauge ajusta la velocidad a la que baja el porcentaje para que llegue a 0 % justo cuando la tensión llega al mínimo. Además, puede retener el porcentaje en un valor bajo (un 1 % por defecto) hasta que la tensión cruza el umbral de vacío (guía de usuario, apartados 1.2.3 y 1.2.4). Sin estos ajustes, el móvil se apagaría con un 5 % en pantalla o se quedaría días en 0 %.

**Las cargas a pulsos complican el reposo.** Un móvil rara vez está quieto del todo: la radio despierta a ráfagas. Texas Instruments documenta un móvil GSM con pulsos de 1 A durante 0,48 ms y 72 mA durante 4,76 ms. Para estos casos el reposo no exige corriente cero, sino una corriente baja, por debajo de C/20. Un pico breve no saca al chip del reposo: la corriente debe mantenerse alta más de 1 segundo (SLUA450A, apartados 2.1 y 3.1). En esa prueba, el error rondó el 4 % al principio y se mantuvo dentro del 2 % entre el 80 % y el 0 % (apartado 3.1). Es una sola prueba, no una garantía general.

**No todos los móviles cuentan culombios.** Algunos indicadores solo usan la tensión y un modelo de la celda. La documentación de Android cita el Nexus 5, con un MAX17048 sin contador de culombios, que solo ofrece el estado de carga con una resolución de 1/256 % (AOSP, «Measure device power»). En los que sí cuentan, Android expone la carga restante en µAh y la corriente en µA. Pero la resolución y la frecuencia de actualización dependen del chip, de la resistencia de medida y de su configuración. El Nexus 6, por ejemplo, actualiza la corriente cada 175,8 ms con pasos de 156,25 µA (misma fuente).

**Las químicas planas son más difíciles.** La guía del MAX17055 señala que la curva de OCV del fosfato de hierro y litio (LiFePO4) es mucho más plana que la de las químicas de cobalto y tiene dos mesetas. Eso hace al algoritmo más sensible a cómo interpreta la tensión (apartado 1.2.8). Cuanto más plana es la curva, peor faro es la tensión.

**El modelo envejece con la celda.** Texas Instruments reconoce que los métodos puros de cuenta de carga y de tensión pierden precisión con el envejecimiento (SLUA364B, resumen). Su propio margen de error estimado crece con cada ciclo sin un nuevo aprendizaje de capacidad (apartado 2.8). Los algoritmos combinados lo compensan reaprendiendo capacidad y resistencia, pero necesitan ocasiones: reposos largos o cargas completas.

## Consecuencias prácticas

Para quien usa el móvil, el porcentaje es una previsión, no un depósito con una varilla. Que suba un punto al dejar un juego, o que baje rápido al abrir la cámara con frío, no indica una avería. Es el indicador recalculando la carga útil con un consumo y una temperatura nuevos.

Los apagones con porcentaje restante tienen la misma raíz. Apple explica que, con impedancia alta, la tensión cae más bajo un pico de consumo. Si cae por debajo del mínimo que necesitan los componentes, el sistema se apaga para protegerlos. Desde el punto de vista del aparato el apagado es intencionado, aunque al usuario le resulte inesperado. Es más probable con poca carga, con la batería envejecida o con frío (Apple, «Batería y rendimiento del iPhone»). El indicador predice con el consumo medio; un pico breve y fuerte puede adelantarse a esa predicción.

Para quien diseña el aparato, la precisión empieza en el hardware. Un *offset* pequeño en la medida de corriente se integra durante semanas. Por eso la resistencia de medida, su colocación y el propio consumo del indicador importan. Y por eso el algoritmo necesita una fuente de verdad que no derive, como la tensión en reposo, y una química con una curva de OCV bien caracterizada.

## Preguntas frecuentes

### ¿Es lo mismo el porcentaje que la «capacidad máxima» o salud de la batería?

No. El porcentaje compara la carga útil que queda con la capacidad útil actual. La capacidad máxima compara la capacidad actual con la que tenía la batería nueva (Apple, «Batería y rendimiento del iPhone»). Una batería envejecida puede marcar 100 % y durar menos: está llena, pero su 100 % es más pequeño.

### ¿Hay que descargar la batería a 0 % para «calibrar» el indicador?

En los dos algoritmos revisados, no es un requisito. Impedance Track aprende la capacidad con dos lecturas en reposo separadas por una descarga de más del 37 % de la capacidad de diseño (SLUA364B, apartado 2.3). ModelGauge m5 se anuncia como un algoritmo que no necesita estados de vacío, de lleno ni de reposo (hoja de datos del MAX17055, p. 1). Si un móvil concreto usa otro método, su comportamiento queda **no verificado** aquí.

### ¿Por qué el porcentaje sube un poco si dejo el móvil un rato?

Porque, al cesar el consumo, la tensión se recupera hacia la OCV y desaparece la caída I · R. El indicador recalcula y encuentra más carga útil. La carga química no ha aumentado; solo ha cambiado cuánta se podrá extraer.

### ¿Es lo mismo un contador de culombios que un medidor de energía?

No. El contador suma carga, en mAh. La energía, en Wh, es la carga multiplicada por la tensión a la que se entrega. Android ofrece las dos magnitudes, la carga en µAh y la energía en nWh. En el Nexus 6 y el Nexus 9, la energía se obtiene multiplicando la carga por una tensión nominal fija de 3,7 V (AOSP, «Measure device power»).

## Fuentes

- [Texas Instruments — Theory and Implementation of Impedance Track Battery Fuel-Gauging Algorithm in bq20zxx Product Family. Informe de aplicación SLUA364B, noviembre de 2005, revisado en diciembre de 2006](https://ti.com/lit/SLUA364). Capacidad química y útil (apartado 1), OCV y tabla por química (2.2), reposo, criterio de 4 µV/s y aprendizaje de Qmax con sus condiciones (2.3), resistencia y su dependencia de la temperatura (2.4), simulación de descarga en pasos del 4 % (2.6), porcentaje continuo (2.7), error estimado (2.8) y limitaciones con el envejecimiento (resumen).
- [Texas Instruments — Theory and Implementation of Impedance Track Battery Fuel-Gauging Algorithm in bq2750x Family. Nota de aplicación SLUA450A, enero de 2008, revisada en noviembre de 2022](https://ti.com/lit/pdf/slua450). Entrada y salida del reposo (apartado 2.1) y prueba con un móvil GSM bajo carga a pulsos (3.1).
- [Analog Devices — Accurate Fuel Gauging Without Battery Characterization Using ModelGauge m5 EZ Algorithm. Nota técnica, agosto de 2019](https://www.analog.com/en/resources/technical-articles/accurate-fuel-gauging-without-battery-characterization-using-modelgauge-m5-ez-algorithm.html). Mezcla de contador y modelo de tensión, más de 200 000 correcciones al día, independencia de la OCV respecto a la edad y resultados de precisión.
- [Maxim Integrated (Analog Devices) — MAX17055 7µA 1-Cell Fuel Gauge with ModelGauge m5 EZ. Hoja de datos, revisión 0, enero de 2017](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX17055.pdf). Características eléctricas de medida y consumo (p. 4), resistencia de medida de referencia (figura 1), VEmpty y precisión publicada (p. 12), deriva del contador y mezcla (p. 15).
- [Maxim Integrated (Analog Devices) — MAX17055 ModelGauge m5 EZ User Guide. Revisión 0, diciembre de 2016](https://wcm-cce.cldnet.analog.com/media/en/technical-documentation/user-guides/max17055-user-guide.pdf). Mezcla (apartado 1.2.1), compensación de vacío y filtrado (1.2.2), convergencia y retención cerca del vacío (1.2.3 y 1.2.4), aprendizaje de capacidad (1.2.6) y LiFePO4 (1.2.8).
- [Android Open Source Project — Measure device power](https://source.android.com/docs/core/power/device). Propiedades de batería en µAh, µA y nWh, dependencia del hardware del indicador, resolución y periodo de actualización del Nexus 6 y 9, *offset* del contador y caso del Nexus 5 sin contador de culombios.
- [Android Developers — BatteryManager](https://developer.android.com/reference/android/os/BatteryManager). Definición de BATTERY_PROPERTY_CAPACITY, CHARGE_COUNTER, CURRENT_NOW, CURRENT_AVERAGE y ENERGY_COUNTER.
- [Apple — iPhone battery and performance («Batería y rendimiento del iPhone»), junio de 2026](https://support.apple.com/en-us/101575). Impedancia, edad química, frío, poca carga, apagados para proteger los componentes y capacidad máxima.
