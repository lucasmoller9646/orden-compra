# Orden de compra de ganado · Grupo Ypoti / Rural Bioenergía S.A. — versión 1.5.4

App para que los comisionistas y el director de originación registren cada compra de ganado desde el corral: datos del lote, pesaje animal por animal, foto de los animales carimbados, video opcional, ubicación del corral, distancia por ruta y transportadora sugerida, precio, comisión y liquidación (IVA 5 % y 10 %, dólares, puesto en estancia); en la segunda página la guía, la factura y los datos bancarios, con una verificación automática por IA. Al completar todo, la orden se envía a pago: los administradores reciben un correo con el PDF y un enlace donde autorizan (contado con urgencia o plazo), y el sistema agrega la fila en **COMPRAS DE DESMAMANTES 9**, avisa a Laura para Asana y crea la tarea en Asana.

Está construido sobre el app **Registro de manejo** (misma base: PWA que se agrega a la pantalla de inicio, funciona sin señal, guarda solo cada 2 segundos en el teléfono y copia a la nube cuando hay señal, PIN de 6 números verificado por el servidor, PDF generado en el teléfono con pdf-lib, mapa OpenStreetMap, servidor en Google Apps Script con planilla + Drive + correo).

## Novedades 1.5.4 (requiere actualizar el servidor)
- **Corrección del servidor**: desde el servidor 1.5, «Enviar a pago» fallaba con `foto is not defined` después de subir el PDF (una definición había quedado dentro de un comentario). Ninguna orden podía enviarse a pago. Corregido: hay que pegar `servidor_compras.gs` de nuevo y publicar una nueva versión.
- **Envío robusto**: si la conexión se corta mientras se envía (Safari en iPhone/iPad corta al apagar la pantalla, cambiar de app o tras ~60 s sin respuesta), el app consulta al servidor si el envío quedó registrado; si quedó, lo toma como enviado sin mandar un segundo correo; si no, reintenta hasta 3 veces. La pantalla se mantiene encendida mientras sube.
- **Pedido más liviano**: con el servidor 1.5.4 las fotos ya no van duplicadas en el envío (el servidor las toma de la copia completa de la orden). Cada respuesta del servidor trae su versión (`srv`).
- En el correo y en el PDF, «Justificación» pasa a llamarse «Comentario al enviar». En la base, `app_version` se guarda como texto (antes la planilla lo convertía en fecha).

## Novedades 1.5.1

- **Video sin límite de duración ni de tamaño** (antes 45 s / 80 MB). El servidor manda cada parte de 3 MB directo a una subida reanudable de la API de Drive, sin armar el archivo en memoria (que tenía tope de ~50 MB). Si se corta la señal, sigue desde la última parte subida.

## Novedades 1.5

- **Autorización firmada electrónicamente**: cuando Pedro (o Mauricio) autoriza, el servidor genera el PDF **"Autorización de pago"** con el texto de la autorización, la forma de pago, el resumen de la orden, la firma (nombre, fecha y hora, PIN personal) y los datos de integridad: huella SHA-256 del PDF de la orden y un **sello HMAC-SHA256** calculado con una clave secreta que solo tiene el servidor (propiedad `FIRMA_SECRETO`, se crea sola). Se guarda en la carpeta de la orden en Drive.
- **Verificación**: el documento trae el enlace `…/exec?verificar=<código>`, que muestra si el sello coincide (✓ auténtica) y si el PDF de la orden sigue igual al que se firmó (si la orden se reenvía con cambios, avisa que hace falta autorización nueva).
- **Dónde va**: se ve al final de la página de autorización (botón "📄 Ver la autorización firmada"), va **adjunta** en el correo de confirmación, en la tarea de **Asana** y su enlace en la planilla (columna URL AUTORIZACION) y en el app (panel "Pago").
- **Asana con adjuntos**: la tarea lleva las **fotos de las facturas, guías y COTAs**, el **PDF de la orden** del comisionista y la **autorización firmada**. En el final de la autorización se ve "✓ Asana: tarea creada con N adjuntos".
- Es firma **electrónica** (identificación por PIN + sello del servidor). Una firma **digital** con certificado (Ley 4017/2010, prestador acreditado) necesitaría el certificado de cada firmante.

## Novedades 1.4

- **Datos para pago** (antes "Datos bancarios del proveedor"): primero se elige **Transferencia** o **Cheque**. Con transferencia se cargan una o más cuentas (titular, banco, número, moneda y documento; ya no hay campo alias). Con cheque no se carga nada más. Va así al PDF, al correo, a la planilla base (columna `medio_pago`) y a la tarea de Asana.
- **La IA verifica sola**: cuando están las fotos de factura, guía y COTA y hay señal, la verificación arranca a los pocos segundos de la última foto o cambio, sin tocar ningún botón (el botón "Verificar" queda para repetirla a mano).
- **En el app solo "Enviar a pago"**: se sacaron "Autorizar / Guardar cambios y autorizar" y "Devolver" del app. Pedro o Mauricio corrigen en el app si hace falta, tocan **Enviar a pago** y autorizan desde el enlace del correo nuevo.
- **Planilla BD COMPRAS**: la columna PROVEEDOR (F) se llena siempre con el **propietario que figura en la COTA** de esa factura (si la IA no lo leyó: vendedor de la guía y, por último, el proveedor cargado en el app). Proveedor, establecimiento, departamento y distrito van en mayúsculas y sin tildes, como las filas existentes.
- **PDF**: se sacó el botón "Compartir PDF (WhatsApp, correo…)" y el archivo JSON de la pantalla de la orden. En "Archivos" el PDF tiene **Ver** y **Compartir**; compartir manda solo el archivo, sin texto ni enlace "blob" como leyenda (igual en el botón PDF de la lista).

## Novedades 1.3

- **Una sola lista de órdenes**: ya no existen "En este teléfono" y "Todas las órdenes". Hay una lista y el botón **⟳ Actualizar** (también se actualiza solo al abrir el app y cada 5 minutos): sube lo que cambió en el teléfono, compara con la nube y trae solo las diferencias. Cada orden aparece **una sola vez** (por su id). Las órdenes que están solo en la nube se muestran con "☁ en la nube" y se bajan completas al tocarlas. Los administradores ven todas; cada intermediario, las suyas.
- **Ingreso solo con el PIN de 6 números**: el app ya no muestra la lista de usuarios; el PIN identifica a la persona y la sesión queda abierta con su nombre. Cada intermediario tiene su PIN (`usuarios_y_claves.txt`); los PIN de 4 números dejan de valer.
- **Intermediario**: el campo se llama "Intermediario (comisionista)". Para los administradores es un buscador: se empieza a escribir y la lista se filtra (lista completa de 50 intermediarios, que solo reciben los administradores). Para el intermediario el campo queda fijo con su nombre.
- **Rematadoras**: si la factura la emite una rematadora (FERUSA, ARROBA REMATES, CORRAL DEL NORTE, AOSTA REMATES u otra casa de remates), la IA lo acepta: el emisor de la factura puede ser distinto del vendedor de la guía y del propietario de la COTA. Si la cuenta bancaria es del productor y la factura de la rematadora, queda un aviso para confirmar a quién se paga.
- **Al autorizar**, el final muestra un ✓ OK por cada paso: planilla de compras (fila), tarea de Asana, correo de la decisión y correo a Laura (o ✗ con el motivo si alguno falló), en la página del enlace y en el app.
- **Planilla BD COMPRAS**: FECHA = fecha de emisión de la factura (una por factura); la columna L va como **TRANSITO** (antes "PENDIENTE").

## Novedades 1.2

- **Precio por rango de peso**: en la compra por kg se elige **Simple** (un precio) o **Por rango** (p. ej. de 0 a 230 kg → 23.500 ₲/kg; de 230,1 kg en adelante → 22.000 ₲/kg). Cada animal pesado se paga con el precio de su rango; el app muestra cuántos animales y kilos caen en cada rango y avisa si alguno queda fuera. "Cómo facturar" sale con una línea por rango (precio por cabeza redondeado a miles) y el PDF, el correo, la planilla (columna Q = "KG PIE (RANGO)") y Asana muestran los rangos.
- **Puntos intermedios en la ruta**: en el mapa de la ruta, "＋ Punto intermedio" deja mover el mapa hasta el camino correcto y "✓ Pasar por aquí" recalcula la ruta por ese punto (se pueden agregar varios, en cualquier orden; "↺ Quitar puntos" vuelve a la ruta directa). Si igual no convence, "✎ Escribir km a mano".
- **Cabezas por camión sin valor por defecto**: hay que cargarlas (es dato obligatorio); recién ahí se calculan los viajes.
- **IA**: controla además que cada **COTA tenga sello** (sin sello = error) y que los documentos **no tengan enmiendas ni errores** (factura enmendada = error; guía o COTA enmendada = aviso). Se **ignora** el campo "Factura de venta o Nota de Remisión" de la guía: con una factura, todas las guías son de esa factura; con varias, las guías se reparten entre las facturas para que cada factura sume sus cabezas. En el app se muestra **solo lo que no está bien** (más "N controles correctos"); el correo de autorización lleva la **lista completa**.
- **Quién autoriza**: el pedido de autorización va **solo a Pedro y Mauricio** (`CONFIG.AUTORIZAN`, o la columna `autoriza` = sí/no de la hoja `usuarios`), con el PDF **una sola vez** (adjunto). Laura y Lucas reciben el correo **después**, con la decisión (Laura además el de Asana). La página del enlace solo acepta las contraseñas de quienes autorizan.
- **Editar al autorizar**: el correo trae "✎ Abrir en el app para corregir y autorizar" (`…/orden-compra/?orden=<id>`). Pedro o Mauricio abren la orden, la corrigen si hace falta y tocan **✅ Guardar cambios y autorizar** (o ↩ Devolver): se guarda el PDF nuevo, la planilla, Asana y salen los correos.
- **Contado con fecha**: al autorizar al contado se puede cargar **"Pagar hasta"** (opcional). Va a Asana (vencimiento y campo "Fecha de pgto"), a la planilla ("ALTA · HASTA 26/10/2026"), al PDF y a los correos.
- **Correcciones**: Asana rechazaba la fecha (el campo fecha ahora va como `{date: …}`); si la orden ya tenía tarea, se actualiza la misma. La hora y el código SENACSA ya no se desarman en la planilla (iban como "1899-12-30T17:36" y "210150003"). La fecha de la compra en BD COMPRAS es la del pesaje (antes podía salir la de la autorización). Una orden devuelta que se reenvía vuelve a quedar pendiente de autorización.

## Novedades 1.1

- **Orden nueva de la página 1**: 1 · datos del lote (con la **cantidad de animales** al inicio y raza Zebú / Híbrido / **Mezclado**) → 2 · negociación (₲ por kg vivo o ₲ por cabeza "punta", y el precio) → 3 · **hoja de pesaje** animal por animal si es por kg (cuenta "12 de 24 pesados"), o solo el **peso promedio estimado** si es por cabeza → 4 · **video del lote carimbado** (obligatorio; ya no hay foto de los animales) → 5 · destino y flete → 6 · comisión (primero la forma: ₲ por cabeza, % o sin comisión; después el monto) → 7 · totales → observaciones.
- **Flete**: contacto de la transportadora con botones Llamar / WhatsApp (Toro Pampa · Pedro Maidana, Gerente de Logística Ganado, 0981 359786; Cerro Porã · Hugo Paniagua, Responsable de flete, 0981 295007); **cabezas por camión** editable y viajes = cabezas ÷ cabezas por camión.
- **Ruta en el mapa**: "Calcular y ver la ruta" dibuja el camino (OSRM) con el corral y la estancia; "✓ Usar estos km" o "✎ No es coherente: escribir a mano".
- **Mapa**: el pin queda fijo en el centro y se mueve el mapa; pinza y doble toque para zoom; satélite por defecto. El resto del app tiene **ancho fijo** (sin zoom de la página) y los campos de fecha y hora se ven bien en iPhone.
- **Cómo facturar** (el precio ya incluye el IVA): animales → cantidad | categoría | precio unitario por cabeza (en la compra por kg se convierte y se redondea a miles) | columna 5 % = valor a facturar | liquidación del IVA 5 % = valor ÷ 21. Comisión → cantidad | "Comisión por N animales hasta {estancia}" | precio por cabeza | columna 10 % = valor | IVA 10 % = valor ÷ 11.
- **Documentos**: se cargan solo fotos, en tres listas: **facturas**, **guías de traslado** y **COTAs** (varias de cada una; las tres obligatorias). Los números los lee la IA (ya no se escriben).
- **IA**: arma el árbol factura → guías (por el "Nº de factura" que cita cada guía) → COTAs (por el "Nº de guía" que cita cada COTA) y controla, además de lo anterior, que **las cabezas de cada factura coincidan con la suma de sus guías**, que cada guía tenga su COTA y que cada COTA corresponda a una guía cargada.
- **Cuentas bancarias**: una o varias; el documento acepta guión (RUC con DV). Cada titular se compara con el emisor de la factura.
- **Planilla**: **una fila por factura** (ID APP = id·F1, id·F2…), con las cabezas, el precio y los kilos de esa factura; flete y comisión prorrateados por cabezas. Si en una edición se quita una factura, su fila queda marcada "ANULADA".
- **Actualización automática**: al abrir el app con señal busca la versión nueva en GitHub, la instala y recarga solo (todo queda guardado en el teléfono). Las órdenes de la versión 1.0 se convierten solas.

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
2. La planilla de compras configurada es la de **producción** (`1YRyv-fKiV0kA_H3sfKwYrpHffSmCfE6ZdacqWuq2JHs`); la hoja tiene que llamarse `BD COMPRAS`. La de prueba era `1sa2Jt4GOyXY28TegRUSx0_PdF1CR-KI61TF4WkodlO8`.
3. Entre a [script.google.com](https://script.google.com) › **Nuevo proyecto**, borre lo que hay, pegue todo `servidor_compras.gs` y guarde (Ctrl+S). Póngale nombre al proyecto (p. ej. `Servidor orden de compra`).
4. `CONFIG` ya viene completo (carpeta, planilla de prueba, correos lucasmoller / mauriciomoller / pedromoller / laura@ypoti.com, proyecto de Asana). Los PIN de cada persona están en `usuarios_y_claves.txt` (se cargan solos en la hoja `usuarios`).
5. **Propiedades del script** (⚙ Configuración del proyecto › Propiedades del script › Agregar): `ANTHROPIC_API_KEY` (sección 4) y `ASANA_TOKEN` (sección 5; el token que ya generó). Nunca van en el código ni en GitHub: si el token quedó pegado en algún chat o correo, conviene regenerarlo en Asana y cargar el nuevo.
6. Elija la función **`probarConfiguracion`** y toque **Ejecutar**. Acepte los permisos (Drive, Sheets, Gmail, conexión externa). En el registro de ejecución ve la URL de la planilla base, la carpeta y si están cargadas las claves. Esto crea la planilla base `Órdenes de compra de ganado (base de datos)` con las hojas `usuarios`, `ordenes`, `pesos`, `log` y `verificaciones_ia`.
7. **Implementar › Nueva implementación › Aplicación web** · *Ejecutar como: Yo* · *Quién tiene acceso: Cualquier persona* › Implementar. Copie la URL que termina en `/exec`.
8. Abra `index.html`, busque `DEFAULT_ENDPOINT` (está al principio del último `<script>`) y pegue esa URL entre comillas. Suba los cuatro archivos del app al repositorio de GitHub Pages.

Para actualizar el servidor después: pegue el `.gs` nuevo y **Implementar › Administrar implementaciones › lápiz › Versión: Nueva versión › Implementar** (la URL /exec no cambia). Para actualizar el app: reemplace `index.html` y `sw.js` en GitHub (cambie `CACHE` en `sw.js` para que los teléfonos tomen la versión nueva).

### Usuarios (hoja `usuarios` de la planilla base)

Columnas: `nombre`, `rol` (`comisionista` o `admin`), `clave` (PIN de 6 números, distinto para cada persona: el PIN identifica a quién ingresa), `mail`, `telefono`, `activo` (`sí`/`no`), `autoriza` (`sí`/`no`; vacío = según `CONFIG.AUTORIZAN`). Para dar de alta a un intermediario: agregue la fila con un PIN nuevo de 6 números (que no repita otro) y avísele el PIN; aparece solo en el buscador de los administradores. Para darlo de baja: `activo = no`. El servidor relee la hoja cada 5 minutos.

Los administradores también reciben los correos de "enviar a pago" (todos los `admin` con `mail`). Si un comisionista tiene `mail`, recibe el correo con la decisión de su orden.

---

## 2 · Cómo se usa (comisionista)

1. Abrir el enlace en el celular (Chrome en Android, Safari en iPhone) y **agregar a la pantalla de inicio**. Ingresar con el PIN de 6 números (hace falta señal solo la primera vez; después la sesión queda en el teléfono con su nombre).
2. **＋ Nueva orden de compra** → página 1 (datos operativos): comisionista (fijo), proveedor, categoría, fecha y hora, localidad, **ubicación del corral** (📡 GPS o 🗺 marcar en el mapa — siempre obligatoria), carimbo, sola marca, raza, **foto de los animales carimbados**, video opcional, estancia de destino, **km por ruta** (se calculan solos con señal; a mano sin señal), si vuelve cargado, transportadora sugerida (ver 3), precio por viaje y cantidad de viajes, **pesaje animal por animal** (Agregar o Enter; cada pesada se puede corregir o quitar), precio por kg o por cabeza, cambio del día (se completa solo), comisión (₲/cabeza, ₲/kg, % o sin comisión) y observaciones. Abajo se ven los totales: total, IVA 5 % (÷ 21), por cabeza, por kg, comisión e IVA 10 % (÷ 11), flete, puesto en estancia y todo en US$.
3. **Documentación ›** → página 2: por cada guía, foto de la **guía de traslado** (GE2) y foto de su **COTA** (certificado SENACSA con QR) — las dos obligatorias; puede haber varias guías por factura —, foto de la **factura** (obligatoria), datos bancarios (titular, banco, cuenta, moneda, RUC/cédula/otro y número, alias). Con señal, la **verificación IA** corre sola; sin señal queda pendiente y corre cuando vuelve la conexión.
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

El app manda al servidor (`verificar_ia`) la foto de la factura, cada par guía + COTA y los datos cargados (proveedor, destino y su código SENACSA, cantidad de animales, total acordado, fecha, titular y documento de la cuenta). El servidor llama a la API con una **herramienta de salida estructurada** (`registrar_lectura`): el modelo solo puede responder con el JSON de los datos leídos (número de factura, timbrado y vigencia, fecha, condición, emisor y RUC, cliente, cantidad, columna del IVA, total, liquidación del IVA; número de guía, validez, propietario y código, establecimiento de origen con código/departamento/distrito, categorías y cantidades, destino con código, consignatario, y la calidad de cada foto). Nunca inventa: si no lee, deja `null` y explica el problema (borrosa, movida, cortada).

Con esos datos el **servidor compara de forma determinista** (función `compararIA_`), para que el resultado no dependa del criterio del modelo:

| Control | OK | AVISO | ERROR |
|---|---|---|---|
| Fotos legibles | calidad buena/regular | mala | ilegible / no se lee |
| Vendedor de la guía y propietario de la COTA = emisor de la factura | similitud ≥ 75 % (tolera Ibañes/Ibañez, orden distinto) | 50–75 % | < 50 % |
| RUC del vendedor en la guía = RUC de la factura | iguales | — | distintos |
| Cabezas de cada factura = animales de sus guías (el Nº de factura citado en la guía se ignora) | iguales | no se leyó | distintos / no se pueden repartir |
| COTA con sello oficial | con sello | no se ve | sin sello |
| Documentos sin enmiendas ni errores | sin enmiendas | guía o COTA enmendada / no se pudo confirmar | factura enmendada |
| Emisor de la factura = proveedor cargado en el app | parecidos | distintos | — |
| La COTA cita la guía ("datos de la Guía N°…") | mismo número | no se leyó | distintos (fotos mezcladas) |
| Mismo establecimiento de origen y misma cantidad en guía y COTA | iguales | — | distintos |
| Guía con adquiriente Rural Bioenergía | sí | — | no |
| Titular de la cuenta = emisor de la factura | ídem | ídem | ídem |
| RUC de la factura = documento del titular (RUC 7538271-7 = CI 7538271) | iguales | no se leyó | distintos |
| Código del propietario en la COTA = RUC de la factura | iguales | distintos / no se leyó | — |
| Timbrado vigente a la fecha de emisión | dentro de la vigencia | no se leyó | fuera de vigencia |
| Fecha de la factura cerca del pesaje | ≤ 15 días | > 15 días | — |
| Factura a nombre de Rural Bioenergía (RUC 80050418-6) | sí | no se leyó | otro cliente |
| Monto de la factura = total acordado | diferencia ≤ 0,5 % | ≤ 2 % | > 2 % |
| IVA 5 % (total ÷ 21) y columna del 5 % | coincide | no se leyó la liquidación | en columna 10 %/exentas o IVA distinto |
| Suma de animales de las COTAs = pesados | iguales | no se leyó | distintos (¿falta una guía con su COTA?) |
| Destino de la COTA = estancia elegida (código SENACSA) | iguales | destino "a definir" / no se leyó | distintos |
| COTA consignada a Rural Bioenergía | sí | — | no |
| COTA dentro del plazo de validez | sí | vencida | — |

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

- **Correo a quienes autorizan** (solo Pedro y Mauricio: `CONFIG.AUTORIZAN` o columna `autoriza` de la hoja `usuarios`): tabla con proveedor, comisionista, fecha y lugar, destino, cabezas y pesos, precio (o rangos), total e IVA, comisión, flete, puesto en estancia, facturas, guías y COTAs, cuentas bancarias; la **verificación IA completa** (errores, avisos y correctos); el **PDF adjunto** (una sola vez); enlaces a las fotos y al video; el botón **Revisar y autorizar el pago** (`…/exec?oc=<token>`) y **Abrir en el app para corregir y autorizar** (`CONFIG.APP_URL?orden=<id>`).
- **Página de autorización**: PIN de 6 números de Pedro o Mauricio, datos de la orden, controles de la IA, enlaces; **Contado** → urgencia baja/media/alta y "pagar hasta" (opcional); **A plazo (cheque diferido)** → días o fecha; observaciones generales; **Autorizar el pago** o **Devolver con observaciones** (exige observaciones). Si otro administrador ya decidió, la página lo muestra y no deja decidir dos veces. Cada decisión lleva sello con hora y código SHA-256.
- **En el app**: Pedro y Mauricio ven en la orden los botones **✅ Autorizar pago** y **↩ Devolver**; si la corrigieron (✎ Editar), el botón dice "Guardar cambios y autorizar" y se manda la versión nueva.
- **Al autorizar**: fila en `BD COMPRAS` (sección 7), tarea en Asana, correo a Laura para Asana, correo con la decisión a todos los administradores (Lucas y Laura incluidos, solo para información) y al comisionista, estado "Pago autorizado" en el app (se actualiza solo cada 5 minutos o con "Consultar estado").

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
| BE… | NRO. GUIA, NRO. COTA, ID APP, RAZA, FORMA DE PAGO, URGENCIA / PLAZO, AUTORIZADO POR, FECHA AUTORIZACION, URL PDF, URL VIDEO, LAT CORRAL, LON CORRAL, COMISIONISTA APP, ESTADO IA | columnas nuevas que el servidor agrega al final la primera vez |

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
- **Seguridad**: los PIN de 6 números viven solo en la hoja `usuarios`; el app guarda el del usuario en el teléfono para firmar los pedidos. 8 PIN incorrectos desde un teléfono lo bloquean 15 minutos y 40 incorrectos en total bloquean todos los ingresos 15 minutos (para que no se pueda adivinar un PIN). Los comisionistas no pueden leer ni modificar órdenes de otros (el servidor lo controla, no solo el app).
- **Un solo teléfono por orden a la vez**: si la misma orden se edita desde dos teléfonos, vale la copia más nueva; el servidor avisa `mas_nueva` y no pisa la más reciente.
- **Dependencias en línea**: fuentes de Google (opcional), OSRM para los km, mapas de OpenStreetMap/Esri, API de Claude, API de Asana. Sin señal el app funciona completo salvo km por ruta, mapa, verificación IA y envío a pago.

---

## 10 · Cambios respecto del app de manejo

Misma base (`sw.js`, almacenamiento, autoguardado, copia en la nube, teclado PIN, mapa, PDF, estilo YPOTI) con estas diferencias: login por usuario (comisionista/administrador) en lugar de contraseña por estancia; una sola tarjeta de formulario en dos páginas; pesaje tecleado animal por animal (no archivo de báscula); verificación IA de documentos; flujo de aprobación de pago con forma/urgencia/plazo en lugar de aprobar/rechazar; escritura en una planilla externa (COMPRAS DE DESMAMANTES 9) y Asana.
