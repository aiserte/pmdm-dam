---
title: "Cafetería del instituto: confirmar el pedido — Enunciado (Sesión 3)"
unidad: UD3
sesion: 3
orden: 3
tipo: enunciado
---

## Contexto

Práctica de la **Sesión 3** (*State hoisting, controles de entrada, diálogos y menús*). Seguimos con la app **Cafetería del instituto**, partiendo de la solución de la sesión 2: la carta con un selector de cantidad en cada producto.

Hoy la app pasa de "carta con contadores" a **app de pedidos**: calcularemos el total, pediremos el nombre del cliente y sus opciones, y confirmaremos el pedido con un diálogo. Para conseguirlo, lo primero es sacar la cantidad de cada tarjeta y subirla a la pantalla (*state hoisting*).

<div class="callout-warning">
Usa lo visto hasta ahora: todo lo de las sesiones 1 y 2 más <em>state hoisting</em>, <code>OutlinedTextField</code>, <code>Switch</code>, <code>Checkbox</code>, <code>Scaffold</code> con <code>TopAppBar</code> y <code>BottomAppBar</code>, <code>DropdownMenu</code>, <code>AlertDialog</code> y <code>Snackbar</code>. Todavía <strong>no</strong> hemos visto <code>LazyColumn</code> ni <code>ViewModel</code>: no los necesitas, todo el estado vive en <code>PantallaCarta</code>.
</div>

<div class="callout-info">
Si no terminaste la práctica de la sesión 2, parte de su solución publicada. Guarda bien el resultado de hoy: es el punto de partida de la sesión 4.
</div>

## Enunciado

### Ejercicio 1 — Recursos nuevos

1. Añade a `res/drawable`, con *Vector Asset → Clip art*, el icono `more_vert` (los tres puntos verticales).
2. Añade a `strings.xml` estos textos y su traducción al inglés en `values-en`:

| Recurso | Texto en español |
|---|---|
| `mas_opciones` | Más opciones |
| `vaciar_pedido` | Vaciar pedido |
| `pedido_vaciado` | Pedido vaciado |
| `total_pedido` | Total: %1$.2f € |
| `unidades_pedido` | Unidades: %1$d |
| `confirmar` | Confirmar |
| `nombre_pedido` | ¿A nombre de quién? |
| `error_nombre` | Escribe un nombre para el pedido |
| `para_llevar` | Para llevar |
| `avisar_listo` | Avisarme cuando esté listo |
| `titulo_confirmacion` | ¿Enviar el pedido? |
| `resumen_pedido` | A nombre de %1$s · %2$d unidades · %3$.2f € |
| `resumen_aqui` | Para tomar aquí |
| `resumen_aviso` | Te avisaremos cuando esté listo |
| `enviar` | Enviar |
| `cancelar` | Cancelar |
| `pedido_enviado` | Pedido de %1$s enviado |

### Ejercicio 2 — State hoisting en `ProductoCard`

1. Convierte `ProductoCard` en un composable **stateless**: elimina su `rememberSaveable` y añade los parámetros `cantidad: Int` y `onCantidadChange: (Int) -> Unit`. Los botones "−" y "+" deben llamar a `onCantidadChange` con el nuevo valor.
2. En `PantallaCarta`, guarda la cantidad de todos los productos en un único estado: un `Map<Int, Int>` cuya clave es el id del nombre del producto (`producto.nombre`), con `rememberSaveable`.
3. Actualiza la preview de `ProductoCard` para que compile (pasándole una cantidad fija y una lambda vacía).
4. Comprueba que los selectores siguen funcionando y que las cantidades se conservan al girar.

### Ejercicio 3 — `Scaffold` dentro de la pantalla

1. Mueve el `Scaffold` de `MainActivity` a `PantallaCarta`. `MainActivity` debe quedar con su tema y una llamada a `PantallaCarta(productos = DatosCarta.productos)`. **Conserva los métodos del ciclo de vida y sus `Log.d` de la sesión 2.**
2. El contenido de la carta (cabecera y tarjetas) va dentro del `Scaffold`, aplicando el `innerPadding` y conservando el desplazamiento vertical.

### Ejercicio 4 — Barra superior con menú

Crea en `ui/carta/BarraSuperiorCarta.kt` un composable `BarraSuperiorCarta(hayPedido: Boolean, onVaciarPedido: () -> Unit, modifier: Modifier = Modifier)` con:

- una `TopAppBar` con el nombre de la app como título y el color `primaryContainer` del tema de fondo;
- en sus acciones, un `IconButton` con el icono `more_vert` (con `contentDescription`) que abre un `DropdownMenu`;
- en el menú, la opción **Vaciar pedido**, deshabilitada si no hay ninguna unidad en el pedido.

Úsala como `topBar` del `Scaffold`. Al vaciar el pedido, todas las cantidades vuelven a 0 y aparece un `Snackbar` con el texto "Pedido vaciado".

### Ejercicio 5 — Barra inferior con el total

Crea en `ui/carta/BarraPedido.kt` un composable `BarraPedido(total: Double, unidades: Int, onConfirmar: () -> Unit, modifier: Modifier = Modifier)` con una `BottomAppBar` que muestre, a la izquierda, el total y el número de unidades del pedido y, a la derecha, un botón **Confirmar**, habilitado solo si hay alguna unidad.

En `PantallaCarta`, calcula el total y las unidades **a partir del estado** (sin guardarlos en variables de estado aparte) y usa `BarraPedido` como `bottomBar`.

### Ejercicio 6 — Datos del pedido

Crea en `ui/carta/DatosPedido.kt` un composable stateless `DatosPedido` que muestre, dentro de una `Card`:

- un `OutlinedTextField` para el nombre, de una línea, con mayúscula inicial en cada palabra y que pueda mostrar el error `error_nombre`;
- una fila con el texto "Para llevar" y un `Switch`;
- una fila con un `Checkbox` y el texto "Avisarme cuando esté listo" (marcado por defecto).

Recibe por parámetro los tres valores, sus tres lambdas de cambio y si debe mostrar el error. Colócalo al final de la carta, después de los productos. Los tres valores se guardan en `PantallaCarta` con `rememberSaveable`. Añade `Modifier.imePadding()` al contenedor desplazable para que el teclado no tape el campo.

### Ejercicio 7 (integrador) — Confirmar con un diálogo

Al pulsar **Confirmar** en la barra inferior:

1. Si el nombre está vacío, el campo se marca con el error y no ocurre nada más.
2. Si hay nombre, aparece un `AlertDialog` (en `ui/carta/DialogoConfirmacion.kt`) con el título "¿Enviar el pedido?" y como texto: el resumen (`resumen_pedido`), "Para llevar" o "Para tomar aquí" según el `Switch`, y "Te avisaremos cuando esté listo" solo si el `Checkbox` está marcado.
3. **Enviar** cierra el diálogo, vacía las cantidades y muestra un `Snackbar` "Pedido de *nombre* enviado". **Cancelar**, tocar fuera o pulsar Atrás solo cierran el diálogo.
4. La visibilidad del diálogo se guarda con `rememberSaveable`.

### Ejercicio 8 — Pruebas en el emulador

Añade a `PRUEBAS.md` una sección **"Sesión 3"** con estas pruebas, su resultado y, si algo falla, qué ocurre:

| # | Prueba | Resultado esperado |
|---|---|---|
| P3-1 | Añadir productos y comprobar la barra inferior | El total y las unidades cambian al momento |
| P3-2 | Escribir un nombre y girar la pantalla | El nombre, el `Switch` y el `Checkbox` se conservan |
| P3-3 | Tocar el campo del nombre con la carta desplazada hasta abajo | El teclado no tapa el campo |
| P3-4 | Pulsar Confirmar sin nombre | El campo muestra el error y no se abre el diálogo |
| P3-5 | Abrir el diálogo y girar la pantalla | El diálogo sigue abierto con el mismo resumen |
| P3-6 | Abrir el diálogo con Logcat filtrado por `tag:CicloVida` | No aparece ningún método del ciclo de vida |
| P3-7 | Activar TalkBack y tocar el botón de tres puntos | Lee "Más opciones" |
| P3-8 | Repetir el flujo completo en inglés | Todos los textos nuevos aparecen traducidos |

### Opcional — "Deshacer" al vaciar

Haz que el `Snackbar` de "Pedido vaciado" tenga una acción **Deshacer** (añade el recurso `deshacer`) que restaure las cantidades que había antes de vaciar.

<div class="callout-info">
Este ejercicio es opcional: no cuenta en los requisitos técnicos ni en los criterios de evaluación.
</div>

## Requisitos técnicos obligatorios

- `ProductoCard`, `DatosPedido`, `BarraPedido` y `DialogoConfirmacion` son **stateless** (no contienen `remember` ni `rememberSaveable`). `BarraSuperiorCarta` solo guarda si el menú está abierto.
- Todo el estado del pedido (cantidades, nombre, para llevar, avisar, diálogo visible) vive en `PantallaCarta` con `rememberSaveable`.
- El total y las unidades se **calculan** a partir de las cantidades, no se guardan como estado aparte.
- `Scaffold` con `topBar`, `bottomBar` y `snackbarHost`; contenido con `innerPadding`, desplazamiento vertical e `imePadding`.
- Ningún `stringResource` dentro de un `onClick` o un `launch`.
- Todos los textos nuevos en `strings.xml`, en español y en inglés; `contentDescription` en el botón del menú.
- `PRUEBAS.md` con la sección "Sesión 3" completa.
- `MainActivity` conserva los métodos del ciclo de vida de la sesión 2.

## Criterios de evaluación

Esta práctica contribuye a los criterios **a)**, **b)** y **g)** del RA2:

- **a)** La estructura de clases sigue el patrón de *state hoisting*: un componente de pantalla con el estado y componentes stateless en ficheros propios.
- **b)** Se usan correctamente los equivalentes Compose de ventanas (`Scaffold`, `TopAppBar`, `BottomAppBar`), menús (`DropdownMenu`), alertas (`AlertDialog`, `Snackbar`) y controles (`OutlinedTextField`, `Switch`, `Checkbox`, `Button`).
- **g)** Se han realizado y documentado las pruebas de interacción: teclado, giro de pantalla con diálogo abierto, validación, accesibilidad e idioma.
- El proyecto compila y se ejecuta sin errores.
