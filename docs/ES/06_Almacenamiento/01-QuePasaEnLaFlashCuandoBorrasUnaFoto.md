---
title: "¿Qué ocurre en las celdas de memoria flash cuando borras una foto?"
section: "06_Almacenamiento"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "Al borrar una foto, las celdas de memoria flash no cambian en ese momento. El sistema de archivos marca su espacio como libre y avisa a la memoria con TRIM, UNMAP o Deallocate. El controlador deja de mapear esas páginas, pero los electrones siguen en su sitio hasta que la recolección de basura borra el bloque entero con alta tensión. En un móvil moderno lo que queda está cifrado, y la garantía real de borrado la da destruir claves, no celdas."
prerequisites:
  - "Qué es un bit y qué es un archivo"
  - "Idea básica de cifrado con clave (ayuda el artículo 10·01)"
translation: "../../EN/06_Storage/01-WhatHappensInFlashWhenYouDeleteAPhoto.md"
---

# ¿Qué ocurre en las celdas de memoria flash cuando borras una foto?

## Pregunta y encuadre

Pulsas «Eliminar» sobre una foto y desaparece de la galería. Parece que el móvil la ha destruido. La pregunta de este artículo es física y concreta: ¿qué les pasa en ese momento, y en los minutos u horas siguientes, a las celdas de memoria flash que guardaban esos bytes?

El recorrido va desde el toque en la pantalla hasta los electrones. Pasa por el sistema de archivos, por el aviso que el sistema operativo envía a la memoria y por el controlador interno que gestiona la flash. Termina en el borrado físico de un bloque y en el papel del cifrado. No es una guía para recuperar archivos ni para borrar un disco, y no compara modelos. La papelera de «Eliminados recientemente» de muchas galerías es una función de la aplicación: aquí se parte del momento en que el archivo se borra de verdad.

La profundidad es intermedia. Basta con saber qué es un bit y qué es un archivo. Para la parte final ayuda saber qué es una clave de cifrado ([artículo 10·01](../10_SeguridadYCriptografia/01-ComoAcuerdanDosAparatosUnaClaveSecreta.md)).

## Respuesta corta

Al borrar una foto no se toca ninguna celda: se borra la referencia al archivo, y los electrones que codifican la foto siguen atrapados hasta que el controlador decide borrar el bloque entero donde estaban, quizá mucho después.

### Resumen de 60 segundos

La memoria flash guarda cada bit como carga eléctrica atrapada en una celda. Tiene dos reglas que lo explican casi todo:

1. **Se escribe por páginas y se borra por bloques.** Una página mide del orden de 8 a 16 kB. Un bloque reúne cientos de páginas y solo se puede borrar entero.
2. **No se sobrescribe en el sitio.** Para cambiar una página hay que escribir la versión nueva en otra página libre. La antigua queda marcada como inválida.

Por eso, al borrar una foto, pasa esto:

1. El sistema de archivos marca como libres los sectores lógicos de la foto. No escribe nada sobre ellos.
2. El sistema operativo puede avisar a la memoria de que esos sectores ya no se usan. Ese aviso se llama TRIM en ATA, UNMAP en SCSI y UFS, y Deallocate en NVMe.
3. El controlador de la memoria quita esas páginas de su tabla de traducción y las marca como inválidas. Si alguien lee esos sectores, el controlador puede devolver ceros.
4. Más tarde, la recolección de basura elige un bloque con muchas páginas inválidas. Copia fuera las páginas válidas y aplica una tensión alta al bloque entero. Solo entonces salen los electrones.

Entre el paso 1 y el paso 4 los datos siguen físicamente en el chip. En un móvil actual están cifrados, y eso cambia qué significa «seguir ahí».

## Modelo mental

Piensa en un cuaderno escrito a bolígrafo con un índice al principio. No puedes borrar una línea. Solo puedes arrancar un pliego entero de muchas hojas. Para «corregir» una página, la copias corregida en una hoja en blanco al final y cambias el índice. Para «tirar» un capítulo, tachas su entrada en el índice.

Las hojas siguen escritas. De vez en cuando alguien recorre el cuaderno, copia las hojas que el índice aún usa a un pliego nuevo y arranca el pliego viejo.

La analogía se rompe en tres puntos. Primero, el índice del cuaderno lo llevas tú, mientras que en la flash hay dos índices: el del sistema de archivos, que maneja el sistema operativo, y el del controlador, que el sistema operativo no ve. Segundo, «arrancar el pliego» no destruye papel: devuelve el bloque a un estado reutilizable aplicando tensión. Y tercero, en un móvil moderno las hojas están escritas en clave. Lo que sobrevive en el cuaderno no se puede leer sin la llave correspondiente.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

La secuencia completa tiene cinco momentos, y solo el último toca las celdas.

1. **La aplicación pide borrar el archivo.** La galería llama al sistema operativo para eliminar el archivo de la foto.
2. **El sistema de archivos libera el espacio.** Quita la entrada del directorio y marca como libres los bloques lógicos que ocupaba la foto. Esos bloques lógicos son números de sector (LBA), no posiciones físicas en el chip. Wei y colaboradores midieron en 2011 que, tras un borrado normal del sistema de archivos, se podía recuperar del chip entre el 4,3 % y el 91,3 % del contenido de los archivos en discos SSD (FAST 2011, tabla 3).
3. **El sistema operativo avisa a la memoria.** Envía un comando de descarte sobre esos LBA: TRIM, UNMAP o Deallocate, según la interfaz. Puede enviarlo al momento o agrupado más tarde. La especificación NVMe lo define como **consultivo**: el controlador «puede» desasignar los rangos, y un controlador conforme puede no hacer nada (NVM Command Set 1.1, apartado 3.3.3).
4. **El controlador actualiza su mapa.** El controlador mantiene una tabla que traduce cada LBA a una página física. Al recibir el descarte, rompe esa relación y marca las páginas físicas como inválidas. En NVMe, una lectura posterior de un bloque desasignado devuelve todo 00h, todo FFh o uno de los dos sin especificar, según lo que declare el campo DLFEAT (apartado 3.3.3.2.1).
5. **La recolección de basura borra el bloque.** Cuando faltan bloques libres, o en momentos de poca actividad, el controlador elige un bloque con pocas páginas válidas. Copia esas páginas válidas a otro bloque y borra el bloque viejo entero (Cai et al., 2017, apartado 2.3). Ahí, y no antes, cambian las celdas de la foto.

### 2. Qué hace cada componente

**La celda.** Es un transistor con una zona aislada que atrapa electrones: una puerta flotante o, en muchos chips actuales, una capa de atrapamiento de carga. La carga atrapada desplaza la tensión umbral del transistor, es decir, la tensión a partir de la cual conduce. Leer es comprobar en qué ventana de tensión umbral cae la celda. Una celda SLC distingue 2 estados (1 bit), una MLC 4 (2 bits) y una TLC 8 (3 bits) (Cai et al., apartado 3.1). En celdas MLC de la generación de 15 a 19 nm, una celda guarda solo del orden de 100 electrones (apartado 4.4).

**Programar.** Programar es meter electrones en la zona aislada por efecto túnel, aplicando una tensión alta y positiva en la puerta de control. Se hace con pulsos que suben la tensión umbral poco a poco y verifican tras cada pulso. Este método solo puede **subir** la tensión umbral, nunca bajarla (Cai et al., apartado 3.4). Por eso una página escrita no se puede reescribir con otro valor.

**Borrar.** Borrar es sacar los electrones hacia el sustrato. La puerta de control se pone a masa y el sustrato a una tensión alta. Todas las celdas de un bloque comparten ese sustrato, así que el borrado afecta al bloque entero a la vez (apartado 3.4). La fuente revisada no da la cifra de esa tensión, de modo que aquí no se da.

**La página y el bloque.** Una página es la unidad de lectura y escritura. Mide de 8 a 16 kB. En MLC, un bloque tiene de 256 a 1024 páginas (Cai et al., apartados 2.1 y 3.2). Multiplicando, un bloque ronda de 2 a 16 MB. Esa cuenta es de este artículo, no de la fuente. Una foto de móvil de unos pocos megabytes ocupa, por tanto, cientos de páginas repartidas por uno o varios bloques que también contienen datos de otros archivos.

**El controlador y su FTL.** La capa de traducción de flash (FTL) es el firmware que hace que la flash parezca un disco normal. Traduce LBA a páginas físicas, escribe siempre en páginas nuevas, marca las viejas como inválidas, recolecta basura y reparte el desgaste entre bloques (Cai et al., apartado 2.3). El sistema operativo no ve nada de esto.

**El espacio oculto.** La memoria tiene más flash física de la que anuncia. En los SSD que estudiaron Wei y colaboradores, entre un 6 % y un 25 % más (apartado 2.2). Ese margen da sitio a la recolección de basura. También es espacio que el sistema operativo no puede leer ni sobrescribir directamente.

**El cifrado.** En los móviles actuales el contenido de los archivos llega a la flash ya cifrado. En Android, todos los aparatos que salen con Android 10 o posterior deben usar cifrado basado en archivos, y la configuración normal cifra el contenido con AES-256 en modo XTS (documentación de AOSP). En iOS, cada archivo se protege con una clave propia, que se guarda envuelta por una clave de clase en los metadatos del archivo (Apple Platform Security).

### 3. Parámetros que importan

| Parámetro | Valor | Escenario y fuente |
|---|---|---|
| Unidad de escritura | página de 8 a 16 kB | flash NAND contemporánea (Cai et al., 2.1) |
| Unidad de borrado | bloque de 256 a 1024 páginas | MLC (Cai et al., 3.2) |
| Estados por celda | 2 / 4 / 8 | SLC / MLC / TLC (Cai et al., 3.1) |
| Electrones por celda | ~100 | MLC de 15 a 19 nm (Cai et al., 4.4) |
| Ciclos de programación y borrado | ~3000 (MLC) · ~1000 (TLC) | por bloque, generación de 15 a 19 nm (Cai et al., 1 y 4) |
| Flash extra no visible | 6 % a 25 % | SSD estudiados en 2011 (Wei et al., 2.2) |
| Copias antiguas de un mismo archivo | hasta 16 | un SSD con 1000 archivos pequeños (Wei et al., figura 1) |
| Lectura tras Deallocate | 00h, FFh o sin especificar | campo DLFEAT de NVMe (NVM Command Set 1.1, 3.3.3.2.1) |
| Cifrado del contenido en Android | AES-256-XTS | configuración por defecto de AOSP |

El desgaste explica por qué el controlador no borra cada bloque en cuanto puede. Cada borrado consume uno de unos pocos miles de ciclos. Borrar un bloque para eliminar una sola página obligaría a copiar el resto del bloque y a gastar un ciclo (Wei et al., apartado 4.2). El controlador prefiere esperar a que un bloque acumule muchas páginas inválidas.

### 4. Diagrama o flujo

```text
Tú           Galería        Sistema de archivos      Controlador (FTL)           Celdas NAND
 │ «Eliminar»    │                   │                       │                         │
 │──────────────>│ borrar archivo    │                       │                         │
 │               │──────────────────>│ entrada fuera         │                         │
 │               │                   │ LBA 8000-8999 libres  │                         │
 │               │                   │ TRIM/UNMAP/Deallocate │                         │
 │               │                   │──────────────────────>│ mapa: LBA -> (nada)     │
 │               │                   │                       │ páginas = inválidas     │  carga intacta
 │               │                   │                       │                         │
 │               │          ... minutos, horas o días ...    │                         │
 │               │                   │                       │ recolección de basura:  │
 │               │                   │                       │ copia páginas válidas ─>│ bloque nuevo
 │               │                   │                       │ borra bloque viejo ────>│ electrones fuera
 │               │                   │                       │ bloque a la lista libre │
```

Un bloque antes y después de la recolección de basura:

```text
Bloque 412 (antes)                    Bloque 97 (libre)        Bloque 412 (después)
┌────────┬────────┬────────┐          ┌────────┬────────┐      ┌────────┬────────┬────────┐
│ foto ✗ │ foto ✗ │ chat ✓ │  copia   │ chat ✓ │ app ✓  │      │ borrada│ borrada│ borrada│
├────────┼────────┼────────┤  ─────>  ├────────┼────────┤      ├────────┼────────┼────────┤
│ foto ✗ │ app ✓  │ foto ✗ │          │ libre  │ libre  │      │ borrada│ borrada│ borrada│
└────────┴────────┴────────┘          └────────┴────────┘      └────────┴────────┴────────┘
✗ = página inválida (aún con carga)    ✓ = página válida        borrada = sin electrones
```

## Límites, excepciones y casos reales

**El aviso de descarte puede no llegar o no cumplirse.** Si el sistema de archivos no envía el descarte, el controlador cree que esas páginas siguen en uso. Las conservará y las copiará en cada recolección de basura. Si lo envía, el controlador puede ignorarlo, porque el comando es consultivo. En 2011, Wei y colaboradores concluyeron que TRIM no tenía un efecto fiable sobre la seguridad de los datos (nota 2). Los controladores han cambiado desde entonces. No hay un estudio equivalente y reciente para móviles revisado aquí, así que el comportamiento actual de cada modelo queda **no verificado**.

**Sobrescribir el archivo no sirve.** En un disco magnético, escribir encima de un archivo destruye el original. En flash, la escritura va a páginas nuevas y la vieja sigue ahí. En el mismo estudio, sobrescribir un solo archivo con distintos métodos dejó recuperable entre el 4 % y el 75 % de su contenido en los SSD (apartado 3.3). NIST SP 800-88 Rev. 2 (2025) lo dice de forma general: en memorias flash con celdas de reserva y reparto de desgaste, el usuario no puede sanear todos los datos anteriores sobrescribiendo (apartado 3.1.1).

**Ni siquiera sobrescribir el disco entero es infalible.** Dos pasadas completas bastaron en la mayoría de discos del estudio de 2011. En uno quedaba cerca del 1 % (1 GB) tras 20 pasadas (apartado 3.2.2). Tres discos informaron de que soportaban un comando de borrado integrado y no lo ejecutaron bien (resumen y apartado 3.2.1).

**Leer «ceros» no significa que las celdas estén vacías.** Tras un Deallocate, el controlador puede devolver ceros por su interfaz normal aunque las páginas sigan cargadas. Leer las celdas requiere desoldar el chip y saltarse el controlador. Es un trabajo de laboratorio, no algo que haga una aplicación.

**El cifrado cambia qué queda.** En un móvil con cifrado por archivos, lo que sobrevive en páginas inválidas es texto cifrado. Para leerlo haría falta la clave del archivo, que depende de claves ligadas al hardware y, en muchas clases de protección, al código del usuario. Apple no afirma en su guía que borrar un archivo destruya su clave. Lo que sí documenta es la otra vía: «Borrar contenidos y ajustes» elimina una clave guardada en una zona de la NAND que se puede borrar de forma directa y segura, y eso deja todos los archivos inaccesibles criptográficamente. En Android, con la configuración por defecto, el núcleo deriva una clave distinta para cada archivo a partir de la clave de clase. Con ciertas optimizaciones para cifrado en línea, se usa una clave por clase en lugar de una por archivo (documentación de AOSP). Por eso este artículo no afirma que una foto borrada sea irrecuperable en todos los casos: afirma que lo que queda está cifrado.

**Las copias que no ves.** La foto puede existir también en miniaturas, cachés o copias en la nube. Borrar el archivo en el móvil no toca esas copias. Están fuera del alcance de este artículo.

## Consecuencias prácticas

Para quien usa el móvil, «borrar» un archivo es una operación de contabilidad, no de destrucción. Es rápida precisamente porque no toca las celdas. El hueco se recupera de verdad más tarde, cuando el controlador recolecta basura.

Si lo que preocupa es que nadie pueda recuperar los datos al vender o regalar el aparato, la herramienta adecuada es el restablecimiento de fábrica de un móvil cifrado. Funciona destruyendo claves, que es lo que NIST SP 800-88 llama borrado criptográfico, una técnica de nivel *purge*. Las aplicaciones que prometen «borrado seguro» sobrescribiendo archivos chocan con la regla de no sobrescribir en el sitio.

Para quien diseña memorias o sistemas, el descarte importa también por rendimiento y duración. Un controlador que sabe qué páginas no valen nada no las copia en cada recolección de basura. Eso reduce la amplificación de escritura, es decir, cuántos bytes escribe la flash por cada byte que pide el sistema (Cai et al., apartado 2.4). Menos amplificación significa menos ciclos gastados y una memoria que dura más.

## Preguntas frecuentes

### ¿Es lo mismo TRIM que borrar las celdas?

No. TRIM, UNMAP y Deallocate son avisos de que unos sectores lógicos ya no contienen nada útil. El borrado físico es una operación interna del controlador sobre bloques enteros. Puede ocurrir mucho después del aviso, o no ocurrir hasta que el controlador necesite espacio. La propia especificación NVMe compara Deallocate con TRIM de ATA y con UNMAP de SCSI (apartado 3.3.3.2.1).

### ¿Por qué una foto borrada se puede recuperar a veces con un programa?

Porque, si el sistema de archivos aún no ha reutilizado esos sectores y el controlador no ha aplicado el descarte, las páginas siguen mapeadas y legibles. El programa lee lo que el sistema de archivos ya no referencia. Cuando el controlador ha aplicado el descarte, la lectura por la interfaz normal devuelve ceros o unos fijos.

### ¿Borrar desgasta la memoria?

Borrar un archivo, no: solo cambia metadatos. Lo que desgasta es el borrado físico de bloques, que forma parte de la recolección de basura. Cada bloque soporta unos pocos miles de ciclos de programación y borrado en las generaciones citadas: unos 3000 en MLC y unos 1000 en TLC (Cai et al.).

### ¿Sirve un imán para borrar una memoria flash?

No. La flash guarda carga eléctrica, no magnetización. En el estudio de 2011, desmagnetizar los chips con campos de 14 000 y 8000 gauss dejó los datos intactos en todos los casos (Wei et al., apartado 3.2.3).

## Fuentes

- [Cai, Y., Ghose, S., Haratsch, E. F., Luo, Y., Mutlu, O. — Error Characterization, Mitigation, and Recovery in Flash-Memory-Based Solid-State Drives. Proceedings of the IEEE, 105(9), septiembre de 2017 (versión arXiv:1706.08642)](https://arxiv.org/pdf/1706.08642). Estados por celda (apartado 3.1), organización en páginas y bloques (2.1 y 3.2), programación por pulsos y borrado por bloque con sustrato común (3.4), FTL y recolección de basura (2.3), amplificación de escritura (2.4), ciclos de vida (1 y 4) y electrones por celda (4.4).
- [Wei, M., Grupp, L. M., Spada, F. E., Swanson, S. — Reliably Erasing Data From Flash-Based Solid State Drives. USENIX FAST, 2011](https://static.usenix.org/event/fast11/tech/full_papers/Wei.pdf). Escritura fuera de sitio y restos digitales (apartado 2.2), flash extra del 6 % al 25 % y hasta 16 copias antiguas (2.2 y figura 1), recuperación tras borrado y sobrescritura de archivos (3.3 y tabla 3), sobrescritura completa (3.2.2), comandos integrados (3.2.1), desmagnetización (3.2.3) y TRIM (nota 2).
- [NVM Express — NVM Command Set Specification, revisión 1.1 (agosto de 2024)](https://nvmexpress.org/wp-content/uploads/NVM-Express-NVM-Command-Set-Specification-Revision-1.1-2024.08.05-Ratified.pdf). Dataset Management como comando consultivo y atributo Deallocate (apartado 3.3.3 y figura 45), lectura de bloques desasignados y campo DLFEAT (3.3.3.2.1 y figura 114) y equivalencia con TRIM y UNMAP.
- [NIST SP 800-88 Rev. 2 — Guidelines for Media Sanitization (septiembre de 2025)](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-88r2.pdf). Niveles *clear*, *purge* y *destroy* (apartado 3.1), límites de la sobrescritura en flash con celdas de reserva (3.1.1) y borrado criptográfico (3.2).
- [Apple — Apple Platform Security: Data Protection in Apple devices](https://support.apple.com/guide/security/sece8608431d/web). Clave por archivo envuelta por una clave de clase en los metadatos, clave del sistema de archivos y borrado de contenidos y ajustes.
- [Apple — Apple Platform Security: Effaceable Storage](https://support.apple.com/guide/security/aside/sec0183122de/1/web/1). Zona de la NAND para claves que se puede direccionar y borrar de forma segura.
- [Android Open Source Project — File-based encryption](https://source.android.com/docs/security/features/encryption/file-based). Obligatoriedad desde Android 10, AES-256-XTS para el contenido y derivación de claves con HKDF-SHA512.
- [Android Open Source Project — Hardware-wrapped keys](https://source.android.com/docs/security/features/encryption/hw-wrapped-keys). Clave por archivo o por clase según la opción de cifrado en línea.
