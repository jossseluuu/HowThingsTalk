# Guía de estilo / Style guide

<div align="center">

**Español** · [English](#english)

</div>

---

## El objetivo del texto / The purpose of the text

HowThingsTalk no es un blog de opinión ni una guía de compra. Es una biblioteca técnica para explicar cómo funciona lo que llevamos en el bolsillo, en la mesa, en la muñeca o en la pared.

HowThingsTalk is not an opinion blog or a buying guide. It is a technical library that explains how the things we carry in our pockets, on our desks, on our wrists, or on our walls actually work.

Cada artículo debe hacer tres cosas bien:

Every article should do three things well:

1. responder una pregunta concreta;
2. explicar el mecanismo real;
3. ser comprensible sin perder rigor.

1. answer one concrete question;
2. explain the real mechanism;
3. remain understandable without losing rigor.

---

## Tonalidad / Tone

La prosa debe ser:

- directa;
- técnica sin ser pedante;
- densa cuando hace falta, breve cuando no;
- honesta con las limitaciones del conocimiento;
- clara para un lector curioso con formación general.

The prose should be:

- direct;
- technical without being pedantic;
- dense when needed, brief when not;
- honest about the limits of current knowledge;
- clear for a curious reader with general technical literacy.

Se prioriza la claridad sobre la “elegancia” artificial. No se escribe para impresionar, sino para explicar.

Clarity is prioritized over artificial elegance. We do not write to impress; we write to explain.

---

## Estructura recomendada del artículo / Recommended article structure

Cada artículo debería poder leerse en capas, sin que el lector pierda el hilo si se salta secciones.

Each article should be readable in layers without losing the thread if the reader skips sections.

### 1. Pregunta y encuadre / Question and framing

Debe quedar claro:

- qué pregunta responde;
- qué nivel de profundidad tiene;
- qué conocimiento previo hace falta;
- qué no se va a cubrir.

It should be clear:

- what question it answers;
- what depth it targets;
- what prerequisite knowledge is needed;
- what will not be covered.

### 2. Respuesta corta / Short answer

Una frase contundente y una explicación de 60 segundos.

A punchy sentence and a 60-second explanation.

### 3. Modelo mental / Mental model

Un ejemplo cotidiano que permita inferir la idea. Debe quedar claro en qué punto la analogía deja de ser válida.

A familiar everyday example that allows the reader to infer the idea. The moment where the analogy stops being valid must be clear.

### 4. Mecanismo técnico / Technical mechanism

Aquí va la explicación de verdad: señales, capas, protocolos, flujo, tiempos, física, condiciones límite, componentes.

This is where the real explanation lives: signals, layers, protocols, flow, timing, physics, limits, and components.

### 5. Límites, excepciones y consecuencias / Limits, exceptions, and impact

Es importante explicar qué no hace, qué se complica en casos reales y por qué la idea funciona mejor en teoría que en la práctica.

It is important to explain what it does not do, what becomes complicated in real-world conditions, and why the idea is cleaner in theory than in practice.

### 6. Referencias / References

Cada artículo debe cerrar con una lista de referencias primarias o de máxima confianza.

Every article should end with a list of primary or highly reliable references.

---

## Reglas de escritura / Writing rules

### 1. Frases cortas y claras / Short, clear sentences

Si una frase tiene más de tres comas, probablemente debería dividirse en dos.

If a sentence has more than three commas, it probably should be split into two.

### 2. Datos con unidad / Data with units

Todo dato físico lleva unidad, escenario y contexto.

Every physical fact should include units, scenario, and context.

Ejemplos:

- `10 ms de retardo`.
- `2,4 GHz en un enlace de 1 m`.
- `1,5 V` en una celda de litio.

Examples:

- `10 ms latency`.
- `2.4 GHz over a 1 m link`.
- `1.5 V` in a lithium cell.

### 3. Analogías transparentes / Transparent analogies

La analogía es útil si deja claro dónde rompe. No se presenta como una explicación completa.

Analogies are useful when it is clear where they break down. They are never presented as a complete explanation.

### 4. Sin marketing / No marketing language

No se usan frases tipo “revolucionario”, “increíble”, “mágico”, “brutal” salvo que realmente sean necesarias y estén respaldadas por la realidad.

Avoid phrases like “revolutionary,” “amazing,” “magic,” or “brutal,” unless they are genuinely necessary and backed by reality.

### 5. No inventar / Do not invent

Si no hay evidencia, no se escribe como si fuera hecho. El texto debe ser honesto con lo que se sabe y con lo que todavía no se ha verificado.

If there is no evidence, it should not be written as if it were fact. The text must be honest about what is known and what remains unverified.

### 6. No mezclar niveles / Do not mix levels

No se salta de la intuición a la ecuación sin explicación. La progresión tiene que ser técnica y pedagógica.

Do not jump from intuition to equations without explanation. The progression must be both technical and pedagogical.

---

## Reglas específicas para diagramas y ejemplos / Diagram and example rules

- Los diagramas deben ayudar a entender el flujo o la relación entre elementos.
- Si un diagrama explica un protocolo, debe mostrar un orden correcto y una dirección clara.
- Si se usa un ejemplo, debe ser un caso realista, no un escenario artificial.
- No se convierten los números en “estilo” si no aportan claridad.

- Diagrams must help explain flow or relationships between elements.
- If a diagram explains a protocol, it must show a correct sequence and a clear direction.
- If an example is used, it must be realistic, not artificial.
- Numbers should not be decorative; they should aid understanding.

---

## Reglas para referencias y fuentes / Source and reference rules

### Fuentes preferidas / Preferred sources

- estándares primarios;
- especificaciones de fabricantes;
- documentación técnica oficial;
- literatura revisada por pares;
- publicaciones de universidades o laboratorios reconocidos.

- primary standards;
- manufacturer specifications;
- official technical documentation;
- peer-reviewed literature;
- publications from recognized universities or laboratories.

### Fuentes que no se utilizan / Sources to avoid

- artículos de prensa sin análisis técnico;
- reseñas de consumo;
- blogposts de segunda mano;
- contenido generado para SEO;
- explicaciones que no citan la base técnica.

- press coverage without technical analysis;
- consumer reviews;
- secondhand blogposts;
- SEO-driven content;
- explanations that do not cite the underlying technical basis.

---

## Qué no se hace en este proyecto / What this project does not do

- no se comparan modelos comerciales;
- no se recomienda qué comprar;
- no se escriben “life hacks” ni consejos vagos;
- no se describen exploits ni procedimientos ofensivos;
- no se presentan opiniones sin pruebas;
- no se convierten los datos en narrativa sensacionalista.

- no product models are compared;
- no buying recommendations are given;
- no “life hacks” or vague advice;
- no exploit procedures or offensive steps;
- no opinions without evidence;
- no sensationalist storytelling around technical data.

---

## Checklist final / Final checklist

Antes de publicar un artículo, verifica:

- [ ] La pregunta es clara y concreta.
- [ ] El texto incluye una respuesta corta y una explicación útil.
- [ ] El modelo mental está presente y es transparente.
- [ ] El mecanismo técnico está explicado con rigor.
- [ ] Hay números con unidad cuando corresponden.
- [ ] Las referencias son verificables.
- [ ] El texto es breve, claro y directo.
- [ ] No hay marketing ni llantos de contenido.
- [ ] El artículo encaja en la estructura del repositorio.

Before publishing an article, verify:

- [ ] The question is clear and concrete.
- [ ] The text includes a short answer and a useful explanation.
- [ ] The mental model is present and transparent.
- [ ] The technical mechanism is explained with rigor.
- [ ] There are numbers with units where relevant.
- [ ] The references are verifiable.
- [ ] The text is brief, clear, and direct.
- [ ] There is no marketing or fluff.
- [ ] The article fits the repository structure.

---

## English

### Summary

This project favors precision over style games. A good article in HowThingsTalk is one that teaches the mechanism clearly, cites trustworthy sources, and speaks directly to the reader’s curiosity.

A good article is not a long essay; it is a precise explanation with the right level of depth.

[← Volver al índice general / Back to the main index](../docs/Index.md)
