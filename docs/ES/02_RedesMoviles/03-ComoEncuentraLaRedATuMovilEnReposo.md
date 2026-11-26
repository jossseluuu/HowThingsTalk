---
title: "¿Cómo encuentra la red a tu móvil para avisarle de una llamada si está en reposo?"
section: "02_RedesMoviles"
language: "es"
status: "published"
order: 3
depth: "intermediate"
summary: "En reposo, la red no sabe en qué antena está tu móvil: solo conoce una zona de varias celdas, la que el móvil declaró al registrarse. Cuando llega una llamada, todas las antenas de esa zona emiten un aviso con la identidad temporal del móvil. El móvil no escucha todo el tiempo: se despierta en unos instantes fijos, calculados a partir de esa identidad, y vuelve a dormir si el aviso no es para él."
prerequisites:
  - "Cómo reparte una antena su capacidad: estación base, canal de control PDCCH e identificadores RNTI (artículo 02·01)"
  - "Cómo pasa el móvil de una antena a otra con una conexión activa (artículo 02·02)"
translation: "../../EN/02_CellularNetworks/03-HowTheNetworkFindsAnIdlePhone.md"
---

# ¿Cómo encuentra la red a tu móvil para avisarle de una llamada si está en reposo?

## Pregunta y encuadre

Tu móvil lleva una hora en el bolsillo con la pantalla apagada. No tiene ninguna conexión abierta con la red. Ha podido cambiar de antena varias veces sin avisar a nadie. Y aun así, cuando alguien te llama, suena en uno o dos segundos. La pregunta es cómo lo consigue la red. ¿Cómo sabe dónde buscarte? ¿Y cómo se entera el móvil de que lo buscan sin tener la radio encendida todo el tiempo?

El mecanismo se llama aviso o radiobúsqueda (en inglés, *paging*). Este artículo lo explica con la norma 5G NR de 3GPP. El 4G (LTE) usa el mismo esquema de zonas, avisos e instantes de escucha, con otros nombres y otra identidad para calcular esos instantes.

- **Profundidad:** intermedia. Hay una fórmula con módulos y un ejemplo con números.
- **Conocimientos previos:** qué es una estación base, el canal de control PDCCH y los identificadores RNTI, explicados en [¿Cómo reparte una antena de telefonía móvil su capacidad entre todos los móviles conectados a la vez?](01-ComoReparteUnaAntenaSuCapacidad.md). Ayuda haber leído [¿Cómo pasa el móvil de una antena a otra sin cortar la conexión?](02-ComoPasaElMovilDeUnaAntenaAOtra.md), que trata el caso contrario: el móvil conectado.
- **Fuera de alcance:** cómo encuentra el móvil la red al encenderse, las reglas exactas con las que elige antena en reposo, cómo se establece después la llamada de voz y la seguridad del registro.

## Respuesta corta

En reposo, la red solo sabe en qué zona de varias celdas estás, así que lanza el aviso por todas las antenas de esa zona, y el móvil lo oye porque se despierta en unos instantes fijos calculados a partir de su identidad temporal.

### Resumen de 60 segundos

Cuando el móvil se registra, la red central le entrega una lista de zonas de seguimiento (*tracking areas*). Cada zona agrupa muchas celdas. Mientras el móvil se mueva dentro de esa lista, no avisa a nadie. Si entra en una celda de una zona que no está en la lista, se registra de nuevo. Además, se registra cada cierto tiempo aunque no se mueva: por defecto, cada 54 minutos.

Cuando llega una llamada o un dato para el móvil, la red central pide a las antenas de esa zona que lo avisen. Cada antena emite un mensaje de aviso con la identidad temporal del móvil.

El móvil no escucha el canal de avisos todo el tiempo. Escucha solo en una ocasión de aviso por ciclo, que suele durar entre fracciones de segundo y un par de segundos. Esa ocasión no es aleatoria: el móvil y la antena la calculan por separado con la misma fórmula y la misma identidad. Así, la antena sabe cuándo hablar y el móvil sabe cuándo despertar.

Si el aviso lleva su identidad, el móvil pide una conexión. Si no, vuelve a dormir hasta la siguiente ocasión. Por eso el móvil gasta poca batería en reposo y por eso la llamada tarda un poco en sonar.

## Modelo mental

Piensa en un aeropuerto grande. Tú estás en la terminal 2, pero nadie sabe en qué puerta. Si te buscan, no recorren la terminal mirando caras: lo anuncian por megafonía en toda la terminal. Y tú no estás pendiente de cada anuncio. Solo prestas atención a los que empiezan en los minutos que te tocan. Si cambias de terminal, pasas por el mostrador para que sepan dónde buscarte.

La terminal es la zona de seguimiento. La megafonía es el aviso que emiten todas las antenas de la zona. El mostrador es el nuevo registro. Los minutos que te tocan son tu ocasión de aviso.

**Dónde se rompe la analogía.** En el aeropuerto oyes todos los anuncios aunque no les prestes atención. El móvil, en cambio, apaga de verdad gran parte de la radio entre ocasiones: si la red hablara fuera de su turno, no lo oiría. Por eso la fórmula del turno tiene que ser la misma en los dos lados. Además, el anuncio no dice tu nombre real. Dice una identidad temporal que la red cambia a menudo, para que nadie pueda seguirte escuchando los avisos.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

**Los tres estados del móvil.** La norma define tres estados de la conexión de radio (TS 38.300, apartado 7.2):

| Estado | Quién sabe dónde está el móvil | Quién inicia el aviso |
|---|---|---|
| Conectado (RRC_CONNECTED) | La estación base conoce su celda | No hace falta aviso |
| Inactivo (RRC_INACTIVE) | La estación base conoce un área de notificación (RNA) | La red de radio (aviso RAN) |
| Reposo (RRC_IDLE) | La red central conoce su lista de zonas de seguimiento | La red central, 5GC (aviso CN) |

En reposo y en inactivo es el propio móvil quien elige su celda: lo que la norma llama acampar en una celda (TS 38.304, apartado 4.1). El móvil escucha el canal de control de esa celda, su información de sistema y, en la mayoría de los casos, los avisos (TS 38.304, apartado 3.1).

**El registro.** El móvil se registra en la zona de seguimiento de la celda en la que acampa. Si se mueve a una celda que no pertenece a ninguna de sus zonas registradas, vuelve a registrarse. En inactivo, si sale de su área de notificación, hace una actualización de área RAN (TS 38.304, apartado 4.1). La norma lo dice así: como la red conoce, en la mayoría de los casos, el conjunto de zonas donde acampa el móvil, puede enviar el aviso por los canales de control de todas las celdas de ese conjunto (TS 38.304, apartado 4.1).

**La secuencia de una llamada entrante, con el móvil en reposo:**

1. La red central recibe algo destinado al móvil: una llamada o un paquete de datos. En reposo, el aviso lo inicia la red central (TS 38.300, apartado 7.2).
2. La red central pide a las estaciones base de las zonas del móvil que lo avisen. Puede empezar por un área menor y ampliarla en intentos sucesivos. Para ello pasa a la estación base el número de intento y el alcance previsto del siguiente (TS 38.300, apartado 9.2.5). La estrategia la elige el operador.
3. Cada estación base calcula cuándo escucha ese móvil y, en ese instante, emite en el canal de control un aviso dirigido al identificador común de avisos, el P-RNTI. Ese aviso señala dónde está el mensaje *Paging*, que lleva la identidad temporal del móvil (TS 38.300, apartado 9.2.5; TS 38.331, apartado 5.3.2).
4. El móvil se despierta en su ocasión, detecta el P-RNTI, lee el mensaje y encuentra su identidad.
5. El móvil inicia el acceso aleatorio y pide establecer una conexión. A partir de ahí, la llamada sigue su curso con el móvil ya conectado.

En inactivo el esquema es el mismo, pero el aviso lo inicia la estación base que guarda el contexto del móvil. Lo envía por las celdas de su área de notificación y con otra identidad: el I-RNTI completo (TS 38.300, apartados 7.2 y 9.2.5; TS 38.331, apartado 5.3.2).

### 2. Qué hace cada componente

**La zona de seguimiento.** Cada celda difunde a qué zona pertenece. La zona se identifica con un TAI: la identidad del operador más un código de zona, el TAC (TS 38.300, apartado 8.2). Al registrarse, el móvil recibe una lista de TAI. Debe hacer un registro por movilidad cuando detecta que el TAI de su celda no está en esa lista (TS 24.501, apartado 5.5.1.3.2).

**El registro periódico.** El temporizador T3512 obliga al móvil a registrarse cada cierto tiempo aunque no se mueva. Su valor lo da el operador al registrarse. Si no lo da, vale 54 minutos. El móvil lo arranca al pasar a reposo. Cuando vence, hace un registro periódico (TS 24.501, tabla 10.2.1). Así la red sabe que el móvil sigue ahí.

**El temporizador del lado de la red.** La red central arranca un temporizador de alcanzabilidad cuando el móvil pasa a reposo. Por defecto dura 4 minutos más que T3512: 58 minutos con el valor por defecto. Si vence sin noticias del móvil, lo normal es que la red deje de avisarlo y arranque otro temporizador. Cuando ese segundo temporizador vence, la red da de baja al móvil de forma implícita (TS 24.501, tabla 10.2.2).

**La identidad temporal.** El móvil no se anuncia con su identidad permanente. La red le asigna una temporal, el 5G-S-TMSI, una cadena de 48 bits (TS 38.304, apartado 7.1). El mensaje de aviso lleva el 5G-S-TMSI para el aviso de la red central o el I-RNTI completo para el aviso RAN (TS 38.331, apartado 5.3.2).

**El mensaje *Paging*.** Viaja por el canal lógico de avisos (PCCH), sin conexión previa con el móvil. Contiene una lista de registros. Cada registro lleva una identidad. Un mismo mensaje puede avisar a varios móviles a la vez. Desde la versión 17 de la norma, cada registro puede indicar además que la causa es una llamada de voz (TS 38.331, apartado 5.3.2).

**Los mensajes cortos.** El mismo P-RNTI sirve también para mensajes cortos que van directamente en el canal de control. Avisan a todos los móviles de que la información de sistema ha cambiado o de que hay una alerta pública de emergencia (ETWS o CMAS). Estos mensajes llegan incluso a móviles conectados (TS 38.300, apartado 9.2.5).

### 3. Parámetros que importan

**El ciclo de recepción discontinua (DRX).** En reposo o inactivo, el móvil solo tiene que vigilar el canal de avisos durante una ocasión de aviso (PO) por ciclo DRX (TS 38.300, apartado 9.2.5). El ciclo puede venir de tres sitios (TS 38.300, apartado 9.2.5; TS 38.304, apartado 7.1):

- un ciclo por defecto que difunde cada celda;
- un ciclo propio del móvil, acordado con la red central;
- en inactivo, un ciclo propio que fija la red de radio.

El móvil usa el más corto de los que le apliquen. El ciclo se mide en tramas de radio. Cada trama dura 10 ms: la norma cuenta 100 tramas por segundo (TS 38.304, apartado 7.4). Los valores habituales del ciclo normal son 32, 64, 128 y 256 tramas, es decir, 0,32, 0,64, 1,28 y 2,56 s (TS 38.331, campo *PagingCycle*; **no verificado** en el texto consultado).

**La fórmula del turno.** La trama de aviso (PF) es la trama de radio en la que puede empezar la ocasión del móvil. Su número de trama (SFN) cumple (TS 38.304, apartado 7.1):

```text
(SFN + PF_offset) mod T = (T div N) · (UE_ID mod N)
i_s = floor(UE_ID / N) mod Ns
UE_ID = 5G-S-TMSI mod 1024
```

- **T:** el ciclo DRX, en tramas.
- **N:** cuántas tramas de aviso hay en cada ciclo. Lo fija la red.
- **PF_offset:** un desplazamiento que fija la red.
- **Ns:** cuántas ocasiones de aviso hay en cada trama de aviso.
- **i_s:** cuál de esas ocasiones usa el móvil.
- **UE_ID:** los 10 bits bajos de la identidad temporal. Si el móvil aún no tiene 5G-S-TMSI, usa 0.

La estación base conoce el 5G-S-TMSI porque se lo pasa la red central con la petición de aviso. Así, los dos lados llegan a la misma trama sin hablar entre sí.

**Un ejemplo.** Valores ilustrativos, no de un operador concreto: T = 128 tramas (1,28 s), N = 32, PF_offset = 0, Ns = 2. Supón que el 5G-S-TMSI del móvil da UE_ID = 700.

```text
UE_ID mod N = 700 mod 32 = 28
T div N     = 128 / 32   = 4
SFN mod 128 = 4 · 28     = 112      → tramas 112, 240, 368, …
i_s = floor(700 / 32) mod 2 = 21 mod 2 = 1   → segunda ocasión de la trama
```

El móvil se despierta cada 1,28 s alrededor de una trama de 10 ms. El resto del ciclo puede dormir. Si la llamada llega justo después de su ocasión, la red tiene que esperar a la siguiente: hasta 1,28 s en este ejemplo, más lo que tarde el resto del proceso.

**Por qué la ocasión dura varias «miradas».** En 5G, la antena emite en varios haces. Una ocasión de aviso es un conjunto de S·X oportunidades consecutivas de vigilar el canal de control. S es el número de bloques de sincronización (SSB) que emite la celda, uno por haz. X es cuántas oportunidades hay por cada uno (TS 38.304, apartado 7.1). Así, la estación base puede repetir el aviso en todos los haces, porque no sabe en qué dirección está el móvil. Si X es mayor que 1 y el móvil detecta el P-RNTI en una de ellas, no tiene que vigilar el resto de esa ocasión (TS 38.304, apartado 7.1).

**DRX extendido (eDRX).** Para aparatos que pueden tardar en contestar, la red puede configurar ciclos mucho más largos. Van de 2,56 s a 10 485,76 s, unas 2,91 horas (TS 38.300, apartado 9.2.10). Por encima de 10,24 s, el móvil no vigila cada ciclo. Solo escucha durante una ventana periódica (PTW) dentro de una hipertrama concreta (PH) (TS 38.300, apartado 9.2.10; TS 38.304, apartado 7.4). Una hipertrama son 1024 tramas, 10,24 s. El ciclo máximo, 1024 hipertramas, da los 10 485,76 s de la norma. Con eDRX, el UE_ID pasa a ser el 5G-S-TMSI módulo 4096 (TS 38.304, apartado 7.1).

**El aviso anticipado (PEI).** Muchos móviles comparten la misma ocasión de aviso. Si uno de ellos tiene un aviso, todos los demás se despiertan para nada. Desde la versión 17, la red puede emitir un aviso anticipado (PEI) poco antes de la ocasión (TS 38.304, apartado 7.2). Los móviles de una ocasión se reparten en subgrupos, hasta 8 por celda (TS 38.300, apartado 9.2.5). El móvil vigila una ocasión de PEI por ciclo. Si no detecta PEI, o el PEI no señala su subgrupo, no tiene que vigilar la ocasión de aviso (TS 38.304, apartado 7.2). La norma lo justifica para reducir el consumo, pero no cuantifica el ahorro.

### 4. Diagrama o flujo

```text
 REGISTRO (antes de la llamada)
 Móvil ── registro en la zona TA 7 ──────────────────────────────► Red central (AMF)
 Móvil ◄──────────────── lista de zonas {TA 7, TA 8} + 5G-S-TMSI ── AMF
 Móvil: pasa a reposo, arranca T3512 (54 min por defecto) y acampa por su cuenta

 RECEPCIÓN DISCONTINUA (ejemplo: T = 1,28 s)
 tiempo ─►  ·····█···········█···········█···········█·····
                 PO          PO          PO          PO      (█ = escucha; · = duerme)

 LLAMADA ENTRANTE
 AMF ── petición de aviso (5G-S-TMSI) ──► gNB A (TA 7), gNB B (TA 7), gNB C (TA 8) …
 gNB A, B, C: calculan PF y PO a partir de UE_ID = 5G-S-TMSI mod 1024
 gNB A, B, C ── en el PO: PDCCH con P-RNTI + mensaje Paging, haz a haz ──► todas sus celdas
 Móvil (acampado en una celda de gNB B): despierta en su PO
                                          ¿está mi 5G-S-TMSI en la lista?
                                          ├─ no → vuelve a dormir
                                          └─ sí → acceso aleatorio + petición de conexión
 Móvil ── conexión establecida ──► gNB B ──► AMF: la red ya sabe la celda exacta
```

## Límites, excepciones y casos reales

- **El tamaño de la zona es un compromiso.** Una lista de zonas grande evita registros, pero cada aviso ocupa el canal de control de muchas celdas. Una lista pequeña ahorra avisos, pero obliga a registrarse más a menudo. La norma da las herramientas, no el tamaño. Cada operador decide.
- **La red puede no encontrarte.** En un sótano o sin cobertura, el móvil no oye el aviso. La red reintenta según su estrategia. Si el temporizador de alcanzabilidad vence, lo normal es que deje de avisar (TS 24.501, tabla 10.2.2). Qué pasa entonces con la llamada, por ejemplo si va al buzón de voz, lo decide el operador.
- **El turno depende de una identidad que cambia.** En 4G, la ocasión de aviso se calculaba con la identidad permanente (IMSI). En 5G se calcula con la temporal, el 5G-S-TMSI (Ericsson, 2019). Además, en 5G es obligatorio renovar el 5G-S-TMSI después de un aviso, y el I-RNTI se usa una sola vez. En 4G, renovar la identidad temporal tras el aviso era opcional (Ericsson, 2019). Así, escuchar los avisos y las ocasiones no revela la identidad permanente del móvil.
- **Un móvil sin identidad temporal.** Antes de registrarse, el móvil usa UE_ID = 0 (TS 38.304, apartado 7.1). En la práctica solo pasa justo al encenderse.
- **Inactivo escucha dos avisos.** Un móvil en inactivo vigila tanto el aviso de la red de radio como el de la red central. Los dos usan la misma identidad para calcular la ocasión, así que coinciden en el tiempo (TS 38.300, apartado 9.2.5).
- **El eDRX no es para un teléfono de uso normal.** Con ciclos de horas, una llamada no llegaría a tiempo. Está pensado para aparatos que toleran esperas largas. Si un móvil concreto lo usa, y con qué valores, depende del aparato y del operador: queda **no verificado** aquí.

## Consecuencias prácticas

- **La llamada tarda en sonar.** Parte del retraso es esperar a la siguiente ocasión de aviso: hasta un ciclo entero. Después llegan el acceso aleatorio, la conexión y el resto del establecimiento de la llamada.
- **El reposo es barato en batería.** El móvil duerme la mayor parte del ciclo y solo despierta alrededor de su ocasión. El PEI y el eDRX recortan todavía más esas escuchas.
- **Moverse en reposo apenas genera tráfico.** Cambiar de celda dentro de la lista de zonas no exige ningún mensaje. Solo cruzar a una zona nueva, o que venza T3512, lo hace.
- **Las notificaciones de las apps usan el mismo camino.** Un mensaje que llega a un móvil en reposo es un dato entrante. Para entregarlo, la red primero tiene que avisar al móvil y esperar a que se conecte.
- **Para quien diseña la red, el aviso es un recurso escaso.** Cada aviso ocupa el canal de control en todas las celdas de la zona. Por eso existen el aviso escalonado por áreas, el agrupamiento de varios móviles en un mismo mensaje y los subgrupos.

## Preguntas frecuentes

### ¿Sabe la red en qué antena estoy cuando el móvil está en reposo?

No. Solo sabe en qué lista de zonas de seguimiento te registraste. La celda exacta la conoce cuando el móvil se conecta. Por eso el aviso se emite en muchas celdas a la vez.

### ¿Es lo mismo que la reselección de celda?

No. La reselección es cómo elige el móvil, por su cuenta, la celda en la que acampa. El aviso es cómo lo busca la red. Están relacionados: el móvil lee los avisos en la celda que ha elegido, y si esa celda está fuera de su lista de zonas, vuelve a registrarse.

### ¿Es lo mismo que la recepción discontinua de un móvil conectado?

Se parecen, pero no son lo mismo. Un móvil conectado también puede dormir entre ventanas de escucha, con ciclos propios de ese estado. Ahí la estación base conoce su celda y le habla por su identificador propio, el C-RNTI. En reposo no hay identificador propio en la celda: todos escuchan el mismo P-RNTI y cada uno busca su identidad dentro del mensaje.

### ¿Puede alguien rastrearme escuchando los avisos?

El diseño de 5G lo dificulta. El aviso lleva una identidad temporal, no la permanente. La ocasión se calcula con esa identidad temporal, y la red debe renovarla tras cada aviso (Ericsson, 2019). Este artículo no entra en ataques concretos.

## Fuentes

- ETSI / 3GPP. *TS 38.304: NR; User Equipment (UE) procedures in Idle mode and RRC Inactive state* (V18.1.0, Rel-18). Apartados 3.1, 4.1, 7.1, 7.2, 7.3 y 7.4. <https://www.etsi.org/deliver/etsi_ts/138300_138399/138304/18.01.00_60/ts_138304v180100p.pdf>
- ETSI / 3GPP. *TS 38.300: NR; NR and NG-RAN Overall description; Stage-2* (V18.3.0, Rel-18). Apartados 7.2, 8.2 y 9.2.5. <https://www.etsi.org/deliver/etsi_ts/138300_138399/138300/18.03.00_60/ts_138300v180300p.pdf>
- 3GPP. *TS 38.300*, apartado 9.2.10 (*Extended DRX for RRC_IDLE and RRC_INACTIVE*), texto de Rel-19 consultado en una copia de la norma. <https://itecspec.com/spec/3gpp-38-300-9-2-10-extended-drx-for-rrc_idle-and-rrc_inactive>
- ETSI / 3GPP. *TS 38.331: NR; Radio Resource Control (RRC) protocol specification* (V17.5.0, Rel-17). Apartado 5.3.2 (mensaje *Paging*). <https://www.etsi.org/deliver/etsi_ts/138300_138399/138331/17.05.00_60/ts_138331v170500p.pdf>
- 3GPP. *TS 24.501: Non-Access-Stratum (NAS) protocol for 5G System (5GS); Stage 3* (Rel-19). Tablas 10.2.1 y 10.2.2 y apartado 5.5.1.3.2, consultados en copias de la norma. <https://itecspec.com/spec/3gpp-24-501-10-list-of-system-parameters> · <https://whatthespec.net/sos/spec/ts-24501/5.5.1.3.2>
- Ericsson. Artículo técnico de Ohlsson, Nakarmi y Hedman sobre la privacidad del aviso en 5G (mayo de 2019). Identidad usada para calcular la ocasión en 4G y en 5G, identidades permitidas en el aviso y renovación obligatoria tras el aviso. <https://www.ericsson.com/en/blog/2019/5/fighting-imsi-catchers-5g-cellular-paging-privacy>
