---
title: "¿Cómo evita el móvil que sus archivos se corrompan si se apaga de golpe?"
section: "06_Almacenamiento"
language: "es"
status: "published"
order: 2
depth: "intermediate"
summary: "Guardar un archivo obliga a escribir en varios sitios del almacenamiento, y un corte entre una escritura y otra dejaría las estructuras del sistema de archivos a medias. Para evitarlo, el sistema de archivos agrupa los cambios en transacciones que se aplican enteras o no se aplican. Lo consigue con un diario (ext4), con copia en escritura y puntos de control (APFS) o con un registro y puntos de control alternos (F2FS). Todos dependen de que la memoria cumpla las órdenes de vaciar su caché. Lo que protegen es la estructura del sistema de archivos, no los últimos datos que aún no se habían confirmado."
prerequisites:
  - "Qué es un archivo y qué es un directorio"
  - "Páginas, bloques y escritura fuera de sitio en la memoria flash (artículo 06·01)"
translation: "../../EN/06_Storage/02-HowFileSystemsSurviveSuddenPowerLoss.md"
---

# ¿Cómo evita el móvil que sus archivos se corrompan si se apaga de golpe?

## Pregunta y encuadre

El móvil se cuelga mientras guarda una foto. O se reinicia a la fuerza, o se queda sin batería de golpe. Al volver a encenderlo, la galería, los chats y los ajustes casi siempre siguen bien. La pregunta de este artículo es por qué: ¿qué hace el sistema de archivos para que un corte en mitad de una escritura no deje el almacenamiento en un estado incoherente?

El recorrido empieza en el problema: una sola operación del usuario se convierte en varias escrituras en sitios distintos. Después explica las tres estrategias que usan los sistemas de archivos de los móviles. Son el diario de ext4, la copia en escritura de APFS y los puntos de control de F2FS. Termina en lo que estas técnicas no garantizan. No es una guía para recuperar datos ni para reparar un disco, y no compara modelos de móvil.

La profundidad es intermedia. Basta con saber qué es un archivo y un directorio. Ayuda haber leído el [artículo 06·01](01-QuePasaEnLaFlashCuandoBorrasUnaFoto.md), que explica por qué la memoria flash escribe por páginas y nunca sobrescribe en el sitio.

## Respuesta corta

El sistema de archivos nunca deja una actualización a medias en el sitio definitivo: primero deja constancia completa del cambio en otro lugar, marca el final con un registro que lleva suma de comprobación, y al arrancar descarta todo lo que no tenga ese registro.

### Resumen de 60 segundos

Guardar una foto nueva exige varias escrituras. Hay que escribir los datos de la foto. Hay que crear su inodo, la ficha con su tamaño y la lista de bloques que ocupa. Hay que marcar esos bloques como ocupados y añadir el nombre al directorio. Si la corriente se va entre dos de esas escrituras, el directorio puede apuntar a una ficha vacía, o pueden quedar bloques ocupados que nadie usa.

La solución es tratar el grupo de escrituras como una transacción: o se aplica entera o no se aplica. Hay tres formas de hacerlo:

1. **Diario (ext4).** Antes de tocar las estructuras, se escribe el cambio completo en una zona aparte llamada diario y se cierra con un bloque de confirmación. Al arrancar se rehacen las transacciones confirmadas y se descartan las demás.
2. **Copia en escritura (APFS).** Nada se modifica en el sitio. Las versiones nuevas se escriben en bloques libres y luego se publica un punto de control que apunta a ellas. Si el punto de control queda a medias, se ignora y vale el anterior.
3. **Registro con puntos de control (F2FS).** Se escribe siempre al final de un registro y se alternan dos copias del punto de control. Al arrancar se elige la copia válida más reciente.

Las tres dependen de una condición: que la memoria escriba de verdad lo que dice haber escrito, en el orden que se le pide.

## Modelo mental

Imagina un contable que lleva un libro mayor. Antes de mover dinero entre cuentas, apunta en un cuaderno aparte todo lo que va a hacer: «restar 50 a la cuenta A, sumar 50 a la cuenta B». Cuando la nota está completa, firma debajo «hecho». Solo entonces corrige el libro mayor. Si le interrumpen, al volver mira el cuaderno. Si la última nota está firmada, la repite entera. Si no está firmada, la tacha. El libro nunca queda con el dinero restado de A y sin sumar a B.

La analogía se rompe en tres puntos. Primero, el contable ve el papel y sabe en qué orden ha escrito. El sistema operativo no: la memoria tiene una caché que puede confirmar una escritura antes de guardarla y puede reordenarlas. Hay que obligarla con órdenes explícitas. Segundo, no todos los sistemas usan cuaderno. APFS y F2FS nunca corrigen el libro en el sitio: escriben una página nueva y cambian un único puntero al final. Tercero, el cuaderno protege la contabilidad, no el contenido de los sobres. Con la configuración por defecto de ext4, el diario guarda los metadatos, no los datos de tus archivos.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

**El problema: una operación, varias escrituras.** Crear un archivo de foto modifica al menos cuatro estructuras: los bloques de datos, el inodo, el mapa de bloques libres y la entrada del directorio. Están en lugares distintos del almacenamiento. Ninguna memoria escribe varios sitios a la vez de forma atómica. Un corte entre dos escrituras produce combinaciones que no deberían existir:

- el directorio contiene el nombre, pero el inodo aún no está escrito;
- el inodo apunta a bloques que el mapa sigue marcando como libres y que otro archivo podría reutilizar;
- los bloques están marcados como ocupados, pero ningún inodo los usa, y ese espacio se pierde.

**El agravante: el orden se pierde por el camino.** Entre la aplicación y las celdas hay dos cachés. La primera es la caché de páginas del sistema operativo, en RAM. La segunda es la caché de escritura de la propia memoria. La documentación del núcleo Linux advierte de que muchos dispositivos avisan de que una escritura ha terminado antes de que los datos lleguen al almacenamiento no volátil. Por eso el sistema operativo tiene que forzar el vaciado de esa caché en `fsync`, en `sync` y al desmontar (documentación del núcleo, *Explicit volatile write back cache control*).

El núcleo dispone de dos herramientas para imponer orden:

- **Vaciado previo (`REQ_PREFLUSH`).** Antes de ejecutar una escritura, la memoria vacía su caché. Así, todo lo escrito antes queda en almacenamiento no volátil.
- **Acceso forzado (`REQ_FUA`).** La memoria solo da por terminada esa escritura cuando sus datos están en almacenamiento no volátil.

Con estas dos órdenes, el sistema de archivos puede garantizar que un bloque concreto llega al soporte después de otros. Todas las estrategias que siguen se construyen sobre esa garantía.

### 2. Qué hace cada componente

**El diario de ext4 (jbd2).** ext4 es uno de los sistemas de archivos que Android mantiene con parches de seguridad frecuentes, junto con F2FS (documentación de AOSP). Su diario funciona así (documentación del núcleo, *Journal (jbd2)*):

1. **Bloque descriptor.** Abre la transacción y lista en qué posición definitiva irá cada bloque que sigue.
2. **Bloques de datos del diario.** Son copias exactas de los bloques de metadatos modificados: inodos, mapas, directorios.
3. **Bloque de confirmación.** Cierra la transacción. Se escribe solo cuando todo lo anterior está en el soporte y fuera de la caché de escritura. Puede llevar una suma de comprobación de la transacción, normalmente CRC32 o CRC32C.
4. **Punto de control.** Más tarde, los bloques se copian a su posición definitiva y la transacción se retira del diario.

Al montar tras un corte, ext4 recorre el diario. Las transacciones con bloque de confirmación válido se rehacen hasta la última confirmada. Si falta la confirmación o la suma no coincide, la transacción se descarta. El resultado es que una actualización de metadatos nunca queda a medias.

ext4 tiene tres modos de datos (guía de administración de ext4):

- **`data=ordered`**, el modo por defecto. Solo los metadatos pasan por el diario. Los datos de los archivos se escriben en su sitio definitivo antes de confirmar los metadatos que apuntan a ellos. Así, un inodo nunca apunta a bloques con restos de otro archivo.
- **`data=journal`**. Datos y metadatos pasan por el diario. Es más lento porque todo se escribe dos veces.
- **`data=writeback`**. No impone orden entre datos y metadatos. Tras un corte, un archivo puede contener datos antiguos.

Las barreras de escritura, que imponen el orden de las confirmaciones con la caché activada, están activadas por defecto. La transacción en curso se confirma como mucho cada 5 segundos por defecto (opción `commit`).

**La copia en escritura de APFS.** APFS es el sistema de archivos de los aparatos de Apple. Su especificación lo dice sin rodeos: los objetos en disco nunca se modifican en el sitio (Apple File System Reference, «Objects»). Para cambiar un objeto se escribe una copia nueva en otro bloque. Cada objeto lleva una suma de comprobación Fletcher de 64 bits y el número de la última transacción que lo modificó. Esos números de transacción solo crecen y nunca se reutilizan.

Las versiones nuevas se hacen visibles mediante **puntos de control**. El estado que vive en memoria se escribe periódicamente en un punto de control, seguido de una copia del superbloque del contenedor, la estructura raíz. Hay dos zonas: una de descriptores, con los superbloques y mapas, y otra de datos, con los objetos del estado. Si escribir un punto de control se interrumpe, ese punto de control no es válido. Al montar se usa el más reciente que sea válido, y el sistema vuelve al último estado coherente (Apple File System Reference, «About Apple File System»).

**El registro y los puntos de control de F2FS.** F2FS es un sistema de archivos de estructura de registro diseñado para memoria flash. Escribe los bloques nuevos en espacio libre en lugar de sobrescribir los antiguos (Lee et al., 2015, apartado 1). Su protección ante cortes tiene dos piezas (apartado 2.7):

- **Dos paquetes de punto de control alternos.** Uno guarda la última versión estable y el otro la intermedia. Cada paquete lleva una cabecera y un pie con un número de versión que crece en cada punto de control. Al montar, un paquete es válido si su cabecera y su pie coinciden. Si los dos son válidos, gana el de versión más alta. F2FS crea un punto de control en momentos como `sync`, el desmontaje o la limpieza en primer plano.
- **Recuperación hacia delante para `fsync`.** Crear un punto de control completo en cada `fsync` sería lento. En su lugar, F2FS escribe solo los bloques de datos y los nodos directos que apuntan a ellos, con una marca especial. Al arrancar, busca los nodos marcados posteriores al último punto de control y aplica esos cambios encima (apartado 2.7.2).

En un Galaxy S4 con Android 4.2.2, F2FS hizo una carga de SQLite hasta 2 veces más rápido que ext4 y escribió alrededor de un 46 % menos de datos (apartado 3.2.1). Son medidas de 2015 en un aparato concreto. Sirven para ver por qué el coste de cada `fsync` importa en un móvil, no para predecir el de un aparato actual.

**La aplicación, la última capa.** El sistema de archivos protege sus propias estructuras. No sabe qué significa el contenido de tus archivos. Las aplicaciones que necesitan atomicidad la construyen encima. SQLite, la base de datos que usan muchas aplicaciones para mensajes, contactos o ajustes, lo hace con un diario de reversión (documentación de SQLite, *Atomic Commit In SQLite*):

1. Copia en un archivo de diario el contenido original de las páginas que va a cambiar.
2. Vacía ese diario al almacenamiento con `fsync`. En el modo por defecto lo hace dos veces: primero el contenido y luego la cabecera con el número de páginas.
3. Escribe los cambios en la base de datos y la vacía al almacenamiento.
4. Borra el diario. Ese borrado es el instante en que la transacción se confirma.

Si al abrir la base encuentra un diario completo, restaura las páginas originales. La base vuelve al estado anterior a la transacción interrumpida.

### 3. Parámetros que importan

| Parámetro | Valor | Escenario y fuente |
|---|---|---|
| Modo de datos por defecto de ext4 | `data=ordered` | guía de administración de ext4 |
| Intervalo máximo de confirmación de ext4 | 5 s | opción `commit`, valor por defecto (guía de administración de ext4) |
| Barreras de escritura en ext4 | activadas | valor por defecto (guía de administración de ext4) |
| Suma del bloque de confirmación | CRC32 o CRC32C, habitualmente | diario jbd2 (documentación del núcleo) |
| Suma de cada objeto en APFS | Fletcher de 64 bits, 8 bytes | campo `o_cksum` (Apple File System Reference) |
| Paquetes de punto de control en F2FS | 2, alternos | Lee et al., apartado 2.1 |
| SQLite en F2FS frente a ext4 | hasta 2× más rápido, ~46 % menos escritura | Galaxy S4, Android 4.2.2, modo WAL (Lee et al., 3.2.1) |
| Vaciados del diario por transacción en SQLite | 2 | modo `synchronous=FULL`, el de por defecto (documentación de SQLite) |
| SSD con fallos ante cortes de corriente | 13 de 15 | 15 SSD de 5 fabricantes, más de 3000 cortes provocados (Zheng et al., 2013) |

### 4. Diagrama o flujo

Una transacción de ext4 en modo `ordered` y lo que pasa si la corriente se va en cada punto:

```text
Tiempo ──────────────────────────────────────────────────────────────────────>

 1. Datos de la foto        2. Diario:               3. Diario:           4. Punto de control:
    a su sitio definitivo      descriptor + inodo,      bloque de            inodo, mapa y
    (vaciar caché)             mapa y directorio        confirmación         directorio a su sitio
                               (vaciar caché)           (FUA)
        │                          │                        │                     │
   corte A                    corte B                  corte C               corte D
        │                          │                        │                     │
        v                          v                        v                     v
 Bloques escritos que       Sin confirmación:        Confirmada: al        Confirmada: al montar
 nadie referencia aún.      se descarta. El          montar se rehace.     se rehace (rehacer dos
 La foto no existe.         archivo no existe.       La foto existe.       veces da lo mismo).
```

En ningún punto queda un directorio que apunte a un inodo a medias. Lo peor que pasa es perder la foto entera, nunca tener media foto con metadatos corruptos.

La alternancia de puntos de control de F2FS:

```text
           Paquete CP n.º 0            Paquete CP n.º 1
           ┌──────────────┐            ┌──────────────┐
 antes     │ versión 41 ✓ │            │ versión 42 ✓ │  <- el más reciente válido
           └──────────────┘            └──────────────┘
 escribe   │ cabecera 43  │  corte
 el 0      │ ...          │  ──>  cabecera 43 ≠ pie 41: paquete 0 no válido
           └──────────────┘
 al montar ──────────────────────>  se usa el paquete 1 (versión 42)
```

## Límites, excepciones y casos reales

**Coherente no significa completo.** Estas técnicas garantizan que las estructuras del sistema de archivos queden coherentes. No garantizan que se conserve lo último que escribiste. En ext4, un corte puede perder hasta los últimos 5 segundos de cambios de metadatos. Con la asignación diferida de bloques puede perder incluso datos más antiguos, porque su escritura no empieza hasta que vence otro plazo del núcleo (guía de administración de ext4). Una aplicación que necesita que algo sobreviva tiene que pedir `fsync` y esperar a que termine.

**La memoria puede no cumplir.** Todo lo anterior supone que la memoria respeta el vaciado de caché y el orden. Zheng y colaboradores provocaron más de 3000 cortes de corriente en 15 SSD de 5 fabricantes. Trece de los quince mostraron fallos (FAST 2013, resumen). Encontraron bits corrompidos en 3 discos y escrituras cortadas a mitad en otros 3. En 8 hallaron escrituras no serializables: el estado final no correspondía a ningún orden posible de las escrituras pedidas. Un disco perdió metadatos internos y otro quedó inservible (tabla 5). Cuatro discos tenían protección contra cortes, como un supercondensador. Dos de ellos no fallaron y los otros dos sí (tablas 3 y 5). El estudio usó SSD de ordenador de 2013, no memorias de móvil. No se ha revisado aquí un estudio equivalente sobre las memorias UFS o eMMC de los móviles actuales, así que su comportamiento ante cortes queda **no verificado**.

**El sistema de archivos no detecta todos los errores de datos.** En ext4 en modo `ordered`, los datos de los archivos no llevan suma de comprobación del diario. SQLite tampoco añade redundancia a la base de datos, y su documentación reconoce que no puede defenderse de un sistema operativo o una memoria que no cumplan lo que prometen (apartados 2 y 9.4).

**La recuperación tiene un coste.** Rehacer el diario o buscar el punto de control válido lleva tiempo al arrancar. Por eso un móvil puede tardar algo más en encenderse tras un reinicio forzado que tras un apagado normal. La magnitud depende del aparato y de cuánto quedó pendiente; no hay una cifra general verificada.

**Un móvil casi nunca pierde la corriente de golpe.** La batería está integrada y el sistema se apaga de forma ordenada cuando se agota. Los casos reales son más bien cuelgues del núcleo y reinicios forzados. En un cuelgue se pierde todo lo que estaba en RAM, incluida la caché de páginas, igual que en un corte. Para el sistema de archivos, la recuperación es la misma.

## Consecuencias prácticas

Para quien usa el móvil, un reinicio forzado rara vez estropea el almacenamiento. Lo que sí puede perderse es lo último que se estaba guardando: la foto de hace un segundo, el mensaje recién escrito o un cambio de ajustes. Que se conserve o no depende de si la aplicación llegó a confirmar la escritura con `fsync`.

Un apagado normal es más seguro que uno forzado por una razón concreta. Al desmontar, el sistema operativo vacía todas las cachés y escribe un punto de control limpio. No queda nada que recuperar al arrancar.

Para quien diseña aparatos y aplicaciones, cada `fsync` cuesta tiempo y escrituras en la flash. SQLite vacía su diario dos veces por transacción en el modo por defecto. Por eso los sistemas de archivos para móviles se esfuerzan en abaratar `fsync`, como la recuperación hacia delante de F2FS. También por eso las aplicaciones agrupan muchos cambios en una sola transacción en lugar de confirmar uno a uno.

## Preguntas frecuentes

### ¿Es lo mismo que la escritura fuera de sitio de la memoria flash?

No, aunque se parecen. La capa de traducción de la flash (FTL) escribe fuera de sitio por una limitación física: no puede sobrescribir una página ([artículo 06·01](01-QuePasaEnLaFlashCuandoBorrasUnaFoto.md)). La copia en escritura de APFS o el registro de F2FS escriben fuera de sitio por una decisión de diseño: mantener intacta la versión anterior hasta que la nueva esté completa. Son dos capas distintas, y cada una protege su propia tabla de traducción.

### ¿El diario guarda una copia de mis fotos?

Con la configuración por defecto de ext4, no. El modo `ordered` solo pasa por el diario los metadatos. Los datos de la foto se escriben una vez, en su sitio definitivo, antes de confirmar los metadatos. El modo `data=journal` sí escribe los datos dos veces, una en el diario y otra en su sitio, y por eso es más lento.

### ¿Por qué hay que «expulsar» un pendrive antes de quitarlo?

Por las cachés. Parte de lo que has copiado puede estar aún en la RAM del ordenador o en la caché de la memoria USB. Expulsar desmonta el sistema de archivos, y el núcleo fuerza que todo llegue al almacenamiento no volátil al desmontar. Si lo quitas antes, interrumpes las escrituras pendientes igual que un corte de corriente.

### ¿Hace falta comprobar el disco tras un corte, como antes?

Con un sistema de archivos con diario o con copia en escritura, normalmente no. Rehacer el diario o elegir el último punto de control válido deja las estructuras coherentes en poco tiempo. Las herramientas de comprobación completa siguen existiendo para otros daños, por ejemplo bloques que la memoria ha corrompido.

## Fuentes

- [The Linux Kernel documentation — ext4 Data Structures and Algorithms: Journal (jbd2)](https://www.kernel.org/doc/html/latest/filesystems/ext4/journal.html). Estructura de la transacción (descriptor, bloques de datos, revocación y confirmación), descarte de transacciones sin confirmación o con suma incorrecta, sumas de comprobación del bloque de confirmación y modo por defecto que solo registra metadatos.
- [The Linux Kernel documentation — ext4 General Information](https://docs.kernel.org/admin-guide/ext4.html). Modos `data=journal`, `data=ordered` y `data=writeback`, intervalo `commit` de 5 s por defecto y posible pérdida de los últimos cambios, barreras de escritura activadas por defecto y reproducción del diario al montar.
- [The Linux Kernel documentation — Explicit volatile write back cache control](https://docs.kernel.org/block/writeback_cache_control.html). Cachés de escritura volátiles que confirman antes de guardar, vaciado obligatorio en `fsync`, `sync` y desmontaje, y semántica de `REQ_PREFLUSH` y `REQ_FUA`.
- [Apple — Apple File System Reference (2020)](https://developer.apple.com/support/apple-file-system/Apple-File-System-Reference.pdf). Protección ante fallos del contenedor, varias copias del superbloque, zonas de descriptores y de datos del punto de control, puntos de control interrumpidos no válidos («About Apple File System»), objetos nunca modificados en el sitio, suma Fletcher de 64 bits e identificadores de transacción crecientes («Objects»).
- [Lee, C., Sim, D., Hwang, J., Cho, S. — F2FS: A New File System for Flash Storage. USENIX FAST, 2015](https://www.usenix.org/conference/fast15/technical-sessions/presentation/lee). Diseño de registro (apartado 1), dos paquetes de punto de control y elección del válido (2.1 y 2.7), recuperación hacia delante para `fsync` (2.7.2) y medidas en un Galaxy S4 (3.1 y 3.2.1).
- [The Linux Kernel documentation — WHAT IS Flash-Friendly File System (F2FS)?](https://docs.kernel.org/filesystems/f2fs.html). Base de estructura de registro, opciones `checkpoint`, `fsync_mode` y `disable_roll_forward`.
- [Zheng, M., Tucek, J., Qin, F., Lillibridge, M. — Understanding the Robustness of SSDs under Power Fault. USENIX FAST, 2013](https://www.usenix.org/system/files/conference/fast13/fast13-final80.pdf). Quince SSD de cinco fabricantes, más de 3000 cortes provocados, trece con fallos (resumen), discos con protección contra cortes (tabla 3) y tipos de fallo por disco (tabla 5).
- [SQLite — Atomic Commit In SQLite](https://www.sqlite.org/atomiccommit.html). Diario de reversión y punto de confirmación (apartados 3.4 a 3.12), recuperación con diario activo (4), doble vaciado en `synchronous=FULL` (6.2), supuestos sobre el almacenamiento (2) y límites ante vaciados que no se cumplen (9.2 y 9.4).
- [Android Open Source Project — Android kernel file system support](https://source.android.com/docs/core/architecture/android-kernel-file-system-support). ext4 y F2FS entre los sistemas de archivos que reciben parches de seguridad frecuentes.
