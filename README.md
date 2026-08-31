<div align="center">

# HowThingsTalk

**Cómo funcionan de verdad los aparatos que llevas encima**

*How everyday devices actually work, underneath*

[Índice general](docs/README.md) · [Hoja de ruta](docs/Roadmap.md) · [Guía de estilo](docs/_meta/StyleGuide.md) · [Glosario](docs/99-Reference/01-Glossary.md)

**Español** · [English](#-english)

</div>

---

## ¿Qué es esto?

Este repositorio pretende ser una biblioteca de explicaciones sobre **cómo funcionan por dentro los aparatos que usamos todos los días**: teléfonos, portátiles, auriculares, relojes, altavoces, tarjetas, bombillas, coches.

Cada una de las entradas responde a **una pregunta concreta y directa** con el detalle necesario para entenderla de verdad, sin quedarse en meras metáforas. El proyecto persigue un doble compromiso deliberado:

> El pacto con el principiante: Abrir la puerta al conocimiento de forma intuitiva, guiando la lectura sin apoyarse en tecnicismos ni artificios.

> El pacto con el experto: Garantizar una solidez técnica impecable, libre de imprecisiones o concesiones al rigor.

Para lograrlo, cada artículo se estructura en **cuatro capas de profundidad diferentes**:
0. **Pregunta, Encuadre y Profundidad:** Cada artículo parte de la pregunta exacta que intenta resolver, declarando explícitamente la profundidad técnica y los conocimientos previos necesarios para seguirlo sin perderse.
1. **La Respuesta Corta:** Una única frase contundente que sintetiza la respuesta entera, seguida de un resumen de 60 segundos para quien solo busca la idea general rápida.
2. **El Modelo Mental:** Una analogía del mundo cotidiano para asentar la intuición, acompañada del punto exacto donde la metáfora se rompe y da paso a la realidad tecnológica.
3. **El Mecanismo Técnico:** El desglose sin concesiones con diagramas de flujo, tramas de datos, protocolos, espectro de radio y los números físicos reales.

Quien quiera bajarse a mitad del camino, puede hacerlo en cualquiera de las capas habiendo entendido el núcleo del problema.

## ¿Qué no es?

- **No es un Blog de Noticias.** Sin lanzamientos ni novedades. Lo que se explica aquí seguirá siendo válido dentro de cinco años.
- **No es un Sitio de Reseñas.** No se comparan modelos comerciales ni se recomienda qué comprar.
- **No es un Repositorio de Tutoriales.** No se explica *cómo configurar* algo, sino *por qué funciona*.
- **No es un Manual de Ingeniería Ofensiva.** Se describen mecanismos de funcionamiento y garantías de seguridad; no se detallan procedimientos para vulnerar sistemas de terceros. La frontera está definida en la [guía de estilo](templates/StyleGuide.md) y se respeta sin excepciones.

## Preguntas que se responden aquí

- ¿Cómo sabe mi móvil donde está aunque tenga el GPS apagado y no esté conectado a ninguna red WiFi?
- ¿Qué ocurre exactamente en el espectro radioeléctrico entre mi teléfono y la antena cuando hago una llamada?
- ¿Por qué protocolo y ruta viaja un archivo al usar AirDrop, y qué lo hace tan rápido comparado con otros métodos?
- ¿Qué ocurre a nivel físico en las celdas de memoria flash cuando "borro" una foto?
- ¿Por qué ocurre al activar el micrófono en unos auriculares Bluetooth la calidad del audio cae en picado y suena diferente?
- ¿Por qué la batería del movil cae drásticamente al 10% cuando estamos a temperaturas bajo cero?
- ¿Por qué una bombilla inteligente necesita enviar datos a un servidor a 2.000 km para encenderse desde el interruptor de la misma habitación?
- ¿Qué señales y trames emite exactamente un teléfono en reposo sobre la mesa?

## ¿Por dónde empiezo?

Si es tu primera visita, el punto de entrada recomendado es [Roadmap](docs/Roadmap.md).
Este artículo aborda la pregunta, el modelo mental, la geometríca y las tramas de radio en un solo texto, estableciendo el nivel de profundidad técnica al que aspira toda la biblioteca.
A partir de ahí, puedes consultar el [indice general](docs/Index.md), que organiza la lectura en diferentes itinerarios temáticos.

## Estructura

```
HowThingsTalk/
├── docs
|    ├── EN
|    |   ├── 00_Foundations
|    |   ├── 01_WiFiNetworks
|    |   ├── 02_CellularNetworks
|    |   ├── 03_ShortRangeRadio
|    |   ├── 04_DataTransfer
|    |   ├── 05_Positioning
|    |   ├── 06_Storage
|    |   ├── 07_AudioAndPeripherals
|    |   ├── 08_PowerAndBatteries
|    |   ├── 09_IOT
|    |   ├── 10_SecurityAndCryptography
|    |   ├── 11_PrivacyAndFootprint
|    |   └── 99_Reference
|    ├── ES
|    |   ├── 00_Fundamentos
|    |   ├── 01_RedesWiFi
|    |   ├── 02_RedesMoviles
|    |   ├── 03_CortoAlcance
|    |   ├── 04_TransferenciaDeDatos
|    |   ├── 05_Geolocalizacion
|    |   ├── 06_Almacenamiento
|    |   ├── 07_AudioYPerifericos
|    |   ├── 08_EnergiaYBaterias
|    |   ├── 09_IOT
|    |   ├── 10_SeguridadYCriptografia
|    |   ├── 11_PrivacidadYHuella
|    |   └── 99_Referencia
|    ├── Index.md
|    └── RoadMap.md
├── guides
|    ├── ContributionGuide.md
|    ├── NamingConvention.md
|    ├── StyleGuide.md
|    └── TaxonomyAndGlosary.md
├── templates
|    └── ArticuleTemplate.md
└── Readme.md
```
Las secciones siguen un orden ascendente: desde los fundamentos en la base, pasando por los protocolos, hasta culminar en sus consecuencias. Cada artículo especifica sus prerrequisitos en la cabecera, si algún concepto se resiste, el contexto necesario está a un solo clic.
Tanto la estructura de carpetas, como la de archivos y artículosutiliza el formato CamelCase, además, se promoverá siempre que los artículos y documentos de la "Biblioteca" se encuentren en formato bilingüe. Para conocer más sobre la motivación y las reglas detalladas, puedes consultar el siguiente [documento](guides/NamingConvention.md).

## Estado

| Sección | Artículos Previstos | Artículos Escritos |
|---|---|---|
| 00 - Fundamentos | X | Y |
| 01 - Redes WiFi | X | Y |
| 02 - Redes Móviles | X | Y |
| 03 - Corto Alcance | X | Y |
| 04 - Transferencia de Datos | X | Y |
| 05 - Geolocalización | X | Y |
| 06 - Almacenamiento | X | Y |
| 07 - Audio y Periféricos | X | Y |
| 08 - Energía y Baterias | X | Y |
| 09 - IOT (Internet of Things) | X | Y |
| 10 - Seguridad y Criptografía | X | Y |
| 11 - Privacidad y Huella | X | Y |

## ¿Cómo se escribe aquí?

La normativa completa se encuentra en el siguiente [documento](/guides/NamingConvention.md). Que en esencia son:

- **Concisión:** Priorizaremos frases breves. Si hay más de tres comas, probablmente podamos dividir la idea en dos frases.
- **Contexto Cuantitativo:** Todo dato exige su unidad, su escenario y contexto exacto.
- **Analogías Transparentes:** Máximo una por concepto, mostrando siempre sus límites teóricos.
- **Jerarquía de Fuentes:** Acudiremos a los estándares primarios (IEEE, 3GPP, Bluetooth SIG, RFC), pasa por los fabricantes y recurre a la literatura científica. Quedan excluidas las fuentas de segunda mano.
- **Honestidad Técnica:** Ante la ausencia de fuente comprobable, marcaremos el dato como "no verificado". La transparencia siempre supera a la especulación.

## Contribuir

El proyecto está abierto a correcciones, datos actualizados y nuevo contenido.

- **Propuestas y Errores:** Abre un issue detallando la sugerencia o el hallazgo.
- **Requisito de Verificación:** Toda corrección debe incluir la fuente que la fundamenta. Sin una referencia contrastable, la modificación no podrá ser integrada.

Para crear, sugerir o escribir un artículo deberemos seguir los siguientes pasos:
1. Comprueba en la [hoja de ruta](docs/RoadMap.md) que no esté en proceso o realizado.
2. Copia la [plantilla](templates/ArticuleTemplate.md) y pegala en la carpeta que toque.
3. Ahora renombraremos el archivo según la [convención](guides/NamingConvention.md).
4. Revisa la [guia de estilo](guides/StyleGuide.md) antes de empezar, esto te ahorrará un tiempo valioso en revisiones.
5. Redacta el artículo dedicándole todo el tiempo que necesites, debes saber que el rigor prima sobre la prisa.
6. Enlaza en el artículo que creaste en el [índice](docs/Index.md). Así podremos encontrarlo disponible al buscar documentos.
7. Finalmente, envía tu *pull request*.

## Licencia

| Qué | Licencia |
|---|---|
| Texto, Imágenes y Diagramas | [CC BY-SA 4.0](LICENSE) |
| Fragmentos de Código y Scripts | MIT |
| Estándares y Papers Citados | De sus autores. Aquí solo se enlazan, no se redistribuyen. |

Puedes copiar, adaptar y republicar el contenido citando la fuente y manteniendo la misma licencia.

---
## English

<details>
<summary><b>Read the full English version</b></summary>

<br>

> **Note on language.** Articles are written in Spanish. Titles, folder names, section
> indexes and this document are bilingual, so you can navigate the repository and judge
> whether a machine translation is worth it. There is no plan to translate the articles
> themselves for now — see [Roadmap](docs/Roadmap.md), *Discarded*.

---

### What is this?

This repository aims to be a comprehensive reference library explaining how the everyday devices we use actually work under the hood: smartphones, laptops, earbuds, smartwatches, speakers, smart cards, lightbulbs, and connected cars.

Every article tackles one specific, direct question with the exact depth required for genuine understanding, going far beyond surface-level metaphors. This project is built on a deliberate dual commitment:

> The Beginner's Pact: Opening the door to technical knowledge intuitively, guiding the reader step-by-step without relying on obscure jargon or artificial fluff.

> The Expert's Pact: Guaranteeing uncompromising technical precision, completely free of inaccuracies or concessions to rigor.

To achieve this, every entry is structured into four distinct layers of depth:
0. **Question, Framing, and Depth:** Each article begins with the exact problem it aims to solve, explicitly stating its technical scope and the prerequisite knowledge required to follow along without getting lost.
1. **The Short Answer:** A single punchy sentence summarizing the entire core convept, followed by a 60-second summary for anyone seeking a quick high-level overview.
2. **The Mental Model:** An everyday real-world analogy to establish intuition, accompanied by the precise boundary where the metaphor breaks down and yields to technological reality.
3. **The Technical Inner Workings:** An uncompromising deep dive featuring flowcharts, data frames, protocol stack breakdowns, RF spectrum behavior, and physical constants.

Readers can exit at any layer along the way while still walking away with a solid grasp of the core issue.

## What is this not?

- **Not a News Blog.** No product launches, leaks, or breaking news. The mechanics explained here will remain valid five years from now.
- **Not a Review Site.** No commercial side-by-side comparisons or purchasing guides.
- **Not a How-To Repository.** We don't cover how to configure a system, but why it works under the hood.
- **Not an Offensive Security Manual.** We break down operational mechanisms and security guarantees, we do not publish exploitation walkthroughs against third-party systems. This boundatary is strictly defined in our [style guide](templates/StyleGuide.md) and enforced without exception.

## Questions Answered Here

- How does my phone know its precise location even with GPS turned off and no active Wi-Fi connections?
- What exactly transpires across the radio spectrum between my handset and the cell tower during a call?
- Which protocols and network routes carry a file via AirDrop, and why is it so much faster than alternative methods?
- What happens at the physical level inside flash memory cells when I "delete" a photo?
- Why does audio quality plummet and sound completely different the moment I activate the microphone on Bluetooth earbuds?
- Why does phone battery capacity collapse so abruptly toward 10% when exposed to sub-zero temperatures?
- Why must a smart lightbulb route data to a server 2,000 km away just to toggle on from a switch in the same room?
- What raw signals and frames does an idle smartphone continuously broadcast while sitting on a table?

## Where do I Start?

If this is your first visit, the recommended entry point is the [Roadmap](docs/Roadmap.md).
This baseline article covers the problem framing, mental model, geometric principles, and RF frames in a single unified text, establishing the target technical depth for the entire library. From there, feel free to navigate the [index](docs/Index.md), which organizes articles into curated learning paths.

## Structure

```
HowThingsTalk/
├── docs
|    ├── EN
|    |   ├── 00_Foundations
|    |   ├── 01_WiFiNetworks
|    |   ├── 02_CellularNetworks
|    |   ├── 03_ShortRangeRadio
|    |   ├── 04_DataTransfer
|    |   ├── 05_Positioning
|    |   ├── 06_Storage
|    |   ├── 07_AudioAndPeripherals
|    |   ├── 08_PowerAndBatteries
|    |   ├── 09_IOT
|    |   ├── 10_SecurityAndCryptography
|    |   ├── 11_PrivacyAndFootprint
|    |   └── 99_Reference
|    ├── ES
|    |   ├── 00_Fundamentos
|    |   ├── 01_RedesWiFi
|    |   ├── 02_RedesMoviles
|    |   ├── 03_CortoAlcance
|    |   ├── 04_TransferenciaDeDatos
|    |   ├── 05_Geolocalizacion
|    |   ├── 06_Almacenamiento
|    |   ├── 07_AudioYPerifericos
|    |   ├── 08_EnergiaYBaterias
|    |   ├── 09_IOT
|    |   ├── 10_SeguridadYCriptografia
|    |   ├── 11_PrivacidadYHuella
|    |   └── 99_Referencia
|    ├── Index.md
|    └── RoadMap.md
├── guides
|    ├── ContributionGuide.md
|    ├── NamingConvention.md
|    ├── StyleGuide.md
|    └── TaxonomyAndGlosary.md
├── templates
|    └── ArticuleTemplate.md
└── Readme.md
```
The sections follow a bottom-up architecture: moving from core physical foundations at the base, ascending through protocols, and culminating in real-world implications. Every article declares its required background knowledge in its header, if a concept feels daunting, the necessary prerequisite context is always a single click away.
All directory trees, filenames, and article paths strictly enforce CamelCase. Furthermore, all articles and reference documents across the library are maintained in a bilingual format. To dive deeper into the design philosophy and full ruleset, check out our [document](guides/NamingConvention.md).

## sTATE

| Section | Planned Articles | Writed Articles |
|---|---|---|
| 00 - Foundations | X | Y |
| 01 - WiFi Networks | X | Y |
| 02 - Cellular Networks | X | Y |
| 03 - Short-Range Radio | X | Y |
| 04 - Data Transfer | X | Y |
| 05 - Positioning | X | Y |
| 06 - Storage | X | Y |
| 07 - Audio and Peripherals | X | Y |
| 08 - Power and Batteries | X | Y |
| 09 - IOT (Internet of Things) | X | Y |
| 10 - Security and Cryptography | X | Y |
| 11 - Privacy and Footprint | X | Y |

## How We Write

Our full editorial standards live in the following [document](/guides/NamingConvention.md). In short:

- **Conciseness:** We prioritize short, crisp sentences. If a sentence packs more than three commas, it should probably be split into two.
- **Quantitative Context:** Every data point demands its unit of measurement, test scenario, and precise environment.
- **Transparent Analogies:** Maximum one metaphor per concept, explicitly marking where its physical validity ends.
- **Primary Sources First:** We cite primary standards bodies first (IEEE, 3GPP, Bluetooth SIG, RFCs), followed by vendor engineering documentation and peer-reviewed literature. Secondary blogs and rewrites are strictly excluded.
- **Technical Honesty:** If an original primary source cannot be verified, we label the data point as "unverified". Transparent gaps are infinitely better than speculation.

## Contributing

This project welcomes community contributors, error reports, data updates, and new articles.

- **Suggestions and Issues:** Open an issue detailing your proposed topic or bug report.
- **Verification Rule:** Every fix or correction must cite its supporting primary source. Unverified modifications will not be merged.

To propose, draft, or submit a new article, follow this workflow:
1. Check the [roadmap](docs/RoadMap.md) to ensure is not already claimed or in progress.
2. Copy [template](templates/ArticuleTemplate.md) into the target topic folder.
3. Rename the file following our [naming rules](guides/NamingConvention.md).
4. Review the [style guide](guides/StyleGuide.md) beforehand, it saves substantial review cycle time.
5. Take all the time you need to write the piece; engineering accuracy always takes priority over speed.
6. Link your newly created article inside the main [index](docs/Index.md) so readers can find it.
7. Submit your *pull request*.

## License

| Asset | License |
|---|---|
| Prose, Images & Diagrams | [CC BY-SA 4.0](LICENSE) |
| Code Snippets & Scripts | MIT |
| Cited Standards & Papers | Held by their respective copyright holders. Linked for reference, not redistributed. |

You are free to copy, adapt, and redistribute this material provided you attribute the source and distribute contributions under the same license.

[🇪🇸 Volver al español](#howthingstalk)

</details>

---