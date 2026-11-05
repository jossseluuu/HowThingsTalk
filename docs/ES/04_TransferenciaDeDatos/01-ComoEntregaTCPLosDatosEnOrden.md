---
title: "¿Cómo entrega TCP los datos en orden si los paquetes llegan desordenados?"
section: "04_TransferenciaDeDatos"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "TCP convierte los datos de la aplicación en un flujo de bytes numerado. Usa confirmaciones, búferes y retransmisiones para entregar ese flujo en orden aunque los segmentos lleguen tarde, repetidos o desordenados."
prerequisites:
  - "Paquetes y capas de red"
  - "Direcciones IP y puertos"
  - "Latencia y rendimiento efectivo"
translation: "../../EN/04_DataTransfer/01-HowTCPDeliversDataInOrder.md"
---

# ¿Cómo entrega TCP los datos en orden si los paquetes llegan desordenados?

## Pregunta y encuadre

Cuando una aplicación envía datos por Internet, la red no garantiza que todos los paquetes recorran exactamente la misma ruta ni que lleguen en el mismo orden. Algunos pueden retrasarse, perderse o aparecer duplicados. Sin embargo, una aplicación que usa TCP recibe un flujo de bytes ordenado y fiable.

La pregunta es cómo consigue TCP esa entrega ordenada. El artículo se centra en el transporte TCP entre dos extremos. No describe cómo una aplicación divide un archivo en mensajes, ni cómo IP enruta los datagramas, ni cómo una aplicación confirma que un archivo guardado es idéntico al original. Se necesita conocer la diferencia entre una dirección IP, un puerto y un paquete. La profundidad es intermedia.

## Respuesta corta

TCP numera cada byte del flujo, almacena temporalmente los datos que llegan fuera de orden y solicita o realiza la retransmisión de los bytes que faltan antes de entregarlos a la aplicación.

### Resumen de 60 segundos

TCP no considera que cada segmento sea un mensaje independiente de la aplicación. Ofrece un flujo continuo de bytes. Cada byte ocupa una posición en una secuencia numérica. Los segmentos transportan rangos de esa secuencia.

El receptor confirma de forma acumulativa cuál es el siguiente byte que espera. Si recibe más adelante bytes posteriores a un hueco, puede guardarlos en un búfer, pero no adelanta la confirmación acumulativa ni entrega a la aplicación un flujo con ese hueco. El emisor interpreta las confirmaciones y, si detecta una pérdida o vence el temporizador de retransmisión, vuelve a enviar los datos pendientes. Cuando llega el tramo que faltaba, el receptor puede unirlo con los datos ya almacenados y avanzar la entrega.

## Modelo mental

Imagina un libro cuyas páginas se envían por separado. Cada página lleva un número y el lector las coloca en una bandeja hasta poder leerlas en orden. Si recibe las páginas 1, 2 y 4, conserva la 4, pero no puede leer el libro completo saltándose la 3. Cuando llega la página 3, puede continuar.

La analogía ayuda a entender el almacenamiento temporal y el orden, pero se rompe en un punto importante: TCP no numera mensajes ni páginas. Numera bytes individuales de un flujo. Un segmento puede contener parte de un mensaje de la aplicación, varios mensajes o una cantidad de bytes que no coincide con las llamadas de escritura del programa.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

1. La aplicación entrega bytes a TCP. Para TCP forman un flujo continuo, sin conservar necesariamente los límites de cada llamada de escritura.
2. TCP divide ese flujo en segmentos adecuados a las condiciones del enlace y a los límites negociados. Cada segmento identifica el número de secuencia de su primer byte.
3. IP transporta cada segmento dentro de un datagrama. La red puede perderlos, duplicarlos o entregarlos en un orden distinto.
4. El TCP receptor comprueba el segmento y usa sus números de secuencia para saber qué posiciones del flujo contiene. Los bytes válidos que llegan después de un hueco pueden quedar almacenados temporalmente.
5. El receptor envía confirmaciones. La confirmación acumulativa indica el siguiente número de secuencia que espera; por tanto, confirma que todos los bytes anteriores ya han llegado.
6. El emisor conserva los datos todavía no confirmados. Si detecta indicios de pérdida o vence el temporizador correspondiente, retransmite los datos necesarios.
7. Cuando se rellena el hueco, el receptor puede avanzar la posición de recepción y entregar a la aplicación los bytes consecutivos disponibles.

### 2. Qué hace cada componente

**Número de secuencia.** Cada byte de datos ocupa una posición en el espacio de secuencia de 32 bits de TCP. El número de secuencia de un segmento identifica el primer byte que transporta. El espacio numérico da la vuelta al alcanzar su límite, por lo que las comparaciones se realizan con aritmética modular.

**Confirmación acumulativa (ACK).** El campo Acknowledgment Number indica el siguiente byte que el receptor espera. Si envía ACK = 5001, está confirmando que ha recibido todos los bytes anteriores a 5001. No significa que haya recibido un paquete con ese número: los números corresponden al flujo de bytes.

**Búfer de recepción.** Mantiene temporalmente los datos recibidos que todavía no pueden formar una secuencia continua desde el siguiente byte esperado. Así, un segmento posterior no tiene que descartarse solo porque otro anterior no haya llegado aún. El espacio disponible se anuncia mediante la ventana de recepción.

**Cola de retransmisión.** El emisor mantiene los datos enviados que aún no han sido confirmados. Cuando una confirmación acumulativa avanza, puede retirar de esa cola la parte ya reconocida. Los datos que siguen pendientes pueden necesitar retransmisión.

**Temporizador y detección de pérdidas.** TCP estima cuándo debería llegar una confirmación y usa un temporizador de retransmisión. También puede inferir una pérdida a partir de confirmaciones duplicadas. El mecanismo clásico de retransmisión rápida descrito en RFC 5681 usa tres ACK duplicados como indicio para retransmitir sin esperar al temporizador. No es una regla que describa todas las implementaciones modernas ni todos los algoritmos de recuperación.

**SACK (Selective Acknowledgment).** Si ambos extremos lo negocian, el receptor puede indicar bloques concretos de datos que ya recibió más allá del hueco. El emisor obtiene así información más precisa y puede retransmitir solo los rangos que faltan. SACK complementa la confirmación acumulativa; no cambia el significado del ACK principal.

### 3. Parámetros que importan

- **Número de secuencia:** es un valor de 32 bits. Identifica posiciones de bytes, no números de paquetes ni mensajes de aplicación (RFC 9293, sección 3.4).
- **MSS (Maximum Segment Size):** limita la cantidad de datos TCP que cabe en un segmento, sin contar las cabeceras TCP e IP. Su valor efectivo depende de la MTU del camino, de las opciones y de la negociación entre extremos.
- **Ventana de recepción (`rwnd`):** expresa cuánto espacio de recepción está dispuesto a aceptar el extremo remoto. Evita que el emisor desborde el búfer del receptor.
- **Ventana de congestión (`cwnd`):** limita cuánto dato puede mantener el emisor en vuelo por motivos de congestión. No es lo mismo que la ventana de recepción. La cantidad que puede enviarse sin nueva confirmación queda limitada por ambas.
- **RTT y temporizador de retransmisión:** el RTT es el tiempo de ida y vuelta de los segmentos y sus confirmaciones. TCP ajusta el temporizador según las estimaciones del RTT y su variabilidad; no existe un único plazo fijo aplicable a todas las conexiones.
- **SACK:** es una extensión negociable. No debe darse por hecho que está disponible en toda conexión TCP.

### 4. Diagrama o flujo

```text
Aplicación emisora
       |
       v
Flujo de bytes -> TCP numera y segmenta
       |
       v
Datagramas IP -> red (pueden perderse o reordenarse)
       |
       v
TCP receptor -> valida números de secuencia
       |
       +--> Datos posteriores al hueco: búfer temporal
       |
       +--> ACK acumulativo: siguiente byte esperado
       |           |
       |           v
       |     TCP emisor: actualiza pendientes
       |           |
       |           +--> pérdida detectada/timeout -> retransmisión
       |
       v
Hueco rellenado -> flujo continuo -> aplicación receptora
```

## Límites, excepciones y casos reales

**TCP ordena bytes, no mensajes.** Si una aplicación realiza dos escrituras, el receptor no tiene por qué obtener dos lecturas con los mismos límites. La aplicación necesita su propio encuadre si debe distinguir mensajes, por ejemplo mediante longitudes o delimitadores definidos por su protocolo.

**La llegada desordenada no demuestra por sí sola una pérdida.** Un segmento puede estar simplemente retrasado o haber seguido una ruta distinta. Las confirmaciones duplicadas son indicios, no una prueba infalible. Por eso los algoritmos combinan varias señales y temporizadores.

**SACK no sustituye al ACK acumulativo.** Describe bloques recibidos fuera de orden, mientras que el ACK principal sigue señalando el primer byte que falta. Su uso depende de que se haya permitido durante el establecimiento de la conexión.

**La recuperación tiene un coste.** Los datos posteriores a un hueco ocupan memoria y no pueden adelantarse a la aplicación como si el flujo estuviera completo. Una pérdida puede añadir latencia aunque el resto de los segmentos ya haya llegado. Si la conexión se interrumpe definitivamente, TCP no puede garantizar que la aplicación reciba el resto del flujo.

**TCP no garantiza la integridad semántica de un archivo.** Su checksum detecta determinados errores en segmentos, pero no sustituye a una suma criptográfica del archivo ni demuestra que el contenido guardado sea el esperado. Esa comprobación pertenece a capas superiores o a la propia aplicación.

**No todo tráfico usa TCP.** UDP y otros transportes tienen modelos distintos. No se debe atribuir a cualquier transferencia de datos la entrega ordenada y fiable que proporciona TCP.

## Consecuencias prácticas

Una descarga TCP puede parecer continua aunque debajo haya retransmisiones, segmentos duplicados o llegadas fuera de orden. La aplicación recibe bytes ordenados y no necesita gestionar cada paquete IP.

El precio de esa abstracción es que una pérdida puede bloquear temporalmente la entrega de bytes posteriores. En una transferencia de archivo suele ser preferible esperar y reconstruir el flujo. En aplicaciones interactivas sensibles al retardo, esa espera puede ser más molesta que perder una pequeña parte de información; por eso algunas aplicaciones eligen transportes o protocolos con garantías diferentes.

También explica por qué el tamaño de un `send()` no predice el tamaño de los segmentos observados en la red, ni el de los `read()` que realiza el receptor. TCP conserva el orden de los bytes, no las fronteras de las operaciones de la aplicación.

## Preguntas frecuentes

### ¿TCP vuelve a ordenar los paquetes IP?

No modifica el orden en que la red los entregó. Usa los números de secuencia para colocar los bytes en sus posiciones y solo presenta a la aplicación el flujo consecutivo disponible.

### ¿Qué ocurre si llega dos veces el mismo segmento?

Los números de secuencia permiten reconocer los bytes repetidos. El receptor no debe entregar una segunda copia de esos bytes al flujo de la aplicación y puede volver a confirmar la posición que espera.

### ¿Un ACK confirma que la aplicación ya guardó los datos en disco?

No. Confirma la recepción en la pila TCP del extremo remoto, no que la aplicación los haya procesado, escrito en disco o validado.

### ¿TCP siempre retransmite exactamente el mismo paquete?

No necesariamente. TCP retransmite los datos pendientes, pero el segmento que los contiene puede construirse de nuevo y no tiene por qué reproducir idéntica la encapsulación de la transmisión anterior.

## Fuentes

- [RFC 9293 — Transmission Control Protocol (TCP), IETF (2022)](https://www.rfc-editor.org/rfc/rfc9293.html). Especificación base: flujo de bytes, números de secuencia, confirmaciones, segmentación y ventanas.
- [RFC 2018 — TCP Selective Acknowledgment Options, IETF (1996)](https://www.rfc-editor.org/rfc/rfc2018.html). Define la negociación y el contenido de las confirmaciones selectivas SACK.
- [RFC 5681 — TCP Congestion Control, IETF (2009)](https://www.rfc-editor.org/rfc/rfc5681.html). Describe control de congestión y el mecanismo clásico de retransmisión rápida basado en ACK duplicados.
