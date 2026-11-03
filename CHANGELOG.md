# Registro de cambios · Changelog

Todos los cambios relevantes del repositorio se anotan aquí, del más reciente al más antiguo. Una tarea automática revisa el repositorio cada día y añade una entrada con lo que haya cambiado; si no ha cambiado nada, no añade entrada.

All notable changes to this repository are recorded here, newest first. An automated task reviews the repository every day and adds an entry with whatever changed; if nothing changed, no entry is added.

Formato basado en / Format based on [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/). Categorías / Categories: **Añadido · Added**, **Cambiado · Changed**, **Corregido · Fixed**, **Eliminado · Removed**.

---

## 2026-11-03 · Revisión automática · Automated review

### Añadido · Added

- Artículo «¿Cómo puede un auricular quitar ruido emitiendo más sonido?» ([`docs/ES/07_AudioYPerifericos/01-ComoQuitaRuidoUnAuricular.md`](docs/ES/07_AudioYPerifericos/01-ComoQuitaRuidoUnAuricular.md)) · Article "How can headphones remove noise by playing more sound?" ([`docs/EN/07_AudioAndPeripherals/01-HowHeadphonesCancelNoise.md`](docs/EN/07_AudioAndPeripherals/01-HowHeadphonesCancelNoise.md)).

## 2026-11-02 · Revisión automática · Automated review

### Añadido · Added

- Artículo «¿Qué decide cuántos bits por segundo caben en una señal de radio?» ([`docs/ES/00_Fundamentos/02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md`](docs/ES/00_Fundamentos/02-QueDecideLaVelocidadDeUnEnlaceDeRadio.md)) · Article "What decides how many bits per second a radio signal can carry?" ([`docs/EN/00_Foundations/02-WhatSetsTheSpeedOfARadioLink.md`](docs/EN/00_Foundations/02-WhatSetsTheSpeedOfARadioLink.md)).

## 2026-10-31 · Revisión automática · Automated review

### Añadido · Added

- Artículo «¿Por qué una señal de radio de más frecuencia llega menos lejos?» ([`docs/ES/00_Fundamentos/01-PorQueMasFrecuenciaLlegaMenosLejos.md`](docs/ES/00_Fundamentos/01-PorQueMasFrecuenciaLlegaMenosLejos.md)) · Article "Why does a higher-frequency radio signal reach a shorter distance?" ([`docs/EN/00_Foundations/01-WhyHigherFrequenciesReachLess.md`](docs/EN/00_Foundations/01-WhyHigherFrequenciesReachLess.md)).

## 2026-10-31 · Repaso de coherencia · Coherence pass

### Añadido · Added

- `CHANGELOG.md`: este registro de cambios. · This changelog.
- `LICENSE.md`: licencias CC BY-SA 4.0 (contenido) y MIT (código), antes enlazadas pero inexistentes. · CC BY-SA 4.0 (content) and MIT (code) licenses, previously linked but missing.
- Guía de estilo: secciones de bilingüismo, consecuencias prácticas, preguntas frecuentes y la frontera de seguridad explícita. · Style guide: bilingualism, practical consequences, FAQ sections and an explicit security boundary.
- Convención de nombres: archivos raíz, pareja ES/EN con el mismo `NN`, tabla de campos del front matter y campo `translation`. · Naming convention: root files, ES/EN pair sharing `NN`, front matter field table and `translation` field.

### Cambiado · Changed

- `docs/ROADMAP.MD` → `docs/RoadMap.md`, como indica la convención de nombres. · As the naming convention requires.
- `templates/ArticuleTemplate.md` → `templates/ArticleTemplate.md` (errata en el nombre · filename typo).
- `guides/TaxonomyAndGlosary.md` → `guides/TaxonomyAndGlossary.md` (errata en el nombre · filename typo).
- Hoja de ruta: solo recoge lo que está en curso, lo publicado y lo descartado; los contadores usan los nombres reales de carpeta y reflejan los archivos reales (0 artículos). · Roadmap: only tracks in-progress, published and discarded items; counters use the real folder names and reflect the real files (0 articles).
- La biblioteca pasa a ser bilingüe también en los artículos: se retira de «Descartado» la traducción al inglés y se descarta en su lugar publicar en un solo idioma. · The library is now bilingual for articles too.
- Plantilla: front matter en cada mitad, títulos de sección unificados con la guía de estilo y la sección de fuentes al final. · Template: front matter in each half, headings aligned with the style guide, sources last.
- Front matter: `section` usa el nombre de carpeta y `depth` usa `intro`/`intermediate`/`advanced` en los dos idiomas. · `section` uses the folder name and `depth` uses the same values in both languages.
- Glosario: entradas ordenadas alfabéticamente y con encabezado bilingüe; secciones en tabla con sus carpetas ES y EN. · Glossary: alphabetical, bilingual headings; sections table with ES and EN folders.
- README e índice: la tabla de estado deja de usar `X`/`Y` y muestra cifras reales. · README and index: status table shows real figures instead of `X`/`Y`.
- README: «Preguntas que se responderán aquí» pasa a «El tipo de preguntas que responde esta biblioteca». · Renamed to describe the kind of questions, not a promise.

### Eliminado · Removed

- Hoja de ruta: la lista pública de artículos propuestos, «Los tres siguientes», «Por investigar» y «Dudas sueltas». Los próximos temas ya no se anuncian; cada artículo aparece en la hoja de ruta al entrar en curso o al publicarse. · Roadmap: the public list of proposed articles, "The next three", "To research" and "Loose ideas". Upcoming topics are no longer announced.

### Corregido · Fixed

- Enlaces rotos en README, índice, hoja de ruta y guías (`docs/README.md`, `docs/Roadmap.md`, `docs/_meta/StyleGuide.md`, `docs/99-Reference/01-Glossary.md`, `templates/StyleGuide.md`, `Readme.md`, `LICENSE`). · Broken links across README, index, roadmap and guides.
- README: «¿Cómo se escribe aquí?» enlazaba a la convención de nombres en vez de a la guía de estilo. · Linked to the naming convention instead of the style guide.
- README: el punto de entrada recomendado era un artículo inexistente. · The recommended entry point was a non-existent article.
- Hoja de ruta: «1 artículo publicado de 107 esqueletos» y el artículo publicado que enlazaba no existían en el repositorio, que no contenía ningún artículo. · "1 published out of 107 skeletons" did not match the repository, which contained no articles.
- Guía de estilo: el ejemplo «1,5 V en una celda de litio» pasa a «3,7 V nominales en una celda de ion-litio». · Lithium cell example voltage corrected.
- Glosario: definiciones de ancho de banda, BSSID, decibelio, GPS, ruido, señal y WiFi precisadas. · Definitions made precise.
- Erratas: «sTATE», «Writed», «trames», «probablmente», «fuentas», «boundatary», «convept», «previso», «llantos de contenido», «bateria», entre otras. · Typos.
- Anclas «Volver al español» que apuntaban a encabezados inexistentes. · "Back to Spanish" anchors pointing at missing headings.

[← Volver al README · Back to the README](README.md)
