---
title: "¿Cómo sabe el móvil que la web a la que se conecta es de verdad quien dice ser?"
section: "10_SeguridadYCriptografia"
language: "es"
status: "published"
order: 2
depth: "intermediate"
summary: "El servidor presenta un certificado: un documento público que une su nombre de dominio con una clave pública y que va firmado por una autoridad de certificación. El móvil sigue la cadena de firmas hasta una raíz que ya trae instalada, comprueba el nombre, las fechas y la revocación, y exige al servidor una firma hecha en ese momento con la clave privada. Solo quien tiene esa clave puede producirla."
prerequisites:
  - "Intercambio de claves de Diffie-Hellman (artículo 10/01)"
  - "Qué significa cifrar datos con una clave, a nivel general"
translation: "../../EN/10_SecurityAndCryptography/02-HowYourPhoneKnowsAWebsiteIsGenuine.md"
---

# ¿Cómo sabe el móvil que la web a la que se conecta es de verdad quien dice ser?

## Pregunta y encuadre

Escribes el nombre de tu banco en el navegador. Entre el móvil y el servidor hay un WiFi, un router y varios operadores. Cualquiera de ellos podría responder en lugar del banco. El [artículo sobre Diffie-Hellman](01-ComoAcuerdanDosAparatosUnaClaveSecreta.md) explicaba cómo dos aparatos acuerdan una clave secreta. También advertía de que ese intercambio no dice con quién estás hablando. Este artículo responde a esa parte: cómo comprueba el móvil que al otro lado está de verdad el dueño del dominio.

Cubre las firmas digitales a nivel conceptual, qué contiene un certificado, la cadena de confianza hasta las raíces que trae el sistema y las comprobaciones que hace el cliente en TLS 1.3. También explica cómo se vigila a las autoridades que emiten certificados: revocación y transparencia de certificados. No cubre las matemáticas internas de cada algoritmo de firma, ni la configuración de un servidor. No describe ataques contra sistemas concretos.

La profundidad es intermedia. Ayuda haber leído el artículo sobre Diffie-Hellman, aunque los conceptos necesarios se repasan aquí.

## Respuesta corta

El servidor enseña un certificado firmado por una autoridad en la que el móvil ya confía y, en el mismo momento, firma la conversación con la clave privada que ese certificado nombra; el móvil comprueba las dos firmas, el nombre y las fechas, y solo entonces sigue adelante.

### Resumen de 60 segundos

Una firma digital usa dos claves. La privada solo la tiene su dueño y sirve para firmar. La pública la puede tener cualquiera y sirve para comprobar firmas. Nadie puede fabricar una firma válida sin la clave privada.

Un certificado es un documento que dice «la clave pública X pertenece a `banco.example`». Lo firma una autoridad de certificación (CA) después de comprobar que quien lo pide controla ese dominio. A menudo la firma la pone una CA intermedia, cuyo propio certificado firma a su vez una CA raíz.

El móvil trae de fábrica una lista de raíces en las que confía. Cuando el servidor envía su certificado, el móvil sigue la cadena de firmas hasta una de esas raíces. Comprueba que el nombre coincide con el que escribiste, que no ha caducado y que no lo han anulado.

El certificado es público y cualquiera puede copiarlo. Por eso hay un último paso. El servidor firma un resumen de toda la conversación con su clave privada. Un impostor puede reenviar el certificado, pero no puede producir esa firma.

## Modelo mental

Piensa en un control de acceso que pide un carné. El carné lleva tu nombre y tu foto. Lo ha expedido un organismo que el vigilante conoce. El vigilante comprueba el sello del organismo, la fecha de caducidad y que la foto se parece a ti. Si todo cuadra, te deja pasar.

La analogía se rompe en tres puntos. Primero, el «carné» digital es público: cualquiera puede tener una copia exacta. Lo que hace las veces de la foto es una prueba en directo, una firma que solo puede hacer quien tiene la clave privada. Segundo, la CA no comprueba quién eres ni si eres honrado. Normalmente solo comprueba que controlas el dominio. Un sitio fraudulento puede tener un certificado perfectamente válido para su propio dominio. Tercero, el vigilante juzga a ojo, pero el móvil hace comprobaciones matemáticas exactas: una firma es válida o no lo es, sin término medio.

## Mecanismo técnico

### 1. Qué ocurre en el sistema

Todo empieza mucho antes de la conexión, cuando el dueño del dominio pide un certificado:

1. **Genera un par de claves.** La privada se queda en el servidor. La pública va en la solicitud.
2. **Demuestra que controla el dominio.** La CA le pide, por ejemplo, publicar un valor concreto en un registro DNS del dominio o en una ruta de su web. El protocolo ACME (RFC 8555) automatiza estos retos, llamados `dns-01` y `http-01` (apartados 8.4 y 8.3). Los métodos permitidos están en el apartado 3.2.2.4 de los requisitos básicos del CA/Browser Forum. Desde septiembre de 2017, la CA también debe consultar el registro CAA del dominio, donde el dueño puede indicar qué CA tiene permiso para emitir.
3. **La CA firma el certificado** y lo registra en varios registros públicos de transparencia (se explican más abajo).

Después, en cada conexión con TLS 1.3:

1. El móvil y el servidor hacen el intercambio de Diffie-Hellman. A partir de ahí, el resto del *handshake* va cifrado.
2. El servidor envía su certificado y los intermedios en el mensaje `Certificate`. Según RFC 9846 (apartado 4.5.1), el certificado propio debe ir primero y cada uno de los siguientes debería certificar al anterior. La raíz se puede omitir, porque el cliente ya la tiene.
3. El servidor envía `CertificateVerify`: una firma, hecha con su clave privada, sobre el resumen (*hash*) de todos los mensajes intercambiados hasta ese momento.
4. El móvil valida la cadena, comprueba el nombre y verifica la firma de `CertificateVerify`. Si algo falla, corta la conexión.

### 2. Qué hace cada componente

**La firma digital.** El firmante calcula un resumen de los datos y lo transforma con su clave privada. El verificador usa la clave pública para comprobar que la firma corresponde a esos datos exactos. Si cambia un solo bit, la comprobación falla. Su seguridad descansa en problemas matemáticos parecidos a los del intercambio de claves: el logaritmo discreto en curvas elípticas (ECDSA, Ed25519) o la factorización (RSA). Una firma Ed25519 mide 64 bytes (RFC 8032, apartado 5.1.6).

**El certificado.** Su formato es X.509 versión 3, perfilado en RFC 5280 (apartado 4.1). La parte firmada se llama `tbsCertificate` e incluye, entre otros campos:

- el número de serie;
- el emisor (*issuer*), es decir, la CA que firma;
- la validez: fecha de inicio y de fin;
- el sujeto y su clave pública (`subjectPublicKeyInfo`);
- las extensiones, como los nombres de dominio y si el certificado puede firmar otros certificados.

Detrás va el algoritmo de firma y la firma de la CA sobre todo lo anterior.

**Los nombres de dominio.** Van en la extensión `subjectAltName`, como entradas de tipo `dNSName` (RFC 5280, apartado 4.2.1.6). RFC 9525 fija cómo se comparan. El cliente usa el nombre que quería visitar, no el que le diga el servidor. Lo compara etiqueta a etiqueta, sin distinguir mayúsculas. El campo antiguo *Common Name* ya no vale para identificar un servicio (RFC 9525, apartado 2). Se admite un comodín, pero solo como etiqueta completa más a la izquierda y cubriendo una única etiqueta (apartado 6.3). Así, `*.banco.example` sirve para `web.banco.example`, pero no para `banco.example` ni para `a.web.banco.example`.

**La CA intermedia y la raíz.** La extensión `basicConstraints` dice si un certificado es de CA. Si no lo es, su clave no debe usarse para comprobar firmas de otros certificados (RFC 5280, apartado 4.2.1.9). Esto impide que el dueño de un certificado normal emita certificados para otros dominios. Las raíces suelen guardarse fuera de línea. Las intermedias hacen el trabajo diario, de modo que una intermedia comprometida se puede retirar sin cambiar la raíz.

**El almacén de raíces.** Es la lista de raíces en las que confía el sistema operativo o el navegador. Cada fabricante mantiene la suya con un programa propio. Chrome, por ejemplo, usa solo su Chrome Root Store para autenticar servidores TLS. La versión 1.8 de su política (febrero de 2026) retira cualquier raíz cuya clave se generó hace más de 15 años (apartado 1.3.1.2). La información sobre las CA se comparte en la base de datos común CCADB.

**La firma `CertificateVerify`.** Es la prueba de posesión. Según RFC 9846 (apartado 4.5.2), el servidor firma la concatenación de cuatro piezas: 64 bytes con el valor 0x20, la cadena de contexto `TLS 1.3, server CertificateVerify`, un byte 0 y el resumen de la conversación. El relleno y la cadena de contexto impiden reutilizar la firma en otro protocolo o en otro papel. El resumen incluye los valores de Diffie-Hellman de los dos lados. Por eso, si alguien en medio los hubiera cambiado, la firma no cuadraría. Si la verificación falla, el cliente debe cortar con una alerta `decrypt_error`.

### 3. Parámetros que importan

- **Comprobaciones por cada certificado de la cadena** (RFC 5280, apartado 6.1.3). La firma se verifica con la clave pública del certificado superior. El periodo de validez incluye la hora actual. El certificado no está revocado. El emisor coincide con el sujeto del certificado superior.
- **Validez máxima de un certificado de servidor.** Según la versión 2.3.1 de los requisitos básicos del CA/Browser Forum (octubre de 2026, apartado 1.2.2): 200 días desde el 15 de marzo de 2026, 100 días desde el 15 de marzo de 2027 y 47 días desde el 15 de marzo de 2029.
- **Reutilización de la validación del dominio.** Mismo calendario: 200 días, después 100 y, desde marzo de 2029, 10 días. Pasado ese plazo, la CA tiene que volver a comprobar que el solicitante controla el dominio.
- **Transparencia en Chrome.** Un certificado debe llevar sellos de registros públicos (SCT): de al menos 2 registros distintos si su vida es de 180 días o menos, y de 3 si es mayor. Al menos dos deben ser de operadores distintos. Si no los lleva, Chrome lo rechaza.
- **Transparencia en Apple.** Exige al menos dos SCT. Con sellos incrustados: 2 de registros distintos para 180 días o menos y 3 para 181 a 398 días.
- **Esquemas de firma en TLS 1.3** (RFC 9846, apartado 4.3.3). Entre otros, `ecdsa_secp256r1_sha256` (0x0403), `rsa_pss_rsae_sha256` (0x0804) y `ed25519` (0x0807). El cliente anuncia cuáles acepta en la extensión `signature_algorithms`.

### 4. Diagrama o flujo

Validación en una conexión TLS 1.3 típica, con una CA intermedia:

```text
ALMACÉN DE RAÍCES DEL MÓVIL (instalado antes de la conexión)
  [Raíz R]  clave pública de R
       |
       | firma
       v
  [Intermedia I]  sujeto: I · emisor: R · cA = verdadero
       |
       | firma
       v
  [Hoja H]  sujeto: banco.example · emisor: I · cA = falso
            clave pública del servidor · válida 200 días · 3 SCT

Móvil                                              Servidor (banco.example)
-----                                              ------------------------
ClientHello  [key_share, signature_algorithms] ------>
             <------ ServerHello  [key_share]
                     (desde aquí, todo cifrado)
             <------ Certificate  [H, I]
             <------ CertificateVerify  [firma con la clave privada de H
                                         sobre el resumen de la conversación]
             <------ Finished

1. ¿La firma de I la verifica la clave de R, que está en el almacén?
2. ¿La firma de H la verifica la clave de I?   ¿I es CA?
3. ¿Hoy está dentro de la validez de H e I?   ¿Ninguno revocado?
4. ¿"banco.example" está en el subjectAltName de H?
5. ¿H trae los SCT exigidos?
6. ¿CertificateVerify se verifica con la clave pública de H?

Todo sí -> Finished del móvil y datos de la aplicación.
Algún no -> alerta y conexión cortada.
```

## Límites, excepciones y casos reales

**El certificado prueba el control del dominio, no la honradez.** Un sitio fraudulento en `banco-seguro.example` puede obtener un certificado válido para ese nombre. El candado solo garantiza que hablas con quien controla el dominio que aparece en la barra. Fijarse en el nombre sigue siendo cosa del usuario.

**Cualquier CA de confianza puede emitir para cualquier dominio.** Si una sola de las CA del almacén se equivoca o la comprometen, podría emitir un certificado para un dominio ajeno. Hay tres defensas. El registro CAA reduce qué CA pueden emitir. La transparencia de certificados hace que cada emisión quede a la vista. Y los programas de raíces retiran a las CA que incumplen.

**La transparencia de certificados.** Un registro de transparencia es un árbol de Merkle al que solo se pueden añadir entradas (RFC 9162, apartado 4). Al recibir un certificado, el registro devuelve un SCT: la promesa de incluirlo en un plazo máximo fijado, el *Maximum Merge Delay*. El navegador exige esos sellos. Así, un certificado emitido a escondidas no funcionaría, y uno emitido en público puede detectarlo el dueño del dominio vigilando los registros. La transparencia no bloquea la emisión indebida; la hace visible.

**La revocación es la pieza más débil.** Si se roba una clave privada, la CA puede revocar el certificado. Avisar a todos los clientes es difícil. Las listas de revocación (CRL, RFC 5280, apartado 5) son listas firmadas de números de serie anulados. Pueden ser grandes. La consulta en línea OCSP, en cambio, revela a la CA qué sitio visitas. Let's Encrypt apagó su servicio OCSP el 6 de agosto de 2025 por ese motivo de privacidad y publica solo CRL. Algunos navegadores comprimen la revocación de toda la web en filtros que descargan periódicamente. Mozilla describió en 2020 cómo CRLite reducía unos 300 MB de datos de revocación a alrededor de 1 MB. Los certificados de vida corta son otra respuesta: un certificado de 47 días deja de valer pronto aunque nadie lo revoque.

**La hora del aparato importa.** La comprobación de validez usa el reloj del móvil. Si la fecha está muy desajustada, certificados correctos parecen caducados o aún no válidos.

**Raíces añadidas por la empresa o por el usuario.** Un dispositivo gestionado puede tener instaladas raíces de la empresa. Con ellas, un equipo de la red corporativa puede presentar certificados que el móvil acepta e inspeccionar el tráfico. Es legítimo en un aparato gestionado, pero cambia en quién confías. Las aplicaciones pueden aplicar reglas propias, como aceptar solo ciertas claves (no verificado de forma general).

**El ordenador cuántico también afecta a las firmas.** El algoritmo de Shor rompería ECDSA, Ed25519 y RSA igual que rompe Diffie-Hellman. El NIST publicó en agosto de 2024 un estándar de firma resistente, ML-DSA (FIPS 204). La urgencia es distinta. Un intercambio de claves débil permite descifrar más tarde el tráfico grabado hoy. Una firma solo tiene que resistir mientras dura la conexión. Por eso la migración del intercambio va por delante de la de los certificados.

## Consecuencias prácticas

El aviso de certificado no válido es la única señal visible de que esta cadena ha fallado. Puede ser un error inocente: un reloj desajustado, un certificado caducado o un portal cautivo de un WiFi público. Pero el móvil no puede distinguirlo de un intento de suplantación. Aceptarlo desactiva justo la parte que protege frente a intermediarios.

Para quien administra un servicio, la validez máxima de 200 días, que bajará a 47 en 2029, hace casi obligatorio renovar de forma automática. Un certificado olvidado deja el servicio inaccesible. También conviene publicar un registro CAA y vigilar los registros de transparencia para descubrir certificados emitidos a tu nombre sin tu permiso.

Para quien diseña aparatos, el almacén de raíces es una pieza viva. Un aparato que nunca se actualiza acumula raíces retiradas y le faltan las nuevas. Con el tiempo deja de poder validar sitios correctos.

## Preguntas frecuentes

### ¿El certificado cifra la conexión?

No. El certificado autentica. El cifrado usa claves que salen del intercambio de Diffie-Hellman. En TLS 1.3, la clave privada del certificado solo se usa para firmar.

### ¿Un certificado «de pago» es más seguro que uno gratuito?

No desde el punto de vista criptográfico. Las comprobaciones que hace el navegador son las mismas. Lo que cambia entre tipos de certificado es cuánto verifica la CA sobre la organización. El navegador no lo usa para decidir si la conexión es segura.

### Si el certificado es público, ¿por qué no puede usarlo un impostor?

Porque no basta con enseñarlo. El servidor tiene que firmar en ese momento un resumen de la conversación con la clave privada. El resumen incluye valores aleatorios nuevos de esa conexión, así que una firma antigua no sirve.

### ¿Qué pasa si roban la clave privada de un servidor?

El ladrón podría hacerse pasar por ese servidor hasta que el certificado se revoque y los clientes se enteren, o hasta que caduque. No podría descifrar conexiones pasadas, porque en TLS 1.3 las claves de sesión salen de secretos de Diffie-Hellman que se borran al terminar.

## Fuentes

- [RFC 5280 — Internet X.509 Public Key Infrastructure Certificate and CRL Profile (IETF, mayo de 2008)](https://www.rfc-editor.org/rfc/rfc5280.html). Campos del certificado (apartado 4.1), `subjectAltName` (4.2.1.6), `basicConstraints` (4.2.1.9), listas de revocación (5) y comprobaciones de cada certificado en la validación de la ruta (6.1.3).
- [RFC 9525 — Service Identity in TLS (IETF, noviembre de 2023)](https://www.rfc-editor.org/rfc/rfc9525.html). Prohibición de usar el *Common Name* (apartado 2), identificadores de referencia (6.1), reglas de comparación y comodines (6.3).
- [RFC 9846 — The Transport Layer Security (TLS) Protocol Version 1.3 (IETF, julio de 2026)](https://www.rfc-editor.org/rfc/rfc9846.html). Esquemas de firma (apartado 4.3.3), mensaje `Certificate` y orden de la cadena (4.5.1), construcción de `CertificateVerify` y alerta `decrypt_error` (4.5.2).
- [RFC 8555 — Automatic Certificate Management Environment (ACME) (IETF, marzo de 2019)](https://www.rfc-editor.org/rfc/rfc8555.html). Retos `http-01` (apartado 8.3) y `dns-01` (8.4) para demostrar el control de un dominio.
- [RFC 8032 — Edwards-Curve Digital Signature Algorithm (EdDSA) (IRTF, enero de 2017)](https://www.rfc-editor.org/rfc/rfc8032.html). Firma Ed25519 de 64 bytes (apartado 5.1.6).
- [RFC 9162 — Certificate Transparency Version 2.0 (IETF, diciembre de 2021)](https://www.rfc-editor.org/rfc/rfc9162.html). Registro como árbol de Merkle de solo añadir, SCT y *Maximum Merge Delay* (apartados 4 y 4.1).
- [CA/Browser Forum — Baseline Requirements for TLS Server Certificates, versión 2.3.1 (4 de octubre de 2026)](https://cabforum.org/working-groups/server/baseline-requirements/requirements/). Calendario de validez máxima y de reutilización de la validación (apartado 1.2.2), métodos de validación del dominio (3.2.2.4) y CAA obligatorio desde la versión 1.4.3 (septiembre de 2017).
- [Chrome Root Program Policy, versión 1.8 (5 de febrero de 2026)](https://googlechrome.github.io/chromerootprogram/crp/policy/). Uso exclusivo del Chrome Root Store y retirada de raíces con claves de más de 15 años (apartado 1.3.1.2).
- [Chrome — Certificate Transparency Policy](https://googlechrome.github.io/CertificateTransparency/ct_policy.html). Número de SCT según la vida del certificado y rechazo de los certificados que no cumplen.
- [Apple — Apple's Certificate Transparency policy (21 de abril de 2025)](https://support.apple.com/en-us/103214). Número de SCT exigidos según la vida del certificado.
- [CCADB — Common CA Database](https://www.ccadb.org/). Base de datos compartida por los programas de raíces.
- [Let's Encrypt — Ending OCSP Support in 2025 (5 de diciembre de 2024)](https://letsencrypt.org/2024/12/05/ending-ocsp/) y [OCSP Service Has Reached End of Life (6 de agosto de 2025)](https://letsencrypt.org/2025/08/06/ocsp-service-has-reached-end-of-life). Motivo de privacidad y paso a CRL.
- [Mozilla Security Blog — CRLite Part 1: All Web PKI Revocations Compressed (9 de enero de 2020)](https://blog.mozilla.org/security/2020/01/09/crlite-part-1-all-web-pki-revocations-compressed/). Compresión de unos 300 MB de revocaciones en alrededor de 1 MB.
- [NIST FIPS 204 — Module-Lattice-Based Digital Signature Standard (13 de agosto de 2024)](https://csrc.nist.gov/pubs/fips/204/final). Estándar de firma ML-DSA.
