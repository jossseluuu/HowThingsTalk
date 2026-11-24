---
title: "¿Cómo pasa el móvil de una antena a otra sin cortar la conexión?"
section: "02_RedesMoviles"
language: "es"
status: "published"
order: 2
depth: "intermediate"
summary: "Mientras está conectado, el móvil mide sin parar la señal de su antena y la de las vecinas. Cuando una vecina supera a la actual por un margen y durante un tiempo que fija la red, envía un informe. La antena actual decide el traspaso, prepara a la nueva por la red y ordena al móvil saltar. El salto corta los datos unas decenas de milisegundos, pero los paquetes en tránsito se reenvían a la nueva antena y no se pierden."
prerequisites:
  - "Cómo reparte una antena su capacidad: estación base, canal de control y C-RNTI (artículo 02·01)"
  - "Por qué la señal cambia al moverse unos centímetros (artículo 00·03)"
translation: "../../EN/02_CellularNetworks/02-HowYourPhoneHandsOverBetweenCellTowers.md"
---

# ¿Cómo pasa el móvil de una antena a otra sin cortar la conexión?

## Pregunta y encuadre

Vas en tren o en coche con una videollamada abierta. En media hora recorres decenas de kilómetros y atraviesas la cobertura de muchas antenas distintas. La llamada sigue. La pregunta es cómo ocurre ese relevo. ¿Quién decide que hay que cambiar de antena, con qué criterio y qué pasa con los datos que estaban de camino en ese instante?

Este artículo explica el traspaso (en inglés, *handover*) con la norma 5G NR de 3GPP. El 4G (LTE) usa el mismo esquema de medidas, eventos y preparación entre estaciones base, con otros nombres de mensajes.

- **Profundidad:** intermedia. Hay una desigualdad con decibelios y algún cálculo con milisegundos.
- **Conocimientos previos:** qué es una estación base, el canal de control y el identificador temporal C-RNTI, explicados en [¿Cómo reparte una antena de telefonía móvil su capacidad entre todos los móviles conectados a la vez?](01-ComoReparteUnaAntenaSuCapacidad.md). Ayuda saber por qué la señal fluctúa al moverse, explicado en [¿Por qué la señal cambia tanto al mover el móvil unos centímetros?](../00_Fundamentos/03-PorQueLaCoberturaCambiaEnPocosCentimetros.md).
- **Fuera de alcance:** cómo encuentra el móvil la red al encenderse, cómo elige antena cuando está en reposo (la reselección de celda, que decide el propio móvil), el paso entre 4G y 5G y los detalles de la red central del operador.

## Respuesta corta

El móvil mide continuamente su antena y las vecinas, avisa cuando una vecina es claramente mejor, y la antena actual prepara a la nueva por la red antes de ordenarle saltar; así el corte dura unas decenas de milisegundos y los datos pendientes se reenvían en lugar de perderse.

### Resumen de 60 segundos

Con una conexión activa, el móvil no decide solo a qué antena hablar. La red le dice qué medir y cuándo avisar. El móvil escucha unas señales de sincronización que cada antena emite de forma periódica. Mide su potencia y la promedia para suavizar las fluctuaciones rápidas.

Un aviso típico es «avísame si una vecina supera a la actual en 3 dB durante 320 ms». Cuando se cumple, el móvil envía un informe de medidas. La estación base actual lo lee y decide si hace el traspaso.

Si lo hace, pide permiso a la estación base de destino por un enlace entre ambas. La de destino reserva recursos y le devuelve una configuración lista para usar. La antena actual se la entrega al móvil. El móvil suelta la antena vieja, se sincroniza con la nueva y le confirma la llegada.

Durante el salto no fluyen datos. La norma limita ese corte a unos 40 ms más la espera hasta la siguiente ocasión de acceso en la celda nueva, en un caso típico. Mientras tanto, la antena vieja reenvía a la nueva los paquetes que no llegó a entregar. Al final, la red central desvía el tráfico a la nueva antena y la vieja libera todo lo que tenía del móvil.

## Modelo mental

Piensa en una carrera de relevos. El corredor que lleva el testigo ve acercarse al siguiente. No le pasa el testigo al primero que aparece, sino al que está claramente en posición. Antes de la entrega, el siguiente ya está avisado y corriendo. Cuando el testigo cambia de mano, la carrera sigue.

El testigo es tu conexión con su contexto: identidad, claves de cifrado y datos pendientes. Los corredores son las estaciones base. El aviso previo es la preparación del traspaso entre estaciones. «Claramente en posición» es el margen y el tiempo que exige la red antes de cambiar.

**Dónde se rompe la analogía.** En un relevo, los corredores se ven y deciden ellos. Aquí, la estación base no ve dónde estás: depende de lo que el móvil mida y le cuente. Además, quien decide el cambio es la estación que suelta el testigo, no la que lo recibe, aunque la nueva puede rechazarlo si no tiene sitio. Y en el traspaso normal hay un instante en que nadie lleva el testigo. El móvil suelta la antena vieja antes de engancharse a la nueva, y por eso hay un breve corte.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

La norma distingue dos tipos de movilidad con el móvil conectado (TS 38.300, apartado 9.2.3.1). Cambiar de **haz** dentro de la misma antena lo resuelven las capas física y MAC, sin mensajes RRC. Cambiar de **celda** exige señalización explícita: eso es el traspaso. La red controla el proceso y el móvil aporta las medidas.

El traspaso típico entre dos estaciones base 5G conectadas entre sí por la interfaz Xn sigue estos pasos (TS 38.300, apartado 9.2.3.2.1, figura 9.2.3.2.1-1):

1. **Configuración de medidas.** La estación base actual (origen) le dice al móvil qué frecuencias medir y qué condiciones deben disparar un informe. El móvil mide y, cuando una condición se cumple, envía un informe de medidas.
2. **Decisión.** La estación de origen decide el traspaso a partir del informe y de su propia información de gestión de recursos.
3. **Petición de traspaso.** El origen envía un mensaje *Handover Request* a la estación de destino por Xn. Incluye la celda objetivo, una clave derivada para el destino, la configuración actual del móvil, sus capacidades y sus sesiones de datos.
4. **Control de admisión.** El destino comprueba si puede aceptar al móvil.
5. **Aceptación.** El destino prepara sus recursos y responde con *Handover Request Acknowledge*. Dentro va, ya construido, el mensaje que recibirá el móvil.
6. **Orden al móvil.** El origen reenvía ese mensaje al móvil: un *RRCReconfiguration*. Como mínimo contiene la identidad de la celda de destino, el nuevo C-RNTI y los algoritmos de seguridad del destino. Puede incluir recursos reservados de acceso aleatorio y la información de sistema de la celda nueva, para que el móvil no tenga que leerla.
7. **Estado de la transferencia.** El origen le dice al destino por qué número de secuencia van los paquetes en cada sentido (*SN Status Transfer*). Así el destino sabe qué falta por entregar y qué ha llegado ya.
8. **Salto.** El móvil se desconecta de la celda vieja, se sincroniza con la nueva, hace el acceso aleatorio y envía *RRCReconfigurationComplete* al destino. Desde ese momento, el móvil ya habla con la antena nueva.
9. a 11. **Cambio de ruta.** El destino pide a la red central (AMF) que desvíe el tráfico (*Path Switch Request*). La UPF, que encamina los datos del usuario, cambia el camino de bajada a la nueva estación. Por el camino viejo envía unos paquetes «marcador de fin» para indicar que ya no llegará nada más por ahí.
12. **Liberación.** El destino avisa al origen (*UE Context Release*) y el origen libera los recursos del móvil.

Entre el paso 5 y el 12, el origen puede reenviar al destino los paquetes de bajada que aún no había entregado (TS 38.300, apartado 9.2.3.2.2). El destino los envía primero, antes que los paquetes nuevos que llegan por la ruta ya cambiada.

### 2. Qué hace cada componente

**Las señales que se miden.** Cada celda emite de forma periódica bloques de sincronización (SS/PBCH, o SSB) que llevan su identificador físico (PCI). El móvil también puede medir señales de referencia CSI-RS si la red lo configura. Sobre ellas calcula tres magnitudes: RSRP (potencia recibida de la señal de referencia), RSRQ (calidad) y SINR (relación señal/interferencia más ruido) (TS 38.331, apartado 5.5.3.1). En FR1, el móvil debe poder medir al menos 8 celdas por frecuencia (TS 38.133 V15.4.0, apartado 9.2.3.1).

**El filtro de capa 3.** Una medida aislada fluctúa mucho por el desvanecimiento. El móvil la suaviza con un promedio exponencial (TS 38.331, apartado 5.5.3.2):

```text
F(n) = (1 − a) · F(n−1) + a · M(n),   con a = 1 / 2^(k/4)
```

M(n) es la medida nueva y F(n) el valor filtrado. El coeficiente k lo fija la red. Su valor por defecto es 4, que da a = 0,5: cada medida nueva pesa la mitad y la historia la otra mitad. El filtrado se hace en decibelios.

**Los eventos.** La red no pide informes constantes. Define condiciones con nombre, llamadas eventos (TS 38.331, apartados 5.5.4.2 a 5.5.4.7). Las más usadas para el traspaso:

| Evento | Se dispara cuando… |
|---|---|
| A2 | la celda actual cae por debajo de un umbral |
| A3 | una vecina supera a la actual en un margen |
| A4 | una vecina supera un umbral |
| A5 | la actual cae por debajo de un umbral y a la vez una vecina supera otro |

**La condición A3, en detalle.** Es el disparador más típico. Se cumple cuando (TS 38.331, apartado 5.5.4.4):

```text
Mn + Ofn + Ocn − Hys  >  Mp + Ofp + Ocp + Off
```

- Mn y Mp: medida filtrada de la vecina y de la celda actual, en dBm (RSRP) o dB (RSRQ, SINR).
- Ofn y Ofp: ajuste por frecuencia; Ocn y Ocp: ajuste por celda concreta. Sirven para favorecer o penalizar celdas. Valen 0 dB si no se configuran.
- Off: el margen (a3-Offset), de −15 a +15 dB en pasos de 0,5 dB.
- Hys: la histéresis, de 0 a 15 dB en pasos de 0,5 dB.

Para dejar de cumplirse, la desigualdad se invierte con la histéresis sumada en lugar de restada. Ese hueco entre «entrar» y «salir» evita que la condición parpadee cuando las dos señales están igualadas.

**El tiempo de disparo (timeToTrigger).** La condición debe cumplirse en todas las medidas filtradas durante un tiempo fijo antes de enviar el informe (TS 38.331, apartado 5.5.4.1). Hay 16 valores posibles, de 0 a 5120 ms: 0, 40, 64, 80, 100, 128, 160, 256, 320, 480, 512, 640, 1024, 1280, 2560 y 5120 ms (TS 38.331, apartado 6.3.2).

**El informe de medidas.** Lleva los resultados de la celda actual y de las vecinas que dispararon el evento. Cada vecina va con su PCI y sus valores de RSRP, RSRQ o SINR ya filtrados (TS 38.331, apartado 5.5.5). El móvil solo lo envía cuando la seguridad de la conexión ya está activa.

**La estación de origen.** Decide el traspaso. La norma define los mensajes, pero no el algoritmo de decisión: qué eventos configurar y con qué valores lo eligen el fabricante y el operador.

**La interfaz Xn.** Une estaciones base 5G entre sí. Por ella viajan la petición, la aceptación, el estado de la transferencia y los datos reenviados. Si dos estaciones no tienen Xn, el traspaso se hace a través de la red central (no se detalla aquí).

**El temporizador T304.** Arranca cuando el móvil empieza a ejecutar la orden de traspaso y se para cuando completa el acceso aleatorio en la celda nueva (TS 38.331, apartados 5.3.5.5.2 y 5.3.5.3). Valores posibles: 50, 100, 150, 200, 500, 1000, 2000 o 10 000 ms. Si vence, el traspaso ha fallado y el móvil inicia un restablecimiento de la conexión con causa «fallo de traspaso» (TS 38.331, apartados 5.3.5.8.3, 5.3.7.2 y 5.3.7.4).

### 3. Parámetros que importan

**Un ejemplo de disparo.** Valores ilustrativos, no de un operador concreto: Off = 2 dB, Hys = 1 dB, sin ajustes por celda ni frecuencia, timeToTrigger = 320 ms. La condición A3 queda:

```text
Mn − 1 > Mp + 2   →   Mn > Mp + 3 dB
```

Si la celda actual se mide a −95 dBm, la vecina tiene que superar −92 dBm durante 320 ms seguidos. A 120 km/h (33,3 m/s), en esos 320 ms el móvil avanza unos 10,7 m. Un margen o un tiempo mayores retrasan el traspaso. Unos menores lo adelantan, pero aumentan el riesgo de ir y volver entre dos celdas.

**El corte durante el salto.** La norma fija cuánto puede tardar el móvil (TS 38.133 V15.4.0, apartado 6.1.1.2, traspaso de FR1 a FR1). El retardo total es el tiempo de procesar el mensaje RRC más el tiempo de interrupción:

```text
T_interrupción = T_búsqueda + T_IU + 20 ms + T_Δ
```

- **T_búsqueda:** 0 ms si la celda de destino es «conocida». Lo es si el móvil la ha identificado durante los últimos 5 segundos. Si no lo es, hay que sumar el tiempo de encontrarla.
- **T_Δ:** tiempo de ajustar el sincronismo fino. Vale una periodicidad de la ventana de medida de SSB de la celda de destino (Trs).
- **T_IU:** espera hasta la primera ocasión de acceso aleatorio en la celda nueva. Depende de cómo configure esas ocasiones la celda de destino.
- **20 ms:** margen fijo de procesado.

Con una celda conocida y Trs = 20 ms: 0 + T_IU + 20 + 20 = **40 ms más T_IU**. Es un límite máximo que debe cumplir el móvil, no un valor medido. El tiempo de procesado RRC que se suma está en TS 38.331, apartado 12; no se da aquí la cifra porque no se ha verificado.

**Pérdida de datos.** En los canales con confirmación (RLC en modo reconocido), el estado de la transferencia y el reenvío permiten un traspaso sin pérdidas. Para ello, el destino debe mantener el mismo reparto de flujos de calidad de servicio entre portadoras (TS 38.300, apartados 9.2.3.2.1 y 9.2.3.2.2).

### 4. Diagrama o flujo

```text
 Móvil                Origen (gNB A)          Destino (gNB B)        Red central (AMF/UPF)
   │ ◄── config. medidas ───│                        │                        │
   │  mide SSB de A y B; filtra; A3 se cumple 320 ms │                        │
   │ ── informe de medidas ─►│                        │                        │
   │                         │ decide traspaso        │                        │
   │                         │ ── Handover Request ──►│ control de admisión    │
   │                         │ ◄── Request Ack ───────│ (incluye la orden)     │
   │ ◄── RRCReconfiguration ─│                        │                        │
   │   (celda B, nuevo C-RNTI, seguridad, RACH)       │                        │
   │                         │ ── SN Status Transfer ►│                        │
   │                         │ ══ reenvío de datos ══►│                        │
   │ ✂ suelta la celda A ····│······ corte: ~40 ms + T_IU ······               │
   │ ── acceso aleatorio + RRCReconfigurationComplete ─►│                      │
   │ ◄═════════════ datos (primero los reenviados) ═════│                      │
   │                         │                        │ ── Path Switch ───────►│
   │                         │ ◄ marcador de fin ─────────────────────────────── UPF
   │                         │                        │ ◄── Path Switch Ack ───│
   │                         │ ◄── UE Context Release │                        │
   │                         │ libera recursos        │                        │
```

## Límites, excepciones y casos reales

- **Ir y volver (*ping-pong*).** En una frontera donde dos celdas llegan casi igual, el desvanecimiento puede hacer que cada una gane por turnos. Cada traspaso cuesta señalización y un corte. La histéresis, el margen y el tiempo de disparo existen para frenar ese vaivén. A cambio, ajustarlos con prudencia retrasa el traspaso cuando de verdad hace falta.
- **Llegar tarde.** Si el móvil se mueve rápido o la señal cae de golpe, por ejemplo al entrar en un túnel, la celda actual puede perderse antes de que llegue la orden. Entonces el traspaso falla. El móvil busca una celda y pide restablecer la conexión. La nueva estación recupera su contexto de la anterior (TS 38.300, apartado 9.2.3.3). El corte es mucho más largo que en un traspaso normal.
- **Traspaso condicional (CHO).** Para no depender de que la orden llegue a tiempo, la red puede enviarla por adelantado (TS 38.300, apartado 9.2.3.4). Prepara una o varias celdas candidatas y entrega al móvil sus configuraciones junto con una condición de ejecución, basada en los eventos A3 o A5. El móvil sigue en la celda actual y evalúa la condición. Cuando se cumple para una candidata, salta a ella por su cuenta, sin esperar más mensajes.
- **Traspaso sin corte de bajada (DAPS).** Con DAPS, el móvil mantiene a la vez las dos conexiones (TS 38.300, apartados 9.2.3.1 y 9.2.3.2.2). Sigue recibiendo de la celda vieja hasta que la libera y sigue transmitiéndole hasta que el acceso a la nueva tiene éxito. Exige que el móvil lleve dos pilas de protocolo en paralelo y desactiva funciones como la agregación de portadoras. No se combina con el traspaso condicional.
- **Movilidad por capas 1 y 2 (LTM).** Desde la versión 18 de la norma, la red puede preconfigurar celdas candidatas por RRC y ordenar el cambio con un mensaje MAC corto, a partir de medidas de capa 1 (TS 38.300, apartado 9.2.3.5). Reduce la señalización en el momento del cambio.
- **Sin acceso aleatorio.** En algunos casos, como el traspaso entre celdas de la misma estación base, el móvil puede reutilizar su avance temporal y saltarse el acceso aleatorio (TS 38.300, apartado 9.2.3.6).
- **Lo que no fija la norma.** Qué eventos se configuran, con qué márgenes y tiempos, y cómo se decide entre varias candidatas lo eligen el fabricante y el operador. Dos redes con el mismo equipo pueden comportarse de forma distinta.

## Consecuencias prácticas

- **El móvil no se cambia «a la antena más fuerte» en cuanto aparece.** Espera a que la vecina sea mejor por un margen y durante un tiempo. Por eso a veces sigues en una celda con poca señal aunque haya otra mejor cerca.
- **Una videollamada aguanta el traspaso, pero no siempre sin rastro.** Un corte de unas decenas de milisegundos con datos reenviados suele absorberse en el búfer de la aplicación. Un fallo de traspaso con restablecimiento es mucho más largo y puede notarse.
- **La señalización crece con la velocidad.** En un tren rápido, los traspasos son más frecuentes y llegan con menos margen. Para estos casos existen el traspaso condicional y el ajuste de parámetros por celda.
- **Las barras pueden saltar de golpe.** Tras un traspaso, el indicador refleja la nueva celda. Un cambio brusco de barras sin moverte casi nada puede ser solo eso.

## Preguntas frecuentes

### ¿Es lo mismo que cuando el móvil en reposo cambia de antena?

No. En reposo no hay una conexión activa que mantener. El propio móvil elige la celda según unas reglas que difunde la red (la reselección de celda), sin pedir permiso ni preparar nada. El traspaso de este artículo solo ocurre con el móvil conectado, y lo decide la red.

### ¿Es lo mismo que el *roaming* entre puntos de acceso WiFi?

No del todo. En WiFi, tradicionalmente es el aparato quien decide cuándo cambiar de punto de acceso y se asocia al nuevo por su cuenta. En la red móvil, la estación de origen decide, prepara a la de destino y le entrega al móvil la configuración ya lista. Hay ampliaciones de WiFi que aceleran ese cambio, pero el reparto de papeles es distinto.

### ¿Cambiar de haz es un traspaso?

No. Dentro de una misma celda, una antena 5G puede emitir varios haces. Cambiar de haz lo resuelven las capas física y MAC, sin mensajes RRC ni cambio de C-RNTI (TS 38.300, apartado 9.2.3.1). El traspaso es el cambio de celda.

### ¿Se pierden datos en el traspaso?

En los flujos con confirmación, en principio no. La estación vieja le dice a la nueva por dónde iba y le reenvía lo pendiente. Durante el corte hay retraso, no pérdida. En flujos sin confirmación, lo que estuviera en el aire en ese instante puede perderse.

## Fuentes

- ETSI / 3GPP. *TS 38.300: NR; NR and NG-RAN Overall description; Stage-2* (V18.6.0, Rel-18). Apartados 9.2.3.1, 9.2.3.2.1, 9.2.3.2.2, 9.2.3.3, 9.2.3.4, 9.2.3.5 y 9.2.3.6. <https://www.etsi.org/deliver/etsi_ts/138300_138399/138300/18.06.00_60/ts_138300v180600p.pdf>
- ETSI / 3GPP. *TS 38.331: NR; Radio Resource Control (RRC) protocol specification* (V15.3.0, Rel-15). Apartados 5.3.5, 5.3.7, 5.5.3, 5.5.4, 5.5.5 y 6.3.2. <https://www.etsi.org/deliver/etsi_ts/138300_138399/138331/15.03.00_60/ts_138331v150300p.pdf>
- ETSI / 3GPP. *TS 38.133: NR; Requirements for support of radio resource management* (V15.4.0, Rel-15). Apartados 6.1.1.2 y 9.2.3.1. <https://www.etsi.org/deliver/etsi_ts/138100_138199/138133/15.04.00_60/ts_138133v150400p.pdf>
