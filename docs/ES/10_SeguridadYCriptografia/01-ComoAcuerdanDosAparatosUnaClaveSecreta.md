---
title: "¿Cómo acuerdan dos aparatos una clave secreta hablando por un canal que cualquiera puede escuchar?"
section: "10_SeguridadYCriptografia"
language: "es"
status: "published"
order: 1
depth: "intermediate"
summary: "Con el intercambio de Diffie-Hellman, cada aparato elige un número secreto, publica un resultado derivado de él y combina el resultado del otro con su propio secreto. Los dos llegan a la misma clave sin enviarla nunca, porque deshacer la operación es inviable con los ordenadores actuales. Para que nadie se haga pasar por el otro extremo, el intercambio tiene que ir firmado."
prerequisites:
  - "Qué significa cifrar datos con una clave, a nivel general"
  - "Capas de una pila de protocolos (glosario)"
translation: "../../EN/10_SecurityAndCryptography/01-HowTwoDevicesAgreeOnASecretKey.md"
---

# ¿Cómo acuerdan dos aparatos una clave secreta hablando por un canal que cualquiera puede escuchar?

## Pregunta y encuadre

Cuando el móvil abre una web con HTTPS, el tráfico pasa por el WiFi de una cafetería, por el router de un operador y por varios centros de datos. Cualquiera de ellos puede copiar cada byte. Aun así, en pocos milisegundos el móvil y el servidor tienen una clave que solo conocen ellos. Nunca se habían comunicado antes y no se la han enviado. La pregunta es cómo es posible.

El artículo explica el intercambio de claves de Diffie-Hellman: la idea matemática, la versión con curvas elípticas que usan hoy casi todas las conexiones (X25519) y cómo lo emplea TLS 1.3. También explica por qué el intercambio solo no basta y hace falta una firma, y qué cambia ante un ordenador cuántico. No cubre cómo se cifran después los datos con esa clave, ni cómo funcionan los certificados por dentro. No describe ataques contra sistemas concretos.

La profundidad es intermedia. Basta saber que cifrar consiste en transformar datos con una clave y tener una idea de las capas de una pila de protocolos, que se definen en el [glosario](../../../guides/TaxonomyAndGlossary.md). La aritmética necesaria se explica sobre la marcha.

## Respuesta corta

Cada aparato mezcla un número secreto propio con un dato público, envía el resultado y repite la mezcla con lo que recibe del otro; los dos llegan al mismo valor, y quien solo escucha no puede deshacer la mezcla para obtenerlo.

### Resumen de 60 segundos

El truco se basa en una operación fácil de hacer y muy difícil de deshacer. Los dos extremos comparten un punto de partida público. Cada uno elige un número secreto al azar y aplica la operación al punto de partida tantas veces como indica su secreto. Envía el resultado al otro sin protegerlo.

Después, cada uno aplica su propio secreto al resultado que ha recibido. Como el orden no importa, los dos obtienen el mismo valor final. Ese valor nunca ha viajado por la red.

Un espía ve el punto de partida y los dos resultados públicos. Para llegar al valor final necesitaría uno de los secretos. Obtenerlo a partir de los resultados públicos es un problema que, con los tamaños actuales, no se sabe resolver en un tiempo razonable con ordenadores clásicos.

El intercambio, por sí solo, no dice con quién estás hablando. Alguien en medio podría hacer un intercambio con cada extremo por separado. Por eso TLS 1.3 añade una firma del servidor sobre toda la negociación. Y como los secretos se tiran al acabar cada conexión, robar más tarde la clave del servidor no permite descifrar el tráfico ya grabado.

## Modelo mental

La imagen clásica es la de la pintura. Dos personas acuerdan en público un color base, por ejemplo amarillo. Cada una añade en privado un color secreto: Ana añade rojo y Benito añade azul. Se intercambian las mezclas a la vista de todos. Después, cada una añade su color secreto a la mezcla que ha recibido. Las dos acaban con el mismo tono: amarillo, rojo y azul. Quien mire solo ha visto amarillo, naranja y verde, y separar una mezcla de pintura en sus colores es prácticamente imposible.

La analogía se rompe en tres puntos. Primero, la dificultad de separar pintura es física y evidente; la de deshacer la operación matemática es una suposición. Nadie ha demostrado que no exista un método rápido, aunque décadas de investigación no lo han encontrado. Además, un ordenador cuántico grande sí lo tendría. Segundo, en la pintura el resultado final es un color; en el protocolo es un número que todavía hay que transformar en claves de cifrado. Tercero, la pintura no protege contra alguien que se ponga en medio y finja ser Benito ante Ana. Esa protección la tiene que añadir otra pieza.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

La idea la publicaron Whitfield Diffie y Martin Hellman en 1976. Su versión usa aritmética modular: trabajar siempre con el resto de dividir entre un número primo `p`. Se elige un número público `g`. Cada parte elige un secreto y calcula `g` elevado a ese secreto, módulo `p`.

Un ejemplo con números de juguete, `p = 23` y `g = 5`:

1. Ana elige en secreto `a = 6` y calcula `A = 5^6 mod 23 = 8`. Envía 8.
2. Benito elige en secreto `b = 15` y calcula `B = 5^15 mod 23 = 19`. Envía 19.
3. Ana calcula `19^6 mod 23 = 2`.
4. Benito calcula `8^15 mod 23 = 2`.

Los dos llegan a 2 porque `(5^15)^6` y `(5^6)^15` son el mismo número: `5^(6·15)`. El espía ve 23, 5, 8 y 19. Para llegar a 2 necesita saber que 8 es 5 elevado a 6. Ese problema se llama **logaritmo discreto**. Con 23 se resuelve probando, pero con números de cientos de dígitos no se conoce forma práctica de resolverlo.

Hoy casi siempre se usa la misma idea sobre una **curva elíptica**. En lugar de elevar un número a una potencia, se «suma» un punto de la curva consigo mismo tantas veces como indica el secreto. Esa operación se llama multiplicación escalar. Deshacerla es el logaritmo discreto en la curva. Con curvas se alcanza la misma seguridad con números mucho más cortos.

### 2. Qué hace cada componente

**El grupo público.** Es el terreno de juego: un primo y un generador en la versión clásica, o una curva y un punto base en la elíptica. Está fijado por un estándar y es igual para todo el mundo. No es secreto.

**El secreto de cada parte.** Es un número aleatorio que no sale nunca del aparato. En X25519 son 32 bytes aleatorios, según RFC 7748 (apartado 6.1). Su calidad depende del generador de números aleatorios: un secreto predecible anula todo el esquema.

**El valor público.** Es el resultado de aplicar el secreto al punto base. Viaja sin cifrar. En TLS 1.3 va dentro de la extensión `key_share`.

**El secreto compartido.** Es el resultado final, igual en los dos extremos. No se usa directamente como clave. TLS 1.3 lo introduce en su programación de claves (*key schedule*), basada en la función de derivación HKDF. De ahí salen claves distintas para cada sentido y para cada fase.

**La firma.** El servidor firma la negociación con la clave privada que corresponde a su certificado. Es el mensaje `CertificateVerify`, que RFC 9846 describe como una firma sobre todo el *handshake*. Si alguien en medio cambia un valor público, la firma no cuadra y la conexión se corta.

### 3. Parámetros que importan

- **Curva X25519.** Definida en RFC 7748. Trabaja módulo el primo `2^255 − 19`, con punto base `u = 9`. Secreto, valor público y secreto compartido miden 32 bytes cada uno. El RFC la sitúa en el nivel de seguridad de unos 128 bits, «ligeramente por debajo» del nivel estándar de 128 bits (apartado 7).
- **Curva X448.** También en RFC 7748, para un nivel de unos 224 bits. Su valor público mide 56 bytes (RFC 9846, apartado 4.3.8.2).
- **Grupos con nombre en TLS 1.3.** Entre otros, `x25519` (código 0x001D) y `secp256r1` (0x0017), según RFC 9846 (apartado 4.3.7).
- **Comprobaciones del valor recibido.** En la versión clásica, cada parte debe comprobar que el valor del otro cumple `1 < Y < p − 1`. En las curvas NIST, que el punto está de verdad en la curva (RFC 9846, apartado 4.3.8). En X25519, RFC 7748 permite comprobar que el secreto compartido no sea todo ceros; RFC 10024 obliga a hacerlo en el modo híbrido.
- **Un solo uso.** RFC 9846 dice que clientes y servidores no deben reutilizar un valor de `key_share` en varias conexiones (apartado 4.3.8).
- **Tamaño en el modo híbrido poscuántico.** Con `X25519MLKEM768` (código 0x11EC, RFC 10024), el valor del cliente mide 1216 bytes (1184 de ML-KEM y 32 de X25519) y el del servidor, 1120 bytes. Frente a los 32 bytes de X25519 solo, son unas 38 veces más.

### 4. Diagrama o flujo

Intercambio en un *handshake* completo de TLS 1.3 con X25519, sin claves precompartidas:

```text
Cliente (móvil)                                      Servidor (web)
---------------                                      --------------
secreto c  (32 bytes al azar)                        secreto s  (32 bytes al azar)
C = c · P   (P = punto base, u = 9)                  S = s · P

ClientHello  [key_share: C] ----------------------->
                                                     K = s · C
             <----------------------- ServerHello  [key_share: S]
             <----------------------- Certificate  (clave pública del servidor)
             <----------------------- CertificateVerify  (firma sobre todo el handshake)
             <----------------------- Finished
K = c · S
comprueba certificado y firma
Finished  ------------------------------------------>

K = c·s·P en los dos extremos  ->  HKDF  ->  claves de tráfico
El espía ve P, C y S, pero no c, s ni K.
c y s se borran al terminar la conexión.
```

## Límites, excepciones y casos reales

**Sin firma no hay protección contra quien se pone en medio.** Un intermediario activo puede hacer un intercambio con el cliente y otro con el servidor. Cada extremo creería tener una clave con el otro, y el intermediario leería todo. Diffie-Hellman protege contra quien escucha, no contra quien suplanta. La autenticación viene del certificado y de la firma `CertificateVerify`. Si el cliente no comprueba el certificado, el intercambio pierde su valor.

**El secreto hacia delante depende de tirar los secretos.** RFC 8446 ya decía que TLS 1.3 eliminó los modos con RSA estático y Diffie-Hellman estático, y que todos sus intercambios de clave basados en clave pública dan secreto hacia delante (*forward secrecy*, apartado 1.2). Eso significa que robar después la clave del certificado no permite descifrar sesiones grabadas. La garantía solo se cumple si los secretos efímeros se borran de verdad. Por eso RFC 9846 prohíbe reutilizar los valores `key_share`. Aun así, para no romper la compatibilidad, exige aceptar al otro extremo aunque los reutilice.

**Los valores recibidos se comprueban.** Un valor público malformado puede forzar un secreto compartido débil o predecible. Por eso los estándares exigen las comprobaciones del apartado anterior. Es una de las razones por las que se usan curvas y formatos fijados por estándares, en lugar de parámetros elegidos por cada aplicación.

**La aleatoriedad es la base.** Si el generador de números aleatorios de un aparato produce secretos repetidos o adivinables, el resto da igual. El esquema supone que `c` y `s` son impredecibles.

**El ordenador cuántico.** El algoritmo de Shor, publicado en 1997, resuelve el logaritmo discreto en tiempo polinómico en un ordenador cuántico suficientemente grande. Proos y Zalka adaptaron el algoritmo a las curvas elípticas en 2003. No se conoce públicamente ninguna máquina capaz de hacerlo con los tamaños de X25519. El riesgo es que alguien grabe hoy tráfico cifrado y lo descifre cuando exista. La respuesta del IETF es el intercambio híbrido de RFC 10024 (agosto de 2026). Combina X25519 con ML-KEM-768, un mecanismo de encapsulado de claves estandarizado por el NIST en FIPS 203 (agosto de 2024). Según RFC 10024, la combinación es segura mientras al menos uno de los dos componentes lo siga siendo.

**ML-KEM no es Diffie-Hellman.** En un encapsulado de claves, el cliente publica una clave y el servidor le devuelve un secreto «encapsulado» con ella. El resultado práctico es el mismo, un secreto compartido sin enviarlo en claro, pero la estructura es distinta. El precio es el tamaño: más de 1 kB por sentido en lugar de 32 bytes.

**Un valor más que hay que adivinar.** El cliente envía su `key_share` antes de saber qué grupos acepta el servidor. Si el servidor no admite ninguno de los propuestos, pide otro con un `HelloRetryRequest`. RFC 9846 (apartado 4.3.8) señala que eso cuesta un viaje de ida y vuelta adicional.

## Consecuencias prácticas

Para quien usa el aparato, una red WiFi abierta o un operador curioso pueden ver con qué servidor se conecta el móvil y cuántos datos intercambia. No pueden ver el contenido de una conexión TLS bien autenticada. El candado del navegador significa precisamente eso: hubo un intercambio de claves y una firma que se pudo verificar.

El aviso de certificado no válido no es un detalle menor. Es el síntoma de que la parte que impide la suplantación ha fallado. Aceptarlo deja el intercambio de claves sin su protección frente a intermediarios.

Para quien diseña aparatos o servicios, la lección es no inventar. Las curvas, los formatos y las comprobaciones están fijados en estándares porque los fallos aparecen en los detalles: valores sin validar, secretos reutilizados o aleatoriedad pobre. También conviene prever espacio para el modo híbrido: un `ClientHello` con 1216 bytes extra de clave puede dejar de caber en un solo paquete de red.

## Preguntas frecuentes

### ¿Es lo mismo que cifrar con la clave pública del servidor?

No. En los modos antiguos con RSA estático, el cliente elegía la clave y la enviaba cifrada con la clave pública del certificado. Quien robara más tarde la clave privada podía descifrar todo lo grabado. En Diffie-Hellman efímero, la clave sale de dos secretos que se tiran al acabar. La clave del certificado solo se usa para firmar, no para proteger la clave de la sesión.

### ¿Diffie-Hellman cifra los datos?

No. Solo produce un secreto compartido. Los datos se cifran después con un algoritmo simétrico, como AES-GCM o ChaCha20-Poly1305, usando claves derivadas de ese secreto.

### ¿Por qué se usan curvas elípticas y no la versión con números primos?

Por tamaño y velocidad. Con curvas, la misma seguridad necesita números mucho más cortos: X25519 trabaja con valores de 32 bytes para unos 128 bits de seguridad. La versión clásica sigue existiendo en TLS 1.3 como grupos de campo finito, pero con valores mucho mayores.

### ¿Un ordenador cuántico rompería también el cifrado AES de los datos?

No de la misma forma. El algoritmo de Shor ataca el logaritmo discreto y la factorización, que son la base del intercambio y de las firmas. El cifrado simétrico no depende de esos problemas. Por eso el cambio urgente se está haciendo en el intercambio de claves.

## Fuentes

- [Diffie, W. y Hellman, M. E. — New Directions in Cryptography. IEEE Transactions on Information Theory, vol. IT-22, n.º 6, noviembre de 1976, p. 644 y siguientes](https://ee.stanford.edu/~hellman/publications/24.pdf). Propuesta original del intercambio con `Y = α^X mod q` y clave común `α^(X_i·X_j) mod q`, basada en la dificultad del logaritmo en un cuerpo finito (sección III).
- [RFC 7748 — Elliptic Curves for Security (IRTF, enero de 2016)](https://www.rfc-editor.org/rfc/rfc7748.html). Primo `2^255 − 19` y punto base `u = 9` (apartado 4.1), curva448 (4.2), entradas y salidas de 32 bytes de X25519 (5), protocolo de intercambio y comprobación opcional del valor todo ceros (6.1), nivel de seguridad (7).
- [RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3 (IETF, agosto de 2018)](https://www.rfc-editor.org/rfc/rfc8446.html). Eliminación de RSA y Diffie-Hellman estáticos y secreto hacia delante en todos los intercambios basados en clave pública (apartado 1.2).
- [RFC 9846 — The Transport Layer Security (TLS) Protocol Version 1.3 (IETF, julio de 2026)](https://www.rfc-editor.org/rfc/rfc9846.html). Sustituye a RFC 8446. Descripción del *handshake* y de `CertificateVerify` (apartado 2), grupos con nombre (4.3.7), extensión `key_share`, prohibición de reutilizarla, coste de un viaje adicional y comprobaciones de los valores recibidos (4.3.8, 4.3.8.1 y 4.3.8.2).
- [RFC 10024 — Post-Quantum Traditional (PQ/T) Hybrid Key Agreement Mechanisms for TLS 1.3 (IETF, agosto de 2026)](https://datatracker.ietf.org/doc/draft-ietf-tls-ecdhe-mlkem/). Definición de intercambio híbrido (apartado 3), tamaños del `key_share` de cliente y servidor (4.1 y 4.2), comprobación obligatoria del valor todo ceros (4.3) y código de `X25519MLKEM768` (7.1).
- [NIST FIPS 203 — Module-Lattice-Based Key-Encapsulation Mechanism Standard (13 de agosto de 2024)](https://csrc.nist.gov/pubs/fips/203/final). Estándar de ML-KEM.
- [Shor, P. W. — Polynomial-Time Algorithms for Prime Factorization and Discrete Logarithms on a Quantum Computer. SIAM Journal on Computing, 26(5), 1484-1509, 1997](https://doi.org/10.1137/S0097539795293172). Algoritmo cuántico para el logaritmo discreto.
- [Proos, J. y Zalka, C. — Shor's discrete logarithm quantum algorithm for elliptic curves. Quantum Information and Computation, 3(4), 317-344, 2003](https://arxiv.org/abs/quant-ph/0301141). Adaptación del algoritmo de Shor a las curvas elípticas.
