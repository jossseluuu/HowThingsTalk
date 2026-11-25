---
title: "¿Cómo ayuda la red móvil al GPS del teléfono a encontrar tu posición en pocos segundos?"
section: "05_Geolocalizacion"
language: "es"
status: "published"
order: 3
depth: "intermediate"
summary: "Un receptor GPS aislado tiene que buscar a ciegas los satélites y después leer su órbita a 50 bit/s, lo que lleva decenas de segundos. El móvil se ahorra casi todo ese trabajo: la red le envía qué satélites hay en el cielo, con qué Doppler llegarán, sus órbitas y la hora aproximada. Así busca en una ventana pequeña, no necesita decodificar el mensaje y puede trabajar con señales mucho más débiles."
prerequisites:
  - "Frecuencia, longitud de onda y decibelios (artículo 00·01)"
  - "Relación señal-ruido (artículo 00·02)"
  - "Cómo calcula el GPS una posición (artículo 05·01)"
translation: "../../EN/05_Positioning/03-HowTheNetworkHelpsGPSFindYouFast.md"
---

# ¿Cómo ayuda la red móvil al GPS del teléfono a encontrar tu posición en pocos segundos?

## Pregunta y encuadre

Un receptor GPS de montaña recién encendido puede tardar bastante en dar su primera posición. El móvil, en cambio, suele situarte en unos segundos, incluso junto a una ventana. Los dos escuchan los mismos satélites. La pregunta es qué hace distinto el móvil.

El artículo explica el GPS asistido (A-GPS, o A-GNSS cuando se usan varias constelaciones): qué le falta a un receptor que arranca solo, qué datos le envía la red, cómo los usa para buscar menos y escuchar señales más débiles, y quién termina haciendo el cálculo. No cubre la localización por WiFi o por celdas sin GPS, que está en el [artículo 05·02](02-ComoTeLocalizaElMovilSinGPS.md). Tampoco cubre las correcciones de precisión centimétrica (RTK, SSR), aunque viajan por los mismos protocolos.

Hace falta conocer el principio del GPS del [artículo 05·01](01-ComoCalculaElGPSTuPosicion.md): código C/A, correlación, pseudodistancias y mensaje de navegación. También ayudan los decibelios ([artículo 00·01](../00_Fundamentos/01-PorQueMasFrecuenciaLlegaMenosLejos.md)) y la relación señal-ruido ([artículo 00·02](../00_Fundamentos/02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md)). La profundidad es intermedia. Las cuentas son multiplicaciones, y se indican como «calculado».

## Respuesta corta

La red le da al móvil por adelantado lo que un GPS aislado tendría que averiguar escuchando: qué satélites buscar, dónde buscarlos, sus órbitas y la hora.

### Resumen de 60 segundos

Un receptor que arranca sin ninguna información no sabe qué satélites tiene encima ni con qué desplazamiento Doppler le llegan. Tiene que probar combinaciones de satélite, retraso de código y frecuencia hasta que una destaca sobre el ruido. Después debe leer las efemérides del propio satélite, que llegan a 50 bit/s. Solo esa lectura puede llevar hasta 36 s.

El móvil pide ayuda a un servidor de localización de la red. El servidor tiene receptores de referencia que siguen todos los satélites sin parar. Le envía la hora aproximada, una posición de partida y, para cada satélite visible, su Doppler esperado y sus órbitas. Con eso el móvil busca en una ventana mucho más estrecha. Y no necesita decodificar el mensaje de navegación: solo tiene que medir el código.

Lo segundo es lo que más importa en interiores. Leer datos exige más señal que detectar el código. Sin tener que leerlos, el móvil puede fijar posición con señales unas 18 dB más débiles que la mínima garantizada a cielo abierto. Las pruebas de 3GPP exigen una posición en 20 s como máximo a ese nivel.

## Modelo mental

Imagina que buscas a un amigo en un estadio lleno, de noche y sin saber en qué grada está. Recorres sección por sección con unos prismáticos. Cuando por fin lo encuentras, te grita su número de asiento letra a letra, muy despacio, entre el ruido del público.

Ahora imagina que te escribe un mensaje: «Tribuna norte, fila 12, llevo gorra roja». Vas directo a una zona pequeña. Y ya no necesitas oírle decir dónde está. Solo tienes que reconocerle, que es mucho más fácil que entender una frase entre gritos.

La analogía se rompe en dos puntos. Tu amigo se mueve poco, pero un satélite GPS se acerca o se aleja de ti a cientos de metros por segundo, y eso cambia la frecuencia con la que llega su señal. Y el mensaje no te dice dónde está tu amigo ahora mismo, sino una descripción de su trayectoria. El móvil tiene que calcular con ella dónde estaba el satélite en el instante exacto en que emitió.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

**El arranque en frío de un receptor aislado.** Sin memoria previa, el receptor no sabe la hora, ni su posición, ni qué satélites están sobre el horizonte. Ocurren tres cosas en serie.

1. **Búsqueda a ciegas.** Para cada código de satélite posible prueba todos los retrasos de código y una gama amplia de frecuencias. Es la adquisición del [artículo 05·01](01-ComoCalculaElGPSTuPosicion.md), pero sin pistas.
2. **Lectura del mensaje.** Una vez enganchado, decodifica el mensaje de navegación. Según la especificación IS-GPS-200, va a 50 bit/s en subtramas de 300 bits, es decir, 6 s por subtrama. El reloj del satélite viaja en la subtrama 1 y las efemérides en las subtramas 2 y 3. Hay que esperar a recibir las tres. Blay, Wang y Akos (2021) señalan que puede llevar hasta 36 s.
3. **Cálculo.** Con cuatro o más satélites leídos, resuelve posición y desfase de reloj.

**El arranque asistido.** El móvil pide ayuda antes o en paralelo a la búsqueda.

1. **Petición.** El móvil solicita datos de asistencia a un servidor de localización. En LTE el servidor se llama E-SMLC y el protocolo es LPP (3GPP TS 37.355).
2. **Asistencia.** El servidor envía la hora de referencia, una posición aproximada, los modelos de reloj y órbita de cada satélite, el modelo de ionosfera y una ayuda de adquisición: Doppler esperado y ventanas de búsqueda.
3. **Búsqueda dirigida.** El móvil busca solo los satélites visibles, en una ventana estrecha de frecuencia y, si tiene la hora fina, también de retraso.
4. **Medida.** Mide el retraso de código y el Doppler de cada satélite. No necesita esperar al mensaje de navegación.
5. **Cálculo.** O lo hace el propio móvil con las órbitas recibidas (modo *UE-based*), o envía las medidas al servidor y lo calcula él (modo *UE-assisted*). TS 36.305 define los dos modos según qué nodo hace el cálculo.

### 2. Qué hace cada componente

**La red de receptores de referencia.** TS 36.305 describe el A-GNSS como apoyado en una red de receptores GNSS de referencia que funciona de forma continua. Esos receptores, a cielo abierto, sí leen el mensaje de todos los satélites. El servidor reparte después ese contenido.

**El servidor de localización.** En el plano de control de LTE es el E-SMLC. Decide qué método usar y puede combinar resultados de varios métodos. La asistencia también puede viajar por el plano de usuario, como datos IP normales: es SUPL, de la Open Mobile Alliance. En ese caso el servidor se llama SLP y el móvil, SET. TS 36.305 indica que LPP funciona sobre los dos caminos.

**La hora de referencia.** Hay dos calidades. La hora gruesa (*coarse time*) tiene un error de segundos. En las pruebas de 3GPP (TS 36.171) es de ±2 s. La hora fina (*fine time*) relaciona la temporización de la celda con la hora GPS con un error de microsegundos: ±10 µs en esas pruebas. No todos los móviles la admiten.

**La posición de referencia.** Es un punto aproximado, normalmente ligado a la celda que da servicio. En las pruebas de 3GPP es un círculo de 3 km de radio con 500 m de incertidumbre en altura.

**El modelo de navegación.** Son las efemérides y las correcciones de reloj: lo mismo que el satélite emite en sus subtramas 1 a 3. Según IS-GPS-200, en operación normal cada juego se transmite durante dos horas y está ajustado para un intervalo de cuatro horas.

**El almanaque.** Es una descripción gruesa de toda la constelación. TS 37.355 indica que sirve para saber qué satélites son visibles y que vale durante unas semanas. En el mensaje GPS completo se tarda 25 tramas en recibirlo entero: 25 × 30 s = 12,5 min (calculado a partir de IS-GPS-200).

**La ayuda de adquisición.** Para cada satélite incluye el Doppler esperado en la posición de referencia, su incertidumbre, el retraso de código esperado, una ventana de búsqueda de código y la elevación y el acimut. El Doppler se expresa como velocidad: el campo cubre de −1024 a +1023,5 m/s en pasos de 0,5 m/s. La incertidumbre va de 2,5 a 40 m/s en la versión básica.

**La ayuda de bits de datos.** TS 37.355 permite enviar los propios bits que el satélite va a emitir. Sirve para «quitarlos» de la señal (*data wipe-off*) y poder integrar durante más tiempo.

### 3. Parámetros que importan

**Doppler y frecuencia.** En L1, cada metro por segundo de velocidad relativa desplaza la frecuencia unos 5,25 Hz (calculado: 1575,42 MHz ÷ c). Inside GNSS cifra en unos 800 m/s la velocidad máxima con la que cambia la distancia a un satélite. Eso son unos ±4,2 kHz (calculado). A eso se suma el error del oscilador del receptor: con un error de 1 ppm, ±1,6 kHz (calculado; 1 ppm es un valor de ejemplo).

**El tamaño de la búsqueda (ejemplo calculado).** Supón celdas de medio chip en código y de 500 Hz en frecuencia. Son valores ilustrativos, no normativos.

| Situación | Celdas de código | Celdas de frecuencia | Celdas por satélite | Satélites a probar |
|---|---|---|---|---|
| Arranque en frío | 2046 (1023 chips) | unas 24 (±5,8 kHz) | unas 49 000 | todos los códigos posibles, hasta 32 |
| Asistido, hora gruesa | 2046 (el código sigue sin acotar) | unas 8 (±40 m/s + oscilador) | unas 16 000 | solo los visibles |
| Asistido, hora fina | unas 80 (±20 chips) | unas 8 | unas 640 | solo los visibles |

La ventana de código con hora fina sale de sumar dos errores. Un error de 10 µs equivale a unos 10 chips. Un error de posición de 3 km equivale como mucho a otros 10 chips, porque un chip son unos 293 m (calculado). Con hora gruesa, en cambio, el móvil no sabe en qué punto del milisegundo del código cae cada satélite. TS 37.355 indica de hecho que la ventana de código cubre la incertidumbre de posición, no la de la hora.

El oscilador queda como factor dominante en la columna de frecuencia. Se suele afirmar que los móviles lo calibran con la portadora de la celda, lo que reduciría aún más la búsqueda. No verificado en una fuente primaria para este artículo.

**Lo débil que puede ser la señal.** El estándar SPS garantiza al menos −158,5 dBW en L1 a cielo abierto, es decir, −128,5 dBm. Las pruebas de sensibilidad de TS 36.171 colocan siete de los ocho satélites a −147 dBm. Son 18,5 dB menos, unas 70 veces menos potencia (calculado). A ese nivel, el móvil debe dar una posición con error horizontal de 100 m o menos en el 95 % de los intentos y en 20 s como máximo. Ese tiempo se cuenta desde la petición y suponiendo que la asistencia ya ha llegado.

**Por qué leer datos exige más señal que medir el código.** Para medir el retraso basta con que la correlación acumulada destaque sobre el ruido. Cuanto más tiempo se acumula, más destaca. Pero cada bit de datos dura 20 ms y puede invertir el signo de la señal. Si el receptor no sabe dónde están esos cambios, no puede acumular de forma coherente más allá de un bit. Inside GNSS explica que, con señal débil, la tasa de errores de bit impide leer la hora de semana de forma fiable. Con asistencia el móvil no necesita leerla. Y con la ayuda de bits puede acumular más de 20 ms: pasar de 20 a 100 ms da hasta unos 7 dB más de relación señal-ruido (calculado, ideal). En la práctica la estabilidad del oscilador y el movimiento recortan esa ganancia.

**La ambigüedad del milisegundo.** El código C/A se repite cada milisegundo. Medir su fase da la distancia salvo un número entero de milisegundos de luz, unos 300 km cada uno (calculado). Un receptor aislado lo resuelve leyendo la hora del mensaje. El asistido usa la posición de referencia. Blay y otros indican que el entero queda resuelto si el error combinado es menor de medio milisegundo de luz, unos 150 km. Una posición de partida de pocos kilómetros cumple de sobra.

**Hora gruesa como quinta incógnita.** Con ±2 s de error en la hora, la posición calculada de cada satélite está mal. Inside GNSS cifra el error en hasta ±1,7 km de distancia geométrica. La solución es añadir el error de hora como quinta incógnita, junto a las tres coordenadas y el desfase de reloj. Blay y otros lo describen así. Hace falta entonces un quinto satélite, y la incertidumbre del resto de incógnitas aumenta algo.

### 4. Diagrama o flujo

```text
 ARRANQUE EN FRÍO (receptor aislado)
   Búsqueda a ciegas: hasta 32 códigos × 2046 retrasos × ~24 frecuencias
        │
        ▼
   Enganche ─► leer subtramas 1-3 a 50 bit/s (hasta 36 s)
        │       (exige señal suficiente para decodificar bits)
        ▼
   4 satélites con efemérides ─► posición

 ARRANQUE ASISTIDO (móvil)
   Receptores de referencia ──► servidor de localización (E-SMLC o SLP)
   (cielo abierto, siguen todos           │
    los satélites sin parar)              │  LPP por el plano de control,
                                          │  o SUPL por datos IP
                                          ▼
   Móvil recibe: hora (±2 s o ±10 µs), posición (~3 km),
                 órbitas y relojes, ionosfera, Doppler esperado y ventanas
        │
        ▼
   Búsqueda dirigida: solo satélites visibles, ventana estrecha
        │
        ▼
   Mide retraso de código + Doppler (no lee el mensaje; vale a −147 dBm)
        │
        ├── UE-based:    el móvil calcula con las órbitas recibidas
        └── UE-assisted: envía medidas, el servidor calcula
        ▼
   Posición (requisito de prueba 3GPP: ≤ 100 m al 95 %, ≤ 20 s)
```

## Límites, excepciones y casos reales

**La asistencia no crea señal.** Bajo tierra o en el centro de un edificio grande puede no llegar nada útil, ni siquiera a −147 dBm. Entonces el móvil recurre a WiFi, a las celdas o a la última posición conocida, como explica el [artículo 05·02](02-ComoTeLocalizaElMovilSinGPS.md).

**Sin conexión de datos.** El A-GPS por el plano de usuario necesita datos IP. Si no hay cobertura de datos, el receptor del móvil funciona como uno aislado. Si usó el GPS hace poco, aún guarda órbitas válidas: las efemérides sirven unas cuatro horas y el almanaque unas semanas. Ese caso intermedio se suele llamar arranque templado o en caliente.

**Los requisitos de 3GPP no son el rendimiento real.** TS 36.171 fija condiciones de laboratorio: ruido blanco, geometría conocida, un multitrayecto modelado. En una calle estrecha el multitrayecto real puede ser peor. Y los 20 s son un máximo, no una cifra típica. Los tiempos típicos de cada fabricante no se han verificado para este artículo.

**Las primeras posiciones son peores.** La prueba de sensibilidad acepta 100 m de error. La de precisión nominal, con señales a −130 dBm, exige 30 m solo con GPS y 15 m con varias constelaciones. El punto azul suele empezar grande y encogerse a medida que el receptor sigue más satélites.

**La hora gruesa cuesta un satélite.** Con la quinta incógnita hacen falta cinco satélites en lugar de cuatro. Con pocos satélites visibles, como en un cañón urbano, eso puede retrasar la primera posición. La hora fina lo evita, pero exige que red y móvil la admitan.

**Datos caducados.** Unas órbitas fuera de su intervalo de ajuste o un almanaque viejo dan predicciones peores. Por eso el servidor también informa del estado de los satélites: TS 37.355 obliga a enviar siempre la lista actual de señales no fiables.

## Consecuencias prácticas

**Para quien usa el móvil.** La primera posición rápida depende tanto de la red como del cielo. Con datos activos, la posición llega en segundos. En modo avión, con el GPS encendido, el móvil todavía puede localizarse, pero tarda más si lleva horas sin usarlo.

**Consumo.** TS 36.305 indica que la asistencia permite gastar menos energía que el GNSS autónomo. Es lógico: el receptor pasa menos tiempo encendido buscando y no tiene que seguir cada satélite durante decenas de segundos para leer su mensaje.

**Llamadas de emergencia.** El modo *UE-assisted* deja el cálculo en el servidor de la red. Así la posición queda directamente en la red, que es quien la necesita en servicios como las llamadas de emergencia. Los requisitos regulatorios de cada país no se tratan aquí.

**Privacidad.** Pedir asistencia implica hablar con un servidor. En el plano de control la red ya conoce la celda del móvil. Por el plano de usuario, la petición pasa por la conexión de datos hasta el servidor que use el sistema. Qué guarda cada proveedor de esas peticiones no se ha verificado para este artículo.

**Para quien diseña.** La sensibilidad de un receptor de móvil no es solo una cuestión de antena. Depende de cuánto puede acumular, y eso depende de qué le cuenta la red.

## Preguntas frecuentes

### ¿Es lo mismo que localizarse por WiFi o por la antena?

No. Aquí la posición sigue saliendo de medir satélites, como en el [artículo 05·01](01-ComoCalculaElGPSTuPosicion.md). La red solo aporta datos para buscarlos y medirlos antes. La localización por WiFi o por celdas, del [artículo 05·02](02-ComoTeLocalizaElMovilSinGPS.md), no mide ningún satélite. Eso sí: la posición de referencia que envía el servidor suele ser una estimación basada en la celda.

### ¿El GPS necesita internet para funcionar?

No. Un receptor GPS solo escucha. Sin asistencia funciona igual, pero tarda más en arrancar en frío y necesita más señal para leer el mensaje de navegación.

### ¿El móvil envía su posición a los satélites?

No. Los satélites no reciben nada del usuario. En el modo *UE-assisted* el móvil envía medidas a un servidor de la red, no a los satélites.

### ¿Qué diferencia hay entre arranque en frío, templado y en caliente?

Depende de qué recuerda el receptor. En frío no tiene ni hora, ni posición, ni órbitas. Con almanaque y hora aproximada sabe qué satélites buscar, pero aún debe leer las efemérides. Con efemérides vigentes de menos de unas cuatro horas puede calcular casi en cuanto engancha la señal. La asistencia equivale a recibir de golpe un arranque en caliente por la red.

## Fuentes

- [U.S. Space Force — IS-GPS-200N, NAVSTAR GPS Space Segment / Navigation User Segment Interfaces (2022)](https://www.gps.gov/technical/icwg/IS-GPS-200N.pdf). Código C/A de 1023 chips a 1,023 Mchip/s y 1 ms (3.2.1.3); mensaje a 50 bit/s, subtramas de 300 bits y 6 s, 25 páginas en las subtramas 4 y 5 (20.3.1–20.3.3); reloj y efemérides en las subtramas 1 a 3; transmisión durante dos horas e intervalo de ajuste de cuatro horas (20.3.4.4).
- [GPS Standard Positioning Service (SPS) Performance Standard, 5.ª edición (2020)](https://www.gps.gov/technical/ps/2020-SPS-performance-standard.pdf). Potencia mínima recibida de −158,5 dBW para el código C/A en L1.
- [3GPP TS 36.305 V17.0.0, Stage 2 functional specification of UE positioning in E-UTRAN (ETSI TS 136 305)](https://www.etsi.org/deliver/etsi_ts/136300_136399/136305/17.00.00_60/ts_136305v170000p.pdf). Definición de los modos *UE-based* y *UE-assisted* (3.1, 8.1.1); red continua de receptores de referencia; ventajas de la asistencia en tiempo de arranque, sensibilidad, consumo y precisión (8.1.1); tipos de asistencia (8.1.2.1); papel del E-SMLC y uso de LPP con SUPL (5.3.3, 6.2.1).
- [3GPP TS 37.355 V17.0.0, LTE Positioning Protocol (LPP) (ETSI TS 137 355)](https://www.etsi.org/deliver/etsi_ts/137300_137399/137355/17.00.00_60/ts_137355v170000p.pdf). Elementos de asistencia GNSS (6.5.2): hora y posición de referencia, modelo de ionosfera, modelo de navegación, almanaque, integridad en tiempo real, ayuda de bits de datos y ayuda de adquisición (Doppler de −1024 a +1023,5 m/s, incertidumbre de 2,5 a 40 m/s, ventana de código).
- [3GPP TS 36.171 V18.0.0, Requirements for support of A-GNSS (ETSI TS 136 171)](https://www.etsi.org/deliver/etsi_TS/136100_136199/136171/18.00.00_60/ts_136171v180000p.pdf). Pruebas de sensibilidad a −147 dBm con hora gruesa (±2 s) y fina (±10 µs): 100 m al 95 % en 20 s (5.1); precisión nominal a −130 dBm: 30 m (5.2) y 15 m con varias constelaciones (6.2); posición de referencia de 3 km (anexo B.1.4).
- [R. Blay, B. Wang y D. M. Akos, «Deriving Accurate Time from Assisted GNSS Using Extended Ambiguity Resolution», NAVIGATION 68(1), 217–229 (2021)](https://navi.ion.org/content/68/1/217). Hasta 36 s para recibir las subtramas 1 a 3; ambigüedad del milisegundo resuelta con un error menor de medio milisegundo de luz (unos 150 km); solución de cinco incógnitas con hora gruesa.
- [Inside GNSS, «Coarse Time Positioning» (2012)](https://insidegnss.com/coarse-time-positioning/). Hora de semana como incógnita adicional; ±2 s de error equivalen a hasta ±1,7 km; velocidad máxima de unos 800 m/s en la distancia a un satélite; fallo de la decodificación con señal débil por la tasa de errores de bit.
