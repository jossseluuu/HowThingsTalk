---
title: "¿Cómo genera la app de autenticación un código válido sin conectarse a internet?"
section: "10_SeguridadYCriptografia"
language: "es"
status: "published"
order: 3
depth: "intermediate"
summary: "Al activar la verificación en dos pasos, el servidor y el móvil guardan el mismo secreto. Cada 30 segundos, los dos calculan por su cuenta un HMAC del secreto y de la hora actual, y lo recortan a 6 cifras. No hace falta red: basta con que los dos relojes coincidan. El servidor acepta el código si sale lo mismo que en su cálculo."
prerequisites:
  - "Qué es una función hash, a nivel general"
  - "Qué es una clave secreta compartida (artículo 10/01, opcional)"
translation: "../../EN/10_SecurityAndCryptography/03-HowAuthenticatorAppsGenerateCodes.md"
---

# ¿Cómo genera la app de autenticación un código válido sin conectarse a internet?

## Pregunta y encuadre

Activas la verificación en dos pasos en una web. Escaneas un código QR con una app de autenticación. Desde entonces, la app muestra un número de 6 cifras que cambia cada 30 segundos. Funciona en modo avión y la web lo acepta. Este artículo explica cómo es posible: qué comparten el móvil y el servidor, qué cálculo hacen los dos y por qué el resultado coincide sin que se hablen.

Cubre el algoritmo TOTP (RFC 6238) y la base sobre la que se construye: HOTP (RFC 4226) y HMAC (RFC 2104). Explica cómo se reparte el secreto al escanear el QR y qué comprueba el servidor. No cubre los códigos que llegan por SMS, que siguen otro mecanismo, ni la configuración de ninguna app o web. Tampoco describe cómo atacar cuentas concretas.

La profundidad es intermedia. Ayuda saber qué es una función hash. El [artículo sobre Diffie-Hellman](01-ComoAcuerdanDosAparatosUnaClaveSecreta.md) explica otra forma de compartir un secreto, pero no es necesario para seguir este.

## Respuesta corta

El móvil y el servidor guardan una copia del mismo secreto y miran la misma hora; cada 30 segundos, los dos aplican al secreto y al número de intervalo un cálculo criptográfico idéntico, y el resultado recortado a 6 cifras es el código.

### Resumen de 60 segundos

El QR que escaneas contiene un secreto aleatorio. El servidor guarda otra copia. Es la única vez que el secreto viaja.

Para generar un código, la app cuenta cuántos intervalos de 30 segundos han pasado desde el 1 de enero de 1970. Ese número es igual en cualquier aparato con el reloj en hora. Luego calcula un HMAC: una especie de huella que mezcla el secreto con ese número. Sin el secreto no se puede predecir la huella, aunque se conozcan muchos códigos anteriores.

La huella mide 20 bytes. La app elige 4 de ellos según una regla fija y los convierte en un número de 6 cifras.

El servidor hace la misma cuenta con su copia del secreto. Si sale lo mismo, el código es válido. Suele aceptar también el intervalo anterior, para tolerar el retraso al teclear o un reloj algo desviado.

## Modelo mental

Imagina dos espías que se separan con una copia del mismo libro de claves. Cada página corresponde a una franja de tiempo. Para identificarse, uno dice la palabra de la página de esa hora. El otro abre su libro por la misma página y comprueba. No necesitan llamarse: les basta con el libro y un reloj.

La analogía se rompe en dos puntos. Primero, no hay libro con páginas escritas de antemano. Cada «página» se calcula en el momento a partir del secreto, así que caben miles de millones de franjas sin ocupar espacio. Segundo, en un libro, quien vea varias páginas no aprende nada de las demás. Aquí pasa lo mismo, pero no por azar: lo garantiza la función HMAC, que no deja deducir el secreto a partir de sus resultados.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

**Alta (una sola vez).**

1. El servidor genera un secreto aleatorio `K` para tu cuenta y lo guarda.
2. Te lo enseña dentro de un QR. El contenido suele ser una dirección `otpauth://`, un formato que formalizó Google y que casi todas las apps reconocen. Lleva el secreto en Base32 y, opcionalmente, el algoritmo, el número de cifras y el intervalo. Su forma es `otpauth://totp/Emisor:cuenta?secret=…&issuer=Emisor`.
3. La app guarda `K`. Para confirmar el alta, la web suele pedirte un primer código.

**Cada inicio de sesión.**

1. La app lee la hora del móvil en segundos Unix.
2. Calcula el número de intervalo `T` (RFC 6238, apartado 4.2):

   `T = floor((hora Unix − T0) / X)`, con `T0 = 0` y `X = 30 s` por defecto.

3. Calcula `HOTP(K, T)`: un HMAC del secreto con `T` como mensaje y un recorte a cifras decimales (RFC 4226, apartado 5).
4. Tú tecleas el código. El servidor repite los pasos 1 a 3 con su copia de `K` y compara.

### 2. Qué hace cada componente

**El secreto compartido `K`.** Es la única pieza que no es pública. RFC 4226 exige al menos 128 bits y recomienda 160 (apartado 4, requisito R6). RFC 6238 aconseja que mida lo mismo que la salida del HMAC: 160 bits con SHA-1 (apartado 5.1). La guía NIST SP 800-63B-4 pide una fuerza mínima de 112 bits (apartado 3.1.4). En Base32 cada carácter lleva 5 bits, así que un secreto de 160 bits ocupa 32 caracteres.

**El contador de tiempo `T`.** Es un entero de 8 bytes (RFC 4226, apartado 5.1). Con `X = 30 s`, a los 59 segundos del 1 de enero de 1970 vale 1 y a los 60 segundos vale 2 (RFC 6238, apartado 4.2). La norma obliga a admitir valores mayores de 32 bits, para que el cálculo siga funcionando después de 2038.

**HMAC.** Es un código de autenticación de mensajes construido con una función hash (RFC 2104, apartado 2). Pasa la clave y el mensaje dos veces por la función hash, mezclados con dos constantes fijas. El resultado depende de cada bit de la clave y del mensaje. Quien no conoce la clave no puede calcularlo. Tampoco puede deducir la clave a partir de resultados anteriores. TOTP usa HMAC-SHA-1 por defecto y admite HMAC-SHA-256 y HMAC-SHA-512 (RFC 6238, apartado 1.2).

**El truncado dinámico.** HMAC-SHA-1 produce 20 bytes. Hay que convertirlos en unas pocas cifras (RFC 4226, apartado 5.3):

1. Se toman los 4 bits bajos del último byte. Dan un desplazamiento entre 0 y 15.
2. Se leen 4 bytes a partir de esa posición.
3. Se pone a cero el bit más alto. Así el número queda en 31 bits y no hay dudas entre aritmética con signo o sin signo.
4. Se calcula el resto de dividir entre 10^cifras.

La norma exige al menos 6 cifras y admite 7 u 8.

**El verificador.** Además de comparar, tiene tres obligaciones. Debe rechazar un código ya usado: RFC 6238 prohíbe aceptar un segundo intento con el mismo código tras una validación correcta (apartado 5.2). Debe limitar los intentos fallidos (RFC 4226, apartado 7.3). Y debe tolerar algo de desfase de reloj sin abrir demasiado la ventana.

### 3. Parámetros que importan

- **Intervalo `X`.** 30 s por defecto y recomendado (RFC 6238, apartados 4.1 y 5.2). El formato `otpauth://` admite 15, 30 o 60 s, según la documentación de Yubico.
- **Cifras.** 6 como mínimo, 7 u 8 opcionales (RFC 4226, apartado 5.3). NIST permite recortar hasta 6 cifras decimales.
- **Ventana de aceptación.** RFC 6238 recomienda admitir como mucho un intervalo hacia atrás por el retraso de la red (apartado 5.2). Para el desfase de reloj, el apartado 6 permite aceptar varios intervalos. Con dos hacia atrás, el desfase máximo tolerado ronda los 89 s.
- **Vida del código según NIST.** No hay una cifra fija. Debe cubrir la deriva esperada del reloj en ambos sentidos, más el retraso de la red y el tiempo de teclear (SP 800-63B-4, apartado 3.1.4).
- **Límite de intentos.** NIST fija un máximo de 100 fallos consecutivos por autenticador y cuenta antes de desactivarlo. Es un techo, y se puede poner uno más bajo (apartado 3.2.2).
- **Probabilidad de acertar a ciegas.** RFC 4226 la estima como `s · v / 10^cifras`, donde `s` es el número de códigos aceptados a la vez y `v` el número de intentos (apartado 6). Con 6 cifras, 3 códigos aceptados y 100 intentos, sale 3 × 10⁻⁴: un 0,03 %. La cuenta es propia, aplicando esa fórmula a esos valores.

### 4. Diagrama o flujo

Ejemplo con los valores de prueba de las normas: el secreto es el texto ASCII `12345678901234567890` (20 bytes) y la hora Unix es 59 s. Los bytes intermedios se han recalculado con esos datos y coinciden con los códigos publicados en RFC 6238 (apéndice B) y RFC 4226 (apéndice D).

```text
ALTA (una vez)
Servidor --- QR: otpauth://totp/...?secret=<K en Base32> ---> App
   guarda K                                                    guarda K

CADA CÓDIGO (en los dos lados, sin comunicarse)
hora Unix = 59 s
T = floor(59 / 30) = 1   ->  00 00 00 00 00 00 00 01   (8 bytes)

HMAC-SHA-1(K, T) =
  75 a4 8a 19 d4 cb e1 00 64 4e 8a [c1 39 7e ea] 74 7a 2d 33 ab
  byte:  0 ...                   11  12 13 14            ...  19

último byte 0xab -> 4 bits bajos = 0xb = 11 -> desplazamiento 11
bytes 11-14     = 0xc1397eea
bit alto a cero = 0x41397eea = 1 094 287 082
mod 10^6        = 287082        (6 cifras)
mod 10^8        = 94287082      (8 cifras, valor de RFC 6238)

App muestra 287082 ---- tú lo tecleas ----> Servidor
                                            calcula lo mismo con su K
                                            para T = 1 (y quizá T = 0)
                                            ¿coincide y no se ha usado?  -> acceso
```

## Límites, excepciones y casos reales

**Todo depende del reloj.** Si la hora del móvil se desvía varios intervalos, los códigos dejan de coincidir aunque el secreto sea correcto. RFC 6238 propone que el servidor anote el desfase detectado en cada aparato y lo compense la próxima vez (apartado 6). Si el desfase crece demasiado, la norma pide recurrir a otra forma de autenticación.

**No resiste el phishing.** El código no sabe a qué web va destinado. Si una página falsa te lo pide, puede reenviarlo al instante a la web real y entrar dentro de los 30 s de validez. NIST lo dice de forma explícita: los autenticadores en los que la persona teclea el resultado no se consideran resistentes al phishing, porque ese resultado no queda ligado a la sesión concreta (SP 800-63B-4, apartado 3.2.5). Las llaves de seguridad y las passkeys sí lo son, porque firman un desafío que incluye el nombre del sitio.

**El servidor guarda el mismo secreto.** A diferencia de una contraseña bien almacenada, que se guarda como un resumen irreversible, el servidor necesita `K` en claro para calcular el código. Si alguien roba la base de datos de secretos, puede generar códigos válidos. Por eso el secreto debe protegerse tanto como cualquier clave.

**El QR es el momento delicado.** El secreto viaja una sola vez, pero viaja entero. Quien vea o fotografíe ese QR tiene una copia del generador. Lo mismo vale para las copias de seguridad de la app: si se guardan los secretos en la nube, la protección pasa a depender de esa cuenta.

**Perder el móvil es perder el generador.** Sin el secreto no hay forma de recalcular los códigos. Por eso las webs ofrecen códigos de recuperación de un solo uso al activar el segundo factor.

**No es lo mismo que un código por SMS.** En un SMS, el servidor genera el código y te lo envía por la red telefónica. NIST califica el uso de la red telefónica para esto como autenticador «restringido» (SP 800-63B-4, apartado 3.2.9). En TOTP, el código no viaja hasta que lo tecleas.

**El formato del QR no es un estándar del IETF.** El esquema `otpauth://` es un formato de facto. Hay un borrador individual en el IETF (versión 03, agosto de 2026) que intenta describirlo, pero no tiene rango de norma.

## Consecuencias prácticas

Para quien lo usa, la app funciona sin cobertura porque no necesita enviar nada: solo necesita el secreto y la hora. Si un día los códigos fallan todos, lo primero es comprobar que la hora del móvil se ajusta de forma automática. Y conviene tratar el código como lo que es, una contraseña que caduca en segundos: no se dicta por teléfono ni se escribe en una página a la que has llegado desde un enlace.

Para quien diseña un servicio, las piezas críticas son tres: generar `K` con suficiente aleatoriedad (128 bits o más), guardarlo protegido en el servidor y aplicar la regla de un solo uso con límite de intentos. La ventana de aceptación es un compromiso. Cada intervalo extra hace la vida más fácil al usuario con el reloj desviado, pero suma un código válido más a lo que puede acertar un atacante.

## Preguntas frecuentes

### ¿Es lo mismo TOTP que HOTP?

Casi. HOTP (RFC 4226) usa un contador que avanza cada vez que pides un código. TOTP (RFC 6238) sustituye ese contador por el número de intervalos de tiempo. Con HOTP, los dos lados pueden perder la cuenta si pulsas el botón sin usar el código. Con TOTP, se desincronizan si se desajusta el reloj.

### ¿Se puede deducir el secreto viendo muchos códigos?

No en la práctica. Cada código sale de un HMAC, y la seguridad de HMAC consiste justo en que sus resultados no revelan la clave. Además, el código conserva solo unos 20 bits de los 160 del HMAC. RFC 4226 concluye que el mejor ataque posible es probar códigos a ciegas (apartado 6).

### ¿Por qué el código dura 30 segundos y no 5?

Por comodidad. Hay que leer el código, cambiar de app y teclearlo, y los relojes no están perfectamente sincronizados. RFC 6238 recomienda 30 s como equilibrio entre seguridad y uso (apartado 5.2).

### ¿Es lo mismo que una passkey?

No. Una passkey usa un par de claves pública y privada. El servidor solo guarda la pública y el aparato firma un desafío ligado al nombre del sitio. Con TOTP, los dos guardan el mismo secreto y el código sirve en cualquier sitio que lo reciba durante su validez.

## Fuentes

- [RFC 6238 — TOTP: Time-Based One-Time Password Algorithm (IETF, mayo de 2011)](https://www.rfc-editor.org/rfc/rfc6238.html). Uso de HMAC-SHA-256 y SHA-512 (apartado 1.2), parámetros `T0` y `X` (4.1), cálculo de `T` y soporte más allá de 2038 (4.2), longitud de la clave (5.1), intervalo recomendado, ventana y un solo uso (5.2), resincronización (6) y valores de prueba (apéndice B).
- [Fe de erratas de RFC 6238, nº 2866 (verificada en noviembre de 2011)](https://www.rfc-editor.org/errata/rfc6238). Secretos de prueba de 20, 32 y 64 bytes según la función hash.
- [RFC 4226 — HOTP: An HMAC-Based One-Time Password Algorithm (IETF, diciembre de 2005)](https://www.rfc-editor.org/rfc/rfc4226). Longitud del secreto (apartado 4, R6), contador de 8 bytes (5.1), truncado dinámico y número de cifras (5.3), ejemplo de cálculo (5.4), análisis de seguridad (6), limitación de intentos (7.3) y valores de prueba (apéndice D).
- [RFC 2104 — HMAC: Keyed-Hashing for Message Authentication (IETF, febrero de 1997)](https://www.rfc-editor.org/rfc/rfc2104.html). Definición de HMAC (apartado 2).
- [NIST SP 800-63B-4 — Digital Identity Guidelines: Authentication and Authenticator Management (julio de 2025)](https://pages.nist.gov/800-63-4/sp800-63b/authenticators). Autenticadores OTP (apartados 3.1.4 y 3.1.5), límite de intentos (3.2.2), resistencia al phishing (3.2.5) y autenticadores restringidos (3.2.9). [Ficha de la publicación](https://csrc.nist.gov/pubs/sp/800/63/B/4/final).
- [Yubico — URI string format (documentación del SDK)](https://docs.yubico.com/yesdk/users-manual/application-oath/uri-string-format.html). Estructura de las direcciones `otpauth://`, parámetros y valores admitidos.
- [IETF — draft-linuxgemini-otpauth-uri, versión 03 (borrador individual, agosto de 2026)](https://datatracker.ietf.org/doc/draft-linuxgemini-otpauth-uri/). Intento de describir el esquema `otpauth://`; sin rango de norma.
