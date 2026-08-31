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
3. Los archivos de artículo deben responder una sola pregunta concreta.
4. Las rutas deben seguir el mismo patrón en español e inglés.
5. No se usan espacios ni nombres ambiguos.

1. Each section has a clear and stable path.
2. Names must be descriptive, brief, and consistent.
3. Article files must answer one single, concrete question.
4. Paths should follow the same pattern in Spanish and English.
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
├── Readme.md
└── ...
```

## Regla de carpetas / Folder rules

### Secciones principales

Cada carpeta de sección usa:

- número de sección con dos dígitos;
- nombre descriptivo de la temática;
- guion bajo para separar número y título;
- el título en el idioma correspondiente.

Examples:

- `docs/ES/05_Geolocalizacion/`
- `docs/EN/05_Positioning/`
- `docs/ES/10_SeguridadYCriptografia/`
- `docs/EN/10_SecurityAndCryptography/`

### Reglas adicionales

- No se usan espacios.
- No se usan caracteres raros.
- Se usa CamelCase / PascalCase o palabras concatenadas con mayúsculas iniciales, según el idioma.
- La sección debe coincidir con el número que aparece en la hoja de ruta.

---

## Regla de archivos / File rules

Los artículos siguen este patrón:

```text
NN-TituloDelArticulo.md
```

Donde:

- `NN` = dos dígitos de orden dentro de la sección;
- `TituloDelArticulo` = título corto y directo;
- `.md` = formato Markdown.

Examples:

- `01-WiFiPositioning.md`
- `02-TrilaterationVsTriangulation.md`
- `09-WhyRadioPassesWalls.md`
- `10-HowTOTPWorks.md`

### Reglas para el nombre

- Solo letras, números y guiones medios.
- Sin espacios, puntos ni subrayados.
- El texto debe ser legible y describir la pregunta del artículo.
- Debe poder entenderse sin abrir el archivo.
- Si el título es largo, se abrevia con precisión; no con “misc” o “stuff”.

---

## Títulos y lenguaje / Titles and language

Los artículos deben nombrarse con la idea central, no con frases genéricas.

Correcto:

- `11-RelativityInGPS.md`
- `08-WhyPhonesDieInCold.md`
- `09-WhatTLSDoesNotProtect.md`

Incorrecto:

- `MiArticulo.md`
- `NotasSobreGPS.md`
- `DocumentacionFinal.md`

El artículo puede estar escrito en español, pero el nombre debe seguir la convención del idioma del contenido. La estructura del repositorio se mantiene bilingüe, no mezclada.

The article title should reflect the central idea, not a generic label.

Correct:

- `11-RelativityInGPS.md`
- `08-WhyPhonesDieInCold.md`
- `09-WhatTLSDoesNotProtect.md`

Incorrect:

- `MiArticulo.md`
- `NotasSobreGPS.md`
- `DocumentacionFinal.md`

The article may be written in Spanish, but the filename must follow the naming convention of the language used in the content. The repository structure remains bilingual, not mixed.

---

## Front matter y metadatos / Front matter and metadata

Cada archivo debe incluir la cabecera de metadatos al principio. El esquema mínimo es:

```yaml
---
title: "Título del artículo"
section: "05-Positioning"
language: "es"
status: "draft"
order: 1
depth: "intermedio"
summary: "Resumen breve de la pregunta y la respuesta."
prerequisites:
  - "Concepto A"
  - "Concepto B"
---
```

### Campos recomendados / Recommended fields

- `title`: título legible del artículo.
- `section`: la sección a la que pertenece.
- `language`: `es` o `en`.
- `status`: `draft`, `in_review`, `published`, `archived`.
- `order`: número dentro de la sección.
- `depth`: `intro`, `intermedio`, `avanzado`.
- `summary`: frase breve de 1-2 líneas.
- `prerequisites`: lista de conceptos previos necesarios.

- `title`: readable title of the article.
- `section`: the section the article belongs to.
- `language`: `es` or `en`.
- `status`: `draft`, `in_review`, `published`, `archived`.
- `order`: article number within the section.
- `depth`: `intro`, `intermediate`, `advanced`.
- `summary`: 1-2 sentence summary.
- `prerequisites`: list of prior concepts needed.

---

## Estado de publicación / Publication status

Los artículos se etiquetan según su estado real:

- `draft`: todavía en proceso de escritura.
- `in_review`: revisado por el autor o la comunidad.
- `published`: publicado y enlazado en la navegación principal.
- `archived`: descartado o reemplazado por otra explicación.

This is the status vocabulary used by the project:

- `draft`: still being written.
- `in_review`: under review.
- `published`: published and linked in the main navigation.
- `archived`: discarded or replaced by another explanation.

---

## Regla de oro / Golden rule

Si un nombre puede confundirse, es un mal nombre. Si un nombre comunica la pregunta del artículo sin que haya que abrirlo, es un buen nombre.

If a name can be confused, it is a bad name. If the name communicates the article’s question without opening it, it is a good name.

---

## English

### Naming rules summary

- Use numbered section folders such as `00_Foundations` or `05_Positioning`.
- Use file names in the form `NN-TitleInPascalCase.md`.
- Keep names concise, descriptive, and technical.
- Never leave spaces or ambiguous labels.
- Match the section, language, and document title to the article’s topic.

Examples:

- `05-Positioning/01-WiFiPositioning.md`
- `05-Positioning/02-TrilaterationVsTriangulation.md`
- `10-SecurityAndCryptography/09-WhatTLSDoesNotProtect.md`

[← Volver al índice general / Back to the main index](../docs/Index.md)
