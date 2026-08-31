---
title_es: "Roadmap"
title_en: "Roadmap"
section: "docs"
updated: 2026-08-31
---

<div align="center">

# Hoja de ruta · Roadmap

**🇪🇸 Español** · [🇬🇧 English](#-english)

</div>

---

## Estado · Status

<sub>Tablas compartidas por las dos versiones. Son fechas, rutas y números: se leen igual en cualquier idioma y se actualizan en un solo sitio.<br>
<i>Shared by both language versions. Dates, paths and numbers read the same either way, and are updated in exactly one place.</i></sub>

**Publicado · Published:** 1 artículo de 107 con esqueleto creado · *1 article of 107 skeletons created*

| Sección · Section | Esqueletos · Skeletons | Escritos · Written |
|---|---|---|
| 00 · Foundations | 8 | 0 |
| 01 · WiFi | 9 | 0 |
| 02 · CellularNetworks | 10 | 0 |
| 03 · ShortRange | 8 | 0 |
| 04 · DataTransfer | 8 | 0 |
| 05 · Positioning | 10 | 1 ✅ |
| 06 · Storage | 8 | 0 |
| 07 · AudioAndPeripherals | 8 | 0 |
| 08 · PowerAndBatteries | 7 | 0 |
| 09 · IoTAndSmartHome | 9 | 0 |
| 10 · SecurityAndCryptography | 8 | 0 |
| 11 · PrivacyAndFootprint | 8 | 0 |
| 99 · Reference | 6 | 1 (borrador · draft) |

### 🔨 En curso · In progress

<sub>Máximo tres a la vez. Si hay ocho abiertos, no hay ninguno. · <i>Three at most. Eight open means none.</i></sub>

| Artículo · Article | Sección · Section | Bloqueo · Blocker |
|---|---|---|
| _(vacío · empty)_ | | |

### ✅ Publicado · Published

| Fecha · Date | Artículo · Article | Sección · Section |
|---|---|---|
| 2026-08-31 | [Cómo te localiza el WiFi sin que te conectes a nada](05-Positioning/01-WiFiPositioning.md) | 05 · Positioning |

<sub>Esta tabla es la que hace visible que el proyecto avanza cuando llevas tres semanas sin publicar. No la borres nunca. · <i>This table is what shows the project is moving during a three-week dry spell. Never delete it.</i></sub>

---

## Qué es este documento

Documento vivo. Aquí está **lo que se está escribiendo, lo que está en cola, lo que hay
que investigar antes de poder escribirlo, y lo que se ha decidido no hacer**.

Lo que ya existe como archivo tiene su propio campo `status` en la cabecera. Este
documento es para lo demás: las ideas que todavía no tienen archivo, y la prioridad
entre las que sí.

### Leyenda

| Marca | Significa |
|---|---|
| 🔨 | En curso ahora mismo |
| 📋 | En cola, decidido, pendiente de turno |
| 🔍 | Hay que investigar antes de poder escribirlo |
| 💭 | Idea suelta, sin decidir si da para artículo |
| ✅ | Publicado |
| ❌ | Descartado, con motivo |

Prioridad: **A** (columna vertebral, sin esto el repositorio cojea) · **B** (buen
artículo, sin urgencia) · **C** (curiosidad, se escribe cuando apetezca).

### 📋 Los tres siguientes

Escribir en este orden. Están elegidos para que el repositorio tenga esqueleto propio
cuanto antes: uno corrige un error de vocabulario que aparece en todas partes, otro es
prerrequisito de media biblioteca, y el tercero cierra el arco del artículo ya publicado.

1. **A** · `05-Positioning/02-TrilaterationVsTriangulation.md` — el error más repetido de todo el campo, y ya está enlazado desde el artículo publicado.
2. **A** · `00-Foundations/03-DecibelsAndPower.md` — sin entender el dBm negativo, media docena de artículos futuros no se sostienen.
3. **A** · `01-WiFi/02-BeaconsAndProbes.md` — completa el mecanismo que el artículo de localización solo esboza.

---

## Propuestas por sección

Artículos **nuevos**, además de los 107 esqueletos ya creados. Cada uno lleva la
pregunta que responde, porque si no se puede formular como pregunta concreta,
probablemente no es un artículo.

### 00 · Foundations

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `09-WhyRadioPassesWalls.md` | La radio y la luz son lo mismo. ¿Por qué una atraviesa una pared y la otra no? |
| A | `10-TheISMBands.md` | ¿Por qué WiFi, Bluetooth, microondas y mandos de garaje comparten los mismos 2,4 GHz? Porque son las bandas que nadie quiso. |
| B | `11-FresnelZone.md` | Ver la antena con los ojos no basta: la señal necesita un elipsoide despejado alrededor de la línea recta. |
| B | `12-DopplerEffect.md` | Por qué la red móvil tiene que corregir tu velocidad, y qué pasa en un AVE a 300 km/h. |
| B | `13-DutyCycling.md` | Cómo un sensor dura diez años con una pila de botón: estando apagado el 99,9% del tiempo. |
| C | `14-TheMicrowaveMyth.md` | No, el microondas no calienta por "resonancia del agua". La explicación real es más interesante. |
| C | `15-WhyLatencyHasAFloor.md` | La velocidad de la luz como límite duro: por qué un ping a Australia nunca bajará de ~100 ms. |

### 01 · WiFi

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `10-WhyHotelWiFiIsBad.md` | CSMA/CA y la tragedia de los comunes: cien personas compartiendo un medio que solo permite hablar a uno a la vez. |
| A | `11-AdvertisedVsRealSpeed.md` | Por qué un router "de 1200 Mbps" nunca te da 1200. PHY rate, overhead y half-duplex. |
| B | `12-ChannelsOneSixEleven.md` | Por qué en 2,4 GHz solo se usan tres canales de los trece disponibles. |
| B | `13-TheMicrowaveInterference.md` | Por qué el WiFi se cae cuando calientas la comida, y por qué solo en 2,4 GHz. |
| B | `14-ManagementFramesAndPMF.md` | Las tramas de gestión iban sin autenticar durante veinte años. Qué permitía eso y qué lo arregló. |
| B | `15-HiddenSSIDs.md` | Ocultar el nombre de la red no oculta nada, y encima empeora la privacidad de quien se conecta. |
| C | `16-WiFiSensing.md` | 802.11bf: usar las alteraciones de la señal para detectar movimiento y respiración a través de paredes. |
| C | `17-TheRouterIsThreeThings.md` | Router, switch, punto de acceso y módem son cuatro aparatos distintos dentro de la misma caja. |

### 02 · CellularNetworks

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `11-Paging.md` | Antes de que suene tu teléfono, la red tiene que encontrarlo. Cómo lo hace sin saber exactamente dónde está. |
| A | `12-WhyNoSignalAtConcerts.md` | La cobertura es capacidad, no potencia. Por qué con cuatro rayas no entra la llamada. |
| A | `13-EmergencyWithoutSIM.md` | Por qué el 112 funciona sin tarjeta, sin saldo y con el teléfono bloqueado. |
| B | `14-WhyBadSignalDrainsBattery.md` | El teléfono sube la potencia de transmisión hasta 200 veces cuando le cuesta llegar a la antena. |
| B | `15-FakeBaseStations.md` | Por qué el diseño de 2G permitía que una antena falsa se hiciera pasar por la red, y qué introdujo 5G para cerrarlo. Conceptual: mecanismo y defensa. |
| B | `16-The5GLabelLie.md` | Cuándo el icono 5G es 5G de verdad, cuándo es NSA sobre 4G y cuándo es puro marketing. |
| B | `17-The3GShutdown.md` | Qué se rompió al apagar el 3G: ascensores, alarmas, coches y lectores de tarjetas. |
| C | `18-MMWaveLimits.md` | Por qué el 5G más rápido no atraviesa ni una ventana. |
| C | `19-TowerDensity.md` | Cuántas antenas hacen falta para cubrir un país, y por qué en el campo hay menos velocidad pero más alcance. |

### 03 · ShortRange

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `09-FrequencyHopping.md` | El salto de frecuencia lo patentó una actriz de Hollywood en 1942 para torpedos. Hoy es el corazón de Bluetooth. |
| A | `10-WhyBluetoothCutsOut.md` | Tu cuerpo es agua, y el agua absorbe 2,4 GHz. Por eso el auricular del lado contrario al bolsillo se corta. |
| A | `11-RelayAttacksAndUWB.md` | Por qué las llaves de coche sin contacto eran vulnerables a la retransmisión de señal, y cómo la medición de tiempo de vuelo lo resuelve. Conceptual. |
| B | `12-ContactlessPayment.md` | Por qué no te cobran dos veces al pasar la tarjeta, y qué pasa si hay dos en la cartera. |
| B | `13-TrackerTagsAndStalking.md` | El problema de diseño de los rastreadores diminutos y las contramedidas que se añadieron después. |
| B | `14-WhyPairingFails.md` | El emparejamiento es un protocolo con estado en los dos extremos. Por qué "olvidar dispositivo" arregla el 90% de los casos. |
| C | `15-TheBluetoothName.md` | Un rey vikingo del siglo X, dos runas superpuestas y por qué el logo es su monograma. |
| C | `16-InfraredNostalgia.md` | El IrDA, el mando de la tele y por qué la luz infrarroja perdió contra la radio. |

### 04 · DataTransfer

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `09-WhyAirDropIsFast.md` | Bluetooth negocia, WiFi transporta. La diferencia entre 2 Mbps y 200. |
| A | `10-ManySmallFilesVsOneBig.md` | Por qué 4000 archivos de 1 MB tardan más que uno de 4 GB. El coste por operación. |
| B | `11-WhyWhatsAppRuinsVideo.md` | Recompresión con pérdida, dos veces, más redimensionado. Qué queda del original. |
| B | `12-ContactHashesInAirDrop.md` | Cómo sabe tu teléfono que quien te envía algo está en tu agenda sin revelar la agenda. Y por qué el método tuvo problemas. |
| B | `13-SyncConflicts.md` | Dos aparatos editan el mismo archivo sin conexión. ¿Quién gana, y quién decide? |
| C | `14-QRCodesAsTransport.md` | El código QR como canal de datos unidireccional: cuántos bytes caben y cómo sobrevive a estar arrugado. |
| C | `15-SneakernetMath.md` | Una furgoneta llena de discos duros sigue teniendo más ancho de banda que la fibra. La cuenta, en serio. |

### 05 · Positioning

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `11-RelativityInGPS.md` | Los satélites corrigen relatividad especial y general a diario. Sin esa corrección, el GPS se desviaría 10 km al día. |
| A | `12-WhyTheBlueDotJumps.md` | El salto al salir del metro: qué fuente estaba mandando y cuál toma el relevo. |
| B | `13-SelectiveAvailability.md` | Hasta el año 2000, el GPS civil llevaba error metido a propósito. Quién lo quitó y por qué. |
| B | `14-COCOMLimits.md` | Todo receptor GPS deja de funcionar por encima de 18 km de altura y 1900 km/h. Para que no sirva de guía de misiles. |
| B | `15-Geofencing.md` | Cómo sabe una app que has entrado en una zona sin tenerte vigilado todo el rato. |
| B | `16-FitnessAppsAndBases.md` | En 2018, un mapa de calor de rutas de running reveló bases militares secretas. Qué falló conceptualmente. |
| C | `17-AltitudeMeasurement.md` | Por qué el móvil sabe en qué piso estás: el barómetro, no el GPS. |
| C | `18-DeadReckoning.md` | Navegar sin ninguna referencia externa, como los submarinos y los aviones antes del GPS. |

### 06 · Storage

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `09-WhereTheGigabytesGo.md` | Por qué un móvil "de 128 GB" tiene 112 libres. Base 10 contra base 2, más el sistema operativo. |
| A | `10-WhyFullDrivesAreSlow.md` | Un SSD al 95% pierde rendimiento de forma brusca. La culpa es de cómo funciona el borrado por bloques. |
| B | `11-DataRetentionUnpowered.md` | Cuánto duran tus fotos en un disco desenchufado en un cajón. La respuesta incomoda. |
| B | `12-BitRotAndECC.md` | Los rayos cósmicos voltean bits en la memoria. Con qué frecuencia, y qué lo corrige. |
| B | `13-QuickFormatIsALie.md` | El formateo rápido no toca los datos: solo tira el índice. |
| B | `14-ImageCompression.md` | JPEG, HEIC y AV1: cómo se tira el 90% de la información sin que el ojo lo note. |
| C | `15-TheWriteAmplification.md` | Escribir 1 KB puede obligar al SSD a escribir 512. Por qué, y qué hace TRIM al respecto. |
| C | `16-StorageMediaLifespan.md` | Papel, cinta, disco, SSD, M-DISC: qué sobrevive cien años y qué no llega a diez. |

### 07 · AudioAndPeripherals

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `09-WhyMicKillsAudioQuality.md` | Al activar el micrófono en Bluetooth, la música pasa a sonar a radio AM. El motivo es un perfil de 2003. |
| A | `10-LatencyBudget.md` | De dónde salen exactamente los 200 ms de retardo: captura, códec, radio, buffer, decodificación. |
| B | `11-WhyANCFailsWithVoices.md` | La cancelación activa mata el ruido del avión y no puede con la conversación de al lado. |
| B | `12-OcclusionEffect.md` | Por qué tu propia voz suena rara con los auriculares puestos, y qué hace el modo transparencia. |
| B | `13-InEarDetection.md` | Cómo sabe el auricular que te lo has quitado: óptico, capacitivo o acelerómetro. |
| B | `14-WhyTWSLagMore.md` | Dos auriculares independientes tienen que sincronizarse entre sí. Eso cuesta milisegundos. |
| C | `15-LosslessOverBluetooth.md` | Por qué "audio sin pérdidas por Bluetooth" es casi siempre falso, y qué cambia LC3. |
| C | `16-PollingRates.md` | 1000 Hz en un ratón gaming: qué significa y a partir de dónde deja de notarse. |

### 08 · PowerAndBatteries

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `08-WhyPhonesDieInCold.md` | Al 10% y −5 °C el móvil se apaga de golpe. La química se ralentiza, no se agota. |
| A | `09-WhyChargingSlowsAt80.md` | La carga rápida no es rápida al final. Corriente constante contra voltaje constante. |
| B | `10-mAhIsMisleading.md` | Comparar mAh entre aparatos con voltajes distintos no significa nada. La unidad correcta es el vatio-hora. |
| B | `11-BatteriesOnPlanes.md` | Por qué en cabina sí y en bodega no. La fuga térmica y por qué hay que verla. |
| B | `12-GaNChargers.md` | Por qué un cargador moderno de 65 W cabe en un puño y el de 2015 pesaba medio kilo. |
| B | `13-PhantomDrain.md` | Un aparato apagado sigue consumiendo. Cuánto, y adónde va. |
| C | `14-USBCIsAMess.md` | Dos cables idénticos que hacen cosas distintas. Qué hay dentro y por qué el logo importa. |
| C | `15-CalibrationMyths.md` | Descargar al 0% para "calibrar" era cierto con níquel y es falso con litio desde hace veinte años. |

### 09 · IoTAndSmartHome

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `10-WhyBulbsNeedInternet.md` | Encender una luz a dos metros pasa por un servidor a 2000 km. Por qué se diseñó así. |
| A | `11-WhenTheServerDies.md` | Casos reales de aparatos convertidos en ladrillos por decisión del fabricante. Qué los previene. |
| B | `12-MiraiAndDefaults.md` | En 2016, cámaras con contraseña de fábrica tumbaron media internet. La lección estructural. |
| B | `13-WhyMatterTookSoLong.md` | Diez años de guerra de estándares y por qué el acuerdo tardó tanto en llegar. |
| B | `14-TenYearBattery.md` | Un sensor de puerta con una CR2032 durante una década. La ingeniería de estar dormido. |
| B | `15-LocalVsCloud.md` | Domótica que funciona sin internet: qué se gana, qué se pierde, qué cuesta montarla. |
| C | `16-TheDashButton.md` | Un botón de plástico con WiFi que pedía detergente. Cómo funcionaba por dentro. |
| C | `17-SmartTVTelemetry.md` | Qué reporta una tele conectada, con qué frecuencia y a quién. |

### 10 · SecurityAndCryptography

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `09-WhatTLSDoesNotProtect.md` | El candado dice que nadie escucha por el camino. No dice que el sitio sea legítimo, ni honesto. |
| A | `10-HowTOTPWorks.md` | Un código de seis dígitos que cambia cada 30 segundos, sin conexión ni comunicación. La magia es un reloj compartido. |
| A | `11-Passkeys.md` | Por qué una clave que nunca sale de tu aparato es mejor que cualquier contraseña. |
| B | `12-WhyFingerprintsArentImages.md` | El sensor no guarda tu huella: guarda un modelo del que no se puede reconstruir. |
| B | `13-RemoteWipeIsKeyDeletion.md` | Borrar 256 GB en un segundo es imposible. Borrar la clave que los descifra, no. |
| B | `14-Entropy.md` | De dónde saca números aleatorios un aparato recién encendido, cuando aún no ha pasado nada. |
| B | `15-PasswordStorage.md` | Qué guarda un servicio cuando guarda tu contraseña, y qué significa que una filtración sea grave o no. |
| C | `16-PostQuantumMigration.md` | "Recolecta ahora, descifra después": por qué la migración ya empezó aunque el ordenador cuántico no exista. |

### 11 · PrivacyAndFootprint

| P | Archivo propuesto | La pregunta |
|---|---|---|
| A | `09-WhatAPhotoKnows.md` | Coordenadas, hora, modelo, orientación y número de serie. El caso del fugitivo localizado por una foto de revista. |
| A | `10-IncognitoDoesNotHide.md` | Qué oculta exactamente el modo privado y de quién. La lista es más corta de lo que la gente cree. |
| B | `11-BrowserFingerprinting.md` | Sin cookies y sin identificador, tu navegador sigue siendo único entre millones. Por qué. |
| B | `12-WhyAppsWantBluetooth.md` | Una app de linterna pide Bluetooth. No es para conectarse a nada: es para saber dónde estás. |
| B | `13-UltrasonicBeacons.md` | Sonidos que tu oído no capta y el micrófono sí, usados para vincular tu móvil con tu televisión. |
| B | `14-SensorSideChannels.md` | Lo que el acelerómetro puede deducir sin permiso de micrófono ni de cámara. Estado de la investigación. |
| C | `15-TheDataBrokerChain.md` | De una app gratuita a un fichero comercial: los saltos que da un dato. |
| C | `16-WhatYouCanActuallyDo.md` | Cierre de la sección: las cinco medidas que cambian algo, y las diez que no. |

### 99 · Reference

| P | Archivo propuesto | Qué es |
|---|---|---|
| A | `07-QuickAnswers.md` | Índice de preguntas frecuentes con enlace directo al artículo que la responde. La puerta de entrada real para mucha gente. |
| B | `08-MythBusting.md` | Tabla de mitos con veredicto y enlace. Muy compartible. |
| B | `09-TimelineOfWireless.md` | Cronología: 1895 Marconi → 1947 celular → 1971 Aloha → 1997 802.11 → hoy. |
| C | `10-RangeComparison.md` | Tabla comparada: alcance, velocidad, consumo y latencia de cada tecnología del repositorio. |

---

## 🔍 Por investigar

Preguntas que interesan pero que **todavía no se pueden escribir** porque falta
documentación fiable, hace falta medir, o no está claro que la respuesta pública sea
completa. Antes de convertir cualquiera en artículo, hay que resolver el bloqueo.

| Pregunta | Qué falta | Dónde mirar |
|---|---|---|
| ¿Cuántos BSSID hay realmente catalogados hoy? | Ninguna plataforma publica cifras actuales | Papers académicos sobre WPS, datos de WiGLE |
| ¿Cuánta precisión da UWB en condiciones reales, no de catálogo? | Mediciones independientes | Publicaciones del FiRa Consortium, papers de ranging |
| ¿Qué margen real de degradación tiene una batería a 500 ciclos? | Los fabricantes dan cifras optimistas | Estudios de laboratorio independientes |
| ¿Qué cambia exactamente 5G SA frente a NSA en identificadores? | Especificación densa, mucha desinformación | 3GPP TS 33.501 |
| ¿Sigue siendo `_nomap` respetado por todas las plataformas? | Nunca se documentó formalmente | Verificar en documentación actual de cada una |
| ¿Cuál es el estado real del audio Bluetooth sin pérdidas? | Marketing contradictorio | Especificación LE Audio / LC3plus |
| ¿Qué precisión tiene el barómetro para detectar plantas? | Depende del calibrado y del clima | Documentación de sensores, pruebas propias |
| ¿Qué se puede deducir del patrón de consumo eléctrico de una casa? | Investigación dispersa | Papers sobre NILM (desagregación de carga) |

## 💭 Dudas sueltas

El colchón. Cosas que se me ocurren y que no sé si dan para artículo, para párrafo
dentro de otro, o para nada.

- ¿Por qué el WiFi de los aviones es tan malo y tan caro? (satélite vs aire-tierra)
- ¿Qué pasa técnicamente en el modo avión de un reloj sin conexión propia?
- ¿Cómo funciona el "toca para pagar" en un reloj sin batería suficiente?
- ¿Por qué algunos coches detectan la llave dentro y otros no?
- ¿Qué hace realmente el "modo de ahorro de datos"?
- ¿Cómo se coordina el tráfico de un estadio lleno? (small cells, DAS)
- ¿Cuánta energía consume una transferencia de 1 GB, de verdad?
- ¿Qué es el "ruido" de fondo de una cámara y por qué las fotos nocturnas son peores?
- ¿Por qué el eSIM no se puede copiar de un móvil a otro?
- ¿Cómo sabe una báscula inteligente tu porcentaje de grasa? (bioimpedancia, y su fiabilidad)

---

## ❌ Descartado

Con el motivo, para no volver a evaluarlo dentro de seis meses.

| Idea | Motivo |
|---|---|
| Guías paso a paso de auditoría de redes | Fuera del alcance definido en `_meta/StyleGuide.md`: se explica el mecanismo, no la receta |
| Herramientas concretas para desanonimizar dispositivos | Igual que la anterior. El repositorio sirve para entender la propia exposición |
| Reseñas y comparativas de productos | Envejecen en meses y el repositorio se convertiría en otra cosa |
| Noticias y novedades del sector | Un repositorio de explicaciones no compite con un feed |
| Tutoriales de configuración (montar una VPN, instalar Home Assistant) | Ya existen mil, y bien hechos. Aquí se explica **por qué** funciona, no cómo instalarlo |
| Sección de programación / código | Diluye la tesis. Si hace falta código, va incrustado en el artículo que lo necesite |
| Traducción de los artículos al inglés | No ahora. Primero contenido. La estructura ya lo permite si algún día compensa |

---

## Cómo usar este archivo

1. **Al empezar algo**, muévelo a 🔨 En curso. Máximo tres.
2. **Al terminar**, quítalo de aquí y añádelo a ✅ Publicado con fecha y enlace.
3. **Al descartar algo**, muévelo a ❌ con el motivo escrito. El motivo es la parte útil.
4. **Al ocurrírsete algo**, va a 💭 sin pensarlo mucho. Ya se filtrará.
5. **Revisa las prioridades** cada mes o dos. Lo que lleva un año en B probablemente sea C.

Regla de higiene: si una entrada de 🔍 lleva más de seis meses bloqueada por falta de
fuentes, o se baja a 💭 o se descarta. Un bloqueo permanente no es una tarea, es un
adorno.

> **Al editar:** las tablas de estado están arriba y son compartidas. Todo lo demás
> existe en dos idiomas. Si tocas una propuesta, tócala en los dos sitios o quedará
> desincronizada.

---

<div align="center">

## 🇬🇧 English

</div>

<details>
<summary><b>Read the full English version</b></summary>

<br>

> **Status tables are shared.** *In progress*, *Published* and the per-section counts
> live [at the top of this file](#estado--status) in a single bilingual copy, because
> they change weekly and duplicating them guarantees they will disagree. Everything
> below exists in both languages.

### What this document is

A living document. It holds **what is being written, what is queued, what needs research
before it can be written, and what has been deliberately ruled out**.

Anything that already exists as a file carries its own `status` field in the front
matter. This document covers the rest: ideas with no file yet, and the priority order
among the ones that do have a file.

### Legend

| Mark | Meaning |
|---|---|
| 🔨 | In progress right now |
| 📋 | Queued, decided, waiting its turn |
| 🔍 | Needs research before it can be written |
| 💭 | Loose idea, undecided whether it makes an article |
| ✅ | Published |
| ❌ | Discarded, with the reason |

Priority: **A** (backbone — without it the repository limps) · **B** (good article, no
rush) · **C** (curiosity, write it when you feel like it).

### 📋 The next three

Write these in order. They are chosen so the repository grows a spine as fast as
possible: one fixes a vocabulary error that appears everywhere, one is a prerequisite
for half the library, and one closes the arc of the article already published.

1. **A** · `05-Positioning/02-TrilaterationVsTriangulation.md` — the most repeated error in the whole field, and already linked from the published article.
2. **A** · `00-Foundations/03-DecibelsAndPower.md` — without understanding negative dBm, half a dozen future articles collapse.
3. **A** · `01-WiFi/02-BeaconsAndProbes.md` — completes the mechanism the positioning article only sketches.

---

### Proposals by section

**New** articles, on top of the 107 skeletons already created. Each carries the question
it answers, because if it cannot be phrased as one concrete question, it is probably not
an article.

#### 00 · Foundations

| P | Proposed file | The question |
|---|---|---|
| A | `09-WhyRadioPassesWalls.md` | Radio and light are the same thing. Why does one go through a wall and the other does not? |
| A | `10-TheISMBands.md` | Why do WiFi, Bluetooth, microwave ovens and garage remotes share the same 2.4 GHz? Because nobody else wanted those bands. |
| B | `11-FresnelZone.md` | Seeing the antenna with your eyes is not enough: the signal needs a clear ellipsoid around the straight line. |
| B | `12-DopplerEffect.md` | Why the cellular network has to correct for your speed, and what happens on a 300 km/h train. |
| B | `13-DutyCycling.md` | How a sensor lasts ten years on a coin cell: by being switched off 99.9% of the time. |
| C | `14-TheMicrowaveMyth.md` | No, microwave ovens do not heat by "water resonance". The real explanation is more interesting. |
| C | `15-WhyLatencyHasAFloor.md` | The speed of light as a hard limit: why a ping to Australia will never drop below ~100 ms. |

#### 01 · WiFi

| P | Proposed file | The question |
|---|---|---|
| A | `10-WhyHotelWiFiIsBad.md` | CSMA/CA and the tragedy of the commons: a hundred people sharing a medium where only one may speak at a time. |
| A | `11-AdvertisedVsRealSpeed.md` | Why a "1200 Mbps" router never gives you 1200. PHY rate, overhead and half-duplex. |
| B | `12-ChannelsOneSixEleven.md` | Why only three of the thirteen 2.4 GHz channels are ever used. |
| B | `13-TheMicrowaveInterference.md` | Why WiFi drops when you heat up lunch, and why only on 2.4 GHz. |
| B | `14-ManagementFramesAndPMF.md` | Management frames went unauthenticated for twenty years. What that allowed, and what fixed it. |
| B | `15-HiddenSSIDs.md` | Hiding the network name hides nothing, and makes privacy worse for whoever connects. |
| C | `16-WiFiSensing.md` | 802.11bf: using signal disturbance to detect motion and breathing through walls. |
| C | `17-TheRouterIsThreeThings.md` | Router, switch, access point and modem are four different devices inside one box. |

#### 02 · CellularNetworks

| P | Proposed file | The question |
|---|---|---|
| A | `11-Paging.md` | Before your phone rings, the network has to find it. How it does that without knowing exactly where it is. |
| A | `12-WhyNoSignalAtConcerts.md` | Coverage is capacity, not power. Why four bars still will not connect the call. |
| A | `13-EmergencyWithoutSIM.md` | Why emergency numbers work with no SIM, no credit and a locked phone. |
| B | `14-WhyBadSignalDrainsBattery.md` | The phone raises transmit power up to 200-fold when the tower is hard to reach. |
| B | `15-FakeBaseStations.md` | Why the 2G design let a fake tower impersonate the network, and what 5G introduced to close it. Conceptual: mechanism and defence. |
| B | `16-The5GLabelLie.md` | When the 5G icon is real 5G, when it is NSA riding on 4G, and when it is pure marketing. |
| B | `17-The3GShutdown.md` | What broke when 3G was switched off: lifts, alarms, cars and card readers. |
| C | `18-MMWaveLimits.md` | Why the fastest 5G cannot get through a window. |
| C | `19-TowerDensity.md` | How many towers it takes to cover a country, and why rural areas get less speed but more range. |

#### 03 · ShortRange

| P | Proposed file | The question |
|---|---|---|
| A | `09-FrequencyHopping.md` | Frequency hopping was patented by a Hollywood actress in 1942, for torpedoes. Today it is the heart of Bluetooth. |
| A | `10-WhyBluetoothCutsOut.md` | Your body is water, and water absorbs 2.4 GHz. That is why the earbud on the far side from your pocket drops out. |
| A | `11-RelayAttacksAndUWB.md` | Why keyless car entry was vulnerable to signal relaying, and how time-of-flight measurement solves it. Conceptual. |
| B | `12-ContactlessPayment.md` | Why you are not charged twice when you tap, and what happens with two cards in the wallet. |
| B | `13-TrackerTagsAndStalking.md` | The design problem of tiny trackers and the countermeasures added afterwards. |
| B | `14-WhyPairingFails.md` | Pairing is a stateful protocol at both ends. Why "forget this device" fixes 90% of cases. |
| C | `15-TheBluetoothName.md` | A tenth-century Viking king, two overlaid runes, and why the logo is his monogram. |
| C | `16-InfraredNostalgia.md` | IrDA, the TV remote, and why infrared lost to radio. |

#### 04 · DataTransfer

| P | Proposed file | The question |
|---|---|---|
| A | `09-WhyAirDropIsFast.md` | Bluetooth negotiates, WiFi carries. The difference between 2 Mbps and 200. |
| A | `10-ManySmallFilesVsOneBig.md` | Why 4,000 files of 1 MB take longer than one of 4 GB. The per-operation cost. |
| B | `11-WhyWhatsAppRuinsVideo.md` | Lossy recompression, twice, plus resizing. What is left of the original. |
| B | `12-ContactHashesInAirDrop.md` | How your phone knows the sender is in your contacts without revealing your contacts. And why the method had problems. |
| B | `13-SyncConflicts.md` | Two devices edit the same file offline. Who wins, and who decides? |
| C | `14-QRCodesAsTransport.md` | The QR code as a one-way data channel: how many bytes fit and how it survives being crumpled. |
| C | `15-SneakernetMath.md` | A van full of hard drives still has more bandwidth than fibre. The actual arithmetic. |

#### 05 · Positioning

| P | Proposed file | The question |
|---|---|---|
| A | `11-RelativityInGPS.md` | The satellites correct for special and general relativity daily. Without it, GPS would drift 10 km a day. |
| A | `12-WhyTheBlueDotJumps.md` | The jump when you leave the metro: which source was in charge, and which takes over. |
| B | `13-SelectiveAvailability.md` | Until 2000, civilian GPS carried deliberate error. Who removed it, and why. |
| B | `14-COCOMLimits.md` | Every GPS receiver stops working above 18 km altitude and 1,900 km/h. So it cannot guide a missile. |
| B | `15-Geofencing.md` | How an app knows you entered an area without watching you the whole time. |
| B | `16-FitnessAppsAndBases.md` | In 2018, a heatmap of running routes revealed secret military bases. What failed conceptually. |
| C | `17-AltitudeMeasurement.md` | How the phone knows which floor you are on: the barometer, not GPS. |
| C | `18-DeadReckoning.md` | Navigating with no external reference, like submarines and aircraft before GPS. |

#### 06 · Storage

| P | Proposed file | The question |
|---|---|---|
| A | `09-WhereTheGigabytesGo.md` | Why a "128 GB" phone has 112 free. Base 10 versus base 2, plus the operating system. |
| A | `10-WhyFullDrivesAreSlow.md` | An SSD at 95% falls off a performance cliff. The cause is how block erasure works. |
| B | `11-DataRetentionUnpowered.md` | How long your photos last on an unplugged drive in a drawer. The answer is uncomfortable. |
| B | `12-BitRotAndECC.md` | Cosmic rays flip bits in memory. How often, and what corrects it. |
| B | `13-QuickFormatIsALie.md` | A quick format never touches the data: it just discards the index. |
| B | `14-ImageCompression.md` | JPEG, HEIC and AV1: how 90% of the information is thrown away without the eye noticing. |
| C | `15-TheWriteAmplification.md` | Writing 1 KB can force the SSD to write 512. Why, and what TRIM does about it. |
| C | `16-StorageMediaLifespan.md` | Paper, tape, disk, SSD, M-DISC: what survives a century and what does not last ten years. |

#### 07 · AudioAndPeripherals

| P | Proposed file | The question |
|---|---|---|
| A | `09-WhyMicKillsAudioQuality.md` | Turn on the Bluetooth microphone and music drops to AM-radio quality. The cause is a profile from 2003. |
| A | `10-LatencyBudget.md` | Exactly where the 200 ms of delay come from: capture, codec, radio, buffer, decode. |
| B | `11-WhyANCFailsWithVoices.md` | Active cancellation kills aircraft rumble and cannot touch the conversation next to you. |
| B | `12-OcclusionEffect.md` | Why your own voice sounds strange with earphones in, and what transparency mode does. |
| B | `13-InEarDetection.md` | How the earbud knows you took it out: optical, capacitive or accelerometer. |
| B | `14-WhyTWSLagMore.md` | Two independent earbuds have to sync with each other. That costs milliseconds. |
| C | `15-LosslessOverBluetooth.md` | Why "lossless audio over Bluetooth" is almost always false, and what LC3 changes. |
| C | `16-PollingRates.md` | 1000 Hz on a gaming mouse: what it means and the point where it stops being noticeable. |

#### 08 · PowerAndBatteries

| P | Proposed file | The question |
|---|---|---|
| A | `08-WhyPhonesDieInCold.md` | At 10% and −5 °C the phone shuts down abruptly. The chemistry slows down; it does not run out. |
| A | `09-WhyChargingSlowsAt80.md` | Fast charging is not fast at the end. Constant current versus constant voltage. |
| B | `10-mAhIsMisleading.md` | Comparing mAh across devices with different voltages means nothing. The right unit is the watt-hour. |
| B | `11-BatteriesOnPlanes.md` | Why the cabin yes and the hold no. Thermal runaway, and why someone has to see it happen. |
| B | `12-GaNChargers.md` | Why a modern 65 W charger fits in a fist and the 2015 one weighed half a kilo. |
| B | `13-PhantomDrain.md` | A switched-off device still draws power. How much, and where it goes. |
| C | `14-USBCIsAMess.md` | Two identical cables that do different things. What is inside, and why the logo matters. |
| C | `15-CalibrationMyths.md` | Draining to 0% to "calibrate" was true with nickel and has been false with lithium for twenty years. |

#### 09 · IoTAndSmartHome

| P | Proposed file | The question |
|---|---|---|
| A | `10-WhyBulbsNeedInternet.md` | Turning on a light two metres away goes via a server 2,000 km away. Why it was designed that way. |
| A | `11-WhenTheServerDies.md` | Real cases of devices bricked by a vendor decision. What prevents it. |
| B | `12-MiraiAndDefaults.md` | In 2016, cameras with factory passwords took down half the internet. The structural lesson. |
| B | `13-WhyMatterTookSoLong.md` | Ten years of standards war, and why agreement took so long to arrive. |
| B | `14-TenYearBattery.md` | A door sensor on one CR2032 for a decade. The engineering of staying asleep. |
| B | `15-LocalVsCloud.md` | Home automation that works without internet: what you gain, what you lose, what it costs to build. |
| C | `16-TheDashButton.md` | A plastic WiFi button that ordered detergent. How it worked inside. |
| C | `17-SmartTVTelemetry.md` | What a connected TV reports, how often, and to whom. |

#### 10 · SecurityAndCryptography

| P | Proposed file | The question |
|---|---|---|
| A | `09-WhatTLSDoesNotProtect.md` | The padlock says nobody is listening en route. It does not say the site is legitimate, or honest. |
| A | `10-HowTOTPWorks.md` | A six-digit code that changes every 30 seconds, with no connection and no communication. The trick is a shared clock. |
| A | `11-Passkeys.md` | Why a key that never leaves your device beats any password. |
| B | `12-WhyFingerprintsArentImages.md` | The sensor does not store your fingerprint: it stores a model that cannot be reconstructed. |
| B | `13-RemoteWipeIsKeyDeletion.md` | Erasing 256 GB in one second is impossible. Erasing the key that decrypts it is not. |
| B | `14-Entropy.md` | Where a freshly booted device gets random numbers, when nothing has happened yet. |
| B | `15-PasswordStorage.md` | What a service stores when it stores your password, and what makes a breach serious or not. |
| C | `16-PostQuantumMigration.md` | "Harvest now, decrypt later": why migration already started even though the quantum computer does not exist. |

#### 11 · PrivacyAndFootprint

| P | Proposed file | The question |
|---|---|---|
| A | `09-WhatAPhotoKnows.md` | Coordinates, time, model, orientation and serial number. The case of the fugitive located by a magazine photo. |
| A | `10-IncognitoDoesNotHide.md` | Exactly what private browsing hides, and from whom. The list is shorter than people assume. |
| B | `11-BrowserFingerprinting.md` | With no cookies and no identifier, your browser is still unique among millions. Why. |
| B | `12-WhyAppsWantBluetooth.md` | A torch app asks for Bluetooth. Not to connect to anything: to know where you are. |
| B | `13-UltrasonicBeacons.md` | Sounds your ear misses and the microphone does not, used to link your phone to your television. |
| B | `14-SensorSideChannels.md` | What the accelerometer can infer with no microphone or camera permission. State of the research. |
| C | `15-TheDataBrokerChain.md` | From a free app to a commercial file: the hops a data point makes. |
| C | `16-WhatYouCanActuallyDo.md` | Closing the section: the five measures that change something, and the ten that do not. |

#### 99 · Reference

| P | Proposed file | What it is |
|---|---|---|
| A | `07-QuickAnswers.md` | An FAQ index linking straight to the article that answers each one. The real front door for many readers. |
| B | `08-MythBusting.md` | A table of myths with verdict and link. Highly shareable. |
| B | `09-TimelineOfWireless.md` | Timeline: 1895 Marconi → 1947 cellular → 1971 Aloha → 1997 802.11 → today. |
| C | `10-RangeComparison.md` | Comparison table: range, speed, power draw and latency for every technology in the repository. |

---

### 🔍 To research

Questions worth answering that **cannot be written yet**, because reliable documentation
is missing, measurement is needed, or it is unclear whether the public answer is
complete. The blocker has to be cleared before any of these becomes an article.

| Question | What is missing | Where to look |
|---|---|---|
| How many BSSIDs are actually catalogued today? | No platform publishes current figures | Academic papers on WPS, WiGLE data |
| What accuracy does UWB give in real conditions, not datasheet ones? | Independent measurements | FiRa Consortium publications, ranging papers |
| What real degradation margin does a battery have at 500 cycles? | Vendor figures are optimistic | Independent laboratory studies |
| What exactly changes between 5G SA and NSA regarding identifiers? | Dense specification, plenty of misinformation | 3GPP TS 33.501 |
| Is `_nomap` still honoured by every platform? | It was never formally documented | Verify against each platform's current documentation |
| What is the real state of lossless audio over Bluetooth? | Contradictory marketing | LE Audio / LC3plus specification |
| How accurate is the barometer at detecting floors? | Depends on calibration and weather | Sensor documentation, own testing |
| What can be inferred from a household's electricity consumption pattern? | Research is scattered | Papers on NILM (load disaggregation) |

### 💭 Loose ideas

The buffer. Things that occur to me and that I do not yet know are an article, a
paragraph inside another one, or nothing at all.

- Why is in-flight WiFi so bad and so expensive? (satellite vs air-to-ground)
- What technically happens in airplane mode on a watch with no connection of its own?
- How does tap-to-pay work on a watch without enough battery to boot?
- Why do some cars detect the key left inside and others do not?
- What does "data saver mode" actually do?
- How is traffic coordinated in a full stadium? (small cells, DAS)
- How much energy does transferring 1 GB really consume?
- What is the background "noise" in a camera, and why are night photos worse?
- Why can an eSIM not be copied from one phone to another?
- How does a smart scale estimate body fat? (bioimpedance, and how reliable it is)

---

### ❌ Discarded

With the reason, so it does not get re-evaluated in six months.

| Idea | Reason |
|---|---|
| Step-by-step network auditing guides | Outside the scope set in `_meta/StyleGuide.md`: explain the mechanism, not the recipe |
| Specific tools for deanonymising devices | Same as above. The repository exists to help understand your own exposure |
| Product reviews and comparisons | They age in months and the repository would become something else |
| Industry news and announcements | A repository of explanations does not compete with a feed |
| Setup tutorials (build a VPN, install Home Assistant) | A thousand already exist, done well. Here we explain **why** it works, not how to install it |
| A programming / code section | It dilutes the thesis. If code is needed, it goes inline in the article that needs it |
| Translating the articles into English | Not now. Content first. The structure already allows it if it ever pays off |

---

### How to use this file

1. **Starting something**: move it to 🔨 In progress. Three at most.
2. **Finishing something**: take it out of here and add it to ✅ Published with date and link.
3. **Discarding something**: move it to ❌ with the reason written down. The reason is the useful part.
4. **New idea**: straight to 💭 without overthinking. It gets filtered later.
5. **Review priorities** every month or two. Anything sitting in B for a year is probably C.

Hygiene rule: if a 🔍 entry has been blocked on sources for more than six months, it
either drops to 💭 or gets discarded. A permanent blocker is not a task, it is
decoration.

> **When editing:** the status tables are shared and live at the top. Everything else
> exists in two languages — if you touch a proposal, touch it in both places or it will
> drift.

<br>

[🇪🇸 Volver al español](#hoja-de-ruta--roadmap)

</details>

---

[← Volver al índice general · Back to the master index](README.md)