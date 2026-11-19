---
title: "¿Cómo reparte una antena de telefonía móvil su capacidad entre todos los móviles conectados a la vez?"
section: "02_RedesMoviles"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "La estación base divide su canal en una cuadrícula de frecuencia y tiempo. Cada 0,5 o 1 ms su planificador decide qué casillas usa cada móvil y se lo anuncia por un canal de control. En subida, el móvil tiene que pedir permiso antes de transmitir. El algoritmo que elige no lo fija la norma: lo diseña cada fabricante."
prerequisites:
  - "Relación señal-ruido, modulación y CQI (artículo 00·02)"
  - "Por qué la señal cambia al moverse unos centímetros (artículo 00·03)"
translation: "../../EN/02_CellularNetworks/01-HowACellTowerSharesItsCapacity.md"
---

# ¿Cómo reparte una antena de telefonía móvil su capacidad entre todos los móviles conectados a la vez?

## Pregunta y encuadre

En un concierto o en una estación de tren, cientos de móviles usan la misma antena. Todos descargan y suben datos a la vez y, sin embargo, sus señales no se pisan. La pregunta es cómo se organiza ese reparto. ¿Quién decide qué parte del canal usa cada móvil, y cada cuánto se toma esa decisión?

Este artículo explica el mecanismo con la norma 5G NR de 3GPP, porque es pública y es la más reciente. El 4G (LTE) se basa en la misma idea de cuadrícula y planificador, con otros valores.

- **Profundidad:** intermedia. Hay cálculos sencillos con microsegundos y kilohercios.
- **Conocimientos previos:** qué son la relación señal-ruido, la modulación y el indicador CQI, explicados en [¿Qué decide cuántos bits por segundo caben en una señal de radio?](../00_Fundamentos/02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md). Ayuda saber por qué la señal fluctúa al moverse, explicado en [¿Por qué la señal cambia tanto al mover el móvil unos centímetros?](../00_Fundamentos/03-PorQueLaCoberturaCambiaEnPocosCentimetros.md).
- **Fuera de alcance:** cómo encuentra el móvil la antena al encenderse, el paso de una antena a otra (traspaso), la red central del operador y la interferencia entre antenas vecinas.

## Respuesta corta

La antena divide su canal en una cuadrícula de bloques de frecuencia y tiempo, y un planificador dentro de la estación base decide cada fracción de milisegundo qué bloques usa cada móvil, en bajada y en subida.

### Resumen de 60 segundos

Una estación base 5G no emite una sola señal para todos. Trocea su canal en miles de subportadoras estrechas, agrupadas de 12 en 12 en «bloques de recursos». En el tiempo, lo trocea en ranuras de 14 símbolos que duran 1 ms o 0,5 ms, según la configuración.

En cada ranura, un programa de la estación base llamado planificador reparte esos bloques. Tiene en cuenta cuántos datos espera cada móvil, qué calidad de señal tiene cada uno y qué exige cada tipo de tráfico. Después publica el reparto en un canal de control. Cada móvil lo lee y busca los mensajes dirigidos a su identificador.

En subida el móvil no puede transmitir cuando quiere. Primero avisa de que tiene datos y luego espera a que la estación base le asigne bloques. Además adelanta su emisión según su distancia, para que todas las señales lleguen a la antena alineadas.

La norma fija el formato de la cuadrícula y de los mensajes. No fija cómo decide el planificador: ese algoritmo lo diseña cada fabricante.

## Modelo mental

Piensa en una oficina de correos con muchas ventanillas y un encargado que reparte turnos. Cada pocos segundos, el encargado mira la cola. Asigna cada ventanilla a un cliente y lo anuncia en una pantalla. Los clientes no se acercan por su cuenta: miran la pantalla y van a la ventanilla que les toca. Quien quiere enviar un paquete tiene que pedir número primero.

Las ventanillas son los bloques de frecuencia. Cada ronda de la pantalla es una ranura de tiempo. El encargado es el planificador y la pantalla es el canal de control. Pedir número es la solicitud de planificación que hace el móvil antes de subir datos.

**Dónde se rompe la analogía.** En correos, todas las ventanillas atienden igual de rápido a cualquier cliente. En radio no: el mismo bloque lleva muchos más bits a un móvil cercano con buena señal que a uno lejano. Además, la calidad de cada móvil cambia de una ranura a otra por el desvanecimiento. Un buen planificador aprovecha esos cambios, algo que el encargado de correos no tiene que hacer. Y la ronda dura 0,5 ms, no segundos.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

**En bajada (de la antena al móvil):**

1. **El móvil informa de su canal.** Mide unas señales de referencia (CSI-RS) y envía a la estación base un informe de estado del canal, que incluye el CQI (3GPP TS 38.300, apartado 5.2.5.1).
2. **El planificador decide.** En cada ranura elige qué móviles reciben datos, en qué bloques de recursos y con qué modulación y código (TS 38.300, apartado 10.1).
3. **La estación base anuncia el reparto.** Envía un mensaje de control (DCI) por el canal PDCCH, marcado con el identificador temporal del móvil, el C-RNTI (TS 38.300, apartado 10.2).
4. **El móvil busca su mensaje.** Prueba a decodificar varias posiciones posibles del canal de control. Solo acepta los mensajes cuyo código de comprobación (CRC) va mezclado con su C-RNTI (TS 38.213, apartado 10.1).
5. **El móvil recibe los datos** en el canal PDSCH, en los bloques indicados.
6. **El móvil confirma.** Responde con un acuse (HARQ-ACK) por el canal PUCCH. Si el bloque llegó mal, la estación base lo retransmite (TS 38.213, apartado 9; TS 38.300, apartado 5.2.5.4).

**En subida (del móvil a la antena):**

1. **Llegan datos al móvil** y no tiene recursos asignados. Eso dispara un informe de búfer (BSR).
2. **El móvil envía una solicitud de planificación (SR).** Es una señal corta por el PUCCH, en recursos que la red le reservó (TS 38.321, apartado 5.4.4).
3. **La estación base concede un permiso inicial** (una «concesión» o *grant*) por el PDCCH.
4. **El móvil envía el BSR** en el canal PUSCH. El BSR dice cuántos datos tiene esperando, agrupados por tipo de tráfico (TS 38.321, apartado 5.4.5).
5. **La estación base concede más recursos** según ese volumen. El móvil transmite sus datos en el PUSCH.

Si el móvil no tiene recursos para enviar la SR, o la ha repetido el máximo de veces sin respuesta, inicia un acceso aleatorio para volver a pedir paso (TS 38.321, apartado 5.4.4).

### 2. Qué hace cada componente

**La cuadrícula de recursos.** 5G NR usa OFDM: el canal se divide en muchas subportadoras estrechas que transmiten en paralelo. En bajada se usa OFDM con prefijo cíclico. En subida se puede usar ese mismo esquema o una variante con ensanchado DFT, que facilita el trabajo al amplificador del móvil (TS 38.300, apartado 5.1). La separación entre subportadoras es 15 kHz × 2^μ, donde μ es la «numerología»: 15, 30, 60, 120 kHz y más (TS 38.211, tabla 4.2-1).

**El bloque de recursos.** Son 12 subportadoras consecutivas (TS 38.211, apartado 4.4.4.1). Es la unidad en la que el planificador reparte la frecuencia (TS 38.300, apartado 10.1). Una portadora tiene como máximo 275 bloques (TS 38.300, apartado 5.1).

**La ranura.** En el tiempo, una trama dura 10 ms y se divide en 10 subtramas de 1 ms. Cada ranura tiene 14 símbolos OFDM con prefijo normal. Con 15 kHz cabe 1 ranura por subtrama; con 30 kHz, 2; con 60 kHz, 4 (TS 38.211, apartado 4.3 y tabla 4.3.2-1). Cuanto más separadas están las subportadoras, más cortos son los símbolos y más decisiones caben por segundo.

**El planificador.** Es parte de la capa MAC de la estación base (gNB). Asigna recursos de forma dinámica en bajada y en subida. Tiene en cuenta el búfer de cada móvil y los requisitos de calidad de servicio de cada flujo. Puede usar también las medidas de radio de la propia estación y los informes de los móviles (TS 38.300, apartado 10.1). La norma no describe ningún algoritmo de decisión.

**El canal de control (PDCCH) y el DCI.** El DCI es el mensaje que dice «este móvil, estos bloques, esta modulación». El móvil vigila el PDCCH siempre que tiene la recepción activa (TS 38.300, apartado 10.2). No sabe de antemano dónde estará su mensaje. Por eso prueba un conjunto de posiciones candidatas, llamado espacio de búsqueda. Con 30 kHz, la norma limita esas pruebas a 36 candidatos por ranura en una sola portadora (TS 38.213, tabla 10.1-2).

**El C-RNTI.** Es un identificador temporal que la estación base asigna a cada móvil conectado. La estación base mezcla el CRC de cada DCI con el C-RNTI del destinatario. Solo ese móvil obtiene un CRC correcto al decodificarlo (TS 38.213, apartado 10.1).

**SR y BSR.** La SR es un aviso mínimo, sin datos: «tengo algo que enviar». El BSR da el volumen, repartido en hasta 8 grupos de canales lógicos (TS 38.321, apartado 5.4.5). Con esos datos el planificador dimensiona la concesión.

**HARQ.** Si un bloque llega con errores, el receptor guarda lo que recibió. La retransmisión envía redundancia nueva que se combina con lo anterior («HARQ con redundancia incremental», TS 38.300, apartados 5.2.5.4 y 5.3.5.4). Varios procesos HARQ funcionan en paralelo, para que la espera de un acuse no detenga la transmisión. El número de procesos lo fija TS 38.214; aquí no se da la cifra porque no se ha verificado.

**El avance temporal (TA).** Las señales de móviles a distintas distancias tardan tiempos distintos en llegar. La estación base mide ese retraso y le dice a cada móvil cuánto debe adelantar su emisión respecto a la trama que recibe (TS 38.211, apartado 4.3.1; TS 38.300, apartado 5.3.5.3). Si el móvil deja de recibir correcciones durante un tiempo configurado, deja de transmitir datos y tiene que volver a sincronizarse con un acceso aleatorio (TS 38.321, apartado 5.2).

**Las concesiones configuradas.** Para tráfico periódico, como la voz, la red puede reservar recursos que se repiten sin un DCI cada vez. En subida admite hasta 12 configuraciones activas por parte de banda; en bajada, hasta 8 (TS 38.300, apartados 10.2 y 10.3).

### 3. Parámetros que importan

**Tamaño de la cuadrícula.** Ejemplos de TS 38.101-1 (tabla 5.3.2-1, Rel-15):

| Canal | Separación | Bloques de recursos | Ranura | Ranuras por segundo |
|---|---|---|---|---|
| 20 MHz | 15 kHz | 106 | 1 ms | 1000 |
| 20 MHz | 30 kHz | 51 | 0,5 ms | 2000 |
| 100 MHz | 30 kHz | 273 | 0,5 ms | 2000 |

Con 100 MHz y 30 kHz, el planificador reparte 273 bloques en cada ranura. Son 546 000 casillas de bloque × ranura por segundo. Cada casilla tiene 12 × 14 = 168 elementos de recurso. Una parte de ellos lleva señales de referencia y control, no datos de usuario.

**Prefijo cíclico.** Es una copia del final de cada símbolo que se pone delante. Absorbe los ecos y las pequeñas diferencias de llegada. Calculado con las fórmulas de TS 38.211 (apartado 5.3.1), con prefijo normal:

| Separación | Símbolo útil | Prefijo cíclico (símbolo típico) |
|---|---|---|
| 15 kHz | 66,7 µs | 4,69 µs |
| 30 kHz | 33,3 µs | 2,34 µs |

**Por qué hace falta el avance temporal: un cálculo.** Un móvil a 3 km de la antena recibe la señal 10 µs tarde (3000 m ÷ 3 × 10⁸ m/s). Si respondiera sin adelantarse, su señal llegaría a la antena 20 µs tarde, contando ida y vuelta. Con 30 kHz, el prefijo cíclico solo absorbe 2,34 µs. Eso equivale a unos 350 m de diferencia de distancia entre dos móviles (2,34 µs × 3 × 10⁸ m/s ÷ 2). Sin TA, cada símbolo del móvil lejano llegaría solapado con el símbolo siguiente de los demás móviles. Las subportadoras dejarían de ser independientes y se interferirían entre sí.

**Cómo decide el planificador: un ejemplo ilustrativo.** Las cifras siguientes son inventadas para mostrar la lógica, no medidas. Dos móviles comparten una antena. A está cerca y su canal permitiría unos 6 bit/s/Hz. B está lejos y le permitiría 1 bit/s/Hz.

| Regla | Qué hace | Resultado |
|---|---|---|
| Turnos iguales | Cada uno recibe la mitad de las ranuras | Justo en tiempo; B recibe 6 veces menos datos que A |
| Máxima tasa | Siempre se atiende a quien más bits lleva ahora | Máximo total; B casi nunca recibe nada |
| Proporcional justa | Se atiende a quien tiene mejor tasa actual en relación con lo que ya ha recibido de media | Ambos avanzan; cada uno tiende a ser atendido en sus momentos de mejor señal |

La regla proporcional justa es conocida desde los sistemas de datos CDMA-HDR. Jalali, Padovani y Pankaj describieron un planificador que elige al siguiente terminal según la tasa que pide y los datos que ya ha recibido (IEEE VTC 2000). Los planificadores comerciales parten de ideas así y añaden prioridades de calidad de servicio. Qué hace exactamente cada fabricante no es público.

### 4. Diagrama o flujo

```text
CUADRÍCULA (100 MHz, 30 kHz): 273 bloques × ranuras de 0,5 ms

 frecuencia ↑
  bloque 272 │ A │ A │ C │ B │ ...
  ...        │ A │ B │ C │ B │
  bloque 1   │ B │ B │ A │ A │
  bloque 0   │ B │ C │ A │ A │
             └───┴───┴───┴───┴──→ tiempo (una columna = una ranura)
  En cada columna el planificador vuelve a decidir.

BAJADA
 Móvil ──CSI (CQI)──────────────────────────────→ gNB
                                                  │ planificador
 Móvil ←──PDCCH: DCI (CRC con C-RNTI) ──────────── gNB
 Móvil ←──PDSCH: datos en los bloques indicados ── gNB
 Móvil ──PUCCH: HARQ-ACK / NACK ────────────────→ gNB  (NACK → retransmisión)

SUBIDA
 Móvil ──PUCCH: SR («tengo datos») ─────────────→ gNB
 Móvil ←──PDCCH: concesión pequeña ─────────────── gNB
 Móvil ──PUSCH: BSR (+ datos si caben) ─────────→ gNB
 Móvil ←──PDCCH: concesión ajustada al volumen ─── gNB
 Móvil ──PUSCH: datos (con avance temporal TA) ─→ gNB
```

## Límites, excepciones y casos reales

- **«Cientos conectados» no es «cientos a la vez en cada ranura».** Un móvil con la pantalla apagada suele estar en reposo o en recepción discontinua (DRX). En ese estado no vigila el PDCCH en todas las ranuras (TS 38.300, apartado 10.2). En cada ranura solo se planifica a una parte de los móviles con datos pendientes. Cuántos móviles conectados soporta una celda depende del equipo y de la configuración; la norma no da una cifra única.
- **El canal de control también se llena.** Cada DCI ocupa recursos del PDCCH. Con muchos móviles pidiendo poco, el cuello de botella puede ser anunciar el reparto, no los datos. Por eso existen las concesiones configuradas para tráfico periódico.
- **La subida tiene un retraso de arranque.** Entre la SR, la primera concesión y el BSR pasan varias ranuras antes de que fluyan los datos. Las concesiones configuradas acortan ese arranque.
- **El informe de canal llega con retraso.** El CQI describe el canal de hace unos instantes. Si el móvil se mueve deprisa, el desvanecimiento cambia antes de que el planificador reaccione. HARQ corrige parte de los errores que eso provoca.
- **Varios móviles en el mismo bloque.** Con varias antenas, la estación base puede enviar datos a dos móviles en los mismos bloques si están en direcciones distintas (multiplexación espacial, TS 38.300, apartado 5.1). La cuadrícula sigue siendo la base del reparto.
- **La interferencia de antenas vecinas no la gestiona este planificador.** Cada celda planifica lo suyo. La coordinación entre celdas existe, pero queda fuera de este artículo.

## Consecuencias prácticas

- **En una zona llena, la velocidad baja aunque la señal sea buena.** Las barras miden la potencia recibida, no cuántas casillas te tocan. Con más móviles activos, cada uno recibe una parte menor de la cuadrícula.
- **Con mala señal también te toca menos.** Un móvil lejano necesita más bloques para los mismos bits. Muchos planificadores compensan parte de esa diferencia, pero no toda.
- **La latencia de subida tiene un suelo.** Pedir permiso cuesta varias ranuras. Por eso numerologías con ranuras más cortas y recursos reservados importan para aplicaciones sensibles al retardo.
- **El mismo hardware puede dar resultados distintos según el operador.** El algoritmo del planificador y su configuración son decisiones del fabricante y del operador, no de la norma.

## Preguntas frecuentes

### ¿Es lo mismo que el reparto del WiFi?

No. En el WiFi clásico, cada aparato escucha el canal y transmite cuando lo encuentra libre, con esperas aleatorias para evitar choques. En la red móvil nadie transmite datos sin permiso: la estación base decide quién usa cada bloque. Las versiones recientes de WiFi (IEEE 802.11ax en adelante) incorporan un reparto similar por subportadoras (OFDMA), pero conviven con el acceso por contienda.

### ¿Tener «todas las barras» garantiza velocidad?

No. Las barras reflejan la potencia de la señal. La velocidad depende también de cuántos bloques te asigne el planificador, y eso depende de cuántos móviles compiten por ellos.

### ¿El móvil oye los datos de los demás?

Recibe la misma señal de radio, pero solo decodifica los bloques que su DCI le asigna. Además, los datos de usuario van cifrados en capas superiores. El identificador C-RNTI sirve para direccionar, no para proteger.

### ¿Por qué no se reparte a partes iguales, sin más?

Porque el mismo bloque vale muchos más bits para un móvil cercano que para uno lejano. Repartir por igual en tiempo desperdicia capacidad; repartir solo por tasa deja sin servicio a los lejanos. Los planificadores buscan un punto intermedio.

## Fuentes

- 3GPP. *TS 38.300: NR; NR and NG-RAN Overall description; Stage-2* (V18.11.0, Rel-18, 2026-09). Apartados 5.1, 5.2.5, 5.3.5, 10.1-10.4. <https://www.3gpp.org/ftp/specs/PDF/2026-09/Rel-18/38_series/38300-ib0.pdf>
- ETSI / 3GPP. *TS 38.211: NR; Physical channels and modulation* (V17.8.0, Rel-17). Tablas 4.2-1 y 4.3.2-1, apartados 4.3.1, 4.4.4.1 y 5.3.1. <https://etsi.org/deliver/etsi_TS/138200_138299/138211/17.08.00_60/ts_138211v170800p.pdf>
- ETSI / 3GPP. *TS 38.213: NR; Physical layer procedures for control* (V17.9.0, Rel-17). Apartados 9 y 10.1, tablas 10.1-2 y 10.1-3. <https://etsi.org/deliver/etsi_TS/138200_138299/138213/17.09.00_60/ts_138213v170900p.pdf>
- ETSI / 3GPP. *TS 38.321: NR; Medium Access Control (MAC) protocol specification* (V18.4.0, Rel-18). Apartados 5.2, 5.3.1, 5.4.1-5.4.5. <https://etsi.org/deliver/etsi_ts/138300_138399/138321/18.04.00_60/ts_138321v180400p.pdf>
- ETSI / 3GPP. *TS 38.101-1: NR; User Equipment (UE) radio transmission and reception; Part 1: Range 1 Standalone* (V15.5.0, Rel-15). Tabla 5.3.2-1. <https://etsi.org/deliver/etsi_TS/138100_138199/13810101/15.05.00_60/ts_13810101v150500p.pdf>
- A. Jalali, R. Padovani, R. Pankaj. «Data throughput of CDMA-HDR a high efficiency-high data rate personal communication wireless system». *IEEE VTC2000-Spring*, vol. 3, pp. 1854-1858. <https://doi.org/10.1109/VETECS.2000.851593>
