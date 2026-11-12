---
title: "Cafetería del instituto: historial y detalle de pedidos — Enunciado (Sesión 3)"
unidad: UD4
sesion: 3
orden: 2
tipo: enunciado
---

## Contexto

Práctica del bloque *Room II: datos que se actualizan solos y relaciones entre tablas* (sesión 3 de la UD4). Partimos de la solución de la práctica 1: la cafetería guarda el último ticket en un fichero JSON y registra la cabecera de cada pedido en la tabla `pedidos` de Room.

Al terminar esta práctica:

- cada pedido se guardará **con sus líneas**, en una segunda tabla relacionada con la primera;
- habrá una pantalla de **historial** con todos los pedidos, que se actualiza sola cuando cambia la base de datos;
- tocando un pedido del historial se abrirá su **detalle**, desde donde se podrá borrar.

El último ticket sigue en su fichero JSON y el diálogo "Último pedido" funciona como hasta ahora.

<div class="callout-warning">
Usa lo visto hasta ahora: todo lo de la UD3 y el bloque 1 más lo de este bloque (consultas que devuelven <code>Flow</code>, <code>stateIn</code>, <code>collectAsStateWithLifecycle</code>, <code>@ForeignKey</code>, <code>@Transaction</code>, <code>@Embedded</code> y <code>@Relation</code>, <code>SavedStateHandle</code> con <code>toRoute</code>). Todavía <strong>no</strong> hemos visto migraciones: al cambiar las entidades, desinstala la app.
</div>

<div class="callout-info">
Si no terminaste la práctica 1, parte de su solución publicada (las dos partes). Haz un <em>commit</em> antes de empezar y otro al terminar.
</div>

## Ejercicio 1 — Dependencia y recursos

1. Añade la librería `lifecycle-runtime-compose` al catálogo de versiones, con la misma versión que `lifecycle-runtime-ktx`, y a `app/build.gradle.kts`, como en el apartado 2.5 de la teoría. Sincroniza Gradle.
2. Añade estos textos en español (`values/strings.xml`) y en inglés (`values-en/strings.xml`):

| Nombre | Español | Inglés |
|---|---|---|
| `historial_pedidos` | Historial de pedidos | Order history |
| `historial_vacio` | Todavía no hay pedidos guardados. | No orders saved yet. |
| `titulo_detalle_pedido` | Pedido n.º %1$d | Order #%1$d |
| `borrar_pedido` | Borrar pedido | Delete order |

## Ejercicio 2 — La tabla de líneas

En el paquete `data/local`:

1. Crea `LineaPedidoEntity.kt`: entidad de la tabla `lineas_pedido` con un `id` autogenerado (`Long`), el id del pedido (columna `pedido_id`), el id del producto (columna `id_producto`), la cantidad y el precio unitario (columna `precio_unitario`). Declara la **clave ajena** hacia `pedidos` con **borrado en cascada** y un **índice** sobre `pedido_id`.
2. Crea `PedidoConLineas.kt`: una clase de resultado con el `PedidoEntity` incrustado y la lista de sus líneas.
3. Añade la nueva entidad a `CafeteriaDatabase`. Mantén la versión 1.
4. **Desinstala la app** del emulador antes de volver a ejecutarla.

## Ejercicio 3 — El DAO

Modifica `PedidoDao` para que tenga:

1. `insertarPedido(pedido)` (es el antiguo `insertar`, con otro nombre) e `insertarLineas(lineas)`, las dos `suspend`.
2. `insertarConLineas(pedido, lineas): Long`, una función **con cuerpo** que inserte el pedido, inserte sus líneas con el id que ha recibido el pedido y devuelva ese id, todo dentro de una **transacción**.
3. `observarTodos()`, que devuelva un `Flow` con todos los pedidos, del más reciente al más antiguo.
4. `observarConLineas(id)`, que devuelva un `Flow` con ese pedido y sus líneas, o `null` si no existe.
5. `borrarPorId(id)`, `suspend`.
6. Se mantienen `contarPorNombre`, `contarTodos` y `borrarTodos`.

## Ejercicio 4 — El repositorio

1. Crea `data/ResumenPedido.kt` con una `data class ResumenPedido` con el id, el nombre, la fecha y hora, las unidades y el total.
2. En `HistorialRepository`:
   - añade una propiedad `historial: Flow<List<ResumenPedido>>` a partir de `observarTodos()`. Cada vez que llegue una lista, escribe en Logcat, con la etiqueta `HistorialRepo`, el mensaje `Consulta del historial: N pedidos`;
   - añade `observarPedido(id: Long): Flow<Ticket?>`, que convierta el pedido con sus líneas en un `Ticket`;
   - cambia `registrar(ticket)` para que guarde también las líneas, con `insertarConLineas`;
   - añade `suspend fun borrar(id: Long)`.
3. Las conversiones de entidad a `ResumenPedido` y a `Ticket` deben ser **funciones de extensión privadas** del fichero del repositorio.

`PedidoViewModel` **no** debe cambiar. Compruébalo.

## Ejercicio 5 — Reutilizar el contenido del ticket

El detalle de un pedido mostrará lo mismo que el diálogo "Último pedido": fecha, nombre, "Para llevar", líneas y total.

1. En `DialogoTicket.kt`, saca ese contenido a un composable **público** `ContenidoTicket(ticket, buscarProducto, modifier)`.
2. Haz que `DialogoTicket` use `ContenidoTicket` y añada debajo, como hasta ahora, la línea de pedidos del cliente.
3. Comprueba que la preview del diálogo sigue viéndose igual.

## Ejercicio 6 — La pantalla del historial

En un paquete nuevo `ui/historial`:

1. `HistorialViewModel.kt`: un `AndroidViewModel` con un `HistorialRepository` y una propiedad `pedidos: StateFlow<List<ResumenPedido>?>` creada con `stateIn`, que escuche a Room solo mientras alguien la mire (con 5 segundos de margen) y valga `null` mientras no haya respuesta.
2. `PantallaHistorial.kt` con:
   - `PantallaHistorial(onVolver, onPedidoClick: (Long) -> Unit, modifier, historialViewModel = viewModel())`: un `Scaffold` con `BarraConVolver` y el título `historial_pedidos`. Lee `pedidos` con `collectAsStateWithLifecycle` y, si no es `null`, muestra `ContenidoHistorial`;
   - `ContenidoHistorial(pedidos, onPedidoClick, modifier)`, stateless: si la lista está vacía, el texto `historial_vacio`; si no, una `LazyColumn` con una `Card` pulsable por pedido, con el nombre, la fecha y hora, las unidades (`unidades_pedido`) y el total (`precio`);
   - una preview de `ContenidoHistorial` con tres pedidos de ejemplo y otra con la lista vacía.

## Ejercicio 7 — La pantalla de detalle

En el paquete `ui/historial`:

1. `DetallePedidoViewModel.kt`: un `AndroidViewModel` que recibe también un `SavedStateHandle` y:
   - lee el id del pedido de la ruta `DetallePedido` (ejercicio 8) y lo deja en una propiedad pública `idPedido`;
   - escribe en Logcat, con la etiqueta `DetalleVM`, `DetallePedidoViewModel creado para el pedido N` al crearse y un mensaje al destruirse (`onCleared`);
   - ofrece `pedido: StateFlow<Ticket?>`, igual que el historial;
   - tiene `buscarProducto(id)`, como `PedidoViewModel`;
   - tiene `borrar(alTerminar: () -> Unit)`, que borra el pedido y, **cuando ha terminado**, llama a `alTerminar`.
2. `PantallaDetallePedido.kt` con `PantallaDetallePedido(onVolver, modifier, detalleViewModel = viewModel())`: un `Scaffold` con `BarraConVolver`, el título `titulo_detalle_pedido` con el id y, si el pedido no es `null`, `ContenidoTicket` y un `OutlinedButton` `borrar_pedido` que borre el pedido y vuelva atrás al terminar.

## Ejercicio 8 — Navegación

1. En `Rutas.kt`, añade `Historial` (sin datos) y `DetallePedido(idPedido: Long)`.
2. En `AppCafeteria`, añade los dos destinos. Ninguno de los dos recibe `pedidoViewModel`, y `composable<DetallePedido>` no lee la ruta: lo hace su `ViewModel`.
3. En `BarraSuperiorCarta`, añade la opción `historial_pedidos` al menú, siempre habilitada, con un parámetro `onVerHistorial`. Pásalo desde `PantallaCarta` (que recibe un parámetro nuevo con el mismo nombre) hasta `AppCafeteria`, que navega a `Historial`.

## Ejercicio 9 — Pruebas en el emulador

Añade a `PRUEBAS.md` una sección **"UD4 · Sesión 3"**. Filtra Logcat con `tag:HistorialRepo | tag:DetalleVM | tag:CicloVida`.

| # | Prueba | Resultado esperado |
|---|---|---|
| P2-1 | Desinstalar la app, instalarla y abrir "Historial de pedidos" | Aparece el texto de historial vacío. Copia el mensaje de Logcat |
| P2-2 | Enviar tres pedidos: "Ana" (2 cafés y 1 croissant), "Luis" (1 bocadillo) y "Ana" (1 zumo). Abrir el historial | Tres tarjetas, la más reciente arriba. ¿Cuántas veces aparece "Consulta del historial" al abrir la pantalla? |
| P2-3 | En el *Database Inspector*, abrir las dos tablas y ejecutar una consulta que muestre solo las líneas del primer pedido | Captura de las dos tablas y copia de la consulta. ¿Qué relación hay entre `pedido_id` y la tabla `pedidos`? |
| P2-4 | Con el historial abierto en el emulador, cambiar en el *Database Inspector* el nombre de un pedido (desmarca antes *Live updates*) | La tarjeta cambia sin tocar la app. Copia el mensaje de Logcat que aparece y explica por qué |
| P2-5 | Con el historial abierto, girar el móvil dos veces | ¿Aparece "Consulta del historial"? Explica por qué |
| P2-6 | Con el historial abierto, pulsar Inicio y volver enseguida (menos de 5 s). Después, pulsar Inicio, esperar 10 s y volver | Anota en qué caso hay una consulta nueva, con los mensajes de `CicloVida` que la rodean, y explica la diferencia |
| P2-7 | Abrir el detalle del primer pedido de "Ana". Cambiar el idioma a inglés y volver a abrirlo | Muestra las líneas con su importe y el total; en inglés, los nombres de los productos traducidos |
| P2-8 | Borrar ese pedido desde el detalle | Se vuelve al historial y el pedido ya no está. Comprueba en el *Database Inspector* qué ha pasado con sus líneas y explica por qué |
| P2-9 | Activar *No conservar actividades* en las opciones de desarrollador. Abrir el detalle de un pedido, pulsar Inicio y volver. Después, desactivar la opción | La app vuelve al mismo detalle con sus datos. Copia los mensajes de `DetalleVM` y explica de dónde sale el id del pedido |
| P2-10 | Quitar el `indices` de `LineaPedidoEntity` y compilar (*Build › Rebuild Project*). Después, volver a ponerlo | Copia el aviso que aparece en la ventana *Build* y explica qué significa |
| P2-11 | Elegir "Olvidar mis datos" y abrir el historial | Historial vacío. Comprueba en el *Database Inspector* el contenido de las dos tablas |

## Opcional — Confirmar antes de borrar

Haz que el botón "Borrar pedido" abra un `AlertDialog` de confirmación ("¿Borrar este pedido?", botones "Borrar" y "Cancelar") y que el pedido solo se borre al confirmar. ¿Dónde debe guardarse si el diálogo está abierto: en el `ViewModel` o en la pantalla? ¿Qué pasa con el diálogo al girar el móvil en cada caso?

<div class="callout-info">
Este ejercicio es opcional: no cuenta en los requisitos técnicos ni en los criterios de evaluación.
</div>

## Requisitos técnicos obligatorios

- `LineaPedidoEntity` con `@ForeignKey` hacia `PedidoEntity`, `onDelete = ForeignKey.CASCADE` e índice sobre `pedido_id`; nombres de columna en minúsculas con guion bajo.
- `insertarConLineas` es una función con cuerpo del DAO marcada con `@Transaction`; `observarConLineas` lleva `@Transaction` y devuelve `Flow<PedidoConLineas?>`.
- Las funciones del DAO que devuelven `Flow` **no** son `suspend`; las demás, sí.
- `HistorialRepository` es la única clase que usa entidades; las conversiones son funciones de extensión privadas. `PedidoViewModel` no ha cambiado.
- `HistorialViewModel` y `DetallePedidoViewModel` usan `stateIn` con `WhileSubscribed(5_000)` y valor inicial `null`; las pantallas los leen con `collectAsStateWithLifecycle`.
- Cada pantalla nueva obtiene su `ViewModel` con `viewModel()` como valor por defecto; el detalle lee el id con `SavedStateHandle` y `toRoute`.
- Se navega atrás después de borrar, no antes.
- `ContenidoTicket` se usa en el diálogo y en el detalle; `ContenidoHistorial` es stateless y tiene previews.
- `PRUEBAS.md` con la sección "UD4 · Sesión 3" completa.

## Criterios de evaluación

Esta práctica contribuye a los criterios **f)** y **g)** del RA2:

- **f)** La app guarda los pedidos con sus líneas en dos tablas relacionadas, de forma atómica, y los muestra a través de repositorios y `ViewModel`, siempre actualizados con lo que hay en la base de datos.
- **g)** Se ha comprobado y documentado en el emulador cuándo consulta la app la base de datos (giros, segundo plano, cambios externos), el borrado en cascada y la recuperación del detalle tras la destrucción de la actividad.
- El proyecto compila y se ejecuta sin errores.
