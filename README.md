# Orden de compra de ganado · Grupo Ypoti / Rural Bioenergía S.A. — versión 1.0

App para que los comisionistas y el director de originación registren cada compra de ganado desde el corral: datos del lote, pesaje animal por animal, foto de los animales carimbados, video opcional, ubicación del corral, distancia por ruta y transportadora sugerida, precio, comisión y liquidación (IVA 5 % y 10 %, dólares, puesto en estancia); en la segunda página la guía, la factura y los datos bancarios, con una verificación automática por IA. Al completar todo, la orden se envía a pago: los administradores reciben un correo con el PDF y un enlace donde autorizan (contado con urgencia o plazo), y el sistema agrega la fila en **COMPRAS DE DESMAMANTES 9**, avisa a Laura para Asana y crea la tarea en Asana.

Está construido sobre el app **Registro de manejo** (misma base: PWA que se agrega a la pantalla de inicio, funciona sin señal, guarda solo cada 2 segundos en el teléfono y copia a la nube cuando hay señal, contraseñas de 4 números verificadas por el servidor, PDF generado en el teléfono con pdf-lib, mapa OpenStreetMap, servidor en Google Apps Script con planilla + Drive + correo).

## Archivos

| Archivo | Qué es | Dónde va |
|---|---|---|
| `index.html`, `sw.js`, `manifest.webmanifest`, `.nojekyll` | El app (un solo archivo + service worker + manifiesto). | Repositorio de GitHub Pages (por ejemplo `lucasmoller9646.github.io/orden-compra/`). |
| `servidor_compras.gs` | El servidor (Google Apps Script). | Proyecto nuevo en script.google.com. |
| `usuarios_y_claves.txt` | Usuarios iniciales y contraseñas. **No subir a GitHub.** | Solo para el administrador. |
| `README.md` | Esta guía. | — |

---

## 1 · Instalar el servidor (Apps Script)

1. La carpeta de Drive ya está configurada: **Banco de datos compra ganado** (`1cKfEixTAyPaOUPz4Sno1RkYYK72xPhYf`). El script corre con la cuenta de quien lo implementa (Lucas), así que no hace falta compartirla con la cuenta de servicio `integracao-ypoti@…` (esa sirve para la integración SAP, no para Apps Script).
2. La planilla de compras configurada es **COMPRAS DE DESMAMANTES (base de prova)** (`1sa2Jt4GOyXY28TegRUSx0_PdF1CR-KI61TF4WkodlO8`), dentro de esa carpeta. Al pasar a producción, cambie `COMPRAS_SHEET_ID` por el ID de la planilla definitiva (la hoja tiene que llamarse `BD COMPRAS`).
3. Entre a [script.google.com](https://script.google.com) › **Nuevo proyecto**, borre lo que hay, pegue todo `servidor_compras.gs` y guarde (Ctrl+S). Póngale nombre al proyecto (p. ej. `Servidor orden de compra`).
4. `CONFIG` ya viene completo (carpeta, planilla de prueba, correos lucasmoller / mauriciomoller / pedromoller / laura@ypoti.com, proyecto de Asana). Revise las contraseñas iniciales en `usuarios_y_claves.txt` y cámbielas si quiere.
5. **Propiedades del script** (⚙ Configuración del proyecto › Propiedades del script › Agregar): `ANTHROPIC_API_KEY` (sección 4) y `ASANA_TOKEN` (sección 5; el token que ya generó). Nunca van en el código ni en GitHub: si el token quedó pegado en algún chat o correo, conviene regenerarlo en Asana y cargar el nuevo.
6. Elija la función **`probarConfiguracion`** y toque **Ejecutar**. Acepte los permisos (Drive, Sheets, Gmail, conexión externa). En el registro de ejecución ve la URL de la planilla base, la carpeta y si están cargadas las claves. Esto crea la planilla base `Órdenes de compra de ganado (base de datos)` con las hojas `usuarios`, `ordenes`, `pesos`, `log` y `verificaciones_ia`.
7. **Implementar › Nueva implementación › Aplicación web** · *Ejecutar como: Yo* · *Quién tiene acceso: Cualquier persona* › Implementar. Copie la URL que termina en `/exec`.
8. Abra `index.html`, busque `DEFAULT_ENDPOINT` (está al principio del último `<script>`) y pegue esa URL entre comillas. Suba los cuatro archivos del app al repositorio de GitHub Pages.

Para actualizar el servidor después: pegue el `.gs` nuevo y **Implementar › Administrar implementaciones › lápiz › Versión: Nueva versión › Implementar** (la URL /exec no cambia). Para actualizar el app: reemplace `index.html` y `sw.js` en GitHub (cambie `CACHE` en `sw.js` para que los teléfonos tomen la versión nueva).

### Usuarios (hoja `usuarios` de la planilla base)

Columnas: `nombre`, `rol` (`comisionista` o `admin`), `clave` (4 números), `mail`, `telefono`, `activo` (`sí`/`no`). Para dar de alta a un comisionista: agregue la fila y avísele su clave. Para darlo de baja: `activo = no`. El servidor relee la hoja cada 5 minutos. Las claves de los administradores tienen que ser distintas entre sí: la página de autorización identifica a quién autoriza por su clave.

Los administradores también reciben los correos de "enviar a pago" (todos los `admin` con `mail`). Si un comisionista tiene `mail`, recibe el correo con la decisión de su orden.

---

## 2 · Cómo se usa (comisionista)

1. Abrir el enlace en el celular (Chrome en Android, Safari en iPhone) y **agregar a la pantalla de inicio**. Ingresar con el nombre y la contraseña de 4 números (hace falta señal solo la primera vez; después la sesión queda en el teléfono).
2. **＋ Nueva orden de compra** → página 1 (datos operativos): comisionista (fijo), proveedor, categoría, fecha y hora, localidad, **ubicación del corral** (📡 GPS o 🗺 marcar en el mapa — siempre obligatoria), carimbo, sola marca, raza, **foto de los animales carimbados**, video opcional, estancia de destino, **km por ruta** (se calculan solos con señal; a mano sin señal), si vuelve cargado, transportadora sugerida (ver 3), precio por viaje y cantidad de viajes, **pesaje animal por animal** (Agregar o Enter; cada pesada se puede corregir o quitar), precio por kg o por cabeza, cambio del día (se completa solo), comisión (₲/cabeza, ₲/kg, % o sin comisión) y observaciones. Abajo se ven los totales: total, IVA 5 % (÷ 21), por cabeza, por kg, comisión e IVA 10 % (÷ 11), flete, puesto en estancia y todo en US$.
3. **Documentación ›** → página 2: foto de cada guía (puede haber varias por factura), foto de la factura, datos bancarios (titular, banco, cuenta, moneda, RUC/cédula/otro y número, alias). Con señal, la **verificación IA** corre sola; sin señal queda pendiente y corre cuando vuelve la conexión.
4. **Guardar y revisar ›** → pantalla de la orden: estado (Borrador si falta algo obligatorio; Completa si no), verificación IA, PDF (Ver / Compartir por WhatsApp), resumen de texto para WhatsApp, Editar, Borrar y **📤 Enviar a pago** (se habilita cuando no faltan datos obligatorios).
5. En cualquier momento se puede tocar **💾 Borrador** o cerrar el app: nada se pierde. El PDF de un borrador lleva en rojo la lista de lo que falta.

El inicio muestra las órdenes de este teléfono (deslizar hacia la izquierda: PDF, Editar, Borrar). **☁ Recuperar mis órdenes de la nube** trae las órdenes propias si se cambia de teléfono. Los administradores tienen además la pestaña **☁ Todas las órdenes**, ven y editan cualquier orden (incluidos los borradores) y pueden registrar compras propias eligiendo el comisionista (o "Otro…").

Estados de una orden: **Borrador** → **Completa** → **Enviada a pago** → **Pago autorizado** / **Devuelta con observaciones**. Si una orden ya enviada se modifica y se vuelve a enviar, los administradores reciben el correo "actualizada" y, si ya estaba autorizada, se actualiza su fila en la planilla de compras.

---

## 3 · Flete: regla de decisión (Guía ToroPampa × Cerro Porã, mayo/2026)

Con los km por ruta el app calcula el precio por viaje de cada transportadora y sugiere la más barata; el comisionista puede cambiarla o elegir **Otro** y escribir nombre y precio.

| Distancia | Toro Pampa | Cerro Porã | Sugerida |
|---|---|---|---|
| 0 a 50 km | 2.800.000 | 1.350.000 | Cerro Porã |
| 51 a 100 km | 2.800.000 | 2.300.000 | Cerro Porã |
| 101 a 110 km | 2.800.000 | 2.800.000 | Indiferente (empate → Toro Pampa) |
| 111 a 160 km | 17.200 ₲/km (1.909.200 a 2.752.000) | 2.800.000 | Toro Pampa |
| 161 a 399 km | 17.200 ₲/km | 16.400 ₲/km | Cerro Porã |
| 400 km o más | 16.800 ₲/km | 16.400 ₲/km | Cerro Porã |

**Retorno cargado → siempre Toro Pampa**, sin excepción. La cantidad de viajes se propone como ⌈cabezas ÷ 52⌉ y se puede corregir. Las tarifas están en `tarifasFlete()` del `index.html` (sin ajuste por combustible): cuando se renegocien, cambiar ahí.

La distancia por ruta se pide al servicio público de OSRM (`router.project-osrm.org`), igual que en la planilla de pesaje; a Yby Pora se suman 6 km desde el acceso y a La Esperanza 21 km desde la entrada. Yby Pytá (Cafetalera) todavía no tiene coordenadas ni código: se cargan los km a mano.

---

## 4 · Verificación IA: cómo generar la clave de la API y cómo funciona

### 4.1 · Crear la clave (una sola vez, 5 minutos)

1. Entre a la consola de la API de Claude: **https://platform.claude.com/** y cree una cuenta de la empresa (o use la de Lucas). Es una cuenta distinta de claude.ai.
2. Cargue crédito (**Billing / Plans & billing** → tarjeta y un saldo inicial, p. ej. US$ 20). La API se cobra por uso; no es una suscripción.
3. **API keys › Create key**. Póngale nombre (`servidor-orden-compra`) y copie la clave (`sk-ant-api03-…`): se muestra una sola vez.
4. En Apps Script: ⚙ **Configuración del proyecto › Propiedades del script › Agregar propiedad** → nombre `ANTHROPIC_API_KEY`, valor la clave. Guardar.
5. Ejecutar › **`probarIA`**: en el registro tiene que aparecer `200` y una respuesta. Si da `401`, la clave está mal copiada; si da `400` con "credit", falta saldo.

Modelo: `claude-sonnet-5-5` (`CONFIG.IA.modelo`), que lee bien manuscritos y formularios. Costo aproximado: cada verificación manda una factura y una o dos guías (fotos de ~2.000 px) y devuelve un JSON corto → del orden de **US$ 0,01 a 0,02 por orden**; con 100 compras al mes, un par de dólares. Para abaratar se puede usar `claude-haiku-5-5` (menos preciso con letra manuscrita); para máxima precisión, `claude-opus-5-5`. Referencia de la API: https://platform.claude.com/docs/en/api/messages/create · imágenes: https://platform.claude.com/docs/en/build-with-claude/vision.

### 4.2 · Qué hace la verificación

El app manda al servidor (`verificar_ia`) la foto de la factura, la(s) foto(s) de la(s) guía(s) y los datos cargados (proveedor, destino y su código SENACSA, cantidad de animales, total acordado, fecha, titular y documento de la cuenta). El servidor llama a la API con una **herramienta de salida estructurada** (`registrar_lectura`): el modelo solo puede responder con el JSON de los datos leídos (número de factura, timbrado y vigencia, fecha, condición, emisor y RUC, cliente, cantidad, columna del IVA, total, liquidación del IVA; número de guía, validez, propietario y código, establecimiento de origen con código/departamento/distrito, categorías y cantidades, destino con código, consignatario, y la calidad de cada foto). Nunca inventa: si no lee, deja `null` y explica el problema (borrosa, movida, cortada).

Con esos datos el **servidor compara de forma determinista** (función `compararIA_`), para que el resultado no dependa del criterio del modelo:

| Control | OK | AVISO | ERROR |
|---|---|---|---|
| Fotos legibles | calidad buena/regular | mala | ilegible / no se lee |
| Nombre de la factura = propietario de la guía | similitud ≥ 75 % (tolera Ibañes/Ibañez, orden distinto) | 50–75 % | < 50 % |
| Titular de la cuenta = emisor de la factura | ídem | ídem | ídem |
| RUC de la factura = documento del titular (RUC 7538271-7 = CI 7538271) | iguales | no se leyó | distintos |
| RUC de la factura = propietario de la guía | iguales | distintos / no se leyó | — |
| Timbrado vigente a la fecha de emisión | dentro de la vigencia | no se leyó | fuera de vigencia |
| Fecha de la factura cerca del pesaje | ≤ 15 días | > 15 días | — |
| Factura a nombre de Rural Bioenergía (RUC 80050418-6) | sí | no se leyó | otro cliente |
| Monto de la factura = total acordado | diferencia ≤ 0,5 % | ≤ 2 % | > 2 % |
| IVA 5 % (total ÷ 21) y columna del 5 % | coincide | no se leyó la liquidación | en columna 10 %/exentas o IVA distinto |
| Animales en la(s) guía(s) = pesados | iguales | no se leyó | distintos |
| Destino de la guía = estancia elegida (código SENACSA) | iguales | destino "a definir" / no se leyó | distintos |
| Guía consignada a Rural Bioenergía | sí | — | no |
| Guía dentro del plazo de validez | sí | vencida | — |

El tipo de animal **no** se compara entre guía y factura (solo se informa). Resultado: **✓ todo coincide**, **⚠ con advertencias** o **✗ corregir**. Los administradores pueden enviar a pago con advertencias o errores escribiendo una justificación (va en el correo y en el PDF); los comisionistas, con errores, tienen que corregir. Cada lectura queda en la hoja `verificaciones_ia` (con los tokens usados). Las tolerancias están en `CONFIG.IA`.

### 4.3 · Sin señal

El comisionista carga todo igual; la verificación queda **"IA: pendiente"**. En cuanto el teléfono tiene conexión (al abrir el app, cada 5 minutos o al volver la señal), el app la ejecuta sola y avisa con un mensaje si algo hay que corregir ("La IA encontró diferencias: revise la verificación"). Si cambian las fotos o los datos clave, la verificación se invalida y se repite; si no cambió nada, no se vuelve a llamar a la API. "Enviar a pago" exige señal (hay que mandar el correo) y, si la verificación no está hecha, la hace en ese momento.

---

## 5 · Integración con Asana

Al autorizar un pago el servidor (a) manda un correo a Laura con todos los datos y (b) **crea la tarea en Asana por API** en el proyecto **TRAZABILIDAD**, sección **Lanzamientos de Compras**, con el mismo formato de las tareas que ya se cargan a mano (ver la tarea "COMPRA DE 24 DMA FABIO…"):

- **Nombre**: `COMPRA DE 24 DMA FABIO CARLOS IBAÑEZ`.
- **Descripción**: `COMPRA DE 24 DMA PRECIO UNITARIO GS. 5.197.548 (22.998 GS/KG · 226 KG PROM.) MONTO TOTAL GS. 124.741.152`, forma de pago, `DATOS PARA PAGO` (titular, CI/RUC, entidad, cuenta, alias, moneda), factura y guía, destino y origen, `FLETE A PAGAR GS. … A …`, comisión, resultado de la IA, quién autorizó y observaciones, enlaces al PDF, fotos y video. El PDF va adjunto.
- **Campos personalizados**: Tipo de Movimiento = Compra · Establecimiento Destino (Ypoti, Yby Pora, Lusipar, Santa Clara, BELA VISTA, LA ESPERENZA, Cerro Memby) · Cantidad de Animales · Prioridade = urgencia (ALTA/MEDIA/BAJA; a plazo → BAJA) · Status = Cerrada · Intermediario (se elige la opción cuyo nombre se parece al comisionista, p. ej. "Ronaldo Salina" → RONALDO SALINAS; si no existe, queda vacío) · Categoria predominante (DMA/DHE/TOR/VAQ/NOV/TER) · Monto total · Fecha de pgto (fecha del plazo o 1/3/7 días según urgencia alta/media/baja; también es la fecha de vencimiento de la tarea) · Fletero (TORO PAMPA S.A., CERRO PORÃ, otras opciones por nombre, o "Otras") · Monto comision.
- **Asignada a** laura@ypoti.com.

Todos los GID están en `CONFIG.ASANA` del servidor (workspace `1200146592749332`, proyecto `1210569584658279`, sección `1211560264927556`). Si prefiere otra sección o proyecto, cambie ahí o cargue `ASANA_PROJECT_GID` / `ASANA_SECTION_GID` en las propiedades del script.

Pasos:

1. **Token de acceso personal.** Ya lo generó (Asana › Configuración › Apps › Tokens de acceso personal). Cárguelo en Apps Script › ⚙ Configuración del proyecto › Propiedades del script → `ASANA_TOKEN`. El token da acceso a todo su Asana: no lo pegue en archivos ni en GitHub; si se filtra, revóquelo en la misma pantalla y genere otro.
2. Ejecutar › **`probarAsana`**: muestra el usuario del token y los proyectos con su GID (control).
3. Si en el futuro agregan opciones nuevas a Intermediario o Fletero en Asana, no hay que tocar nada: el servidor las busca por nombre. Si cambian los campos (los borran o renombran), actualizar los GID en `CONFIG.ASANA` (se ven con `probarAsana` o en la API: `GET /projects/<gid>?opt_fields=custom_field_settings.custom_field.name,custom_field_settings.custom_field.gid`).

Si el token no está cargado, nada falla: Laura recibe el correo y registra a mano. Referencia: https://developers.asana.com/docs/personal-access-token · https://developers.asana.com/reference/createtask.

---

## 6 · Enviar a pago y autorización

- **Correo a los administradores** (todos los `admin` con `mail` + copia a `MAIL_COPIA`): tabla con proveedor, comisionista, fecha y lugar, destino, cabezas y pesos, precio, total e IVA, comisión, flete, puesto en estancia, factura y guía, cuenta bancaria, verificación IA; el **PDF adjunto**; enlaces al PDF, a las fotos y al video en Drive; y el botón **Revisar y autorizar el pago** (`…/exec?oc=<token>`).
- **Página de autorización**: contraseña de 4 números (identifica al administrador), datos de la orden, controles de la IA, enlaces; **Contado** → urgencia baja/media/alta; **A plazo (cheque diferido)** → días o fecha; observaciones generales; **Autorizar el pago** o **Devolver con observaciones** (exige observaciones). Si otro administrador ya decidió, la página lo muestra y no deja decidir dos veces. Cada decisión lleva sello con hora y código SHA-256.
- **Al autorizar**: fila en `BD COMPRAS` (sección 7), correo a Laura, tarea en Asana, correo con la decisión a los administradores y al comisionista, estado "Pago autorizado" en el app (se actualiza solo cada 5 minutos o con "Consultar estado").

---

## 7 · Planilla COMPRAS DE DESMAMANTES 9 (hoja BD COMPRAS)

Una fila por factura, en la primera fila libre, con las **mismas fórmulas** que usan las filas existentes. La fila se identifica por la columna nueva **ID APP**: si la orden se edita y se reenvía, se actualiza la misma fila. Varias guías por factura van en **NRO. GUIA** separadas por `;`.

| Col. | Columna | Qué escribe el app |
|---|---|---|
| C | ANO | año de la fecha |
| D | FECHA | fecha del pesaje |
| E | PROPRIETARIO | `RURAL BIOENERGIA S.A` |
| F | PROVEEDOR | proveedor (mayúsculas) |
| G, H, I | ESTABLECIMENTO ORIGEN, DEPARTAMENTO, DISTRITO | leídos de la guía por la IA (si no hay lectura, la localidad cargada) |
| J | ESTABLECIMENTO DE DESTINO | estancia de destino (`YPOTI`, `BELLA VISTA`, `YBY PORÃ`…) |
| K | FECHA DE CHEGADA | **vacío · lo cargan los capataces** |
| L | STATUS | `PENDIENTE` (los capataces lo pasan a `ENTREGUE`) |
| O | (categoría) | `DESMAMANTES MACHOS`, `TORETON`, … |
| P | CANTIDAD TOTAL | cabezas |
| Q | CATEGORIA | `KG PIE` o `CABEZA` |
| R | PRECIO LIQUIDADO | precio por cabeza (total ÷ cabezas) |
| S, T, V, X, Y | fórmulas | iguales a las filas anteriores |
| U | PESO PROMEDIO SIN DESBASTE | promedio del pesaje en el corral |
| W | PESO PROMEDIO BASCULA | **vacío · lo cargan los capataces** |
| Z, AA | CANTIDAD DE ACOPLADOS, VALOR POR ACOPLADO | viajes y precio por viaje |
| AB–AM | fórmulas | iguales (AD = comisión según la forma: `P×tasa`, `T×tasa` o `S×%`) |
| AN | CAMBIO DEL DIA | el cambio usado en el app (se puede corregir a mano) |
| AO–AS, AU–AW | fórmulas | iguales |
| AT | VALOR AGRO | **vacío · lo carga finanzas** |
| AX | INTERMEDIARIO | comisionista |
| AY | FLETERO | transportadora |
| AZ | KMS PERCORRIDOS | km por ruta |
| BA | NRO. FACTURA DE COMPRA | `001-001-0000545` |
| BB, BC, BD | PAIS, MES, SOLA MARCA | `PARAGUAY`, mes, `SI`/`NO` |
| BE… | NRO. GUIA, ID APP, RAZA, FORMA DE PAGO, URGENCIA / PLAZO, AUTORIZADO POR, FECHA AUTORIZACION, URL PDF, URL VIDEO, LAT CORRAL, LON CORRAL, COMISIONISTA APP, ESTADO IA | columnas nuevas que el servidor agrega al final la primera vez |

La hoja `CONSULTA FACTURA` sigue funcionando igual (consulta por número de factura).

---

## 8 · Dónde queda cada cosa en Drive

`<carpeta del app>/ordenes/<fecha>_<proveedor>_<id>/` → `orden_<id>.json` (la orden completa con fotos, es la copia de la nube), `orden_compra_<…>.pdf`, `fotos/` (animales, factura, guía 1, 2…), `video_<…>.mp4`. La planilla base tiene las hojas `ordenes` (una fila por orden, se actualiza con cada copia y con cada decisión), `pesos` (una fila por animal), `verificaciones_ia`, `log` y `usuarios`.

Los archivos de Drive se comparten "con el enlace" para que los enlaces del PDF y de los correos abran sin pedir cuenta de Google. Si prefiere restringirlos, quite `compartir_()` en el servidor.

---

## 9 · Límites y detalles técnicos

- **Video**: hasta 80 MB (≈ 45 s en 1080p). Queda en el teléfono (IndexedDB) y se sube por partes de 3 MB cuando hay señal; el servidor lo arma en Drive y el PDF toma el enlace (botón con cámara). El PDF generado antes de la subida dice "Video: pendiente de subir"; al abrir la orden de nuevo se regenera con el enlace.
- **Fotos**: la de los animales se guarda a 1.400 px; la factura y las guías a 2.000 px (para que la IA las lea bien). Todo va dentro del JSON de la orden (copia en la nube y recuperación).
- **Cambio del día**: el servidor lo toma de `open.er-api.com` (referencial, se guarda 6 horas) o del valor manual `CONFIG.CAMBIO_MANUAL`; el comisionista puede corregirlo. Es el que va al PDF y a la columna AN.
- **Correos**: Apps Script permite 100 correos por día con cuenta Gmail común y 1.500 con Google Workspace. Cada orden usa 1 (envío) + 2 (decisión y Laura).
- **Seguridad**: las claves de 4 números viven solo en la hoja `usuarios`; el app guarda la del usuario en el teléfono para firmar los pedidos. Después de 8 claves incorrectas en 15 minutos el servidor bloquea ese usuario 15 minutos. Los comisionistas no pueden leer ni modificar órdenes de otros (el servidor lo controla, no solo el app).
- **Un solo teléfono por orden a la vez**: si la misma orden se edita desde dos teléfonos, vale la copia más nueva; el servidor avisa `mas_nueva` y no pisa la más reciente.
- **Dependencias en línea**: fuentes de Google (opcional), OSRM para los km, mapas de OpenStreetMap/Esri, API de Claude, API de Asana. Sin señal el app funciona completo salvo km por ruta, mapa, verificación IA y envío a pago.

---

## 10 · Cambios respecto del app de manejo

Misma base (`sw.js`, almacenamiento, autoguardado, copia en la nube, teclado PIN, mapa, PDF, estilo YPOTI) con estas diferencias: login por usuario (comisionista/administrador) en lugar de contraseña por estancia; una sola tarjeta de formulario en dos páginas; pesaje tecleado animal por animal (no archivo de báscula); verificación IA de documentos; flujo de aprobación de pago con forma/urgencia/plazo en lugar de aprobar/rechazar; escritura en una planilla externa (COMPRAS DE DESMAMANTES 9) y Asana.
