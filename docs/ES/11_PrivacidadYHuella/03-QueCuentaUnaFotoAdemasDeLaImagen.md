---
title: "¿Qué cuenta una foto del móvil, además de lo que se ve en ella?"
section: "11_PrivacidadYHuella"
language: "es"
status: "published"
order: 3
depth: "intermediate"
summary: "Cada foto del móvil lleva dentro un bloque de datos Exif que nadie ve al mirarla. Ahí suelen ir la fecha y hora exactas con su zona horaria, la marca y el modelo del aparato y, si la cámara tiene permiso de ubicación, las coordenadas del lugar con precisión de calle. Ese bloque viaja con el archivo a donde se envíe, salvo que alguien lo quite por el camino."
prerequisites:
  - "Cómo calcula un receptor GPS su posición (artículo 05·01)"
  - "Cómo se localiza el móvil sin GPS (artículo 05·02)"
translation: "../../EN/11_PrivacyAndFootprint/03-WhatAPhotoRevealsBeyondTheImage.md"
---

# ¿Qué cuenta una foto del móvil, además de lo que se ve en ella?

## Pregunta y encuadre

Una foto parece solo una imagen. Pero el archivo que guarda el móvil contiene más cosas que píxeles. La pregunta es concreta: ¿qué información viaja escondida dentro del archivo de una foto, dónde está guardada y quién puede leerla cuando la compartes?

El artículo explica el formato Exif: cómo se mete en un archivo JPEG o HEIC, cómo se codifican la fecha y las coordenadas y qué precisión tienen. Explica también qué hacen iOS y Android para no repartir la ubicación sin querer. No es una guía paso a paso para limpiar fotos ni compara aplicaciones. Tampoco trata el reconocimiento de lugares a partir del contenido de la imagen, que es otro problema.

La profundidad es intermedia. Ayuda saber cómo obtiene el móvil su posición, con GPS ([artículo 05·01](../05_Geolocalizacion/01-ComoCalculaElGPSTuPosicion.md)) o sin él ([artículo 05·02](../05_Geolocalizacion/02-ComoTeLocalizaElMovilSinGPS.md)). No hace falta saber programar: los bytes que aparecen se explican uno a uno.

## Respuesta corta

Una foto del móvil lleva pegada una ficha técnica legible por cualquiera que tenga el archivo: cuándo se hizo, con qué aparato y, si la cámara tiene permiso de ubicación, dónde, con una resolución de unos 30 cm.

### Resumen de 60 segundos

Al disparar, la cámara no guarda solo la imagen. Añade al principio del archivo un bloque con formato **Exif**, una norma de las asociaciones japonesas CIPA y JEITA que usan casi todas las cámaras y móviles.

Ese bloque es una lista de campos con número y valor. Los más habituales son:

1. **Cuándo.** Fecha y hora del disparo, al segundo, y la diferencia con la hora universal.
2. **Con qué.** Marca, modelo, versión del software y, a veces, datos del objetivo.
3. **Cómo.** Exposición, apertura, sensibilidad, flash y orientación.
4. **Dónde.** Latitud, longitud, altitud e incluso hacia dónde apuntaba el móvil. Solo si la cámara tiene permiso de ubicación.
5. **Una miniatura.** Una copia pequeña de la imagen.

Nada de esto se ve al mirar la foto. Pero viaja dentro del archivo: si envías el archivo original, el receptor puede leerlo. Algunos servicios lo quitan al subir la foto y otros no. iOS permite compartir sin ubicación, y Android oculta las coordenadas a las aplicaciones que no tienen un permiso específico.

## Modelo mental

Piensa en una foto revelada en papel con una etiqueta adhesiva pegada por detrás. La etiqueta la escribió la cámara: fecha, hora, modelo y la dirección donde estabas. Quien mira la foto de frente no la ve. Quien recibe la foto y le da la vuelta, sí.

Compartir el archivo original es como entregar la foto con la etiqueta puesta. Algunos intermediarios la despegan antes de pasarla. Otros la dejan.

La analogía se rompe en dos puntos. Primero, la etiqueta de papel está a la vista de quien la busca; el bloque Exif solo lo lee un programa que entiende el formato, aunque eso incluye a cualquier visor de fotos de ordenador. Segundo, una etiqueta de papel solo tiene lo que se escribió una vez. El archivo puede llevar copias del mismo dato en varios sitios, como la miniatura o bloques propios del fabricante. Borrar un campo no garantiza haber borrado todos.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

Cuando pulsas el disparador, la secuencia es esta:

1. **El sensor captura y el procesador comprime.** El resultado es una imagen JPEG o HEIC.
2. **La cámara reúne los datos del momento.** Lee el reloj del sistema y la zona horaria, los ajustes de exposición y el modelo del aparato. Si la aplicación de cámara tiene permiso de ubicación, pide también la última posición al sistema. Según Apple, esa posición puede salir del GPS, de redes WiFi, de la red móvil o de Bluetooth.
3. **La cámara escribe el bloque Exif.** Organiza esos datos como una pequeña estructura de tablas, heredada del formato de imagen TIFF.
4. **El bloque se coloca al principio del archivo.** En JPEG va justo detrás del marcador de inicio de imagen. En HEIC va dentro de la estructura de cajas de metadatos del formato.
5. **El archivo se guarda en la galería.** A partir de ahí, cualquier copia exacta del archivo lleva el bloque consigo.

### 2. Qué hace cada componente

**El contenedor JPEG.** Un archivo JPEG es una sucesión de segmentos, cada uno precedido por un marcador de dos bytes. La norma JPEG (ITU-T T.81) reserva los marcadores `FFE0` a `FFEF` para datos de aplicación, y cada segmento lleva un campo de longitud de 16 bits. Exif usa el segmento **APP1** (`FFE1`), que debe ir inmediatamente después del inicio de imagen (`FFD8`). Como la longitud cabe en 16 bits, el bloque no puede pasar de 64 KB, y la norma Exif lo recuerda expresamente (CIPA DC-008, apartado 4.7.2).

**La cabecera.** Dentro de APP1, los primeros seis bytes son `Exif` seguido de dos bytes a cero. Luego viene una cabecera TIFF de 8 bytes. Dice si los números van con el byte más significativo primero (`MM`) o al revés (`II`), y dónde empieza la primera tabla.

**Las tablas (IFD).** Cada tabla, llamada IFD, es una lista de entradas de 12 bytes. Cada entrada dice: número de campo (2 bytes), tipo de dato (2 bytes), cuántos valores tiene (4 bytes) y el valor o, si no cabe, dónde está (4 bytes). La norma define varias tablas encadenadas:

- **IFD 0:** datos básicos de la imagen principal, como marca (campo 271), modelo (272) y software (305).
- **IFD Exif:** a la que apunta el campo 34665. Contiene la fecha de disparo (`DateTimeOriginal`, 36867), su desfase horario (`OffsetTimeOriginal`, 36881), la exposición y otros datos. Aquí están también los campos opcionales de número de serie del cuerpo (42033), modelo del objetivo (42036), nombre del propietario (42032) e identificador único de imagen (42016).
- **IFD GPS:** a la que apunta el campo 34853. Contiene latitud, longitud, altitud, hora GPS, dirección de la imagen y error horizontal estimado.
- **IFD 1:** la miniatura, una pequeña imagen JPEG completa dentro del bloque.

**La nota del fabricante.** El campo `MakerNote` (37500) es un bloque de bytes cuyo contenido decide cada fabricante. La norma no lo describe. Puede contener datos internos de la cámara que ningún visor estándar muestra.

**El permiso de ubicación.** La cámara solo escribe la IFD GPS si el sistema le da la posición. En iPhone, Apple indica que la ubicación se guarda únicamente si la cámara tiene acceso a los servicios de localización.

### 3. Parámetros que importan

- **Formato de las coordenadas.** La latitud (campo 2 de la IFD GPS) y la longitud (campo 4) se guardan como tres números racionales: grados, minutos y segundos. Cada racional son dos enteros de 4 bytes, numerador y denominador, así que cada coordenada ocupa 24 bytes. Dos campos aparte (1 y 3) dicen si es norte o sur y este u oeste.
- **Resolución del dato.** Un segundo de latitud equivale a unos 30,9 m sobre el terreno. Si el denominador de los segundos es 100, el valor se guarda en pasos de 0,01″, es decir, unos 0,31 m. En longitud el paso se encoge con el coseno de la latitud: a 40° de latitud, un segundo son unos 23,7 m. El denominador lo elige quien escribe el archivo; 100 es un ejemplo habitual, no una regla de la norma.
- **Precisión real.** La resolución del número no es la precisión de la medida. GPS.gov indica que los receptores GPS de una frecuencia de buena calidad logran 2,168 m o mejor en horizontal el 95 % del tiempo, según datos de la FAA. También advierte que la precisión real depende de la atmósfera, de los obstáculos al cielo y de la calidad del receptor. En interiores, con posición por WiFi o por red móvil, el error suele ser mayor. Exif tiene un campo para declararlo, `GPSHPositioningError` (campo 31), aunque es opcional.
- **Fecha y hora.** `DateTimeOriginal` es texto con formato `AAAA:MM:DD HH:MM:SS`, al segundo. `OffsetTimeOriginal` añade el desfase respecto a la hora universal, por ejemplo `+01:00`. Con los dos se sabe el instante exacto y, de paso, la zona horaria en la que estaba el aparato.
- **Tamaño.** Todo el bloque APP1 cabe en 64 KB, miniatura incluida. Los datos extra que no caben pueden ir en segmentos APP2 adicionales (CIPA DC-008, apartado 4.7.3).
- **Versión de la norma.** Exif 3.0 (CIPA DC-008-2023) se publicó en 2023. Añade un tipo de dato de texto UTF-8 para escribir en cualquier idioma, nuevos campos como autor y editor, y una guía sobre qué campos actualizar o conservar al editar una imagen.

### 4. Diagrama o flujo

Estos son los primeros bytes reales de un JPEG con Exif generado para este artículo, con datos de ejemplo:

```text
FF D8                  Inicio de imagen (SOI)
FF E1  00 D6           Segmento APP1, 214 bytes de longitud
45 78 69 66 00 00      "Exif" + 00 00
4D 4D  00 2A           Cabecera TIFF: orden "MM" y número mágico 42
00 00 00 08            La IFD 0 empieza 8 bytes después de la cabecera
```

Y así se encadenan las tablas dentro del bloque:

```text
APP1 "Exif"
 └─ Cabecera TIFF
     └─ IFD 0 ── Make = "Marca"   Model = "Modelo"   Software = "..."
         ├─ campo 34665 ──► IFD Exif
         │                   DateTimeOriginal   = "2026:11:14 10:30:00"
         │                   OffsetTimeOriginal = "+01:00"
         │                   MakerNote          = [bytes del fabricante]
         ├─ campo 34853 ──► IFD GPS
         │                   GPSLatitudeRef  = "N"
         │                   GPSLatitude     = 40/1, 25/1, 1234/100  → 40° 25′ 12,34″
         │                   GPSLongitudeRef = "W"
         │                   GPSLongitude    = 3/1, 42/1, 5678/100   → 3° 42′ 56,78″
         └─ siguiente ───► IFD 1: miniatura JPEG
FF DB ... FF DA ...        A partir de aquí, la imagen comprimida
```

Las coordenadas del ejemplo, `40/1, 25/1, 1234/100`, equivalen a 40,420094° de latitud norte.

## Límites, excepciones y casos reales

**HEIC guarda lo mismo con otro envoltorio.** Los iPhone pueden capturar en HEIF desde iOS 11, según la Biblioteca del Congreso de EE. UU. HEIF (ISO/IEC 23008-12) no tiene segmentos APP1: organiza el archivo en cajas tipadas y lleva los metadatos, Exif incluido, dentro de su estructura de metadatos. El contenido Exif es el mismo; cambia dónde está.

**La ubicación del archivo no es la de la galería.** Apple indica que se puede ocultar o cambiar la ubicación de una foto en la aplicación Fotos. Pero las copias que ya enviaste son archivos independientes: modificar la foto en tu galería no cambia esas copias.

**Las coordenadas no son una prueba.** Exif es texto y números que cualquier programa puede reescribir. La norma no incluye firma ni protección frente a cambios. Una foto con coordenadas indica dónde dijo estar el aparato que la escribió, no dónde estaba con certeza.

**Quitar la ubicación no siempre quita todo.** La fecha con su desfase horario, el modelo del aparato y la nota del fabricante siguen ahí si solo se quita la IFD GPS. La miniatura puede conservar una versión anterior de la imagen si el programa que recortó la foto no la regeneró. Exif 3.0 añade precisamente una guía sobre qué campos actualizar al editar, porque no todos los programas lo hacían.

**Cada servicio hace una cosa distinta.** Hay servicios que eliminan los metadatos al subir una foto y otros que conservan el archivo tal cual, sobre todo cuando se envía «como documento» o «como archivo». No hay una norma que obligue a ninguna de las dos cosas, y el comportamiento de cada servicio puede cambiar con el tiempo (no verificado servicio por servicio).

**Android protege a nivel de aplicación, no de archivo.** La documentación de Android indica que, con el almacenamiento aislado, el sistema oculta la ubicación de las fotos a las aplicaciones por defecto. Una aplicación para Android 10 (nivel de API 29) o superior que quiera los metadatos sin recortar tiene que declarar el permiso `ACCESS_MEDIA_LOCATION` y pedirlo al usuario en ejecución. Además debe solicitar los bytes originales con `MediaStore.setRequireOriginal()`. Eso protege frente a aplicaciones instaladas en el propio móvil, pero no cambia el archivo: si se copia por cable o se envía tal cual, las coordenadas siguen dentro.

## Consecuencias prácticas

Para quien comparte fotos, lo importante es saber que el archivo habla más que la imagen. Una foto hecha en casa con la ubicación activada lleva la dirección de casa con precisión de portal. Una serie de fotos con fecha, hora y coordenadas dibuja una rutina. Apple lo resume en su guía de seguridad personal: quien recibe fotos con ubicación puede acceder a esos datos y saber dónde se tomaron.

Los controles existen en tres niveles. En la cámara, se puede negar el permiso de ubicación y las fotos nuevas no la llevarán. Al compartir, iOS permite desactivar la ubicación en «Opciones» antes de enviar, y lo indica sobre el botón antes de enviar. En las aplicaciones, Android exige un permiso aparte para leer la ubicación de las fotos.

Para quien diseña aplicaciones o servicios, el punto clave es decidir de forma explícita qué pasa con los metadatos al recibir una imagen. Conservarlos todos reparte datos que el usuario quizá no sabía que enviaba. Quitarlos todos pierde información útil, como la orientación o la fecha. Exif 3.0 da una guía para editar con criterio en lugar de copiar o borrar a ciegas.

## Preguntas frecuentes

### ¿Si desactivo la ubicación en el móvil, la foto ya no dice nada?

Dice menos, pero no nada. Sin la IFD GPS siguen la fecha y la hora al segundo, el desfase horario, la marca, el modelo, la versión de software y los ajustes de la cámara. La imagen en sí también puede revelar el lugar, pero eso ya no depende de Exif.

### ¿Es lo mismo la ubicación de la foto que el historial de ubicaciones del móvil?

No. El historial lo guarda el sistema o un servicio en la nube, separado de las fotos. La ubicación Exif va dentro de cada archivo y viaja con él. Puedes tener una sin la otra.

### ¿Una red social que quita los metadatos los borra también de sus servidores?

No se puede saber desde fuera. Que el archivo que ven los demás no lleve Exif no dice nada sobre lo que el servicio guardó al recibir el original. Depende de su política, no de la norma Exif.

### ¿Por qué hay coordenadas aunque estuviera en un sótano sin GPS?

Porque la cámara no exige GPS. Usa la posición que le dé el sistema, que puede venir de WiFi o de la red móvil, como explica el [artículo 05·02](../05_Geolocalizacion/02-ComoTeLocalizaElMovilSinGPS.md). También puede ser una posición reciente, guardada antes de entrar. Por eso las coordenadas de una foto pueden estar desplazadas decenas de metros.

## Fuentes

- [CIPA DC-008-2016 / JEITA CP-3451D — Exchangeable image file format for digital still cameras: Exif Version 2.31 (CIPA y JEITA, julio de 2016)](https://people.cs.umass.edu/~liberato/courses/2017-spring-compsci365/files/DC-008-Translation-2016-E.pdf). Copia alojada por la Universidad de Massachusetts. Segmento APP1 tras el SOI, identificador `Exif`, límite de 64 KB y cabecera TIFF (apartado 4.7.2), segmentos APP2 (4.7.3) y números de campo de las IFD 0, Exif, GPS e IFD 1 (tablas 17 a 21).
- [CIPA DC-008-Translation-2023 — Exif Version 3.0 (CIPA, 2023), página oficial de descarga](https://www.cipa.jp/std/documents/download_e.html?DC-008-Translation-2023-E). Versión vigente de la norma.
- [IPTC — Exif 3.0 released, featuring UTF-8 support (1 de junio de 2023)](https://iptc.org/news/exif-3-0-released-featuring-utf-8-support/). Resumen de los cambios de Exif 3.0: tipo UTF-8, nuevos campos y guía de actualización de campos al editar (anexo H).
- [ITU-T T.81 / ISO/IEC 10918-1 — Digital compression and coding of continuous-tone still images (ITU, 1992)](https://www.w3.org/Graphics/JPEG/itu-t81.pdf). Estructura de marcadores JPEG, marcadores APPn `FFE0`–`FFEF` y campo de longitud de 16 bits.
- [Library of Congress — High Efficiency Image File Format (HEIF)](https://www.loc.gov/preservation/digital/formats/fdd/fdd000525.shtml). HEIF como formato de cajas basado en ISO/IEC 23008-12, transporte de metadatos y captura en iPhone desde iOS 11.
- [Apple — Guía de seguridad personal: gestionar los metadatos de ubicación en Fotos](https://support.apple.com/guide/personal-safety/manage-location-metadata-in-photos-ips0d7a5df82/web). Origen de la ubicación, compartir sin ubicación, ocultarla en Fotos y negar el permiso a la cámara.
- [Android Developers — Access media files from shared storage](https://developer.android.com/training/data-storage/shared/media). Ocultación de la ubicación con almacenamiento aislado, permiso `ACCESS_MEDIA_LOCATION` y `MediaStore.setRequireOriginal()`.
- [GPS.gov — GPS Accuracy](https://www.gps.gov/systems/gps/performance/accuracy/). 2,168 m o mejor en horizontal el 95 % del tiempo con receptores de una frecuencia de buena calidad, y factores que degradan la precisión.
