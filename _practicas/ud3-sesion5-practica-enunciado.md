---
title: "Cafetería del instituto: varias pantallas — Enunciado (Sesión 5)"
unidad: UD3
sesion: 5
orden: 5
tipo: enunciado
---

## Contexto

Práctica de la **Sesión 5** (*Navegación entre pantallas con Navigation Compose*). Partimos de la solución de la sesión 4: una sola pantalla con la carta filtrable, los datos del pedido y el diálogo de confirmación, con todo el estado en `PedidoViewModel`.

Hoy la cafetería pasa a tener **cuatro pantallas**:

| Pantalla | Contenido |
|---|---|
| **Carta** | Cabecera, filtro y productos. Tocar una tarjeta abre su detalle. La barra inferior muestra el total y un botón "Ver pedido". |
| **Detalle** | El producto en grande (icono, nombre, categoría, descripción y precio) con el selector de cantidad. Flecha ← para volver. |
| **Resumen** | Las líneas del pedido con su subtotal, los datos del cliente y el botón "Confirmar", que abre el diálogo de la sesión 3. Flecha ← para volver. |
| **Pedido enviado** | Un mensaje de agradecimiento con el nombre del cliente y un botón "Volver a la carta". |

Las tres primeras comparten el **mismo** pedido: lo que se añada en el detalle debe verse en la carta y en el resumen.

<div class="callout-warning">
Usa lo visto hasta ahora: todo lo de las sesiones 1 a 4 más Navigation Compose (rutas <code>@Serializable</code>, <code>NavHost</code>, <code>navigate</code>, <code>navigateUp</code>, <code>popBackStack</code>, <code>launchSingleTop</code>, <code>popUpTo</code>, <code>toRoute</code>). Todavía <strong>no</strong> guardamos nada al cerrar la app: eso es de la sesión 6.
</div>

<div class="callout-info">
Si no terminaste la práctica de la sesión 4, parte de su solución publicada. Para no tener que mover ficheros, las pantallas nuevas van también en el paquete <code>ui/carta</code>; solo la navegación tiene paquete propio, <code>ui/navegacion</code>.
</div>

## Enunciado

### Ejercicio 1 — Dependencias y recursos

1. Añade Navigation Compose, `kotlinx-serialization-json` y el plugin de serialización, como se explica en el apartado 5.3 de la teoría, y sincroniza Gradle.
2. Añade dos iconos con *Vector Asset*: `arrow_back` y `check_circle`.
3. Añade estos textos en español y en inglés:

| Recurso | Español |
|---|---|
| `ver_pedido` | Ver pedido |
| `volver` | Volver |
| `titulo_resumen` | Tu pedido |
| `pedido_vacio` | Todavía no has añadido nada al pedido. |
| `linea_pedido` | %1$d × %2$s |
| `titulo_enviado` | ¡Pedido enviado! |
| `mensaje_enviado` | Gracias, %1$s. Ya estamos preparando tu pedido. |
| `volver_carta` | Volver a la carta |

El texto `pedido_enviado` de la sesión 3 ya no se usará (el aviso pasa a ser una pantalla); puedes borrarlo.

### Ejercicio 2 — Las rutas

Crea `ui/navegacion/Rutas.kt` con una ruta `@Serializable` por pantalla: `Carta` y `Resumen` sin argumentos (`data object`), `Detalle` con el id del producto y `PedidoEnviado` con el nombre del cliente (`data class`).

### Ejercicio 3 — Componentes reutilizables

1. **`SelectorCantidad.kt`**: saca de `ProductoCard` la fila con los botones − y + y la cantidad a un composable stateless `SelectorCantidad(cantidad, onCantidadChange, modifier)`. `ProductoCard` lo usa alineado a la derecha.
2. **`ProductoCard`**: añade un parámetro `onClick: () -> Unit` y haz que la tarjeta entera sea pulsable (`Card(onClick = onClick, ...)`).
3. **`BarraPedido`**: generalízala para que sirva en la carta y en el resumen. En lugar de `onConfirmar`, recibe el texto del botón (`@StringRes textoBoton: Int`) y su acción (`onBotonClick`).
4. **`BarraConVolver.kt`**: una `TopAppBar` que recibe el título como `String` y muestra, en su parámetro `navigationIcon`, una flecha ← que llama a `onVolver`.

### Ejercicio 4 — El `ViewModel` para varias pantallas

En `PedidoUiState.kt` añade una `data class LineaPedido(val producto: Producto, val cantidad: Int)`. En `PedidoViewModel` añade:

- `fun buscarProducto(id: Int): Producto?`
- una propiedad calculada `lineasPedido: List<LineaPedido>` con los productos cuya cantidad es mayor que 0.

El resto del `ViewModel` no cambia.

### Ejercicio 5 — Pantalla de la carta

Modifica `PantallaCarta`:

1. Ya no obtiene el `ViewModel` con `viewModel()`: lo recibe como parámetro **sin valor por defecto**, junto con dos lambdas, `onProductoClick: (Int) -> Unit` y `onVerPedido: () -> Unit`.
2. La barra inferior usa `BarraPedido` con el texto "Ver pedido".
3. Los datos del pedido y el diálogo de confirmación **salen** de esta pantalla (pasan al resumen).
4. Ajusta `ContenidoCarta` y su preview: recibe las cantidades, la categoría seleccionada y `onProductoClick`, en lugar del `PedidoUiState` completo.

### Ejercicio 6 — Pantalla de detalle

Crea `PantallaDetalle(idProducto, pedidoViewModel, onVolver, modifier)`:

- busca el producto con `buscarProducto` (si no existe, no dibuja nada);
- `Scaffold` con `BarraConVolver`, cuyo título es el nombre del producto;
- un `ContenidoDetalle` stateless con preview: icono de 120 dp, nombre, categoría, descripción, precio y `SelectorCantidad`, centrados y con desplazamiento vertical para que se vea entero en horizontal.

### Ejercicio 7 — Pantalla de resumen

Crea `PantallaResumen(pedidoViewModel, onVolver, onPedidoEnviado: (String) -> Unit, modifier)`:

- `Scaffold` con `BarraConVolver` ("Tu pedido") y `BarraPedido` con el botón "Confirmar" (`pedidoViewModel::confirmar`);
- un `ContenidoResumen` stateless con una `LazyColumn`: el texto `pedido_vacio` si no hay líneas, una `FilaPedido` por línea (cantidad × nombre a la izquierda, subtotal a la derecha) y `DatosPedido` al final;
- el `DialogoConfirmacion` de la sesión 3. Al pulsar "Enviar": envía el pedido en el `ViewModel` y llama a `onPedidoEnviado` con el nombre del cliente.

### Ejercicio 8 — Pantalla de pedido enviado

Crea `PantallaPedidoEnviado(nombre, onVolverCarta, modifier)`, centrada vertical y horizontalmente: icono `check_circle` de 96 dp, el título "¡Pedido enviado!", el mensaje con el nombre y un botón "Volver a la carta".

### Ejercicio 9 — La navegación

1. Crea `ui/navegacion/AppCafeteria.kt` con el `NavController`, el `PedidoViewModel` (obtenido **una sola vez**, fuera del `NavHost`) y el `NavHost` con las cuatro pantallas.
2. Conecta las acciones:
   - tocar un producto → `Detalle` con su id;
   - "Ver pedido" → `Resumen`, sin repetirla si se pulsa dos veces (`launchSingleTop`);
   - flecha ← → `navigateUp()`;
   - enviar el pedido → `PedidoEnviado`, quitando antes de la pila todo lo que haya encima de la carta (`popUpTo<Carta>()`);
   - "Volver a la carta" → `popBackStack()`.
3. `MainActivity` dibuja `AppCafeteria()` dentro del tema.

### Ejercicio 10 — Pruebas en el emulador

Añade a `PRUEBAS.md` una sección **"Sesión 5"**. En las pruebas de pila, dibuja la pila de pantallas (como en la teoría) antes y después de la acción.

| # | Prueba | Resultado esperado |
|---|---|---|
| P5-1 | Abrir el detalle del café, pulsar + dos veces y volver con la flecha | La carta muestra 2 cafés y el total correcto |
| P5-2 | Pulsar "Ver pedido" dos veces muy rápido y luego Atrás. Repetir quitando temporalmente `launchSingleTop` | Con `launchSingleTop`, un solo Atrás vuelve a la carta. Documenta qué pasa sin él |
| P5-3 | En el resumen, pulsar "Confirmar" sin nombre | Aparece el error del nombre y no se abre el diálogo |
| P5-4 | Enviar un pedido y, en "Pedido enviado", pulsar el botón Atrás del sistema | Vuelve a la carta (no al resumen) y el pedido está vacío |
| P5-5 | Abrir el resumen con el pedido vacío | Se ve el mensaje `pedido_vacio` y "Confirmar" está desactivado |
| P5-6 | Girar la pantalla en el detalle de un producto con cantidad | Se mantiene la misma pantalla, el mismo producto y la cantidad |
| P5-7 | Con Logcat filtrado por `tag:PedidoVM`, recorrer las cuatro pantallas | "PedidoViewModel creado" aparece una sola vez |
| P5-8 | Experimento: cambia temporalmente `PantallaDetalle` para que obtenga su propio `viewModel()` dentro de `composable<Detalle>`. Abre el detalle de un producto que tenga cantidad | Documenta qué cantidad se ve y qué muestra Logcat al entrar y salir del detalle; explícalo con el apartado 5.8. **Deshaz el cambio** |
| P5-9 | Con *No conservar actividades*: añadir cantidad en el detalle de un producto, pulsar Inicio y volver | Documenta qué pantalla aparece y qué pasa con la cantidad; explícalo con el apartado 5.9 |

### Opcional — Transiciones animadas

Añade al `NavHost` los parámetros `enterTransition`, `exitTransition`, `popEnterTransition` y `popExitTransition` para que las pantallas entren deslizándose desde la derecha al avanzar y desde la izquierda al volver (pista: `slideIntoContainer` y `slideOutOfContainer` con `AnimatedContentTransitionScope.SlideDirection`).

<div class="callout-info">
Este ejercicio es opcional: no cuenta en los requisitos técnicos ni en los criterios de evaluación.
</div>

## Requisitos técnicos obligatorios

- Dependencias de navegación y serialización añadidas mediante el catálogo de versiones, con el plugin declarado en la raíz y aplicado en `app`.
- Cuatro rutas `@Serializable` en `Rutas.kt`, dos de ellas con argumentos.
- Un único `NavHost` en `AppCafeteria`; ninguna pantalla recibe el `NavController` (solo lambdas).
- Un único `PedidoViewModel`, obtenido fuera del `NavHost` y pasado como parámetro sin valor por defecto.
- Por la ruta del detalle solo viaja el id del producto.
- `launchSingleTop` en "Ver pedido" y `popUpTo<Carta>()` al enviar el pedido.
- `SelectorCantidad`, `BarraConVolver`, `ContenidoCarta`, `ContenidoDetalle` y `ContenidoResumen` stateless, los tres últimos con preview.
- Textos nuevos en español e inglés.
- `PRUEBAS.md` con la sección "Sesión 5" completa, incluidas las pilas dibujadas.

## Criterios de evaluación

Esta práctica contribuye a los criterios **a)**, **b)** y **g)** del RA2:

- **a)** La app tiene una estructura de navegación clara: rutas tipadas, un punto central (`AppCafeteria`) y pantallas que no dependen del `NavController`; el estado se comparte mediante un único `ViewModel`.
- **b)** Se usan correctamente las ventanas de la app (`Scaffold` por pantalla, `TopAppBar` con navegación hacia atrás, barras inferiores) y los controles necesarios en cada una.
- **g)** Se ha comprobado y documentado en el emulador el comportamiento de la pila de pantallas, de los giros y de la destrucción de la actividad.
- El proyecto compila y se ejecuta sin errores.
