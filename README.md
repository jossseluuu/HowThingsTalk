<div align="center">

# HowThingsTalk

**Cómo funcionan de verdad los aparatos que llevas encima**

*How everyday devices actually work, underneath*

![Estado](https://img.shields.io/badge/estado-en%20construcci%C3%B3n-orange)
![Artículos](https://img.shields.io/badge/art%C3%ADculos-1%20de%20206-blue)
![Idioma](https://img.shields.io/badge/idioma-espa%C3%B1ol-red)
![Licencia](https://img.shields.io/badge/texto-CC%20BY--SA%204.0-green)
![Código](https://img.shields.io/badge/c%C3%B3digo-MIT-lightgrey)

[Índice general](docs/README.md) · [Hoja de ruta](docs/Roadmap.md) · [Guía de estilo](docs/_meta/StyleGuide.md) · [Glosario](docs/99-Reference/01-Glossary.md)

**🇪🇸 Español** · [🇬🇧 English](#-english)

</div>

---

## Qué es esto

Una biblioteca de explicaciones sobre **cómo funcionan por dentro los aparatos que
usamos todos los días**: teléfonos, portátiles, auriculares, relojes, altavoces,
tarjetas, bombillas, coches.

Cada entrada responde a **una pregunta concreta** con el detalle suficiente para
entenderla de verdad, no para quedarse en la metáfora. El listón es doble y deliberado:

> Alguien con curiosidad y sin formación técnica debe llegar al final.
> Alguien del gremio no debe encontrar nada falso.

Eso se consigue escribiendo por capas. Cada artículo empieza por una frase que resume
la respuesta entera, sigue con un párrafo para quien tenga prisa, y solo entonces baja
al mecanismo completo y a los números. Quien quiera bajarse a mitad de camino, puede.

## Qué no es

- **No es un blog de noticias.** Nada de lanzamientos ni novedades. Lo que se explica
  aquí sigue siendo cierto dentro de cinco años.
- **No es un sitio de reseñas.** No se compara ningún producto ni se recomienda nada.
- **No es un repositorio de tutoriales.** No se explica *cómo instalar*, se explica
  *por qué funciona*.
- **No es un manual ofensivo.** Se describen mecanismos y garantías; no se dan
  procedimientos para aplicárselos a terceros. La línea está escrita en la
  [guía de estilo](docs/_meta/StyleGuide.md) y se respeta sin excepciones.

## Preguntas que se responden aquí

- ¿Cómo sabe mi móvil dónde está aunque tenga el GPS apagado y no esté conectado a
  ninguna WiFi? → [ya escrito](docs/05-Positioning/01-WiFiPositioning.md)
- ¿Qué pasa exactamente entre el teléfono y la antena cuando llamo?
- ¿Por dónde viaja un archivo cuando lo mando por AirDrop, y por qué es tan rápido?
- ¿Qué hace la memoria del móvil cuando "borro" una foto?
- ¿Por qué al activar el micrófono los auriculares suenan a radio vieja?
- ¿Por qué el móvil se apaga al 10% cuando hace frío?
- ¿Por qué una bombilla necesita un servidor a 2000 km para encenderse?
- ¿Qué va emitiendo un teléfono parado encima de la mesa?

## Por dónde empezar

Si es tu primera visita, lee **[Cómo te localiza el WiFi sin que te conectes a
nada](docs/05-Positioning/01-WiFiPositioning.md)**. Toca radio, protocolos, geometría y
privacidad de una sentada, y marca el nivel de detalle al que aspira todo lo demás.

Después, el [índice general](docs/README.md) propone tres itinerarios temáticos según
lo que te interese: cómo te localizan, cómo viaja un archivo por el aire, o qué pasa
dentro del teléfono.

## Estructura

```
docs/
├── README.md                    Índice general y rutas de lectura
├── Roadmap.md                   Qué se escribe ahora, qué está en cola, qué se descartó
├── _meta/                       Plantilla, guía de estilo, convención de nombres
├── _assets/                     Diagramas e imágenes compartidas
│
├── 00-Foundations/              Ondas, espectro, decibelios, propagación, modulación
├── 01-WiFi/                     Tramas 802.11, beacons, asociación, WPA3, mesh
├── 02-CellularNetworks/         Celdas, registro, handover, IMSI/IMEI, SIM, VoLTE
├── 03-ShortRange/               Bluetooth, BLE, GATT, NFC, RFID, UWB
├── 04-DataTransfer/             AirDrop, WiFi Direct, compresión, cifrado, sincronización
├── 05-Positioning/              WPS, GNSS, fingerprinting, fusión de sensores, E911
├── 06-Storage/                  NAND, wear leveling, borrado, cifrado en reposo
├── 07-AudioAndPeripherals/      Códecs, latencia, ANC, audio espacial, periféricos
├── 08-PowerAndBatteries/        Litio, ciclos, carga rápida, Qi, disipación
├── 09-IoTAndSmartHome/          Zigbee, Thread, Matter, LoRaWAN, MQTT
├── 10-SecurityAndCryptography/  Claves, TLS, Secure Enclave, biometría, postcuántica
├── 11-PrivacyAndFootprint/      Emisiones, aleatorización de MAC, metadatos, permisos
└── 99-Reference/                Glosario, siglas, bandas, unidades, estándares
```

Las secciones están ordenadas **de abajo arriba**: la física primero, los protocolos
después, y las consecuencias al final. Cada artículo declara en su cabecera qué da por
sabido, así que si algo se atraganta, el prerrequisito está a un clic.

Las carpetas y los nombres de archivo van en inglés y CamelCase (`05-Positioning/03-GNSS.md`);
los títulos son bilingües dentro del documento. El motivo y las reglas completas están
en [`_meta/NamingConvention.md`](docs/_meta/NamingConvention.md).

## Estado

| Sección | Artículos previstos | Escritos |
|---|---|---|
| 00 · Foundations | 15 | 0 |
| 01 · WiFi | 17 | 0 |
| 02 · CellularNetworks | 19 | 0 |
| 03 · ShortRange | 16 | 0 |
| 04 · DataTransfer | 15 | 0 |
| 05 · Positioning | 18 | **1** |
| 06 · Storage | 16 | 0 |
| 07 · AudioAndPeripherals | 16 | 0 |
| 08 · PowerAndBatteries | 15 | 0 |
| 09 · IoTAndSmartHome | 17 | 0 |
| 10 · SecurityAndCryptography | 16 | 0 |
| 11 · PrivacyAndFootprint | 16 | 0 |
| 99 · Reference | 10 | 1 (borrador) |

Cada artículo lleva un campo `status` en su cabecera:

| Estado | Significa |
|---|---|
| `planned` | Solo el esqueleto. Título reservado, contenido pendiente. |
| `draft` | Publicable pero incompleto. |
| `reviewed` | Releído con las fuentes delante. |
| `done` | No se le ocurre nada más a quien lo escribió. |

**Nada se queda sin publicar por no estar terminado.** Un borrador visible se corrige;
uno en tu carpeta local, no.

## Cómo se escribe aquí

Las reglas completas están en [`_meta/StyleGuide.md`](docs/_meta/StyleGuide.md). El
resumen:

- **Frases cortas.** Si una frase tiene tres comas, probablemente son dos frases.
- **Todo dato lleva unidad y contexto.** Nunca "mucho alcance": *"hasta ~30 m en
  interior, ~100 m en exterior con línea de visión directa"*.
- **Una analogía por concepto, con la costura visible.** Después de cada analogía, se
  dice en qué deja de valer.
- **Fuentes primarias.** Estándares primero (IEEE, 3GPP, Bluetooth SIG, RFC),
  documentación del fabricante después, papers luego. Nunca blogs que citan blogs.
- **Si no encuentras la fuente, escribe "no verificado".** Un hueco reconocido vale más
  que un dato inventado.

## Contribuir

Se aceptan correcciones, datos actualizados y artículos nuevos.

**Para proponer un tema o señalar un error**, abre un *issue*. Si es una corrección,
incluye la fuente que la respalda: sin fuente no se puede verificar, y un dato sin
verificar no entra.

**Para escribir un artículo:**

1. Comprueba en la [hoja de ruta](docs/Roadmap.md) que no esté ya asignado.
2. Copia [`_meta/ArticleTemplate.md`](docs/_meta/ArticleTemplate.md) a la carpeta que toque.
3. Nombra el archivo según la [convención](docs/_meta/NamingConvention.md).
4. Lee la [guía de estilo](docs/_meta/StyleGuide.md) una vez. Ahorra mucho retrabajo.
5. Añade el enlace al `README.md` de su sección y actualiza la hoja de ruta.
6. Abre un *pull request*.

No hace falta que el artículo esté perfecto. Un `draft` honesto es mejor que un archivo
vacío esperando inspiración.

## PDFs y publicación web

Los PDFs **no se escriben a mano ni se versionan**: se generan desde el Markdown. Así
nunca hay dos versiones del mismo texto diciendo cosas distintas.

```bash
./scripts/generar-pdf.sh docs/05-Positioning/01-WiFiPositioning.md
```

Salen a `pdf/`, que está ignorado por Git. Para publicar una tanda, se adjuntan a un
*release*.

El repositorio también se puede navegar como web con buscador:

```bash
pip install mkdocs-material
mkdocs serve
```

## Licencia

| Qué | Licencia |
|---|---|
| Texto, imágenes y diagramas | [CC BY-SA 4.0](LICENSE) |
| Fragmentos de código y scripts | MIT |
| Estándares y papers citados | De sus autores. Aquí solo se enlazan, no se redistribuyen. |

Puedes copiar, adaptar y republicar el contenido citando la fuente y manteniendo la
misma licencia.

---

## 🇬🇧 English

<details>
<summary><b>Read the full English version</b></summary>

<br>

> **Note on language.** Articles are written in Spanish. Titles, folder names, section
> indexes and this document are bilingual, so you can navigate the repository and judge
> whether a machine translation is worth it. There is no plan to translate the articles
> themselves for now — see [Roadmap](docs/Roadmap.md), *Discarded*.

### What this is

A library of explanations about **how the devices we use every day actually work
underneath**: phones, laptops, headphones, watches, speakers, cards, bulbs, cars.

Each entry answers **one concrete question** in enough depth to genuinely understand it,
rather than stopping at the metaphor. The bar is deliberately double:

> A curious reader with no technical background should reach the end.
> A professional should find nothing false in it.

That is achieved by writing in layers. Every article opens with a single sentence
containing the whole answer, follows with a paragraph for readers in a hurry, and only
then descends into the full mechanism and the hard numbers. Anyone who wants to get off
halfway can.

### What this is not

- **Not a news blog.** No launches, no announcements. What is explained here will still
  be true in five years.
- **Not a review site.** No products are compared or recommended.
- **Not a tutorial collection.** It does not explain *how to install*; it explains
  *why it works*.
- **Not an offensive manual.** Mechanisms and guarantees are described; procedures for
  applying them to other people are not. The boundary is written into the
  [style guide](docs/_meta/StyleGuide.md) and is honoured without exception.

### Questions answered here

- How does my phone know where it is with GPS off and no WiFi connection?
  → [already written, in Spanish](docs/05-Positioning/01-WiFiPositioning.md)
- What exactly happens between the phone and the tower during a call?
- Where does a file actually travel during an AirDrop, and why is it so fast?
- What does the phone's memory do when I "delete" a photo?
- Why does audio drop to AM-radio quality the moment the microphone turns on?
- Why does the phone shut down at 10% battery in the cold?
- Why does a light bulb need a server 2,000 km away to turn on?
- What is an idle phone on a table broadcasting?

### Where to start

First visit: **[How WiFi locates you without connecting to
anything](docs/05-Positioning/01-WiFiPositioning.md)**. It covers radio, protocols,
geometry and privacy in one sitting, and sets the level of detail everything else aims
for.

After that, the [master index](docs/README.md) offers three reading paths: how you get
located, how a file travels through the air, and what happens inside the phone.

### Structure

```
docs/
├── README.md                    Master index and reading paths
├── Roadmap.md                   In progress, queued, researching, discarded
├── _meta/                       Template, style guide, naming convention
├── _assets/                     Shared diagrams and images
│
├── 00-Foundations/              Waves, spectrum, decibels, propagation, modulation
├── 01-WiFi/                     802.11 frames, beacons, association, WPA3, mesh
├── 02-CellularNetworks/         Cells, attach, handover, IMSI/IMEI, SIM, VoLTE
├── 03-ShortRange/               Bluetooth, BLE, GATT, NFC, RFID, UWB
├── 04-DataTransfer/             AirDrop, WiFi Direct, compression, encryption, sync
├── 05-Positioning/              WPS, GNSS, fingerprinting, sensor fusion, E911
├── 06-Storage/                  NAND, wear levelling, deletion, encryption at rest
├── 07-AudioAndPeripherals/      Codecs, latency, ANC, spatial audio, input devices
├── 08-PowerAndBatteries/        Lithium, cycles, fast charging, Qi, thermals
├── 09-IoTAndSmartHome/          Zigbee, Thread, Matter, LoRaWAN, MQTT
├── 10-SecurityAndCryptography/  Keys, TLS, Secure Enclave, biometrics, post-quantum
├── 11-PrivacyAndFootprint/      Emissions, MAC randomization, metadata, permissions
└── 99-Reference/                Glossary, acronyms, bands, units, standards
```

Sections run **bottom-up**: physics first, protocols next, consequences last. Every
article declares its prerequisites in the front matter, so if something is hard going,
the missing piece is one click away.

Folders and filenames are English CamelCase (`05-Positioning/03-GNSS.md`); titles are
bilingual inside the document. Full rules in
[`_meta/NamingConvention.md`](docs/_meta/NamingConvention.md).

### Status

| Section | Planned | Written |
|---|---|---|
| 00 · Foundations | 15 | 0 |
| 01 · WiFi | 17 | 0 |
| 02 · CellularNetworks | 19 | 0 |
| 03 · ShortRange | 16 | 0 |
| 04 · DataTransfer | 15 | 0 |
| 05 · Positioning | 18 | **1** |
| 06 · Storage | 16 | 0 |
| 07 · AudioAndPeripherals | 16 | 0 |
| 08 · PowerAndBatteries | 15 | 0 |
| 09 · IoTAndSmartHome | 17 | 0 |
| 10 · SecurityAndCryptography | 16 | 0 |
| 11 · PrivacyAndFootprint | 16 | 0 |
| 99 · Reference | 10 | 1 (draft) |

Every article carries a `status` field in its front matter:

| Status | Meaning |
|---|---|
| `planned` | Skeleton only. Title reserved, content pending. |
| `draft` | Publishable but incomplete. |
| `reviewed` | Re-read with the sources open. |
| `done` | Nothing left to add, in the writer's view. |

**Nothing is withheld for being unfinished.** A visible draft gets corrected; one
sitting in a local folder does not.

### How writing works here

Full rules in [`_meta/StyleGuide.md`](docs/_meta/StyleGuide.md). The short version:

- **Short sentences.** If a sentence has three commas, it is probably two sentences.
- **Every figure carries units and context.** Never "long range": *"up to ~30 m indoors,
  ~100 m outdoors with line of sight"*.
- **One analogy per concept, with the seam visible.** Every analogy is followed by where
  it stops holding.
- **Primary sources.** Standards first (IEEE, 3GPP, Bluetooth SIG, RFC), vendor
  documentation second, papers third. Never blogs quoting blogs.
- **If the source cannot be found, write "unverified".** An acknowledged gap beats an
  invented figure.

### Contributing

Corrections, updated figures and new articles are welcome. **Articles are written in
Spanish**; issues and pull requests in English are perfectly fine.

**To propose a topic or report an error**, open an issue. For corrections, include the
source: without one it cannot be verified, and unverified figures do not go in.

**To write an article:**

1. Check the [roadmap](docs/Roadmap.md) to make sure it is not already taken.
2. Copy [`_meta/ArticleTemplate.md`](docs/_meta/ArticleTemplate.md) into the right folder.
3. Name the file per the [convention](docs/_meta/NamingConvention.md).
4. Read the [style guide](docs/_meta/StyleGuide.md) once. It saves a lot of rework.
5. Add the link to the section `README.md` and update the roadmap.
6. Open a pull request.

The article does not need to be perfect. An honest `draft` beats an empty file waiting
for inspiration.

### PDFs and web publishing

PDFs are **never hand-written or committed**: they are generated from the Markdown, so
there are never two versions of the same text disagreeing with each other.

```bash
./scripts/generar-pdf.sh docs/05-Positioning/01-WiFiPositioning.md
```

Output goes to `pdf/`, which Git ignores. To publish a batch, attach them to a release.

The repository can also be browsed as a searchable website:

```bash
pip install mkdocs-material
mkdocs serve
```

### Licence

| What | Licence |
|---|---|
| Text, images and diagrams | [CC BY-SA 4.0](LICENSE) |
| Code snippets and scripts | MIT |
| Cited standards and papers | Their authors'. Linked here, never redistributed. |

You may copy, adapt and republish the content as long as you credit the source and keep
the same licence.

<br>

[🇪🇸 Volver al español](#howthingstalk)

</details>

---

<div align="center">
<sub>Si algo aquí ha quedado obsoleto o es falso, <b>abre un issue</b>. Es la forma más útil de contribuir.<br>
<i>If anything here is outdated or wrong, <b>open an issue</b>. It is the most useful way to contribute.</i></sub>
</div>