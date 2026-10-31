# Convención de nombres / Naming convention

<div align="center">

**Español** · [English](#english)

</div>

---

## Objetivo / Goal

El repositorio necesita que los archivos, carpetas y artículos se comporten como un sistema ordenado: fácil de navegar, fácil de localizar y fácil de mantener en dos idiomas.

The repository needs files, folders, and articles to behave like a well-ordered system: easy to navigate, easy to locate, and easy to maintain in two languages.

---

## Principios básicos / Core principles

1. Cada sección tiene una ruta clara y estable.
2. Los nombres deben ser descriptivos, breves y consistentes.
3. Cada archivo de artículo responde una sola pregunta concreta.
4. Las rutas siguen el mismo patrón en español e inglés.
5. No se usan espacios ni nombres ambiguos.

1. Each section has a clear and stable path.
2. Names must be descriptive, brief, and consistent.
3. Each article file answers one single, concrete question.
4. Paths follow the same pattern in Spanish and English.
5. No spaces or ambiguous names are used.

---

## Estructura general / General structure

```text
HowThingsTalk/
├── docs/
│   ├── ES/
│   │   ├── 00_Fundamentos/
│   │   ├── 01_RedesWiFi/
│   │   └── ...
│   ├── EN/
│   │   ├── 00_Foundations/
│   │   ├── 01_WiFiNetworks/
│   │   └── ...
│   ├── Index.md
│   └── RoadMap.md
├── guides/
├── templates/
├── CHANGELOG.md
├── LICENSE.md
└── README.md
```

---

## Archivos raíz / Root files

Los archivos que GitHub reconoce por su nombre van en mayúsculas: `README.md`, `CHANGELOG.md` y `LICENSE.md`. El resto de documentos del proyecto (`docs/Index.md`, `docs/RoadMap.md`, las guías y la plantilla) usan PascalCase. La extensión es siempre `.md`, en minúsculas.

Files that GitHub recognizes by name are uppercase: `README.md`, `CHANGELOG.md`, and `LICENSE.md`. Every other project document (`docs/Index.md`, `docs/RoadMap.md`, the guides, and the template) uses PascalCase. The extension is always lowercase `.md`.

---

## Regla de carpetas / Folder rules

### Secciones principales / Main sections

Cada carpeta de sección usa:

- número de sección con dos dígitos;
- guion bajo para separar número y título;
- título descriptivo en PascalCase, en el idioma de la carpeta (`ES` o `EN`).

Each section folder uses:

- a two-digit section number;
- an underscore between number and title;
- a descriptive PascalCase title in the folder's language (`ES` or `EN`).

Ejemplos / Examples:

- `docs/ES/05_Geolocalizacion/` ↔ `docs/EN/05_Positioning/`
- `docs/ES/10_SeguridadYCriptografia/` ↔ `docs/EN/10_SecurityAndCryptography/`

### Reglas adicionales / Additional rules

- Sin espacios.
- Sin tildes, eñes ni caracteres especiales: `Geolocalizacion`, no `Geolocalización`.
- Las siglas se mantienen como se escriben habitualmente (`WiFi`, `IOT`).
- El número de la carpeta coincide con el de la hoja de ruta y es el mismo en `ES` y `EN`.

- No spaces.
- No accents, `ñ`, or special characters.
- Acronyms keep their usual spelling (`WiFi`, `IOT`).
- The folder number matches the roadmap and is the same in `ES` and `EN`.

---

## Regla de archivos / File rules

Los artículos siguen este patrón:

Articles follow this pattern:

```text
NN-TituloDelArticulo.md
```

Donde / Where:

- `NN` = dos dígitos de orden dentro de la sección / two-digit order within the section;
- `TituloDelArticulo` = título corto y directo en PascalCase / short, direct PascalCase title;
- `.md` = formato Markdown / Markdown format.

### Un artículo, dos archivos / One article, two files

Cada artículo existe en las dos carpetas de idioma, con **el mismo `NN`** y el nombre escrito en el idioma de su contenido:

Every article exists in both language folders, with **the same `NN`** and a name written in the language of its content:

| ES | EN |
|---|---|
| `docs/ES/05_Geolocalizacion/01-LocalizacionPorWiFi.md` | `docs/EN/05_Positioning/01-WiFiPositioning.md` |
| `docs/ES/05_Geolocalizacion/02-TrilateracionVsTriangulacion.md` | `docs/EN/05_Positioning/02-TrilaterationVsTriangulation.md` |
| `docs/ES/10_SeguridadYCriptografia/02-ComoFuncionaTOTP.md` | `docs/EN/10_SecurityAndCryptography/02-HowTOTPWorks.md` |

La pareja ES/EN se identifica por la sección y el `NN`, que es el siguiente número libre de la sección. Los dos archivos se crean a la vez.

The ES/EN pair is identified by section and `NN`, which is the next free number in the section. Both files are created together.

### Reglas para el nombre / Name rules

- Solo letras sin tilde, números y un único guion medio tras `NN`.
- Sin espacios, puntos ni guiones bajos en el título.
- Empieza por mayúscula (PascalCase).
- Describe la pregunta del artículo y se entiende sin abrir el archivo.
- Si el título es largo, se abrevia con precisión; nunca con «Varios» o «Misc».

- Unaccented letters, digits, and a single hyphen after `NN` only.
- No spaces, dots, or underscores in the title.
- Starts with a capital letter (PascalCase).
- Describes the article's question and is understandable without opening the file.
- Long titles are shortened precisely; never with "Misc" or "Stuff".

---

## Títulos y lenguaje / Titles and language

Los artículos se nombran con la idea central, no con etiquetas genéricas.

Articles are named after their central idea, not a generic label.

Correcto / Correct:

- `03-RelatividadEnElGPS.md` · `03-RelativityInGPS.md`
- `01-PorQueElMovilSeApagaConFrio.md` · `01-WhyPhonesDieInCold.md`
- `01-LoQueTLSNoProtege.md` · `01-WhatTLSDoesNotProtect.md`

Incorrecto / Incorrect:

- `MiArticulo.md`
- `NotasSobreGPS.md`
- `DocumentacionFinal.md`

La estructura del repositorio es bilingüe, no mezclada: un archivo de `docs/ES/` está escrito y nombrado en español, y uno de `docs/EN/`, en inglés.

The repository structure is bilingual, not mixed: a file in `docs/ES/` is written and named in Spanish, and one in `docs/EN/` in English.

---

## Front matter y metadatos / Front matter and metadata

Cada artículo empieza con una cabecera YAML. Las claves y los valores controlados (`language`, `status`, `depth`) son iguales en los dos idiomas, para que puedan procesarse de forma automática; solo `title`, `summary` y `prerequisites` se escriben en el idioma del artículo.

Every article starts with a YAML header. Keys and controlled values (`language`, `status`, `depth`) are identical in both languages so they can be processed automatically; only `title`, `summary`, and `prerequisites` are written in the article's language.

```yaml
---
title: "Cómo te localiza el WiFi sin que te conectes a ninguna red"
section: "05_Geolocalizacion"
language: "es"
status: "draft"
order: 1
depth: "intermediate"
summary: "Resumen breve de la pregunta y la respuesta."
prerequisites:
  - "Concepto A"
  - "Concepto B"
translation: "../../EN/05_Positioning/01-WiFiPositioning.md"
---
```

### Campos / Fields

| Campo / Field | Valor / Value |
|---|---|
| `title` | Título legible, en el idioma del artículo / Readable title, in the article's language |
| `section` | Nombre exacto de la carpeta de sección / Exact section folder name |
| `language` | `es` · `en` |
| `status` | `draft` · `in_review` · `published` · `archived` |
| `order` | El número `NN`, sin ceros a la izquierda / The `NN` number, without leading zeros |
| `depth` | `intro` · `intermediate` · `advanced` |
| `summary` | 1-2 frases / 1-2 sentences |
| `prerequisites` | Lista de conceptos previos / List of prior concepts |
| `translation` | Ruta relativa a la versión en el otro idioma / Relative path to the other-language version |

---

## Estado de publicación / Publication status

Los artículos se etiquetan según su estado real:

- `draft`: todavía en proceso de escritura.
- `in_review`: escrito y pendiente de revisión.
- `published`: revisado, con sus dos versiones (ES y EN) y enlazado en el índice.
- `archived`: descartado o sustituido por otra explicación.

Articles are labeled by their real status:

- `draft`: still being written.
- `in_review`: written and awaiting review.
- `published`: reviewed, with both versions (ES and EN), and linked from the index.
- `archived`: discarded or replaced by another explanation.

---

## Regla de oro / Golden rule

Si un nombre puede confundirse, es un mal nombre. Si un nombre comunica la pregunta del artículo sin que haya que abrirlo, es un buen nombre.

If a name can be confused, it is a bad name. If the name communicates the article's question without opening it, it is a good name.

---

## English

### Naming rules summary

- Section folders: `NN_Title`, e.g. `docs/ES/05_Geolocalizacion/` and `docs/EN/05_Positioning/`.
- Article files: `NN-TitleInPascalCase.md`, named in the language of their folder.
- Each article exists in both `ES` and `EN` with the same `NN`.
- Root files GitHub recognizes are uppercase (`README.md`, `CHANGELOG.md`, `LICENSE.md`); everything else is PascalCase with a lowercase `.md`.
- No spaces, accents, or ambiguous labels.

Examples:

- `docs/EN/05_Positioning/01-WiFiPositioning.md`
- `docs/EN/05_Positioning/02-TrilaterationVsTriangulation.md`
- `docs/EN/10_SecurityAndCryptography/01-WhatTLSDoesNotProtect.md`

[Volver al español · Back to Spanish](#convención-de-nombres--naming-convention)

[← Volver al índice general / Back to the master index](../docs/Index.md)
