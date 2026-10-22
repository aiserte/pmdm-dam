---
title: "Listas eficientes y ViewModel"
unit: UD3
order: 4
duration: "2h"
---

## Objetivos

- Mostrar listas largas de forma eficiente con `LazyColumn` y `LazyRow`, usando claves (`key`) para identificar cada elemento.
- Combinar en una misma lista elementos de distinto tipo (cabecera, filtros, productos, formulario).
- Entender qué es un `ViewModel`, cuánto vive comparado con la `Activity` y por qué se usa en las apps reales.
- Separar la app en capas: **estado de la interfaz** (`data class`), **lógica** (`ViewModel`) e **interfaz** (composables stateless).
- Añadir una dependencia nueva al proyecto con Gradle.
- Saber qué mecanismo conserva cada dato frente a giros de pantalla y a la destrucción de la app.

<div class="callout-info">
Esta sesión trabaja sobre todo el criterio <strong>a)</strong> del RA2, "generar la estructura de clases necesaria para la aplicación": al terminar, la cafetería tendrá una estructura de clases como la de una app profesional (modelo de datos, estado de la interfaz, <code>ViewModel</code> y pantallas).
</div>

## 4.1 El problema de `Column` con muchos elementos

Hasta ahora la carta era una `Column` con `forEach`:

```kotlin
Column(modifier = Modifier.verticalScroll(rememberScrollState())) {
    productos.forEach { producto -> ProductoCard(producto, ...) }
}
```

Con 4 productos no se nota, pero `Column` **crea y dibuja todos sus hijos**, se vean o no. Con una carta de 200 productos, la app tardaría en abrirse y gastaría memoria y batería en tarjetas que el usuario quizá nunca llegue a ver.

`LazyColumn` (*columna perezosa*) solo compone los elementos que caben en pantalla, más unos pocos de margen. Cuando el usuario desplaza la lista, compone los que entran y descarta los que salen. Es el equivalente en Compose al `RecyclerView` del Android clásico, pero sin adaptadores ni *ViewHolders*.

## 4.2 `LazyColumn`: items, claves y separación

```kotlin
LazyColumn(
    contentPadding = PaddingValues(16.dp),
    verticalArrangement = Arrangement.spacedBy(12.dp)
) {
    item {
        CabeceraCarta()
    }
    items(items = productos, key = { producto -> producto.id }) { producto ->
        ProductoCard(producto = producto, ...)
    }
    item {
        DatosPedido(...)
    }
}
```

- Dentro de las llaves de `LazyColumn` **no se escriben composables directamente**: se usa un pequeño lenguaje propio (*DSL*) con dos funciones, `item { }` para un elemento suelto e `items(lista) { }` para uno por cada elemento de una lista. Dentro de sus lambdas sí van los composables.
- Se pueden combinar libremente: cabecera, filtros, lista de productos y formulario, en el orden que se quiera, todo dentro de la **misma** lista desplazable.
- `contentPadding` añade margen alrededor del contenido sin recortar el desplazamiento (a diferencia de `Modifier.padding`).
- `LazyColumn` ya se desplaza sola: **no** hace falta `verticalScroll`.

### Por qué la clave (`key`)

Sin `key`, Compose identifica cada elemento por su **posición**. Si la lista cambia (se filtra, se reordena, se borra un elemento), el elemento que estaba en la posición 2 puede pasar a ser otro producto, y Compose puede mezclar su estado o hacer animaciones extrañas. Con `key = { it.id }` cada elemento se identifica por un valor **único y estable**. Por eso, en la práctica de hoy, `Producto` recibe un campo `id`.

<div class="callout-warning">
<strong>Error muy habitual:</strong> meter una <code>LazyColumn</code> dentro de una <code>Column</code> con <code>verticalScroll</code>. La app se cierra con <code>IllegalStateException: Vertically scrollable component was measured with an infinity maximum height constraints</code>. Una lista perezosa necesita saber cuánto espacio tiene, y un contenedor desplazable le ofrece altura infinita. La solución es no anidarlas: si hay más elementos antes o después de la lista, se meten <strong>dentro</strong> de la propia <code>LazyColumn</code> con <code>item { }</code>, como en el ejemplo anterior.
</div>

## 4.3 `LazyRow` y filtros con `FilterChip`

`LazyRow` es lo mismo en horizontal. Un uso típico es una fila de filtros que se puede desplazar de lado si no cabe:

```kotlin
LazyRow(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
    item {
        FilterChip(
            selected = seleccionada == null,
            onClick = { onSeleccionar(null) },
            label = { Text(stringResource(R.string.categoria_todas)) }
        )
    }
    items(Categoria.entries) { categoria ->
        FilterChip(
            selected = seleccionada == categoria,
            onClick = { onSeleccionar(categoria) },
            label = { Text(stringResource(categoria.nombre)) }
        )
    }
}
```

- `FilterChip` es un control de Material 3 pensado para filtros: una "pastilla" que se marca al seleccionarla.
- `Categoria.entries` es la lista de todos los valores de un `enum class` de Kotlin.
- Una `LazyRow` **sí** puede ir dentro de un `item` de una `LazyColumn`: tienen direcciones distintas y no hay conflicto.

Para cuadrículas existen `LazyVerticalGrid` y `LazyHorizontalGrid`, que funcionan igual pero reciben el número de columnas (`columns = GridCells.Fixed(2)`).

<div class="callout-info">
<strong>La posición de la lista se conserva sola.</strong> <code>LazyColumn</code> guarda internamente su posición de desplazamiento con <code>rememberLazyListState()</code>, que usa <code>rememberSaveable</code>: al girar la pantalla, la lista vuelve al mismo sitio. Si necesitáis controlarla (por ejemplo, para volver arriba con un botón), podéis crear el estado vosotros, <code>val estadoLista = rememberLazyListState()</code>, pasarlo con <code>LazyColumn(state = estadoLista)</code> y llamar a <code>estadoLista.animateScrollToItem(0)</code> dentro de un <code>scope.launch</code> (es una función <code>suspend</code>, como <code>showSnackbar</code>).
</div>

## 4.4 El problema de `PantallaCarta`: demasiadas responsabilidades

Recordemos cómo quedó `PantallaCarta` al final de la sesión 3. Este es un resumen de su código:

```kotlin
@Composable
fun PantallaCarta(productos: List<Producto>, modifier: Modifier = Modifier) {
    // 1. Seis estados guardados en la propia pantalla
    var cantidades by rememberSaveable { mutableStateOf(mapOf<Int, Int>()) }
    var nombre by rememberSaveable { mutableStateOf("") }
    var paraLlevar by rememberSaveable { mutableStateOf(false) }
    var avisar by rememberSaveable { mutableStateOf(true) }
    var intentoConfirmar by rememberSaveable { mutableStateOf(false) }
    var mostrarConfirmacion by rememberSaveable { mutableStateOf(false) }

    // 2. Cálculos y reglas del pedido
    val unidades = cantidades.values.sum()
    val total = productos.sumOf { (cantidades[it.nombre] ?: 0) * it.precio }
    val errorNombre = intentoConfirmar && nombre.isBlank()

    // 3. Y además, toda la interfaz
    Scaffold(
        bottomBar = {
            BarraPedido(
                total = total,
                unidades = unidades,
                onConfirmar = {
                    intentoConfirmar = true                              // regla del pedido...
                    if (nombre.isNotBlank()) mostrarConfirmacion = true  // ...escondida en un onClick
                }
            )
        }
        // ... barra superior, carta, datos del pedido, diálogo, snackbar
    ) { /* ... */ }
}
```

Funciona, pero una sola función hace tres trabajos muy distintos: **guarda** los datos del pedido, **decide** las reglas (cuánto se paga, cuándo se puede confirmar) y **dibuja** la pantalla. Eso trae cuatro problemas concretos en nuestra app:

1. **Las reglas del negocio están mezcladas con la interfaz.** La regla "no se puede confirmar un pedido sin nombre" está escondida dentro de la lambda `onConfirmar` de un botón. Si mañana el pedido también se pudiera confirmar desde otro sitio, habría que copiar esa regla.
2. **Otras pantallas no pueden ver el pedido.** En la sesión 5 añadiremos una pantalla de detalle de producto (con su propio botón "Añadir") y una pantalla de resumen del pedido. Las dos necesitan leer y modificar las mismas cantidades, pero esas cantidades son variables locales de `PantallaCarta` y, cuando el usuario cambie de pantalla, `PantallaCarta` ni siquiera estará en la interfaz. Es el mismo problema que la tarjeta "encerrada" de la sesión 3, pero ahora entre pantallas.
3. **`rememberSaveable` tiene límites.** Cada variable se guarda por separado en el `Bundle`, que solo admite tipos sencillos y datos pequeños. Una carta con cientos de productos, o datos que vengan de una base de datos, no caben ahí.
4. **La lógica no se puede probar por separado.** Para comprobar que el total se calcula bien hay que ejecutar la app y pulsar botones; no hay una clase a la que preguntarle "¿cuánto suman 2 cafés y 1 croissant?".

La solución es la misma idea que el *state hoisting*, llevada un paso más allá: sacar el estado y las reglas **fuera de la interfaz**, a una clase que no dibuja nada. En Android esa clase se llama **`ViewModel`**.

<div class="callout-info">
<strong>Una comparación con una cafetería de verdad.</strong> El camarero (la interfaz) enseña la carta, apunta lo que pides y te dice cuánto es; pero el pedido no lo guarda en la cabeza, sino en la comanda de la caja (el <code>ViewModel</code>). Si cambia el camarero a mitad del pedido, el nuevo mira la comanda y sigue donde iba el anterior. Eso es exactamente lo que pasará al girar la pantalla: la interfaz se destruye y se crea otra nueva, pero el pedido sigue en el <code>ViewModel</code>.
</div>

## 4.5 Qué es un `ViewModel`

Un `ViewModel` es **una clase de Kotlin normal**, sin nada de Compose, que hereda de la clase `ViewModel` de la librería *Jetpack Lifecycle* de Android. Guarda el estado de **una pantalla** y las funciones que lo modifican. La versión más pequeña posible para nuestra cafetería sería esta:

```kotlin
class PedidoViewModel : ViewModel() {

    var unidades by mutableStateOf(0)
        private set

    fun anadirUnidad() {
        unidades++
    }
}
```

No dibuja nada, no tiene `@Composable` y no sabe qué botones hay en pantalla. Solo guarda un dato (`unidades`) y ofrece una función para cambiarlo (`anadirUnidad`). La interfaz lo usaría así:

```kotlin
@Composable
fun PantallaCarta(pedidoViewModel: PedidoViewModel = viewModel()) {
    Button(onClick = { pedidoViewModel.anadirUnidad() }) {
        Text("Unidades: ${pedidoViewModel.unidades}")
    }
}
```

Hasta aquí parece lo mismo que un `remember`. La diferencia está en **quién crea el objeto y cuánto vive**.

### Quién lo crea: `viewModel()`

Vosotros **nunca** escribís `PedidoViewModel()`. Se pide a Android con la función `viewModel()`, y Android lo guarda en un almacén asociado a la `Activity` (el *ViewModelStore*). Lo que hace `viewModel()` es:

```
Primera vez que se abre la pantalla:
  PantallaCarta → viewModel() → ¿hay un PedidoViewModel guardado?
                                NO → lo crea y lo guarda

Después de girar (Activity nueva, PantallaCarta nueva):
  PantallaCarta → viewModel() → ¿hay un PedidoViewModel guardado?
                                SÍ → devuelve ESE MISMO objeto
```

Por eso el pedido sobrevive al giro sin ningún `rememberSaveable`: la interfaz nueva no crea un pedido vacío, sino que recupera el objeto que ya tenía los datos.

### Cuánto vive: más que la `Activity`

```
Activity:   creada ─► destruida/recreada ─► destruida/recreada ─► finalizada
            (abrir)   (girar pantalla)      (cambiar idioma)      (cerrar)

ViewModel:  creado ──────── el MISMO objeto todo el tiempo ────► onCleared()
```

En la sesión 2 visteis en Logcat que al girar la pantalla la `Activity` pasa por `onDestroy` y se crea otra con `onCreate`. Si añadimos al `ViewModel` un mensaje al crearse (en su bloque `init`) y otro al destruirse (en su método `onCleared()`), Logcat muestra algo así al abrir la app y girarla:

```
CicloVida   D  onCreate (savedInstanceState es null)
CicloVida   D  onStart
CicloVida   D  onResume
PedidoVM    D  PedidoViewModel creado   ← al dibujarse la pantalla
CicloVida   D  onPause                  ← se gira la pantalla
CicloVida   D  onStop
CicloVida   D  onDestroy                ← la Activity muere...
CicloVida   D  onCreate (savedInstanceState trae datos guardados)
CicloVida   D  onStart                  ← ...y nace otra, pero el ViewModel
CicloVida   D  onResume                    NO vuelve a crearse
```

El `ViewModel` se crea después de `onResume` porque se pide con `viewModel()` desde `PantallaCarta`, y la interfaz de Compose se compone por primera vez cuando la ventana ya es visible. "PedidoViewModel creado" aparece **una sola vez**, por muchas veces que giréis. Solo cuando la pantalla se cierra de verdad (y la `Activity` no va a volver a crearse) Android llama a `onCleared()` y el `ViewModel` desaparece. Lo comprobaréis en la práctica (prueba P4-2).

### Qué va dentro y qué no

| Va en el `ViewModel` | No va en el `ViewModel` |
|---|---|
| El estado de la pantalla: las cantidades del pedido, el nombre, si es para llevar, la categoría elegida | Composables, `Modifier`, colores… nada de la interfaz |
| Las reglas: calcular el total, decidir si se puede confirmar, vaciar, enviar | Referencias a la `Activity` o a su `Context` |
| Más adelante: el acceso a los datos (base de datos en la UD4, red en la UD5) | Estado puramente visual: si el menú de tres puntos está abierto, la posición de la lista, el `SnackbarHostState` |

¿Por qué no se puede guardar la `Activity` dentro del `ViewModel`? Porque el `ViewModel` vive **más** que ella. Al girar, Android destruye la `Activity` vieja; si el `ViewModel` siguiera apuntando a ella, esa `Activity` (con toda su interfaz) no se podría liberar de la memoria. Es lo que se llama una **fuga de memoria**, y se repetiría con cada giro.

## 4.6 Añadir la dependencia con Gradle

El `ViewModel` para Compose está en una librería que la plantilla **no** incluye. Hay que añadirla, y es un buen momento para recordar cómo se gestionan las dependencias en un proyecto Android moderno.

**Paso 1.** Abrid `gradle/libs.versions.toml` (el **catálogo de versiones**: un único sitio con las librerías y sus versiones). En la sección `[libraries]` añadid:

```
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleRuntimeKtx" }
```

`version.ref = "lifecycleRuntimeKtx"` reutiliza la versión de *Lifecycle* que la plantilla ya usa: así todas las librerías de la familia van con la misma versión.

**Paso 2.** Abrid `app/build.gradle.kts` y, dentro de `dependencies { }`, añadid:

```kotlin
implementation(libs.androidx.lifecycle.viewmodel.compose)
```

Los guiones del nombre en el catálogo se convierten en puntos al usarlo desde Gradle.

**Paso 3.** Pulsad **Sync Now** en la barra amarilla que aparece arriba. Hasta que Gradle no sincronice, Android Studio no reconocerá las clases nuevas.

<div class="callout-warning">
Si el nombre <code>lifecycleRuntimeKtx</code> no existe en la sección <code>[versions]</code> de vuestro catálogo (puede cambiar entre versiones de la plantilla), buscad la versión que usa la librería <code>lifecycle-runtime-ktx</code> y referenciad esa. También podéis escribir la versión directamente con <code>version = "..."</code>, copiando la misma que tenga <code>lifecycle-runtime-ktx</code>.
</div>

## 4.7 Construir el `ViewModel` de la cafetería, paso a paso

Vamos a sacar de `PantallaCarta` todo lo que no sea dibujar. Lo haremos en cinco pasos.

### Paso 1: decidir qué es estado y qué no

Repasamos todo lo que guardaba o calculaba `PantallaCarta` en la sesión 3 y lo clasificamos:

| En la sesión 3 | ¿Qué es? | ¿Dónde va ahora? |
|---|---|---|
| `cantidades`, `nombre`, `paraLlevar`, `avisar` | Datos del pedido | Estado, en el `ViewModel` |
| `intentoConfirmar`, `mostrarConfirmacion` | Situación del proceso de confirmar | Estado, en el `ViewModel` |
| Categoría seleccionada en el filtro (nueva hoy) | Elección del usuario | Estado, en el `ViewModel` |
| `total`, `unidades`, `errorNombre` | Se **calculan** a partir del estado | Propiedades calculadas del `ViewModel` (no se guardan) |
| `snackbarHostState`, `scope`, menú de tres puntos abierto | Detalles de cómo se muestra la pantalla | Se quedan en la interfaz con `remember` |

Esta clasificación es lo más importante de la sesión. Una pregunta ayuda a decidir: **si el usuario se fuera a otra pantalla y volviera, ¿esperaría encontrar este dato igual?** El pedido y el nombre, sí: van al `ViewModel`. Si el menú de tres puntos estaba abierto, no: se queda en la interfaz.

### Paso 2: agrupar el estado en una `data class`

En lugar de seis variables sueltas, todo el estado de la pantalla se reúne en una sola clase. Por convención se llama `NombrePantallaUiState` (*estado de la interfaz*):

```kotlin
data class PedidoUiState(
    val cantidades: Map<Int, Int> = emptyMap(),   // id del producto -> cantidad
    val nombre: String = "",
    val paraLlevar: Boolean = false,
    val avisar: Boolean = true,
    val intentoConfirmar: Boolean = false,
    val mostrarConfirmacion: Boolean = false,
    val categoria: Categoria? = null              // null = todas las categorías
)
```

- **Todos los campos son `val`** (no se pueden modificar) y **todos tienen valor por defecto**. Así, `PedidoUiState()` sin argumentos describe la pantalla tal como se ve al abrir la app: pedido vacío, sin nombre, "Avisarme" marcado y todas las categorías visibles.
- Para "cambiar" algo se crea **otro objeto** con `copy`, la función que las `data class` traen de serie (UD2). `copy` copia todos los campos y cambia solo los que le indicáis:

```kotlin
val inicio = PedidoUiState()                                // pedido vacío
val conCafe = inicio.copy(cantidades = mapOf(1 to 1))      // 1 café (id 1)
val conNombre = conCafe.copy(nombre = "Ana")               // 1 café, a nombre de Ana
val paraLlevar = conNombre.copy(paraLlevar = true)         // ... y para llevar

// inicio, conCafe y conNombre NO han cambiado: cada copy crea un objeto nuevo
```

¿Por qué tanto empeño en no modificar nada? Por lo mismo que con el `Map` de la sesión 3: Compose se entera de un cambio cuando el estado **pasa a ser otro objeto**. Si modificáramos un objeto por dentro, seguiría siendo "el mismo" y la pantalla no se actualizaría.

### Paso 3: la clase `PedidoViewModel` y su estado

```kotlin
class PedidoViewModel : ViewModel() {

    // Los productos de la carta (en la UD4 vendrán de una base de datos)
    val productos: List<Producto> = DatosCarta.productos

    // TODO el estado de la pantalla, en una sola variable observable
    var uiState by mutableStateOf(PedidoUiState())
        private set
}
```

Línea a línea:

- **`: ViewModel()`**: hereda de la clase de Android. Es lo que permite que `viewModel()` lo cree, lo guarde y lo conserve al girar.
- **`mutableStateOf(PedidoUiState())`**: el mismo `mutableStateOf` de la sesión 2. Guarda un `PedidoUiState` y **avisa a Compose** cada vez que se le asigna otro. El valor inicial es el pedido vacío.
- **Sin `remember`**: `remember` existe para que un valor sobreviva a que Compose vuelva a ejecutar una función `@Composable`. Aquí no estamos dentro de ninguna función `@Composable`, sino en un objeto que vive mucho más que cualquier recomposición. Escribir `remember` aquí ni siquiera compilaría.
- **`private set`**: cualquiera puede **leer** `uiState`, pero solo el código de dentro de `PedidoViewModel` puede **asignarle** un valor nuevo. Si en un composable alguien escribiera `pedidoViewModel.uiState = PedidoUiState()`, Android Studio lo marcaría como error: *Cannot assign to 'uiState': the setter is private*. Así nos aseguramos de que el pedido solo cambia a través de las funciones del `ViewModel`, que son las que aplican las reglas.

### Paso 4: propiedades calculadas

El total, las unidades, el error del nombre y la lista de productos que se ven según el filtro **no se guardan**: se calculan cada vez que alguien los lee.

```kotlin
    val unidades: Int
        get() = uiState.cantidades.values.sum()

    val total: Double
        get() = productos.sumOf { (uiState.cantidades[it.id] ?: 0) * it.precio }

    val errorNombre: Boolean
        get() = uiState.intentoConfirmar && uiState.nombre.isBlank()

    val productosVisibles: List<Producto>
        get() = uiState.categoria?.let { cat -> productos.filter { it.categoria == cat } } ?: productos
```

- `val total: Double get() = ...` es una propiedad con **getter personalizado**: no ocupa memoria ni tiene valor propio; cada vez que se escribe `pedidoViewModel.total`, se ejecuta el cálculo.
- Como el cálculo lee `uiState` (que es un estado de Compose), la `BarraPedido` que muestra el total se recompone sola en cuanto cambia una cantidad.
- ¿Por qué no guardar el total en `PedidoUiState`? Porque habría que acordarse de actualizarlo en **cada** función que toque las cantidades (añadir, quitar, vaciar, enviar…). Basta con olvidarlo en una para que la barra muestre un total que no corresponde con el pedido. Lo que se puede calcular, se calcula.

Fijaos en un detalle de `productosVisibles` y `total`: la lista visible depende del filtro, pero el total se calcula siempre sobre **todos** los productos. Si filtráis por "Bebidas", el croissant que habíais pedido no se ve, pero se sigue pagando.

### Paso 5: una función por cada cosa que puede hacer el usuario

La interfaz no cambia el estado: **pide** cambios llamando a funciones del `ViewModel`. Cada acción del usuario tiene su función:

| El usuario... | La interfaz llama a... | Qué cambia en `uiState` |
|---|---|---|
| pulsa "+" en el café (que tenía 1) | `cambiarCantidad(1, 2)` | `cantidades` pasa a tener `1 to 2` |
| escribe en el campo del nombre | `cambiarNombre("Ana")` | `nombre = "Ana"` |
| activa "Para llevar" | `cambiarParaLlevar(true)` | `paraLlevar = true` |
| elige el filtro "Bebidas" | `seleccionarCategoria(Categoria.BEBIDA)` | `categoria = BEBIDA` |
| elige "Vaciar pedido" en el menú | `vaciarPedido()` | `cantidades` vacío |
| pulsa "Confirmar" | `confirmar()` | `intentoConfirmar = true` y, si hay nombre, `mostrarConfirmacion = true` |
| pulsa "Cancelar" en el diálogo | `cancelarConfirmacion()` | `mostrarConfirmacion = false` |
| pulsa "Enviar" en el diálogo | `enviarPedido()` | `cantidades` vacío y el diálogo se cierra |

Todas siguen el mismo patrón: **crear un estado nuevo con `copy` y asignarlo a `uiState`**:

```kotlin
    fun cambiarCantidad(idProducto: Int, cantidad: Int) {
        uiState = uiState.copy(cantidades = uiState.cantidades + (idProducto to cantidad))
    }

    fun cambiarNombre(nombre: String) {
        uiState = uiState.copy(nombre = nombre)
    }

    fun vaciarPedido() {
        uiState = uiState.copy(cantidades = emptyMap())
    }
```

La función más interesante es `confirmar`, porque ahora la regla del pedido vive en el `ViewModel` y no en un `onClick`. Comparad:

```kotlin
// Sesión 3: la regla, dentro de la interfaz
onConfirmar = {
    intentoConfirmar = true
    if (nombre.isNotBlank()) mostrarConfirmacion = true
}

// Sesión 4: la regla, en el ViewModel; la interfaz solo avisa
fun confirmar() {
    uiState = uiState.copy(
        intentoConfirmar = true,
        mostrarConfirmacion = uiState.nombre.isNotBlank()   // solo se abre el diálogo si hay nombre
    )
}
```

`cambiarParaLlevar`, `cambiarAvisar` y `seleccionarCategoria` son iguales que `cambiarNombre`, cambiando el campo; `cancelarConfirmacion` y `enviarPedido` siguen la misma idea. Las escribiréis en la práctica.

Para terminar la clase, dos mensajes de Logcat que os permitirán **ver** el ciclo de vida del `ViewModel`:

```kotlin
private const val TAG = "PedidoVM"

class PedidoViewModel : ViewModel() {

    init {
        Log.d(TAG, "PedidoViewModel creado")      // se ejecuta al crear el objeto
    }

    // ... estado, propiedades calculadas y eventos

    override fun onCleared() {
        super.onCleared()
        Log.d(TAG, "PedidoViewModel destruido (onCleared)")   // Android lo llama al destruirlo
    }
}
```

El bloque `init` de Kotlin se ejecuta cuando se construye el objeto (como el cuerpo de un constructor en Java); `onCleared()` es el equivalente en el `ViewModel` del `onDestroy()` de la `Activity`.

<div class="callout-info">
En la documentación oficial veréis muchas veces el estado del <code>ViewModel</code> expuesto con <code>StateFlow</code> en lugar de <code>mutableStateOf</code>. Las dos opciones son válidas; usamos <code>mutableStateOf</code> porque ya lo conocéis y no añade conceptos nuevos. Trabajaremos con <em>Flows</em> en la UD4, cuando la base de datos nos los devuelva.
</div>

## 4.8 Conectar el `ViewModel` con la interfaz

### Obtener el `ViewModel` en la pantalla

```kotlin
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun PantallaCarta(
    modifier: Modifier = Modifier,
    pedidoViewModel: PedidoViewModel = viewModel()
) {
    val estado = pedidoViewModel.uiState
    // ...
}
```

- `viewModel()` se pone como **valor por defecto de un parámetro**. En la app nadie le pasa nada, así que se usa `viewModel()`; pero deja la puerta abierta a pasar otro `ViewModel` (por ejemplo, en pruebas automáticas).
- `PantallaCarta` ya **no recibe la lista de productos**: ahora es el `ViewModel` quien sabe de dónde salen. Por eso en `MainActivity` la llamada queda en `PantallaCarta(modifier = Modifier.fillMaxSize())`.
- `val estado = pedidoViewModel.uiState` lee el estado. Como `uiState` es un `mutableStateOf`, esta lectura hace que `PantallaCarta` se recomponga cada vez que el `ViewModel` asigne un estado nuevo.

<div class="callout-warning">
<strong>Nunca</strong> escribáis <code>val pedidoViewModel = PedidoViewModel()</code> dentro de un composable. Cada recomposición crearía un <code>ViewModel</code> nuevo con el pedido vacío: el usuario pulsaría "+" y la cantidad volvería a 0 al instante. Es el mismo error que <code>mutableStateOf</code> sin <code>remember</code> de la sesión 2. Usad siempre <code>viewModel()</code>.
</div>

### Recorrido completo de un toque

¿Qué pasa exactamente cuando el usuario pulsa "+" en el café, que tenía 1 unidad?

```
1. ProductoCard (café)   onCantidadChange(2)      "quiero que haya 2"
        │
2. ContenidoCarta        onCantidadChange(1, 2)   le añade el id del café
        │
3. PedidoViewModel       cambiarCantidad(1, 2)
        │                uiState = uiState.copy(...)  → estado NUEVO: café = 2
        ▼
4. Compose ve que uiState ha cambiado y recompone SOLO lo que lo lee:
   - la tarjeta del café             → muestra 2
   - BarraPedido (total y unidades)  → "Total: 2,40 €", "Unidades: 2"
```

Es el flujo unidireccional de la sesión 3 (**el estado baja, los eventos suben**), pero ahora el estado no vive en un composable sino en el `ViewModel`, en lo más alto de la pantalla.

### Pasar funciones del `ViewModel`: referencias a funciones

Para conectar los eventos, en lugar de escribir una lambda que solo llama a una función, Kotlin permite pasar **la propia función** con `::`:

| Con lambda | Con referencia a función | Tipo que espera el parámetro |
|---|---|---|
| `onConfirmar = { pedidoViewModel.confirmar() }` | `onConfirmar = pedidoViewModel::confirmar` | `() -> Unit` |
| `onNombreChange = { pedidoViewModel.cambiarNombre(it) }` | `onNombreChange = pedidoViewModel::cambiarNombre` | `(String) -> Unit` |
| `onCantidadChange = { id, n -> pedidoViewModel.cambiarCantidad(id, n) }` | `onCantidadChange = pedidoViewModel::cambiarCantidad` | `(Int, Int) -> Unit` |

Las dos columnas hacen lo mismo. La referencia funciona cuando los parámetros de la función coinciden exactamente con los del tipo que se espera. Cuando el evento tiene que hacer **algo más** que llamar a una función, hace falta la lambda. Por ejemplo, al vaciar el pedido también se muestra un *Snackbar*, y eso es cosa de la interfaz:

```kotlin
onVaciarPedido = {
    pedidoViewModel.vaciarPedido()                                    // la lógica: ViewModel
    scope.launch { snackbarHostState.showSnackbar(mensajeVaciado) }   // lo visual: interfaz
}
```

### Pantalla con `ViewModel` + contenido stateless

Las previews tienen un problema con el `ViewModel`: la preview no es una app real, no hay `Activity` ni almacén de `ViewModels`. Por eso la pantalla se divide en dos funciones:

| Función | Qué hace | ¿Tiene preview? |
|---|---|---|
| `PantallaCarta` | Obtiene el `ViewModel`, monta el `Scaffold` con sus barras, conecta los eventos y muestra el diálogo y los *snackbars*. | No: necesita un `ViewModel` real. |
| `ContenidoCarta` | Dibuja la `LazyColumn` a partir de valores y lambdas que recibe por parámetro. No sabe que existe un `ViewModel`. | Sí: basta con pasarle un `PedidoUiState` de ejemplo. |

```kotlin
// Dentro del Scaffold de PantallaCarta
ContenidoCarta(
    productos = pedidoViewModel.productosVisibles,
    estado = estado,
    errorNombre = pedidoViewModel.errorNombre,
    onCantidadChange = pedidoViewModel::cambiarCantidad,
    onCategoriaChange = pedidoViewModel::seleccionarCategoria,
    onNombreChange = pedidoViewModel::cambiarNombre,
    onParaLlevarChange = pedidoViewModel::cambiarParaLlevar,
    onAvisarChange = pedidoViewModel::cambiarAvisar,
    modifier = Modifier.padding(innerPadding)
)
```

La preview de `ContenidoCarta` se escribe con datos inventados, sin `ViewModel`:

```kotlin
ContenidoCarta(
    productos = DatosCarta.productos,
    estado = PedidoUiState(cantidades = mapOf(1 to 2, 3 to 1), nombre = "Ana"),
    errorNombre = false,
    onCantidadChange = { _, _ -> },   // lambdas que no hacen nada
    onCategoriaChange = {},
    onNombreChange = {},
    onParaLlevarChange = {},
    onAvisarChange = {}
)
```

### Antes y después

| | Sesión 3 | Sesión 4 |
|---|---|---|
| Dónde vive el pedido | Seis `rememberSaveable` en `PantallaCarta` | Un `PedidoUiState` dentro de `PedidoViewModel` |
| Dónde están las reglas (total, confirmar…) | Mezcladas con la interfaz | En funciones y propiedades del `ViewModel` |
| Qué queda en `PantallaCarta` | Todo | Obtener el `ViewModel`, el `Scaffold`, el diálogo y el `SnackbarHostState` |
| Qué pasa al girar | Cada `rememberSaveable` se guarda y se restaura | El `ViewModel` es el mismo objeto: no hay nada que restaurar |
| Qué componentes han cambiado | — | Ninguno: `ProductoCard`, `BarraPedido`, `DatosPedido`… ya eran stateless desde la sesión 3 |

La última fila es la recompensa del *state hoisting*: como los componentes ya recibían todo por parámetro, cambiar de dónde sale el estado no les afecta en absoluto.

## 4.9 ¿Qué sobrevive a qué?

Con esta sesión ya conocéis cuatro formas de guardar estado. Cada una sobrevive a cosas distintas:

| Mecanismo | Recomposición | Giro de pantalla | La app en segundo plano y Android la elimina | Cerrar la app |
|---|---|---|---|---|
| `remember` | Sí | No | No | No |
| `rememberSaveable` | Sí | Sí | **Sí** | No |
| `ViewModel` | Sí | **Sí** | **No** | No |
| `ViewModel` + `SavedStateHandle` | Sí | Sí | Sí | No |
| Persistencia (DataStore, base de datos) | Sí | Sí | Sí | **Sí** (sesión 6 y UD4) |

La fila del `ViewModel` tiene una trampa: sobrevive a los giros, pero **no** a que Android elimine la app en segundo plano para liberar memoria, porque en ese caso se destruye todo el proceso. Lo comprobaréis en la práctica con la opción *No conservar actividades* de la sesión 2, que simula esa situación: destruye la actividad en cuanto se sale de ella, sin que sea un giro, y el `ViewModel` se destruye con ella. Para los datos que no deben perderse en ese caso, el `ViewModel` puede recibir un **`SavedStateHandle`**, un pequeño almacén que se guarda en el mismo `Bundle` que usa `rememberSaveable`.

<div class="callout-info">
<strong>Regla práctica para elegir:</strong> el estado visual y local (un menú abierto) con <code>remember</code>; datos pequeños de un componente que deben sobrevivir a todo (el texto de un campo aislado) con <code>rememberSaveable</code>; el estado y la lógica de una pantalla, en su <code>ViewModel</code>; lo que el usuario espera encontrar al volver a abrir la app, en persistencia.
</div>

## Tabla resumen de la sesión

| Necesito... | Uso |
|---|---|
| Una lista vertical larga y eficiente | `LazyColumn { item { } ; items(lista, key = { it.id }) { } }` |
| Una fila horizontal desplazable | `LazyRow { items(...) { } }` |
| Un filtro seleccionable | `FilterChip(selected, onClick, label)` |
| Una cuadrícula | `LazyVerticalGrid(columns = GridCells.Fixed(n)) { ... }` |
| Añadir una librería | `libs.versions.toml` + `implementation(...)` en `build.gradle.kts` + *Sync Now* |
| Agrupar el estado de una pantalla | `data class XxxUiState(...)` con valores por defecto, cambios con `copy` |
| Estado y lógica que sobreviven al giro | `class XxxViewModel : ViewModel()` con `var uiState by mutableStateOf(...) private set` |
| Obtener el `ViewModel` en un composable | `viewModel: XxxViewModel = viewModel()` como parámetro |
| Pasar una función como lambda | Referencia a función: `viewModel::nombreFuncion` |
| Saber cuándo se destruye el `ViewModel` | Sobrescribir `onCleared()` |

<div class="callout-practica">
<strong>Ejercicio de sesión</strong> (en un proyecto de pruebas, 15 minutos):

1. Crea una <code>LazyColumn</code> que muestre los números del 1 al 1000 (pista: <code>items((1..1000).toList()) { }</code>) y comprueba que la app abre al instante.
2. Mete esa <code>LazyColumn</code> dentro de una <code>Column</code> con <code>verticalScroll</code>, ejecuta y busca en Logcat el error. Deshaz el cambio.
3. Crea un <code>ContadorViewModel</code> con un <code>var contador by mutableStateOf(0)</code> con <code>private set</code> y una función <code>incrementar()</code>. Úsalo desde un botón, gira la pantalla y comprueba que el contador se conserva sin <code>rememberSaveable</code>.
</div>

## Para la próxima sesión

La cafetería sigue siendo una sola pantalla. En la sesión 5 la convertiremos en una app de **varias pantallas** con *Navigation Compose*: la carta, el detalle de cada producto y el resumen del pedido, compartiendo el mismo `PedidoViewModel`.
