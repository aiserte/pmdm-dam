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

Al final de la sesión 3, `PantallaCarta` hacía de todo:

- guardaba seis estados con `rememberSaveable`;
- contenía la lógica del pedido: calcular el total, validar el nombre, vaciar, enviar…;
- y además dibujaba la interfaz.

Esto tiene tres problemas: el fichero crece sin control, la lógica no se puede probar sin la interfaz y `rememberSaveable` solo sirve para datos pequeños y sencillos. En las apps reales la lógica y el estado de una pantalla se sacan a una clase aparte: el **`ViewModel`**.

## 4.5 Qué es un `ViewModel`

Un `ViewModel` es una clase de la librería de Android (*Jetpack Lifecycle*) que guarda el estado de una pantalla y la lógica que lo modifica. Su característica clave es **cuánto vive**:

```
Activity:   creada ─► destruida/recreada ─► destruida/recreada ─► finalizada
            (abrir)   (girar pantalla)      (cambiar idioma)      (cerrar)

ViewModel:  creado ──────── el MISMO objeto todo el tiempo ────► onCleared()
```

Cuando la pantalla gira, la `Activity` se destruye y se crea otra (sesión 2), pero el `ViewModel` **no**: la nueva `Activity` recibe exactamente el mismo objeto, con todos sus datos. Solo se destruye cuando la pantalla se cierra definitivamente; en ese momento Android llama a su método `onCleared()`.

| Va en el `ViewModel` | No va en el `ViewModel` |
|---|---|
| El estado de la pantalla (el pedido, el nombre, el filtro) | Composables ni nada de la interfaz |
| La lógica (calcular el total, validar, vaciar, enviar) | Referencias a la `Activity` o a su `Context` (provocaría fugas de memoria al girar) |
| Más adelante: el acceso a datos (base de datos en la UD4, red en la UD5) | Estado puramente visual (si un menú está abierto, la posición de la lista) |

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

## 4.7 Estado de la interfaz y `ViewModel`

### Paso 1: el estado de la pantalla en una `data class`

En lugar de seis variables sueltas, el estado de la pantalla se agrupa en una sola clase inmutable:

```kotlin
data class PedidoUiState(
    val cantidades: Map<Int, Int> = emptyMap(),
    val nombre: String = "",
    val paraLlevar: Boolean = false,
    val avisar: Boolean = true,
    val intentoConfirmar: Boolean = false,
    val mostrarConfirmacion: Boolean = false,
    val categoria: Categoria? = null       // null = todas las categorías
)
```

Los valores por defecto describen la pantalla al abrirse. Para cambiar algo **no se modifica** el objeto: se crea uno nuevo con `copy`, la función que las `data class` traen de serie (UD2):

```kotlin
val nuevo = estado.copy(nombre = "Ana")   // igual que "estado" salvo el nombre
```

### Paso 2: el `ViewModel`

```kotlin
class PedidoViewModel : ViewModel() {

    val productos: List<Producto> = DatosCarta.productos

    // Estado observable por Compose; solo el ViewModel puede cambiarlo
    var uiState by mutableStateOf(PedidoUiState())
        private set

    // Valores calculados a partir del estado
    val unidades: Int
        get() = uiState.cantidades.values.sum()

    val total: Double
        get() = productos.sumOf { (uiState.cantidades[it.id] ?: 0) * it.precio }

    // Eventos: lo único que la interfaz puede pedir
    fun cambiarCantidad(idProducto: Int, cantidad: Int) {
        uiState = uiState.copy(cantidades = uiState.cantidades + (idProducto to cantidad))
    }

    fun vaciarPedido() {
        uiState = uiState.copy(cantidades = emptyMap())
    }

    // ... resto de eventos
}
```

Tres ideas importantes:

- **`private set`**: cualquiera puede **leer** `uiState`, pero solo el propio `ViewModel` puede **cambiarlo**. La interfaz no puede modificar el estado por su cuenta: tiene que pedirlo llamando a una función (`cambiarCantidad`, `vaciarPedido`…). Es el flujo unidireccional de la sesión 3, llevado a una clase.
- **`mutableStateOf`** fuera de un composable: aquí **no** hace falta `remember`. `remember` sirve para que un dato sobreviva a las recomposiciones de una función; un `ViewModel` ya vive mucho más que cualquier recomposición.
- **Propiedades calculadas** (`get() = ...`): `total` y `unidades` se recalculan cada vez que se leen. Como leen `uiState`, que es un estado de Compose, la interfaz que las muestra se recompone sola cuando el pedido cambia.

<div class="callout-info">
En la documentación oficial veréis muchas veces el estado del <code>ViewModel</code> expuesto con <code>StateFlow</code> en lugar de <code>mutableStateOf</code>. Las dos opciones son válidas; usamos <code>mutableStateOf</code> porque ya lo conocéis y no añade conceptos nuevos. Trabajaremos con <em>Flows</em> en la UD4, cuando la base de datos nos los devuelva.
</div>

## 4.8 Conectar el `ViewModel` con la interfaz

```kotlin
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun PantallaCarta(
    modifier: Modifier = Modifier,
    pedidoViewModel: PedidoViewModel = viewModel()
) {
    val estado = pedidoViewModel.uiState

    Scaffold(
        bottomBar = {
            BarraPedido(
                total = pedidoViewModel.total,
                unidades = pedidoViewModel.unidades,
                onConfirmar = pedidoViewModel::confirmar
            )
        }
        // ...
    ) { innerPadding ->
        ContenidoCarta(
            productos = pedidoViewModel.productosVisibles,
            estado = estado,
            onCantidadChange = pedidoViewModel::cambiarCantidad,
            // ...
            modifier = Modifier.padding(innerPadding)
        )
    }
}
```

- **`viewModel()`** no crea un objeto nuevo cada vez: la primera vez lo crea y, en las siguientes (también tras girar la pantalla), devuelve **el mismo**. Por eso nunca se escribe `PedidoViewModel()` directamente.
- Se pone como **parámetro con valor por defecto**: en la app se usa el valor por defecto, pero se podría pasar otro `ViewModel` (por ejemplo, en pruebas).
- **`pedidoViewModel::confirmar`** es una **referencia a función**: en lugar de escribir la lambda `{ pedidoViewModel.confirmar() }`, se pasa la propia función. Funciona si los parámetros coinciden con los que espera la lambda: `cambiarCantidad(idProducto: Int, cantidad: Int)` encaja con un parámetro de tipo `(Int, Int) -> Unit`.

### Pantalla con `ViewModel` + contenido stateless

La estructura recomendada separa la pantalla en dos funciones:

| Función | Qué hace | ¿Tiene preview? |
|---|---|---|
| `PantallaCarta` | Obtiene el `ViewModel`, monta el `Scaffold`, conecta eventos y muestra diálogos y *snackbars*. | No: necesita un `ViewModel` real. |
| `ContenidoCarta` | Dibuja la lista a partir de valores y lambdas que recibe por parámetro. Stateless. | Sí: basta con pasarle un `PedidoUiState` de ejemplo. |

Así las previews siguen funcionando con datos inventados, por ejemplo `PedidoUiState(cantidades = mapOf(1 to 2, 3 to 1), nombre = "Ana")`.

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
