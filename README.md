<div align="center">

# HowThingsTalk

**Cómo funcionan de verdad los aparatos que llevas encima**

*How everyday devices actually work, underneath*

[Índice general](docs/Index.md) · [Hoja de ruta](docs/RoadMap.md) · [Guía de estilo](guides/StyleGuide.md) · [Glosario](guides/TaxonomyAndGlossary.md) · [Cambios](CHANGELOG.md)

**Español** · [English](#english)

</div>

---

## ¿Qué es esto?

Este repositorio pretende ser una biblioteca de explicaciones sobre **cómo funcionan por dentro los aparatos que usamos todos los días**: teléfonos, portátiles, auriculares, relojes, altavoces, tarjetas, bombillas y coches.

Cada entrada responde a **una pregunta concreta y directa** con el detalle necesario para entenderla de verdad, sin quedarse en meras metáforas. El proyecto persigue un doble compromiso deliberado:

> **El pacto con el principiante:** abrir la puerta al conocimiento de forma intuitiva, guiando la lectura sin apoyarse en tecnicismos ni artificios.

> **El pacto con el experto:** garantizar una solidez técnica impecable, libre de imprecisiones y de concesiones al rigor.

Para lograrlo, cada artículo se estructura en **cuatro capas de profundidad**:

0. **Pregunta, encuadre y profundidad:** cada artículo parte de la pregunta exacta que intenta resolver y declara la profundidad técnica y los conocimientos previos necesarios para seguirlo sin perderse.
1. **La respuesta corta:** una única frase contundente que sintetiza la respuesta entera, seguida de un resumen de 60 segundos para quien solo busca la idea general.
2. **El modelo mental:** una analogía del mundo cotidiano para asentar la intuición, acompañada del punto exacto donde la metáfora se rompe y da paso a la realidad tecnológica.
3. **El mecanismo técnico:** el desglose sin concesiones, con diagramas de flujo, tramas de datos, protocolos, espectro de radio y los números físicos reales.

Quien quiera bajarse a mitad del camino puede hacerlo en cualquiera de las capas habiendo entendido el núcleo del problema.

Cada artículo se escribe en **español y en inglés**: la versión española vive en `docs/ES/` y la inglesa en `docs/EN/`, con el mismo número de orden en ambas carpetas.

## ¿Qué no es?

- **No es un blog de noticias.** Sin lanzamientos ni novedades. Lo que se explica aquí seguirá siendo válido dentro de cinco años.
- **No es un sitio de reseñas.** No se comparan modelos comerciales ni se recomienda qué comprar.
- **No es un repositorio de tutoriales.** No se explica *cómo configurar* algo, sino *por qué funciona*.
- **No es un manual de ingeniería ofensiva.** Se describen mecanismos de funcionamiento y garantías de seguridad; no se detallan procedimientos para vulnerar sistemas de terceros. La frontera está definida en la [guía de estilo](guides/StyleGuide.md) y se respeta sin excepciones.

## El tipo de preguntas que responde esta biblioteca

- ¿Cómo sabe mi móvil dónde está aunque tenga el GPS apagado y no esté conectado a ninguna red WiFi?
- ¿Qué ocurre exactamente en el espectro radioeléctrico entre mi teléfono y la antena cuando hago una llamada?
- ¿Por qué protocolo y ruta viaja un archivo al usar AirDrop, y qué lo hace tan rápido comparado con otros métodos?
- ¿Qué ocurre a nivel físico en las celdas de memoria flash cuando «borro» una foto?
- ¿Por qué, al activar el micrófono en unos auriculares Bluetooth, la calidad del audio cae en picado y suena diferente?
- ¿Por qué la batería del móvil cae drásticamente al 10 % cuando estamos bajo cero?
- ¿Por qué una bombilla inteligente necesita enviar datos a un servidor a 2000 km para encenderse desde el interruptor de la misma habitación?
- ¿Qué señales y tramas emite exactamente un teléfono en reposo sobre la mesa?

## ¿Por dónde empiezo?

La biblioteca ya contiene diez artículos publicados en dos idiomas. El punto de entrada es el [índice general](docs/Index.md), que organiza las secciones. La [hoja de ruta](docs/RoadMap.md) recoge lo publicado y lo que está en curso.

## Estructura

```text
HowThingsTalk/
├── docs/
│   ├── ES/
│   │   ├── 00_Fundamentos/
│   │   ├── 01_RedesWiFi/
│   │   ├── 02_RedesMoviles/
│   │   ├── 03_CortoAlcance/
│   │   ├── 04_TransferenciaDeDatos/
│   │   ├── 05_Geolocalizacion/
│   │   ├── 06_Almacenamiento/
│   │   ├── 07_AudioYPerifericos/
│   │   ├── 08_EnergiaYBaterias/
│   │   ├── 09_IOT/
│   │   ├── 10_SeguridadYCriptografia/
│   │   ├── 11_PrivacidadYHuella/
│   │   └── 99_Referencia/
│   ├── EN/
│   │   ├── 00_Foundations/
│   │   ├── 01_WiFiNetworks/
│   │   ├── 02_CellularNetworks/
│   │   ├── 03_ShortRangeRadio/
│   │   ├── 04_DataTransfer/
│   │   ├── 05_Positioning/
│   │   ├── 06_Storage/
│   │   ├── 07_AudioAndPeripherals/
│   │   ├── 08_PowerAndBatteries/
│   │   ├── 09_IOT/
│   │   ├── 10_SecurityAndCryptography/
│   │   ├── 11_PrivacyAndFootprint/
│   │   └── 99_Reference/
│   ├── Index.md
│   └── RoadMap.md
├── guides/
│   ├── ContributionGuide.md
│   ├── NamingConvention.md
│   ├── StyleGuide.md
│   └── TaxonomyAndGlossary.md
├── templates/
│   └── ArticleTemplate.md
├── CHANGELOG.md
├── LICENSE.md
└── README.md
```

Las secciones siguen un orden ascendente: desde los fundamentos en la base, pasando por los protocolos, hasta culminar en sus consecuencias. Cada artículo especifica sus prerrequisitos en la cabecera; si algún concepto se resiste, el contexto necesario está a un solo clic.

Las carpetas y los archivos usan PascalCase, sin espacios, tildes ni eñes, y cada artículo existe en las dos lenguas. La motivación y las reglas detalladas están en la [convención de nombres](guides/NamingConvention.md).

## Estado

<sub>Se recalcula automáticamente a partir de los archivos reales. El detalle está en la [hoja de ruta](docs/RoadMap.md).</sub>

| Sección | Artículos publicados |
|---|---|
| 00 · Fundamentos | 3 |
| 01 · Redes WiFi | 0 |
| 02 · Redes Móviles | 0 |
| 03 · Corto Alcance | 0 |
| 04 · Transferencia de Datos | 2 |
| 05 · Geolocalización | 1 |
| 06 · Almacenamiento | 0 |
| 07 · Audio y Periféricos | 2 |
| 08 · Energía y Baterías | 0 |
| 09 · IoT y Smart Home | 0 |
| 10 · Seguridad y Criptografía | 1 |
| 11 · Privacidad y Huella | 1 |
| 99 · Referencia | 0 |
| **Total** | **10** |

## ¿Cómo se escribe aquí?

La normativa completa está en la [guía de estilo](guides/StyleGuide.md). En esencia:

- **Concisión:** priorizamos frases breves. Si hay más de tres comas, probablemente podamos dividir la idea en dos frases.
- **Contexto cuantitativo:** todo dato exige su unidad, su escenario y su contexto exacto.
- **Analogías transparentes:** máximo una por concepto, mostrando siempre sus límites.
- **Jerarquía de fuentes:** acudimos primero a los estándares primarios (IEEE, 3GPP, Bluetooth SIG, RFC), después a la documentación de los fabricantes y a la literatura científica revisada. Quedan excluidas las fuentes de segunda mano.
- **Honestidad técnica:** ante la ausencia de fuente comprobable, marcamos el dato como «no verificado». La transparencia siempre supera a la especulación.

## Contribuir

El proyecto está abierto a correcciones, datos actualizados y contenido nuevo. El proceso completo está en la [guía de contribución](guides/ContributionGuide.md).

- **Propuestas y errores:** abre un issue detallando la sugerencia o el hallazgo.
- **Requisito de verificación:** toda corrección debe incluir la fuente que la fundamenta. Sin una referencia contrastable, la modificación no podrá integrarse.

Para escribir un artículo:

1. Comprueba en la [hoja de ruta](docs/RoadMap.md) que no esté en curso ni publicado.
2. Copia la [plantilla](templates/ArticleTemplate.md): la parte española en `docs/ES/<sección>/` y la inglesa en `docs/EN/<sección>/`.
3. Nombra los dos archivos según la [convención de nombres](guides/NamingConvention.md), con el mismo número de orden.
4. Revisa la [guía de estilo](guides/StyleGuide.md) antes de empezar; te ahorrará tiempo en revisiones.
5. Redacta el artículo dedicándole todo el tiempo que necesites: el rigor prima sobre la prisa.
6. Enlaza las dos versiones en el [índice](docs/Index.md) para que se puedan encontrar.
7. Envía tu *pull request*.

## Licencia

| Qué | Licencia |
|---|---|
| Texto, imágenes y diagramas | [CC BY-SA 4.0](LICENSE.md) |
| Fragmentos de código y scripts | [MIT](LICENSE.md) |
| Estándares y artículos citados | De sus autores. Aquí solo se enlazan, no se redistribuyen. |

Puedes copiar, adaptar y republicar el contenido citando la fuente y manteniendo la misma licencia.

---

## English

<details>
<summary><b>Read the full English version</b></summary>

<br>

### What is this?

This repository aims to be a reference library explaining how the everyday devices we use actually work under the hood: smartphones, laptops, earbuds, smartwatches, speakers, smart cards, lightbulbs and connected cars.

Every article tackles one specific, direct question with the exact depth required for genuine understanding, going beyond surface-level metaphors. The project is built on a deliberate dual commitment:

> **The beginner's pact:** opening the door to technical knowledge intuitively, guiding the reader step by step without relying on obscure jargon or artificial fluff.

> **The expert's pact:** guaranteeing uncompromising technical precision, free of inaccuracies or concessions to rigor.

To achieve this, every entry is structured into **four layers of depth**:

0. **Question, framing and depth:** each article begins with the exact problem it aims to solve, explicitly stating its technical scope and the prerequisite knowledge required to follow along.
1. **The short answer:** a single punchy sentence summarizing the core concept, followed by a 60-second summary for anyone seeking a quick overview.
2. **The mental model:** an everyday analogy to establish intuition, together with the precise point where the metaphor breaks down and yields to technological reality.
3. **The technical mechanism:** an uncompromising deep dive with flowcharts, data frames, protocol stacks, RF spectrum behavior and real physical figures.

Readers can stop at any layer and still walk away with a solid grasp of the core issue.

Every article is written in **Spanish and English**: the Spanish version lives in `docs/ES/` and the English one in `docs/EN/`, with the same order number in both folders.

### What is this not?

- **Not a news blog.** No product launches, leaks or breaking news. The mechanics explained here will remain valid five years from now.
- **Not a review site.** No commercial comparisons or buying guides.
- **Not a how-to repository.** We don't cover how to configure a system, but why it works.
- **Not an offensive security manual.** We explain operating mechanisms and security guarantees; we do not publish exploitation walkthroughs against third-party systems. This boundary is defined in the [style guide](guides/StyleGuide.md) and enforced without exception.

### The kind of questions this library answers

- How does my phone know where it is even with GPS turned off and no WiFi connection?
- What exactly happens across the radio spectrum between my handset and the cell tower during a call?
- Which protocols and routes carry a file via AirDrop, and why is it so much faster than other methods?
- What happens at the physical level inside flash memory cells when I "delete" a photo?
- Why does audio quality plummet the moment I turn on the microphone on Bluetooth earbuds?
- Why does a phone battery collapse abruptly around 10% in sub-zero temperatures?
- Why must a smart lightbulb send data to a server 2,000 km away just to turn on from a switch in the same room?
- What signals and frames does an idle smartphone broadcast while sitting on a table?

### Where do I start?

The library now contains ten articles published in two languages. The entry point is the [master index](docs/Index.md), which organizes the sections. The [roadmap](docs/RoadMap.md) lists what has been published and what is in progress.

### Structure

```text
HowThingsTalk/
├── docs/
│   ├── ES/
│   │   ├── 00_Fundamentos/
│   │   ├── 01_RedesWiFi/
│   │   ├── 02_RedesMoviles/
│   │   ├── 03_CortoAlcance/
│   │   ├── 04_TransferenciaDeDatos/
│   │   ├── 05_Geolocalizacion/
│   │   ├── 06_Almacenamiento/
│   │   ├── 07_AudioYPerifericos/
│   │   ├── 08_EnergiaYBaterias/
│   │   ├── 09_IOT/
│   │   ├── 10_SeguridadYCriptografia/
│   │   ├── 11_PrivacidadYHuella/
│   │   └── 99_Referencia/
│   ├── EN/
│   │   ├── 00_Foundations/
│   │   ├── 01_WiFiNetworks/
│   │   ├── 02_CellularNetworks/
│   │   ├── 03_ShortRangeRadio/
│   │   ├── 04_DataTransfer/
│   │   ├── 05_Positioning/
│   │   ├── 06_Storage/
│   │   ├── 07_AudioAndPeripherals/
│   │   ├── 08_PowerAndBatteries/
│   │   ├── 09_IOT/
│   │   ├── 10_SecurityAndCryptography/
│   │   ├── 11_PrivacyAndFootprint/
│   │   └── 99_Reference/
│   ├── Index.md
│   └── RoadMap.md
├── guides/
│   ├── ContributionGuide.md
│   ├── NamingConvention.md
│   ├── StyleGuide.md
│   └── TaxonomyAndGlossary.md
├── templates/
│   └── ArticleTemplate.md
├── CHANGELOG.md
├── LICENSE.md
└── README.md
```

Sections follow a bottom-up order: from the physical foundations at the base, through protocols, up to their real-world consequences. Every article declares its prerequisites in its header; if a concept feels daunting, the necessary context is one click away.

Folders and files use PascalCase, with no spaces or accented characters, and every article exists in both languages. The rationale and full ruleset are in the [naming convention](guides/NamingConvention.md).

### Status

<sub>Recalculated automatically from the actual files. Details in the [roadmap](docs/RoadMap.md).</sub>

| Section | Published articles |
|---|---|
| 00 · Foundations | 3 |
| 01 · WiFi Networks | 0 |
| 02 · Cellular Networks | 0 |
| 03 · Short-Range Radio | 0 |
| 04 · Data Transfer | 2 |
| 05 · Positioning | 1 |
| 06 · Storage | 0 |
| 07 · Audio and Peripherals | 2 |
| 08 · Power and Batteries | 0 |
| 09 · IoT and Smart Home | 0 |
| 10 · Security and Cryptography | 1 |
| 11 · Privacy and Footprint | 1 |
| 99 · Reference | 0 |
| **Total** | **10** |

### How we write

The full editorial rules are in the [style guide](guides/StyleGuide.md). In short:

- **Conciseness:** we prioritize short sentences. If a sentence has more than three commas, it should probably be split in two.
- **Quantitative context:** every data point carries its unit, scenario and exact context.
- **Transparent analogies:** at most one per concept, always marking where it stops being valid.
- **Source hierarchy:** primary standards first (IEEE, 3GPP, Bluetooth SIG, RFCs), then vendor documentation and peer-reviewed literature. Second-hand sources are excluded.
- **Technical honesty:** if a data point cannot be verified against a source, it is labeled "unverified". Transparency always beats speculation.

### Contributing

The project welcomes corrections, updated data and new content. The full process is in the [contribution guide](guides/ContributionGuide.md).

- **Suggestions and issues:** open an issue describing the proposal or the finding.
- **Verification rule:** every correction must cite its supporting source. Changes without a verifiable reference will not be merged.

To write an article:

1. Check the [roadmap](docs/RoadMap.md) to make sure it is not in progress or already published.
2. Copy the [template](templates/ArticleTemplate.md): the Spanish part into `docs/ES/<section>/` and the English part into `docs/EN/<section>/`.
3. Name both files following the [naming convention](guides/NamingConvention.md), with the same order number.
4. Read the [style guide](guides/StyleGuide.md) first; it saves review time.
5. Take all the time you need: accuracy always takes priority over speed.
6. Link both versions from the [index](docs/Index.md) so readers can find them.
7. Submit your *pull request*.

### License

| Asset | License |
|---|---|
| Prose, images and diagrams | [CC BY-SA 4.0](LICENSE.md) |
| Code snippets and scripts | [MIT](LICENSE.md) |
| Cited standards and papers | Held by their authors. Linked for reference, not redistributed. |

You are free to copy, adapt and republish this material provided you credit the source and keep the same license.

[Volver al español · Back to Spanish](#howthingstalk)

</details>

---
