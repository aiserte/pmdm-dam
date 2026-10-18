---
title: "Cafetería del instituto: carta completa y ViewModel — Enunciado (Sesión 4)"
unidad: UD3
sesion: 4
orden: 4
tipo: enunciado
---

## Contexto

Práctica de la **Sesión 4** (*Listas eficientes y ViewModel*). Seguimos con la app **Cafetería del instituto**, partiendo de la solución de la sesión 3: la app de pedidos con barra superior, total, datos del cliente y diálogo de confirmación.

Hoy no añadimos casi nada que el usuario vea, salvo una carta más grande con filtros por categoría. El trabajo de esta sesión es **por dentro**: cambiar la `Column` por una lista eficiente y sacar todo el estado y la lógica del pedido de `PantallaCarta` a un `ViewModel`. Al terminar, la app debe comportarse **igual que antes** (o mejor), pero con una estructura de clases profesional.

<div class="callout-warning">
Usa lo visto hasta ahora: todo lo de las sesiones 1 a 3 más <code>LazyColumn</code>, <code>LazyRow</code>, <code>FilterChip</code>, la dependencia de <code>ViewModel</code>, <code>PedidoUiState</code> y referencias a funciones (<code>::</code>). Todavía <strong>no</strong> hemos visto navegación entre pantallas: todo sigue en una sola pantalla.
</div>

<div class="callout-info">
Si no terminaste la práctica de la sesión 3, parte de su solución publicada. Guarda bien el resultado de hoy: es el punto de partida de la sesión 5.
</div>

## Enunciado

### Ejercicio 1 — Un modelo más completo

1. En `data/Producto.kt`, crea un `enum class Categoria(@StringRes val nombre: Int)` con tres valores: `BEBIDA`, `COMIDA` y `BOLLERIA`, cada uno con su recurso de texto (`categoria_bebida` → "Bebidas", `categoria_comida` → "Comida", `categoria_bolleria` → "Bollería"). Añade también el recurso `categoria_todas` → "Todo".
2. Añade a `Producto` dos propiedades nuevas: `id: Int` (primera propiedad) y `categoria: Categoria` (última).
3. Amplía `DatosCarta` hasta **al menos 8 productos**, con ids únicos y de las tres categorías. Puedes reutilizar los iconos que ya tienes. Sugerencia de productos nuevos: té, agua mineral, napolitana de chocolate y sándwich mixto.
4. Añade los textos nuevos (nombres, descripciones y categorías) en español y en inglés.

### Ejercicio 2 — `LazyColumn`

En `PantallaCarta`, sustituye la `Column` con `verticalScroll` por una `LazyColumn` que contenga, en este orden:

- la cabecera, como `item`;
- los productos, con `items` y usando `producto.id` como `key`;
- los datos del pedido, como `item`.

Usa `contentPadding` para el margen de 16 dp y mantén el `imePadding`. A partir de ahora, la clave del mapa de cantidades es `producto.id` (ya no `producto.nombre`).

### Ejercicio 3 — Filtro por categorías con `LazyRow`

Crea en `ui/carta/FiltroCategorias.kt` un composable stateless `FiltroCategorias(seleccionada: Categoria?, onSeleccionar: (Categoria?) -> Unit, modifier: Modifier = Modifier)` con una `LazyRow` de `FilterChip`: uno "Todo" (seleccionado cuando `seleccionada` es `null`) y uno por cada categoría. Colócalo como `item` en la `LazyColumn`, entre la cabecera y los productos. Al elegir una categoría, la carta solo muestra los productos de esa categoría.

<div class="callout-warning">
El filtro solo cambia qué productos se <strong>ven</strong>. Las cantidades de los productos ocultos se mantienen y siguen contando en el total y en las unidades del pedido.
</div>

### Ejercicio 4 — Dependencia de `ViewModel`

Añade la librería `lifecycle-viewmodel-compose` al catálogo `gradle/libs.versions.toml` y a `app/build.gradle.kts`, como se explica en el apartado 4.6 de la teoría, y sincroniza Gradle.

### Ejercicio 5 — Estado de la interfaz y `ViewModel`

Crea el paquete `ui/carta` (ya existe) con dos ficheros nuevos:

1. `PedidoUiState.kt`: una `data class PedidoUiState` con todo el estado del pedido que antes estaba en `PantallaCarta` (cantidades, nombre, para llevar, avisar, intento de confirmar, diálogo visible) **más** la categoría seleccionada (`Categoria?`, `null` = todas). Todos los campos con valor por defecto.
2. `PedidoViewModel.kt`: una clase `PedidoViewModel : ViewModel()` con:
   - la lista de productos (`DatosCarta.productos`);
   - `var uiState by mutableStateOf(PedidoUiState())` con `private set`;
   - propiedades calculadas: `productosVisibles` (según la categoría), `unidades`, `total` y `errorNombre`;
   - una función por cada evento: `cambiarCantidad(idProducto, cantidad)`, `cambiarNombre`, `cambiarParaLlevar`, `cambiarAvisar`, `seleccionarCategoria`, `vaciarPedido`, `confirmar`, `cancelarConfirmacion` y `enviarPedido`;
   - un `Log.d` con la etiqueta `"PedidoVM"` en un bloque `init` ("PedidoViewModel creado") y otro en `onCleared()` ("PedidoViewModel destruido").

### Ejercicio 6 — Pantalla con `ViewModel` y contenido stateless

1. Divide `PantallaCarta` en dos funciones:
   - `PantallaCarta(modifier, pedidoViewModel: PedidoViewModel = viewModel())`: obtiene el `ViewModel`, monta el `Scaffold` con sus barras y su `SnackbarHost`, y muestra el diálogo de confirmación. **No** debe quedar ningún `rememberSaveable`.
   - `ContenidoCarta(...)`: stateless; recibe la lista de productos visibles, el `PedidoUiState`, si hay error en el nombre y las lambdas de los eventos, y dibuja la `LazyColumn`.
2. Conecta los eventos con **referencias a funciones** del `ViewModel` siempre que los parámetros coincidan (por ejemplo, `onConfirmar = pedidoViewModel::confirmar`).
3. `MainActivity` llama a `PantallaCarta(modifier = Modifier.fillMaxSize())` (ya no le pasa los productos).
4. Mueve las previews a `ContenidoCarta`, con un `PedidoUiState` de ejemplo que tenga cantidades y nombre.

### Ejercicio 7 — Pruebas en el emulador

Añade a `PRUEBAS.md` una sección **"Sesión 4"** con estas pruebas. En las de Logcat, filtra con `tag:CicloVida | tag:PedidoVM` para ver a la vez la `Activity` y el `ViewModel`.

| # | Prueba | Resultado esperado |
|---|---|---|
| P4-1 | Desplazar la carta completa, en vertical y en horizontal | Se llega a todos los productos y a los datos del pedido |
| P4-2 | Abrir la app y girar la pantalla dos veces | La `Activity` se destruye y se crea cada vez; "PedidoViewModel creado" aparece **una sola vez** |
| P4-3 | Hacer un pedido, escribir el nombre, elegir una categoría y girar | Se conserva todo, sin ningún `rememberSaveable` |
| P4-4 | Desplazar la carta hasta la mitad y girar | La lista vuelve a la misma posición |
| P4-5 | Añadir un croissant, filtrar por "Bebidas" | El croissant no se ve, pero sigue contando en el total |
| P4-6 | Con *No conservar actividades*: hacer un pedido, pulsar Inicio y volver | Documenta qué pasa con el pedido y qué muestra Logcat, y explícalo con la tabla del apartado 4.9 |

### Opcional — `SavedStateHandle`

Haz que el pedido sobreviva también a la prueba P4-6: el `ViewModel` recibe un `SavedStateHandle` en su constructor (`class PedidoViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel()`; `viewModel()` se lo pasa automáticamente), guarda en él las cantidades y el nombre cada vez que cambian, y los recupera al crearse.

<div class="callout-info">
Este ejercicio es opcional: no cuenta en los requisitos técnicos ni en los criterios de evaluación.
</div>

## Requisitos técnicos obligatorios

- `Producto` con `id` y `categoria`; `enum class Categoria` con recursos de texto; al menos 8 productos con textos en español e inglés.
- `LazyColumn` con `item`/`items`, `key` por `id` y `contentPadding`; ninguna `Column` con `verticalScroll` en la pantalla.
- `FiltroCategorias` stateless con `LazyRow` y `FilterChip`.
- Dependencia añadida mediante el catálogo de versiones.
- `PedidoUiState` (data class con valores por defecto) y `PedidoViewModel` con `uiState` de solo lectura desde fuera (`private set`), propiedades calculadas y una función por evento.
- `PantallaCarta` sin `rememberSaveable`; `ContenidoCarta` stateless con preview.
- Al menos tres eventos conectados con referencias a funciones (`::`).
- `PRUEBAS.md` con la sección "Sesión 4" completa.

## Criterios de evaluación

Esta práctica contribuye a los criterios **a)**, **b)** y **g)** del RA2:

- **a)** La app tiene una estructura de clases por capas: modelo (`Producto`, `Categoria`, `DatosCarta`), estado de la interfaz (`PedidoUiState`), lógica (`PedidoViewModel`) e interfaz (composables stateless).
- **b)** Se usan correctamente las listas perezosas (`LazyColumn`, `LazyRow`) y los controles de filtro (`FilterChip`).
- **g)** Se ha comprobado y documentado en el emulador y en Logcat la diferencia entre el ciclo de vida de la `Activity` y el del `ViewModel`.
- El proyecto compila y se ejecuta sin errores.
