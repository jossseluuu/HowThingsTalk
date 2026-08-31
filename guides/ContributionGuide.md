# Guía de contribución / Contribution guide

<div align="center">

**Español** · [English](#english)

</div>

---

## ¿Qué es este proyecto? / What is this project?

HowThingsTalk es una biblioteca técnica de explicaciones sobre cómo funcionan los aparatos del día a día. La prioridad no es publicar rápido, sino publicar bien: cada artículo debe responder una pregunta concreta, ser verificable y mantenerse útil dentro de varios años.

HowThingsTalk is a technical library explaining how everyday devices work under the hood. The priority is not speed, but correctness: every article must answer a concrete question, be verifiable, and remain useful for years.

## Qué vale como contribución / What counts as a contribution

Se aceptan:

- correcciones de dato, explicación o lenguaje;
- nuevas entradas para temas no cubiertos;
- mejoras de claridad, enlaces, diagramas o referencias;
- ampliaciones de un artículo ya publicado;
- propuestas de estructura, taxonomía o terminología.

The project welcomes:

- corrections to data, explanations, or wording;
- new entries for uncovered topics;
- improvements in clarity, links, diagrams, or references;
- expansions to published articles;
- suggestions around structure, taxonomy, or terminology.

No se aceptan:

- contenido basado en rumores, artículos de prensa sin fuente primaria;
- comparativas de productos o reseñas de compra;
- tutoriales de configuración sin explicación del mecanismo;
- pasos de explotación o vulneración de sistemas ajenos;
- afirmaciones sin fuente verificable.

The project does not accept:

- rumor-based content or press articles without a primary source;
- product comparisons or buying guides;
- setup tutorials without explanation of the underlying mechanism;
- exploit steps or attacks against third-party systems;
- claims without verifiable evidence.

---

## Flujo recomendado / Recommended workflow

### 1. Comprueba la hoja de ruta / Check the roadmap

Antes de escribir, confirma que el tema no está ya en proceso ni publicado en [docs/RoadMap.md](../docs/RoadMap.md).

Before writing, confirm the topic is not already in progress or published in [docs/RoadMap.md](../docs/RoadMap.md).

### 2. Revisa la guía de estilo / Review the style guide

La escritura del proyecto sigue una línea muy específica: directa, técnica, precisa y fácil de seguir sin perder rigor. Revisa [StyleGuide.md](StyleGuide.md) antes de empezar.

The project follows a very specific line of writing: direct, technical, precise, and easy to follow without sacrificing rigor. Review [StyleGuide.md](StyleGuide.md) before beginning.

### 3. Usa la plantilla / Use the template

Copia [templates/ArticuleTemplate.md](../templates/ArticuleTemplate.md) y trabaja desde ahí. No hagas un artículo desde cero dejando la estructura al azar.

Copy [templates/ArticuleTemplate.md](../templates/ArticuleTemplate.md) and work from there. Do not start writing a new article from scratch without following the structure.

### 4. Nombra el archivo correctamente / Name the file correctly

El nombre debe seguir la convención del proyecto. Consulta [NamingConvention.md](NamingConvention.md).

The filename must follow the repository convention. Review [NamingConvention.md](NamingConvention.md).

### 5. Escribe con la pregunta clara / Write with a clear question

Cada artículo debe responder una pregunta concreta, no un tema amplio o una lista de cosas. Si la pregunta no se puede formular de forma nítida, probablemente no es un artículo.

Each article must answer one concrete question, not a broad topic or a loose checklist. If the question cannot be stated clearly, it is probably not an article.

### 6. Verifica las afirmaciones / Verify all claims

Todo dato técnico debe tener contexto: unidad, escenario y fuente. Si no hay una referencia primaria verificable, no se incorpora.

Every technical claim should include context: unit, scenario, and source. If there is no verifiable primary reference, it does not belong in the article.

### 7. Actualiza la navegación / Update navigation

Cuando el artículo se publique o se complete una primera versión, hay que enlazarlo desde [docs/Index.md](../docs/Index.md) y, si procede, reflejar el avance en [docs/RoadMap.md](../docs/RoadMap.md).

When an article is published or reaches a first complete draft, it must be linked from [docs/Index.md](../docs/Index.md) and, when relevant, reflected in [docs/RoadMap.md](../docs/RoadMap.md).

### 8. Abre un pull request / Open a pull request

Antes de enviar la propuesta, revisa:

- nombre del archivo;
- ruta correcta en la sección;
- estructura del artículo;
- referencias con fuentes primarias;
- enlaces del índice y la hoja de ruta;
- consistencia con el resto del repositorio.

Before submitting, review:

- filename and path;
- correct section placement;
- article structure;
- references with primary sources;
- links in the index and roadmap;
- consistency with the rest of the repository.

---

## Requisitos mínimos de calidad / Minimum quality bar

Todo artículo debe cumplir, como mínimo:

1. Responder una pregunta concreta y legible.
2. Tener 1-2 párrafos de contexto, no un bloque de introducción caótico.
3. Explicar el mecanismo real, no solo una metáfora.
4. Incluir números físicos con unidad, cuando sean relevantes.
5. Usar analogías solo como apoyo y marcar claramente sus límites.
6. Citar fuentes primarias o documentación técnica verificable.
7. Ser autocontenido para un lector de nivel intermedio.
8. Mantener el mismo tono y rigor que el resto del repositorio.

Every article must, at minimum:

1. Answer one concrete and readable question.
2. Include 1-2 paragraphs of context rather than a chaotic intro block.
3. Explain the real mechanism, not just a metaphor.
4. Include physical numbers with units when relevant.
5. Use analogies only as support and clearly mark their limits.
6. Cite primary sources or verifiable technical documentation.
7. Be self-contained for an intermediate-level reader.
8. Maintain the same tone and rigor as the rest of the project.

---

## Fuentes que se aceptan / Acceptable sources

Se prioriza lo siguiente:

- estándares primarios: IEEE, 3GPP, IETF RFC, Bluetooth SIG, USB-IF, ISO, W3C, etc.;
- documentación técnica de fabricantes y especificaciones de protocolo;
- artículos de investigación revisados por pares;
- documentación oficial de hardware, software y protocolos.

The following are prioritized:

- primary standards: IEEE, 3GPP, IETF RFC, Bluetooth SIG, USB-IF, ISO, W3C, etc.;
- technical documentation from manufacturers and protocol specifications;
- peer-reviewed research papers;
- official hardware, software, and protocol documentation.

Se excluyen:

- blogs de segunda mano;
- artículos “popular science” sin cita técnica;
- rumores, comentarios sin datos;
- copias de otros artículos sin referencia original.

The following are excluded:

- secondary blogs;
- “popular science” articles without technical citation;
- rumors, comments without data;
- copied explanations without an original reference.

---

## Estructura de una contribución / Contribution structure

### Propuesta de tema / Topic proposal

Si propones un artículo nuevo, incluye:

- pregunta exacta que responde;
- por qué es útil para la biblioteca;
- nivel de profundidad esperado;
- fuentes preliminares;
- sección donde encaja.

If proposing a new article, include:

- the exact question it answers;
- why it is useful for the library;
- expected depth level;
- preliminary sources;
- the section where it fits.

### Corrección / Fix

Si corriges un texto existente, añade:

- la afirmación original;
- la fuente que la corrige;
- la explicación breve del error.

If editing an existing text, provide:

- the original claim;
- the source that corrects it;
- a brief explanation of the error.

---

## Checklist antes de hacer PR / PR checklist

- [ ] El tema no duplicaba un artículo ya publicado o en curso.
- [ ] El archivo sigue la convención de nomenclatura.
- [ ] La estructura del artículo corresponde a la plantilla.
- [ ] La pregunta es nítida y específica.
- [ ] Hay una explicación del mecanismo real, no solo una metáfora.
- [ ] Hay fuentes verificables en la sección final.
- [ ] Los datos tienen unidad y contexto.
- [ ] El texto no presenta marketing ni opiniones sin base técnica.
- [ ] El índice y la ruta del artículo están actualizados.

- [ ] The topic does not duplicate a published or in-progress article.
- [ ] The file follows the naming convention.
- [ ] The article structure matches the template.
- [ ] The question is clear and specific.
- [ ] The real mechanism is explained, not just the metaphor.
- [ ] There are verifiable sources in the final section.
- [ ] Data points include units and context.
- [ ] The text does not contain marketing language or unsupported opinions.
- [ ] The index and article path are updated.

---

## Política de calidad / Quality policy

El proyecto prioriza la honestidad sobre la exuberancia. Si la respuesta no está comprobada, se marca como “no verificado” o se omite. No se fuerza una explicación elegante cuando la realidad es más compleja.

The project prioritizes honesty over exuberance. If a claim is not verified, it must be marked as “unverified” or omitted. We do not force a neat explanation when the reality is more complex.

En otras palabras: si un dato no se puede defender con una referencia decente, no pertenece a la biblioteca.

In other words: if a fact cannot be defended with a decent source, it does not belong in the library.

---

## English

### Contribution guidelines

This repository is a technical reference rather than an opinion blog. Contributions are expected to be clear, rigorous, and source-backed.

Before writing:

1. Check the roadmap and index.
2. Follow the naming convention.
3. Use the article template.
4. Write one specific question at a time.
5. Verify every claim with a reliable source.
6. Update the relevant navigation file before the PR.

A good contribution is not “smart-sounding”; it is correct, readable, and useful to a curious reader who wants to understand the mechanism, not just the headline.

[← Volver al índice general / Back to the main index](../docs/Index.md)
