<!--
Plantilla de artículo / Article template

Cada artículo se escribe dos veces: copia la parte ES a docs/ES/<Sección>/NN-TituloEnEspanol.md
y la parte EN a docs/EN/<Section>/NN-TitleInEnglish.md, con el mismo NN en las dos.
Each article is written twice: copy the ES part to docs/ES/<Sección>/NN-TituloEnEspanol.md
and the EN part to docs/EN/<Section>/NN-TitleInEnglish.md, with the same NN in both.

Reglas / Rules: ../guides/NamingConvention.md · ../guides/StyleGuide.md
Borra este comentario y las instrucciones en cursiva al escribir. / Delete this comment and the italic instructions when writing.
-->

<!-- ===================== ES: copiar desde aquí ===================== -->

---
title: "Título del artículo"
section: "NN_NombreDeSeccion"
language: "es"
status: "draft"
order: 1
depth: "intermediate"
summary: "Resumen breve de la pregunta y la respuesta."
prerequisites:
  - "Concepto previo"
translation: "../../EN/NN_SectionName/NN-ArticleTitle.md"
---

# Título del artículo

## Pregunta y encuadre

_¿Cuál es la pregunta exacta que responde este artículo? ¿Qué queda explícitamente fuera del alcance? ¿Qué profundidad aborda? ¿Qué conocimientos previos hacen falta para seguir la lectura sin perderse?_

## Respuesta corta

_Una frase contundente que sintetiza la respuesta completa._

### Resumen de 60 segundos

_Una visión directa y clara para asimilar la idea global sin necesidad de leer el artículo completo._

## Modelo mental

_Construye la intuición con una analogía del mundo cotidiano. Debe asentar el concepto y delimitar con precisión el punto exacto donde la metáfora se rompe y da paso a la realidad tecnológica._

## Mecanismo técnico

### 1. Qué ocurre en el sistema

_Describe el flujo general del mecanismo, la secuencia cronológica de pasos y los elementos que intervienen._

### 2. Qué hace cada componente

_Desglosa cada parte del sistema y su función dentro del conjunto._

### 3. Parámetros que importan

_Incluye, si es relevante:_

- _frecuencia o banda de operación;_
- _ancho de banda o tasa de transferencia;_
- _potencia o nivel de señal;_
- _latencia;_
- _consumo energético;_
- _tolerancias y márgenes de error;_
- _protocolo o capa afectada._

### 4. Diagrama o flujo

_Incorpora un esquema textual o un pseudodiagrama._

```text
Fuente -> Transmisor -> Canal -> Receptor -> Decodificación -> Resultado
```

## Límites, excepciones y casos reales

_¿Qué cambia al llevar la teoría a la práctica? ¿Qué condiciones impiden que el sistema funcione igual en todos los entornos? ¿Qué variables reales hacen que el comportamiento se desvíe del modelo ideal?_

## Consecuencias prácticas

_¿Por qué es determinante esto para el usuario, el diseño del aparato o la experiencia real?_

## Preguntas frecuentes

### ¿Es esto lo mismo que…?

_Establece la distinción clave frente a un concepto afín o fácil de confundir._

## Fuentes

- _[Estándar o normativa primaria]_
- _[Documentación oficial del fabricante]_
- _[Artículo científico o publicación técnica verificada]_

<!-- ===================== ES: copiar hasta aquí ===================== -->

<!-- ===================== EN: copy from here ===================== -->

---
title: "Article title"
section: "NN_SectionName"
language: "en"
status: "draft"
order: 1
depth: "intermediate"
summary: "Short summary of the question and the answer."
prerequisites:
  - "Prior concept"
translation: "../../ES/NN_NombreDeSeccion/NN-TituloDelArticulo.md"
---

# Article title

## Question and framing

_What is the exact question this article answers? What is explicitly out of scope? What depth does it target? What prior knowledge is required to follow along without getting lost?_

## Short answer

_A punchy sentence summarizing the full answer._

### 60-second summary

_A direct, clear overview to grasp the overall idea without reading the full article._

## Mental model

_Build intuition with an everyday analogy. It should ground the concept and mark precisely where the metaphor breaks down and yields to technological reality._

## Technical mechanism

### 1. What happens in the system

_Describe the general flow of the mechanism, the chronological sequence of steps, and the elements involved._

### 2. What each component does

_Break down each part of the system and its role within the whole._

### 3. Key parameters

_Include, if relevant:_

- _frequency or operating band;_
- _bandwidth or data rate;_
- _power or signal level;_
- _latency;_
- _energy consumption;_
- _tolerances and margins of error;_
- _affected protocol or layer._

### 4. Diagram or flow

_Include a text schema or a pseudo-diagram._

```text
Source -> Transmitter -> Channel -> Receiver -> Decoding -> Result
```

## Limits, exceptions, and real-world cases

_What changes when theory meets practice? What conditions prevent the system from behaving the same in every environment? What real-world variables make it deviate from the ideal model?_

## Practical consequences

_Why does this matter for the user, the device design, or the real-world experience?_

## Frequently asked questions

### Is this the same as…?

_Draw the key distinction from a closely related or easily confused concept._

## Sources

- _[Primary standard or regulation]_
- _[Official manufacturer documentation]_
- _[Verified paper or technical publication]_

<!-- ===================== EN: copy up to here ===================== -->
