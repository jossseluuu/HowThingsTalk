---
title: "¿Cómo calcula un receptor GPS su posición a partir de las señales de los satélites?"
section: "05_Geolocalizacion"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "Cada satélite GPS emite su hora y su órbita. El receptor mide cuánto tardan esas señales en llegar, las convierte en distancias y resuelve cuatro incógnitas a la vez: las tres coordenadas y el error de su propio reloj. Por eso necesita al menos cuatro satélites, no tres."
prerequisites:
  - "Frecuencia, longitud de onda y decibelios (artículo 00·01)"
  - "Relación señal-ruido (artículo 00·02)"
  - "Multitrayecto (artículo 00·03)"
translation: "../../EN/05_Positioning/01-HowGPSComputesYourPosition.md"
---

# ¿Cómo calcula un receptor GPS su posición a partir de las señales de los satélites?

## Pregunta y encuadre

Un móvil o un reloj con GPS no envía nada a los satélites. Solo escucha. Aun así, en unos segundos sabe dónde está con un error de pocos metros. La pregunta es cómo convierte unas señales de radio muy débiles en una posición.

El artículo explica el cálculo básico de un receptor GPS de uso civil: qué emite cada satélite, cómo se mide el tiempo de viaje de la señal y por qué hacen falta cuatro satélites. No cubre cómo se localiza un móvil con el GPS apagado (por WiFi o por la red móvil), ni la ayuda que la red le da al GPS para arrancar antes, ni las técnicas de precisión centimétrica. Los efectos de la relatividad se mencionan, pero no se desarrollan. Otros sistemas de satélites (Galileo, GLONASS, BeiDou) siguen el mismo principio, pero aquí se usan las cifras del GPS.

Conviene conocer los decibelios ([artículo 00·01](../00_Fundamentos/01-PorQueMasFrecuenciaLlegaMenosLejos.md)) y la relación señal-ruido ([artículo 00·02](../00_Fundamentos/02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md)). La profundidad es intermedia: hay una ecuación, explicada paso a paso.

## Respuesta corta

El receptor mide cuánto tarda en llegar la señal de cada satélite, la convierte en una distancia y busca el único punto y la única corrección de su reloj que encajan con al menos cuatro de esas distancias a la vez.

### Resumen de 60 segundos

Cada satélite GPS lleva relojes atómicos. Emite una señal que dice, en la práctica, «soy el satélite N, esta es mi hora exacta y esta es mi órbita». La señal viaja a la velocidad de la luz y tarda unos 70 a 85 milisegundos en llegar al suelo.

El receptor compara la hora que trae la señal con la hora de su propio reloj. La diferencia, multiplicada por la velocidad de la luz, da una distancia. Si el reloj del receptor fuera perfecto, tres distancias bastarían para fijar un punto. Pero su reloj es de cuarzo barato y puede estar desviado en microsegundos o más. Un microsegundo de error equivale a unos 300 metros.

La solución es tratar ese error de reloj como una incógnita más. Con cuatro satélites hay cuatro ecuaciones y cuatro incógnitas: latitud, longitud, altura y desfase del reloj. El receptor las resuelve por aproximaciones sucesivas. Con más satélites, promedia y descarta medidas malas.

## Modelo mental

Imagina que estás en un campo a oscuras. Varios campanarios repartidos por el horizonte dan la hora con una campanada exacta, y tú sabes dónde está cada uno. El sonido tarda en llegar: si oyes la campanada de las doce 3 segundos tarde, ese campanario está a unos 1000 metros. Con varios campanarios puedes situarte.

Ahora supón que tu reloj de pulsera adelanta un poco y no sabes cuánto. Todas tus distancias saldrán mal en la misma cantidad. Aun así, con un campanario más puedes averiguar a la vez dónde estás y cuánto adelanta tu reloj. Solo hay una combinación que hace encajar todas las medidas.

La analogía se rompe en tres puntos. Los «campanarios» se mueven a casi 4 km/s, así que el receptor necesita saber dónde estaba cada uno en el instante exacto de la emisión. La señal no se oye por encima del ruido: llega por debajo de él, y hay que extraerla con un código conocido. Y la «campanada» no es un sonido aislado: es un patrón continuo que el receptor alinea con una copia propia para medir el retraso.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

El sistema tiene tres partes. El segmento espacial lo forman los satélites. El de control, las estaciones en tierra que vigilan las órbitas y los relojes y les suben correcciones. El de usuario, los receptores, que solo reciben.

Según gps.gov, Estados Unidos se compromete a mantener al menos 24 satélites operativos el 95 % del tiempo. Vuelan en órbita media, a unos 20 200 km de altura, repartidos en seis planos orbitales. Según el servicio de guardacostas de EE. UU. (NAVCEN), las órbitas son circulares, con una inclinación de 55° y un periodo de 11 horas y 58 minutos.

Un cálculo de posición sigue estos pasos:

1. **Adquisición.** El receptor busca qué satélites están en el cielo. Para cada uno prueba retrasos de código y desplazamientos Doppler hasta que una correlación destaca sobre el ruido.
2. **Seguimiento.** Una vez enganchado, mantiene alineada su copia del código con la señal recibida. Ese alineamiento es la medida de tiempo.
3. **Lectura del mensaje de navegación.** Decodifica los datos que el satélite emite a 50 bit/s: su reloj, sus correcciones y su órbita precisa (las efemérides).
4. **Pseudodistancias.** Para cada satélite calcula (hora de recepción − hora de emisión) × c.
5. **Solución.** Con cuatro o más pseudodistancias y las posiciones de los satélites resuelve posición y desfase de reloj.

### 2. Qué hace cada componente

**Los relojes de los satélites.** Son la referencia de todo el sistema. El segmento de control mide su deriva y la transmite como parámetros de corrección dentro del mensaje de navegación. El receptor aplica esa corrección antes de calcular.

**El código C/A.** Cada satélite modula la portadora L1 (1575,42 MHz) con un código pseudoaleatorio propio de 1023 chips que se repite cada milisegundo. Su ritmo es de 1,023 millones de chips por segundo. Los códigos de satélites distintos se parecen muy poco entre sí. Eso permite que todos compartan la misma frecuencia: el receptor separa a cada satélite multiplicando la señal por la copia de su código.

**El mensaje de navegación.** Va superpuesto al código a 50 bit/s. Según el estándar de prestaciones SPS, se organiza en tramas de cinco subtramas, y cada trama se repite cada 30 s. Contiene las correcciones de reloj, las efemérides del propio satélite y un almanaque, que es una descripción aproximada de toda la constelación. Las efemérides sirven para calcular dónde estaba el satélite cuando emitió.

**El correlador del receptor.** Genera una réplica del código y la desliza en el tiempo. Cuando la réplica coincide con la señal recibida, la correlación da un pico. La posición de ese pico dentro del milisegundo del código mide el retraso con una fracción de chip. Un chip dura 1/1,023 MHz ≈ 0,98 µs, que equivale a unos 293 m de recorrido (calculado). Por eso el receptor necesita interpolar dentro del chip para bajar a metros.

**El reloj del receptor.** Es un oscilador de cuarzo. No es preciso, pero sí bastante estable durante el cálculo. Su error es igual para todos los satélites medidos en el mismo instante. Esa propiedad es la que hace posible despejarlo.

**El algoritmo de navegación.** Resuelve el sistema de ecuaciones, normalmente por mínimos cuadrados iterativos o con un filtro que combina medidas sucesivas.

### 3. Parámetros que importan

**La ecuación.** Para cada satélite *i*, con posición conocida (xᵢ, yᵢ, zᵢ):

```text
ρᵢ = √[(xᵢ − x)² + (yᵢ − y)² + (zᵢ − z)²] + c·b
```

- ρᵢ es la pseudodistancia medida, en metros.
- (x, y, z) es la posición del receptor: tres incógnitas.
- b es el desfase del reloj del receptor, en segundos: la cuarta incógnita.
- c = 299 792 458 m/s, la velocidad de la luz (valor exacto por definición del SI).

Se llama *pseudo*distancia porque incluye el término c·b. No es la distancia real hasta que se conoce b. Con cuatro satélites hay cuatro ecuaciones y una única solución útil. El receptor parte de una posición aproximada, linealiza el sistema, corrige y repite. Suele converger en pocas iteraciones.

**Tiempos y distancias (calculado a partir de la altura de 20 200 km y un radio terrestre medio de 6371 km).**

| Situación | Distancia al satélite | Tiempo de viaje |
|---|---|---|
| Satélite en el cénit | 20 200 km | 67 ms |
| Satélite en el horizonte | unos 25 800 km | 86 ms |
| Error de reloj de 1 µs | — | equivale a 300 m |
| Error de reloj de 1 ns | — | equivale a 30 cm |

**Lo débil que es la señal.** El estándar SPS garantiza una potencia mínima recibida de −158,5 dBW para el código C/A en L1. Es una potencia de unos 1,4 × 10⁻¹⁶ W (calculado). El ruido térmico a 290 K es de unos −204 dBW por cada hercio. En los 2 MHz aproximados que ocupa la señal, el ruido suma unos −141 dBW (calculado, sin contar el ruido propio del receptor). La señal queda unos 17 dB por debajo del ruido.

**Por qué aun así se recupera.** El código va a 1,023 Mchip/s y los datos a 50 bit/s. La relación es de 20 460 chips por bit, unos 43 dB (calculado). Al correlar con el código correcto, el receptor concentra la energía del bit y el ruido se queda repartido. Esa ganancia de procesado convierte una señal enterrada en el ruido en una medida utilizable. Es el mismo balance de señal-ruido del [artículo 00·02](../00_Fundamentos/02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md), pagado con una tasa de datos ínfima.

**Precisión obtenida.** El estándar SPS de 2020 indica que, con la precisión de señal de 2018, los receptores bien diseñados lograban 3 m o menos en horizontal y 5 m o menos en vertical el 95 % del tiempo. Es una observación de rendimiento, no el valor mínimo garantizado. Los valores garantizados del estándar no se han podido verificar para este artículo.

**Geometría.** La misma calidad de medida da posiciones peores si los satélites están agrupados en una zona del cielo. Las esferas se cortan en ángulos rasantes y un pequeño error de distancia mueve mucho el punto de corte. Los receptores expresan este efecto con la dilución de precisión (DOP): un número que multiplica el error de distancia para dar el error de posición. Cuanto más repartidos están los satélites, menor es.

### 4. Diagrama o flujo

```text
 Satélite i (reloj atómico + efemérides)
   │  L1 = 1575,42 MHz
   │  código C/A propio (1023 chips, 1 ms) + datos a 50 bit/s
   ▼
 Viaje de 67–86 ms  ──►  llega a −158,5 dBW, ~17 dB bajo el ruido
   │
   ▼
 Receptor
   ├─ Correlador: desliza la réplica del código ─► pico = retraso medido
   ├─ Decodifica el mensaje ─► hora de emisión + posición del satélite
   └─ Su reloj de cuarzo ─► hora de recepción (con desfase b desconocido)
   │
   ▼
 ρᵢ = (t_recepción − t_emisión) × c      (para cada satélite)
   │
   ▼
 4 o más ecuaciones  ─►  mínimos cuadrados iterativos
   │
   ▼
 (x, y, z) + b   ─►  posición y, de regalo, la hora exacta
```

## Límites, excepciones y casos reales

**La ionosfera y la troposfera.** La señal no viaja siempre a c. Al cruzar la ionosfera se retrasa en función de la densidad de electrones, que cambia con la hora del día y la actividad solar. La troposfera añade otro retraso, que depende de la presión y la humedad. Un receptor de una sola frecuencia corrige la ionosfera con un modelo aproximado que el propio satélite emite. Lo que el modelo no capta queda como error. Los receptores de doble frecuencia pueden medir este retraso, porque depende de la frecuencia.

**Multitrayecto.** En una ciudad la señal rebota en fachadas. Si llega solo el rebote, el receptor mide un camino más largo que el real. Si llegan los dos, el pico de correlación se deforma y el retraso medido se desplaza. Es el mismo fenómeno del [artículo 00·03](../00_Fundamentos/03-PorQueLaCoberturaCambiaEnPocosCentimetros.md), pero aquí no solo cambia la potencia: falsea la distancia. Por eso la posición «salta» de acera en calles estrechas.

**Interiores.** Con −158,5 dBW como punto de partida, cada pared o techo resta decibelios a una señal que ya está bajo el ruido. Dentro de un edificio el receptor suele perder parte de los satélites o todos.

**Arranque en frío.** Sin almanaque ni efemérides, el receptor debe buscar a ciegas y luego leer el mensaje de navegación a 50 bit/s. Una trama completa tarda 30 s. Por eso un receptor aislado, sin ayuda externa, puede tardar decenas de segundos en dar la primera posición. Los móviles reducen mucho ese tiempo descargando esos datos por la red, un mecanismo que queda fuera de este artículo.

**Relojes de los satélites y relatividad.** Los relojes en órbita no marchan al mismo ritmo que en el suelo, por la velocidad del satélite y por la diferencia de gravedad. El sistema compensa estos efectos en el diseño y en las correcciones que emite. Sin esa compensación, el error crecería día a día. Las cifras exactas no se han verificado para este artículo.

**Cuatro satélites son el mínimo, no lo normal.** Con la constelación actual suele haber bastantes más en vista a cielo abierto. Los satélites de más sirven para promediar, para detectar una medida incoherente y para mejorar la geometría.

## Consecuencias prácticas

El receptor GPS es pasivo. No emite nada hacia los satélites, así que el sistema admite un número ilimitado de usuarios y los satélites no saben quién los escucha. La privacidad de la posición depende de qué hace después el aparato con ella, no del GPS.

Un receptor con posición resuelta también conoce la hora con mucha precisión, porque despeja b. Por eso el GPS se usa para sincronizar redes, no solo para navegar.

La precisión depende del cielo visible. A cielo abierto da pocos metros. Entre edificios altos, la geometría empeora y aparecen rebotes. Dentro de casa, la señal puede no llegar. Por eso los móviles combinan el GPS con otras fuentes de posición.

Para quien diseña un aparato, la antena y el primer amplificador son críticos. La señal ya llega bajo el ruido. Cualquier decibelio de ruido añadido por el receptor se resta directamente del margen.

## Preguntas frecuentes

### ¿El GPS es lo mismo que la triangulación?

No exactamente. La triangulación mide ángulos. El GPS mide distancias, a partir de tiempos, y resuelve además el error de reloj. El nombre correcto del principio es trilateración con un término de reloj.

### ¿Por qué no bastan tres satélites?

Bastarían si el receptor tuviera un reloj atómico sincronizado con el sistema. Con un reloj de cuarzo, el desfase desconocido es una cuarta incógnita. Algunos receptores usan tres satélites si fijan la altura por otro medio, por ejemplo sobre el mar, pero eso es una restricción añadida, no una medida.

### ¿El GPS necesita datos móviles o internet?

No para funcionar. Toda la información necesaria viene en la señal de los satélites. Los datos móviles solo aceleran el arranque y ayudan cuando la señal es débil.

### ¿Los satélites saben dónde estoy?

No. Solo emiten. El cálculo se hace entero en el receptor.

## Fuentes

- [GPS.gov — Space Segment](https://www.gps.gov/systems/gps/space/). Compromiso de 24 satélites operativos el 95 % del tiempo, seis planos orbitales y altura de unos 20 200 km.
- [U.S. Coast Guard NAVCEN — Global Positioning System Overview](https://navcen.uscg.gov/global-positioning-system-overview). Órbitas circulares a 20 200 km, inclinación de 55° y periodo de 11 h 58 min.
- [IS-GPS-200N — Interface Specification, NAVSTAR GPS Space Segment / Navigation User Segment Interfaces (2022)](https://www.gps.gov/technical/icwg/IS-GPS-200N.pdf). Definición del código C/A: código Gold de 1 ms y 1023 chips a 1,023 Mchip/s (apartado 3.2.1.3).
- [GPS Standard Positioning Service (SPS) Performance Standard, 5.ª edición (2020)](https://www.gps.gov/technical/ps/2020-SPS-performance-standard.pdf). Frecuencia L1 de 1575,42 MHz, datos LNAV a 50 bit/s, estructura de tramas, potencia mínima recibida de −158,5 dBW (apartado 2.2) y precisión observada de 3 m horizontal y 5 m vertical al 95 % (resumen ejecutivo).
- [BIPM — El Sistema Internacional de Unidades (SI), 9.ª edición](https://www.bipm.org/en/publications/si-brochure). Valor exacto de la velocidad de la luz, c = 299 792 458 m/s.
