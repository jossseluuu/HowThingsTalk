---
title: "¿Cómo puede una web reconocer tu navegador sin usar cookies?"
section: "11_PrivacidadYHuella"
language: "es"
status: "published"
order: 4
depth: "intermediate"
summary: "Cada navegador revela al conectarse decenas de detalles corrientes: versión, idioma, zona horaria, tamaño de pantalla, tipos de letra o cómo dibuja un texto su tarjeta gráfica. Ninguno identifica a nadie por separado, pero juntos forman una combinación que suele ser rara o única. Una web puede calcular esa huella en cada visita y reconocer el navegador sin guardar nada en él. Por eso borrar las cookies no la elimina."
prerequisites:
  - "Qué ve tu operador aunque uses HTTPS (artículo 11·02)"
  - "Por qué el móvil cambia su dirección MAC (artículo 11·01)"
translation: "../../EN/11_PrivacyAndFootprint/04-HowWebsitesRecognizeYouWithoutCookies.md"
---

# ¿Cómo puede una web reconocer tu navegador sin usar cookies?

## Pregunta y encuadre

Una cookie es una etiqueta que la web deja en tu navegador para reconocerlo en la siguiente visita. Si la borras, la web pierde esa pista. Pero hay otra forma de reconocer un navegador que no guarda nada en él. La pregunta es concreta: ¿cómo puede una web saber que eres el mismo visitante sin dejar ninguna marca en tu aparato?

El artículo explica la huella digital del navegador (*browser fingerprinting*): qué datos la forman, cómo se mide cuánto identifican y por qué algunas defensas funcionan y otras empeoran las cosas. No es una guía para construir un sistema de rastreo ni compara navegadores. Tampoco trata el rastreo por dirección IP, que ya se ve en el [artículo 11·02](../11_PrivacidadYHuella/02-QueVeTuOperadorAunqueUsesHTTPS.md).

La profundidad es intermedia. Ayuda saber qué viaja en una petición web ([artículo 11·02](../11_PrivacidadYHuella/02-QueVeTuOperadorAunqueUsesHTTPS.md)) y la idea de identificador estable del [artículo 11·01](../11_PrivacidadYHuella/01-PorQueElMovilCambiaSuDireccionMAC.md). La única herramienta matemática es el bit, y se explica sobre la marcha.

## Respuesta corta

Tu navegador no lleva un número de serie, pero la suma de sus rasgos corrientes funciona como si lo llevara: una combinación que suele compartir con muy pocos navegadores, que se puede volver a medir en cada visita y que no se borra con las cookies.

### Resumen de 60 segundos

Para mostrar bien una página, el navegador cuenta cosas de sí mismo. Algunas las envía siempre en la petición: nombre y versión, sistema operativo e idiomas preferidos. Otras las puede consultar el código JavaScript de la página: tamaño de pantalla, zona horaria, tipos de letra disponibles o tarjeta gráfica.

Cada rasgo, por separado, lo comparten millones de personas. Juntos, la cosa cambia. En el primer estudio a gran escala, de 2010, el 83,6 % de 470 161 navegadores tenía una combinación que no compartía con ningún otro de la muestra.

La técnica más potente consiste en pedir al navegador que dibuje un texto o una figura fuera de la vista y leer los píxeles resultantes. Distintos sistemas, tipos de letra y tarjetas gráficas producen diferencias mínimas, invisibles para el ojo pero medibles.

La web resume todo en un código y lo compara con los que ya ha visto. No guarda nada en tu aparato, así que borrar cookies no sirve. Lo que sí reduce la huella es que muchos navegadores respondan exactamente lo mismo.

## Modelo mental

Piensa en un portero de discoteca sin lista de nombres. No te pide el carné, pero se fija en detalles: abrigo verde, zapatillas rojas, gafas redondas, mochila con una chapa. Ninguno es raro por sí solo. La combinación sí lo es. La semana siguiente te reconoce sin haberte apuntado en ningún sitio.

Si te cambias de abrigo, puede dudar. Pero si todo lo demás coincide, probablemente acierte. Y si todos los clientes llevaran el mismo uniforme, no tendría nada a lo que agarrarse.

La analogía se rompe en dos puntos. Primero, el portero solo ve lo que llevas a la vista. Una web puede hacer preguntas activas a tu navegador, como pedirle que dibuje algo, y el resultado depende de piezas internas que nadie ve, como la tarjeta gráfica. Segundo, el portero compara de memoria y se equivoca con facilidad. Una web compara códigos exactos contra millones de visitas guardadas, y puede medir con precisión lo raro que es cada rasgo.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

La W3C, el organismo que estandariza la web, distingue tres formas de reconocer un navegador por sus rasgos (nota *Mitigating Browser Fingerprinting in Web Specifications*, apartado 3):

1. **Huella pasiva.** Usa solo lo que llega en la propia petición, sin ejecutar código: cabeceras HTTP como `User-Agent` o `Accept-Language`, el orden de esas cabeceras y datos de red como la dirección IP.
2. **Huella activa.** La página ejecuta JavaScript para preguntar más cosas: tamaño de ventana, tipos de letra, rendimiento o cómo se dibuja un gráfico. La W3C señala una diferencia clave: este tipo es potencialmente detectable desde el propio aparato, porque el código se ejecuta ahí.
3. **Estado parecido a una cookie.** La web guarda algo en el aparato por una vía distinta de las cookies y lo recupera después. No es huella en sentido estricto, porque sí deja marca.

Una visita típica sigue esta secuencia:

1. **Llega la petición.** El servidor anota las cabeceras y la dirección IP.
2. **Se ejecuta el script.** Recoge decenas de propiedades del navegador y del sistema.
3. **Se dibuja fuera de la vista.** El script pide al navegador que pinte un texto o una escena 3D en un lienzo invisible y lee los píxeles.
4. **Se resume.** Todos los valores se combinan en un código corto con una función resumen (*hash*).
5. **Se compara.** El servidor busca ese código, o uno muy parecido, entre los que ya ha visto.

### 2. Qué hace cada componente

**Las cabeceras HTTP.** Viajan en cada petición. La cabecera `User-Agent` decía tradicionalmente el navegador con su versión exacta, el sistema operativo y, en móviles, a veces el modelo. Por eso Chrome la recortó a partir de 2022. Según la documentación de Google, ahora envía una cadena reducida como esta:

```text
Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.0.0 Mobile Safari/537.36
```

La versión de Android, el modelo (`K`) y la versión menor del navegador son valores fijos, iguales para todos. Los datos detallados siguen disponibles mediante *Client Hints*, pero el servidor tiene que pedirlos con la cabecera `Accept-CH`. Por defecto solo se envían tres: marca y versión principal, si es un móvil y el nombre del sistema operativo. El modelo del aparato (`Sec-CH-UA-Model`) es un dato de «alta entropía» que hay que solicitar.

**Las propiedades consultables con JavaScript.** Resolución y profundidad de color de la pantalla, zona horaria, idiomas, número de núcleos del procesador, si las cookies están activadas o la lista de complementos instalados. En 2010, la lista de complementos y la de tipos de letra eran los rasgos que más identificaban (Eckersley, apartado 4).

**El dibujo en un lienzo (*canvas fingerprinting*).** HTML5 permite dibujar en un elemento `<canvas>` y leer después sus píxeles. Mowery y Shacham lo demostraron en 2012: dibujaron una frase con una tipografía conocida y una escena 3D con WebGL, y resumieron los píxeles en un código. Las diferencias venían del sistema operativo, la versión del navegador, la tarjeta gráfica, los tipos de letra instalados, el ajuste fino de las letras a los píxeles (*subpixel hinting*) y el suavizado de bordes. Con solo 300 participantes, una frase en Arial produjo 50 dibujos distintos. Las imágenes 3D parecían idénticas a simple vista, pero dieron 50 versiones distintas en píxeles.

**El servidor que compara.** Guarda las huellas vistas y busca coincidencias. Como la huella cambia con el tiempo, no basta con buscar el código exacto: hay que enlazar una huella nueva con la antigua cuando solo ha cambiado un rasgo, como la versión del navegador tras una actualización.

### 3. Parámetros que importan

- **La medida es el bit.** Cada bit de información divide por dos el grupo de navegadores candidatos. Un rasgo que comparte la mitad de la población aporta 1 bit. Uno que comparte 1 de cada 1000 aporta unos 10 bits, porque 2¹⁰ = 1024. La W3C lo resume así: un bit más suele preocupar poco, pero unos treinta y pocos bits bastan para distinguir a cada persona del planeta. Por comparación, 2³³ son unos 8600 millones.
- **Los bits no se suman sin más.** Los rasgos están relacionados entre sí: el sistema operativo condiciona los tipos de letra, y la versión del navegador condiciona sus capacidades. Eckersley advierte que la información combinada es menor que la suma de cada rasgo por separado.
- **Lo que mide cada rasgo.** En los tres grandes estudios recogidos por la revisión de Laperdrix y otros (tabla 3), la entropía de algunos rasgos fue:

  | Rasgo | Panopticlick (2010) | AmIUnique (2016) | Hiding in the Crowd (2018) |
  |---|---|---|---|
  | Lista de complementos | 15,4 bits | 11,1 bits | 9,5 bits |
  | Lista de tipos de letra | 13,9 bits | 8,4 bits | 6,9 bits |
  | User-Agent | 10,0 bits | 9,8 bits | 7,2 bits |
  | Dibujo en lienzo | no medido | 8,3 bits | 8,5 bits |
  | Pantalla y color | 4,8 bits | 4,9 bits | 4,8 bits |
  | Zona horaria | 3,0 bits | 3,3 bits | 0,2 bits |

  La zona horaria apenas aporta en el estudio de 2018 porque casi todos los visitantes eran de Francia, con la misma zona horaria. La entropía depende de quién forma la muestra.
- **Cuántos navegadores son únicos.** Panopticlick (2010): 83,6 % de 470 161 navegadores, y 94,2 % entre los que tenían Flash o Java. AmIUnique (2016): 89,4 % de 118 934, y un 81 % en móviles. Hiding in the Crowd (2018), con 2 067 942 huellas de uno de los quince sitios más visitados de Francia: 33,6 % en total, 35,7 % en ordenadores y 18,5 % en móviles.
- **Estabilidad.** La huella cambia. En 2010, el 37,4 % de 8833 visitantes repetidos mostró algún cambio entre visitas separadas más de 24 horas. Aun así, un método simple de enlace acertó el 99,1 % de las veces que se atrevió a dar una respuesta, con un 0,86 % de falsos positivos (Eckersley, apartado 5.2). El sistema FP-Stalker, con una huella cada tres días, siguió a cada aparato una media de 51,8 días, y a un 26 % durante más de 100 días (revisión de Laperdrix, apartado 3.2.2).

### 4. Diagrama o flujo

```text
Navegador                                        Servidor de la web
---------                                        ------------------
Petición HTTP ───────────────────────────────►  Cabeceras: User-Agent,
                                                 Accept-Language, orden...
                                                 + dirección IP        (pasiva)

◄─────────────────────────────── Página + script

Script en el navegador:
  pantalla 1920x1080, color 24 bits
  zona horaria Europe/Madrid
  idiomas es-ES, en
  tipos de letra disponibles
  dibujo en lienzo ─► píxeles ─► código
  tarjeta gráfica (WebGL)
        │
        ▼
  resumen (hash) ─────────────────────────────►  ¿Visto antes?   (activa)
                                                   ├─ igual ───► mismo navegador
                                                   ├─ cambia 1 rasgo ► probable
                                                   └─ nuevo ───► alta en la base
```

## Límites, excepciones y casos reales

**Único en una muestra no es único en el mundo.** Panopticlick y AmIUnique recogían visitas de personas interesadas en la privacidad, que suelen tener configuraciones poco comunes. Eckersley lo advirtió y no extrapoló sus cifras a toda la web. Con una muestra general de dos millones de huellas, la proporción de únicos bajó a un tercio. La revisión de Laperdrix atribuye la diferencia al tipo de público y al tamaño de la muestra.

**Los móviles se parecen más entre sí.** Un mismo modelo de teléfono sale de fábrica con el mismo hardware, los mismos tipos de letra y la misma pantalla. Por eso, en el estudio de 2018, solo el 18,5 % de los móviles era único. La W3C recoge esta observación: los móviles tienen grupos de anonimato bastante mayores que los ordenadores.

**Los complementos desaparecieron, el dibujo no.** En 2010, la lista de complementos era el rasgo más revelador. Los navegadores actuales ya no admiten Flash ni Java, y ese rasgo pierde peso con el tiempo. El dibujo en lienzo y la información de la tarjeta gráfica ocuparon su lugar: en 2018, el lienzo fue el rasgo con más entropía.

**La huella pasiva se está reduciendo.** El recorte del `User-Agent` y el paso a *Client Hints* hacen que los detalles finos ya no viajen solos: el servidor tiene que pedirlos. Eso no los oculta, pero los vuelve visibles. La W3C recomienda este enfoque: que el servidor tenga que solicitar los datos para que la recogida se pueda detectar (buena práctica 8).

**Disfrazarse puede delatar.** Un complemento que cambia el `User-Agent` para decir «Linux» mientras JavaScript sigue diciendo «Windows» crea una combinación que casi nadie tiene. La revisión de Laperdrix (apartado 4.1.1) documenta que esas incoherencias hacen al usuario más fácil de reconocer, no menos. Ser raro es justo lo que la huella busca.

**Hay dos estrategias de defensa.** Una es la uniformidad: que todos los usuarios respondan igual. Tor Browser bloquea el dibujo en lienzo y WebGL, quita complementos e incluye su propio juego de tipos de letra. Firefox ofrece un modo parecido desde la versión 59. La otra es la aleatorización: añadir un ruido pequeño al dibujo o a otros valores para que la huella cambie en cada sesión. La uniformidad falla si el usuario se sale del grupo, por ejemplo cambiando ajustes. El ruido falla si se puede detectar o promediar. La W3C prefiere estandarizar o recortar valores antes que aleatorizarlos.

**No hay solución completa.** La W3C considera inverosímil eliminar la huella del todo con medios técnicos desplegados a gran escala. Habla de mitigaciones, no de soluciones.

## Consecuencias prácticas

Para quien navega, la conclusión principal es que borrar cookies o usar una ventana privada no basta para no ser reconocido. Lo que reduce la huella es parecerse a mucha gente: usar un navegador común en su configuración por defecto, o uno diseñado para que todos sus usuarios respondan igual. Cada ajuste o complemento poco habitual aporta bits.

La huella tampoco depende de la red. Una VPN cambia la dirección IP, pero no cambia los tipos de letra ni cómo dibuja la tarjeta gráfica.

No todo uso es rastreo. La revisión de Laperdrix (apartado 5) recoge su uso para detectar fraude y bots: si alguien entra en tu cuenta desde un navegador con una huella nunca vista, el sistema puede pedir una verificación extra. La misma técnica sirve para proteger y para seguir.

Para quien diseña especificaciones o navegadores, la W3C propone reglas concretas. Cada nueva función debe exponer solo la información que necesita. Si una API da el nivel de batería como 0,81234567, basta con 0,8 o con una categoría como «baja» o «cargando». Los datos detallados deben pedirse de forma explícita para que la recogida sea visible.

## Preguntas frecuentes

### ¿Es lo mismo que una cookie?

No. Una cookie es un dato que la web guarda en tu navegador y recupera después. La huella no guarda nada: se vuelve a calcular en cada visita a partir de lo que el navegador ya es. Por eso no aparece en la lista de datos de sitios y no se borra con ellos.

### ¿Es lo mismo que la dirección MAC?

No. La dirección MAC identifica la tarjeta de red ante el punto de acceso WiFi y no sale de la red local, como explica el [artículo 11·01](../11_PrivacidadYHuella/01-PorQueElMovilCambiaSuDireccionMAC.md). La huella del navegador se forma con rasgos del software y del hardware que la web puede ver desde cualquier lugar de internet.

### ¿El modo incógnito cambia mi huella?

Apenas. Empieza sin cookies y no guarda historial, pero el navegador, el sistema, la pantalla y la tarjeta gráfica son los mismos. Algunos navegadores aplican protecciones extra en ese modo (no verificado navegador por navegador).

### ¿Puede una web saber mi nombre con la huella?

No por sí sola. La huella permite reconocer el mismo navegador en visitas distintas o en webs distintas que compartan el sistema de rastreo. El nombre aparece cuando, en alguna de esas visitas, inicias sesión o das un dato personal. A partir de ahí, lo que se reconoció sin nombre puede quedar unido a él.

## Fuentes

- [W3C — Mitigating Browser Fingerprinting in Web Specifications (nota de grupo, Privacy Working Group, 25 de septiembre de 2025)](https://www.w3.org/TR/fingerprinting-guidance/). Definición, huella pasiva, activa y de tipo cookie (apartado 3), niveles de éxito de las mitigaciones (4.1), entropía y «treinta y pocos bits» (5), buenas prácticas de estandarización, recorte de valores y solicitud explícita (6.1 a 6.4), e inviabilidad de eliminarla del todo (1.3).
- [Eckersley, P. — How Unique Is Your Web Browser? (Privacy Enhancing Technologies Symposium, 2010)](https://gwern.net/doc/www/coveryourtracks.eff.org/f56d1c28d08db1a7c3b0eeba1a51a40fede53f57.pdf). Copia en gwern.net del PDF que publicó la EFF. 470 161 huellas, 83,6 % y 94,2 % únicas, 18,1 bits, atributos medidos (tabla 1), cambios entre visitas y algoritmo de enlace (apartado 5.2).
- [Mowery, K. y Shacham, H. — Pixel Perfect: Fingerprinting Canvas in HTML5 (W2SP, IEEE, 2012)](https://www.ieee-security.org/TC/W2SP/2012/papers/w2sp12-final4.pdf). Método del dibujo en lienzo, causas de las diferencias, 300 participantes y 50 dibujos distintos de una frase en Arial.
- [Laperdrix, P., Bielova, N., Baudry, B. y Avoine, G. — Browser Fingerprinting: A Survey (ACM Transactions on the Web, 2020; versión arXiv)](https://arxiv.org/pdf/1905.01051). Cifras comparadas de Panopticlick, AmIUnique y Hiding in the Crowd (tablas 2 y 3), FP-Stalker (apartado 3.2.2), defensas por uniformidad, aleatorización y bloqueo, problema de las incoherencias (apartado 4) y usos contra el fraude (apartado 5).
- [Gómez-Boix, A., Laperdrix, P. y Baudry, B. — Hiding in the Crowd: an Analysis of the Effectiveness of Browser Fingerprinting at Large Scale (The Web Conference, 2018)](https://hal.archives-ouvertes.fr/hal-01718234). Estudio original de las 2 067 942 huellas; las cifras de este artículo se toman de la revisión anterior.
- [Google — Privacy Sandbox: User-Agent reduction](https://privacysandbox.google.com/protections/user-agent). Cadena `User-Agent` reducida, tres *Client Hints* enviados por defecto, solicitud de datos de alta entropía con `Accept-CH` y motivación frente a la huella pasiva.
