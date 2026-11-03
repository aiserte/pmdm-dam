---
title: "Cafetería del instituto: el último ticket y el historial — Enunciado (Sesiones 1 y 2)"
unidad: UD4
sesion: 1
orden: 1
tipo: enunciado
---

## Contexto

Práctica del bloque *Almacenamiento local: ficheros JSON y bases de datos con Room* (sesiones 1 y 2 de la UD4). Partimos de la solución de la última práctica de la UD3: la cafetería con cuatro pantallas, un `PedidoViewModel` compartido (`AndroidViewModel`) y un `PreferenciasRepository` que recuerda el nombre del cliente.

Cuando el cliente envía un pedido, la app lo olvida. Al terminar esta práctica:

- **Parte A:** guardará el **ticket del último pedido** en un fichero JSON y se podrá consultar desde el menú de la carta, aunque se haya cerrado la app.
- **Parte B:** registrará **cada pedido enviado** en una base de datos SQLite con Room, y el ticket mostrará cuántos pedidos lleva hechos ese cliente.

<div class="callout-warning">
Usa lo visto hasta ahora: todo lo de la UD3 más lo de este bloque (<code>filesDir</code>, <code>File</code>, <code>withContext(Dispatchers.IO)</code>, <code>@Serializable</code> y <code>Json</code>, entidades, DAO, base de datos, <code>Application</code> propia con <code>by lazy</code>). Todavía <strong>no</strong> hemos visto consultas que devuelvan <code>Flow</code> ni relaciones entre tablas: no los necesitas.
</div>

<div class="callout-info">
Si no terminaste la última práctica de la UD3, parte de su solución publicada. La parte A se hace en la primera sesión del bloque y la parte B en la segunda. Haz un <em>commit</em> al terminar cada parte.
</div>

## Parte A · El último ticket en un fichero JSON

### Ejercicio 1 — Recursos

Añade estos textos en español (`values/strings.xml`) y en inglés (`values-en/strings.xml`). El último lo usarás en la parte B:

| Nombre | Español | Inglés |
|---|---|---|
| `ultimo_pedido` | Último pedido | Last order |
| `titulo_ticket` | Tu último pedido | Your last order |
| `ticket_para_llevar` | Para llevar | Takeaway |
| `ticket_para_tomar` | Para tomar aquí | Eat in |
| `producto_desconocido` | Producto %1$d | Product %1$d |
| `cerrar` | Cerrar | Close |
| `pedidos_cliente` | Pedidos de %1$s: %2$d | Orders by %1$s: %2$d |

### Ejercicio 2 — El modelo del ticket

Crea `data/Ticket.kt` con dos clases `@Serializable`:

1. `LineaTicket` con el **id** del producto, la cantidad y el precio unitario en el momento del pedido.
2. `Ticket` con el nombre del cliente, si es para llevar, la fecha y hora del pedido (`Long`, en milisegundos, obtenida con `System.currentTimeMillis()`), la lista de líneas y el total.

<div class="callout-warning">
La línea <strong>no</strong> puede guardar el <code>Producto</code> ni su nombre como recurso (<code>R.string...</code>). Repasa el apartado 1.4 de la teoría para entender por qué.
</div>

### Ejercicio 3 — El repositorio del ticket

Crea `data/TicketRepository.kt` con una clase `TicketRepository(directorio: File)` que use el fichero `ultimo_ticket.json` dentro de esa carpeta y tenga:

1. Un `Json` privado configurado con `prettyPrint` e `ignoreUnknownKeys`.
2. `suspend fun guardar(ticket: Ticket)`, que sobrescribe el fichero.
3. `suspend fun cargar(): Ticket?`, que devuelve `null` si el fichero no existe, si su contenido no es un ticket válido o si hay un error de lectura. En los dos últimos casos, escribe un aviso en Logcat (`Log.w`) con la etiqueta `TicketRepo`.
4. `suspend fun borrar()`.

Las tres funciones deben ejecutar el acceso al fichero en `Dispatchers.IO`.

### Ejercicio 4 — El `ViewModel` guarda y carga el ticket

1. En `PedidoUiState`, añade `ultimoTicket: Ticket? = null` y `mostrarTicket: Boolean = false`.
2. En `PedidoViewModel`, crea un `TicketRepository` con la carpeta `filesDir` de la `Application`.
3. En el `init`, dentro de la misma coroutine que carga las preferencias, carga el ticket guardado y cópialo al estado. Muestra en Logcat (`PedidoVM`) lo que se ha cargado.
4. En `enviarPedido()`, **antes** de vaciar el pedido, construye el `Ticket` a partir de `lineasPedido`, `total`, el nombre y "para llevar". Ponlo en el estado y guárdalo en el fichero (con `viewModelScope`).
5. Crea `verTicket()` y `cerrarTicket()`, que cambian `mostrarTicket`.
6. Haz que `olvidarDatos()` borre también el ticket (del estado y del fichero).

### Ejercicio 5 — Ver el último pedido

1. Crea `ui/carta/DialogoTicket.kt` con un composable stateless `DialogoTicket(ticket, buscarProducto: (Int) -> Producto?, onCerrar, modifier)` que muestre un `AlertDialog` con:
   - título `titulo_ticket`;
   - la fecha y hora del pedido, formateadas según el idioma del móvil (pista: `java.text.DateFormat.getDateTimeInstance(DateFormat.SHORT, DateFormat.SHORT).format(Date(...))`);
   - el nombre y "Para llevar" o "Para tomar aquí";
   - una fila por línea, con el formato `linea_pedido` de la UD3 (`2 × Café`) y el importe de la línea. Si el producto ya no existe en la carta, se muestra `producto_desconocido` con su id;
   - el total, con el recurso `total_pedido`;
   - un único botón `cerrar`.
2. Añade una preview del diálogo con un ticket de ejemplo que incluya una línea de un producto inexistente.
3. En `BarraSuperiorCarta`, añade una opción **"Último pedido"** al menú, habilitada solo si hay ticket.
4. En `PantallaCarta`, conecta la opción con `verTicket()` y muestra `DialogoTicket` cuando `mostrarTicket` sea `true` y haya ticket.

## Parte B · El historial de pedidos con Room

### Ejercicio 6 — Room en el proyecto

Añade Room y el plugin KSP al catálogo de versiones y a los dos `build.gradle.kts`, como en el apartado 1.10 de la teoría, y sincroniza Gradle.

### Ejercicio 7 — Entidad, DAO y base de datos

En un paquete nuevo `data/local`:

1. `PedidoEntity.kt`: entidad de la tabla `pedidos` con un `id` autogenerado (`Long`), el nombre, si es para llevar (columna `para_llevar`), la fecha y hora (columna `fecha_hora`), el número de unidades y el total.
2. `PedidoDao.kt`: un DAO con estas funciones `suspend`:
   - `insertar(pedido)`, que devuelve el id asignado;
   - `contarPorNombre(nombre)`, que devuelve cuántos pedidos tiene ese nombre;
   - `contarTodos()`, que devuelve cuántos pedidos hay en total;
   - `borrarTodos()`.
3. `CafeteriaDatabase.kt`: la base de datos, versión 1, sin exportar el esquema, con una función que devuelve el `PedidoDao`.

### Ejercicio 8 — Una única base de datos para toda la app

1. Crea, en el paquete principal, una clase `CafeteriaApp` que herede de `Application` y tenga una propiedad `database` creada con `by lazy` y `Room.databaseBuilder`, con el fichero `cafeteria.db`.
2. Regístrala en `AndroidManifest.xml` con el atributo `android:name`.

### Ejercicio 9 — El repositorio del historial

Crea `data/HistorialRepository.kt` con una clase `HistorialRepository(pedidoDao: PedidoDao)` con:

- `suspend fun registrar(ticket: Ticket): Long`, que convierte el ticket en un `PedidoEntity` (las unidades son la suma de las cantidades de sus líneas) y lo inserta;
- `suspend fun contarPedidos(nombre: String): Int`;
- `suspend fun contarTodos(): Int`;
- `suspend fun borrarTodo()`.

### Ejercicio 10 — El `ViewModel` usa el historial

1. En `PedidoViewModel`, obtén la base de datos de `CafeteriaApp` y crea el `HistorialRepository`.
2. En `enviarPedido()`, dentro de la misma coroutine que guarda el ticket, **registra** el pedido en el historial. Escribe en Logcat el id que le ha asignado la base de datos y cuántos pedidos hay en total.
3. Añade `pedidosCliente: Int? = null` a `PedidoUiState`. En `verTicket()`, además de abrir el diálogo, consulta cuántos pedidos tiene el cliente del ticket y guárdalo en ese campo.
4. `DialogoTicket` recibe un parámetro nuevo `pedidosCliente: Int?` y, si no es `null`, muestra una línea con el texto `pedidos_cliente`.
5. Haz que `olvidarDatos()` borre también el historial.

### Ejercicio 11 — Pruebas en el emulador

Añade a `PRUEBAS.md` una sección **"UD4 · Sesiones 1 y 2"**. Filtra Logcat con `tag:PedidoVM | tag:TicketRepo`.

**Parte A**

| # | Prueba | Resultado esperado |
|---|---|---|
| P1-1 | Desinstalar la app, instalarla de nuevo y abrir el menú de la carta | "Último pedido" aparece desactivado. Copia el mensaje de Logcat de la carga |
| P1-2 | Enviar un pedido de 2 cafés y 1 croissant, a nombre de "Ana", para llevar. Volver a la carta y abrir "Último pedido" | El diálogo muestra fecha, nombre, "Para llevar", las dos líneas con su importe y el total |
| P1-3 | Cerrar la app deslizándola fuera de las apps recientes y volver a abrirla | El ticket sigue disponible y es idéntico |
| P1-4 | Con *Device Explorer*, abrir `ultimo_ticket.json` | Copia su contenido y compáralo con el del fichero de DataStore de la UD3: ¿por qué uno se puede leer y el otro no? |
| P1-5 | Cambiar el idioma del emulador a inglés y abrir "Último pedido" | Los nombres de los productos aparecen en inglés. Explica por qué funciona |
| P1-6 | Dañar el fichero (apartado 1.6 de la teoría), cerrar la app y volver a abrirla | La app **no** se cierra; "Último pedido" aparece desactivado. Copia el aviso de Logcat |

**Parte B**

| # | Prueba | Resultado esperado |
|---|---|---|
| P1-7 | Enviar tres pedidos: dos de "Ana" y uno de "Luis". Abrir "Último pedido" | El diálogo muestra cuántos pedidos lleva el último cliente. Copia los mensajes de Logcat con los ids asignados |
| P1-8 | Abrir *App Inspection › Database Inspector* | Captura de la tabla `pedidos` con sus filas. ¿Cómo aparece la columna `para_llevar`? |
| P1-9 | En el *Database Inspector*, ejecutar una consulta propia que muestre solo los pedidos de "Ana" ordenados del más reciente al más antiguo | Copia la consulta y el resultado |
| P1-10 | Con *Device Explorer*, abrir `/data/data/<paquete>/databases/` | Anota qué ficheros hay y explica para qué sirve cada uno |
| P1-11 | Quitar el `android:name` del manifiesto y ejecutar. Después, volver a ponerlo | Copia el error de Logcat y explica por qué ocurre |
| P1-12 | Sin desinstalar la app, añadir a `PedidoEntity` una propiedad nueva (por ejemplo, `val avisar: Boolean = true`) y ejecutar. Después, desinstalar la app y volver a ejecutar. Por último, quita la propiedad y desinstala otra vez | Copia el error del primer intento y explica por qué desaparece al desinstalar |
| P1-13 | Elegir "Olvidar mis datos" | Comprueba qué ha pasado con `ultimo_ticket.json`, con el fichero de DataStore y con la tabla `pedidos` |

### Opcional — Total gastado por cliente

Añade al DAO una consulta que devuelva **cuánto ha gastado en total** un cliente (pista: `SELECT SUM(total) ...`) y muéstralo también en el diálogo del ticket. ¿Qué devuelve `SUM` si el cliente no tiene ningún pedido? ¿Qué tipo de retorno debe tener la función del DAO para que eso no cierre la app?

<div class="callout-info">
Este ejercicio es opcional: no cuenta en los requisitos técnicos ni en los criterios de evaluación.
</div>

## Requisitos técnicos obligatorios

**Parte A**

- `Ticket` y `LineaTicket` son `@Serializable` y la línea guarda el id del producto y el precio del momento, no recursos ni objetos `Producto`.
- `TicketRepository` recibe una carpeta (`File`), no un `Context`, y todo su acceso al disco se hace con `withContext(Dispatchers.IO)`.
- `cargar()` devuelve `null` sin cerrar la app si el fichero no existe, está dañado o no se puede leer.
- El ticket se construye antes de vaciar el pedido y se guarda con `viewModelScope`.
- Opción "Último pedido" en el menú, deshabilitada si no hay ticket, y diálogo stateless con preview.

**Parte B**

- Room añadido con el catálogo de versiones: `room-runtime` con `implementation` y `room-compiler` con `ksp`.
- Entidad, DAO y base de datos en `data/local`; todas las funciones del DAO son `suspend` y las consultas usan parámetros (`:nombre`), nunca texto concatenado.
- Una única base de datos, creada con `by lazy` en `CafeteriaApp` y registrada en el manifiesto.
- `HistorialRepository` es la única clase que usa el `PedidoDao`; el `ViewModel` no conoce `PedidoEntity`.
- "Olvidar mis datos" borra las preferencias, el fichero del ticket y el historial.
- Ningún composable accede a ficheros, a la base de datos ni a los repositorios.
- `PRUEBAS.md` con la sección "UD4 · Sesiones 1 y 2" completa.

## Criterios de evaluación

Esta práctica contribuye a los criterios **f)** y **g)** del RA2:

- **f)** La app se conecta con dos almacenes de datos (un fichero JSON y una base de datos SQLite gestionada con Room), a través de repositorios, y los datos persisten tras cerrar la app.
- **g)** Se ha comprobado y documentado en el emulador qué se guarda en cada almacén, cuándo se pierde y qué ocurre ante un fichero dañado, una `Application` mal registrada o un cambio de esquema.
- El proyecto compila y se ejecuta sin errores.
