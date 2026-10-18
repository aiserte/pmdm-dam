---
title: "Cafetería del instituto: carta completa y ViewModel — Solución (Sesión 4)"
unidad: UD3
sesion: 4
orden: 4
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Solo se muestran los ficheros nuevos o que cambian respecto a la solución de la sesión 3. No cambian: <code>CabeceraCarta.kt</code>, <code>ProductoCard.kt</code>, <code>BarraSuperiorCarta.kt</code>, <code>BarraPedido.kt</code>, <code>DatosPedido.kt</code>, <code>DialogoConfirmacion.kt</code> y el tema. Gracias al <em>state hoisting</em> de la sesión 3, ninguno de los componentes stateless ha necesitado ni una línea de cambio.
</div>

### Estructura final

```
com.iesejemplo.cafeteria/
├── MainActivity.kt
├── data/
│   ├── Producto.kt            ← + id, + categoria, + enum Categoria
│   └── DatosCarta.kt          ← 8 productos
└── ui/carta/
    ├── PedidoUiState.kt       ← nuevo: estado de la pantalla
    ├── PedidoViewModel.kt     ← nuevo: estado + lógica
    ├── PantallaCarta.kt       ← PantallaCarta (con ViewModel) + ContenidoCarta (stateless)
    ├── FiltroCategorias.kt    ← nuevo: LazyRow de FilterChip
    └── (resto de componentes sin cambios)
```

### Ejercicio 1 — `data/Producto.kt`

```kotlin
package com.iesejemplo.cafeteria.data

import androidx.annotation.DrawableRes
import androidx.annotation.StringRes
import com.iesejemplo.cafeteria.R

enum class Categoria(@StringRes val nombre: Int) {
    BEBIDA(R.string.categoria_bebida),
    COMIDA(R.string.categoria_comida),
    BOLLERIA(R.string.categoria_bolleria)
}

data class Producto(
    val id: Int,
    @StringRes val nombre: Int,
    @StringRes val descripcion: Int,
    @DrawableRes val icono: Int,
    val precio: Double,
    val categoria: Categoria
)
```

### Ejercicio 1 — `data/DatosCarta.kt`

```kotlin
package com.iesejemplo.cafeteria.data

import com.iesejemplo.cafeteria.R

object DatosCarta {
    val productos = listOf(
        Producto(1, R.string.producto_cafe, R.string.desc_cafe, R.drawable.baseline_local_cafe_24, 1.20, Categoria.BEBIDA),
        Producto(2, R.string.producto_bocadillo, R.string.desc_bocadillo, R.drawable.baseline_lunch_dining_24, 2.80, Categoria.COMIDA),
        Producto(3, R.string.producto_croissant, R.string.desc_croissant, R.drawable.baseline_bakery_dining_24, 1.50, Categoria.BOLLERIA),
        Producto(4, R.string.producto_zumo, R.string.desc_zumo, R.drawable.baseline_local_drink_24, 2.00, Categoria.BEBIDA),
        Producto(5, R.string.producto_te, R.string.desc_te, R.drawable.baseline_local_cafe_24, 1.10, Categoria.BEBIDA),
        Producto(6, R.string.producto_agua, R.string.desc_agua, R.drawable.baseline_local_drink_24, 0.80, Categoria.BEBIDA),
        Producto(7, R.string.producto_napolitana, R.string.desc_napolitana, R.drawable.baseline_bakery_dining_24, 1.60, Categoria.BOLLERIA),
        Producto(8, R.string.producto_sandwich, R.string.desc_sandwich, R.drawable.baseline_lunch_dining_24, 2.20, Categoria.COMIDA)
    )
}
```

### Ejercicio 1 — Recursos nuevos

Añadir a `res/values/strings.xml`:

```xml
    <string name="categoria_todas">Todo</string>
    <string name="categoria_bebida">Bebidas</string>
    <string name="categoria_comida">Comida</string>
    <string name="categoria_bolleria">Bollería</string>

    <string name="producto_te">Té</string>
    <string name="desc_te">Negro, verde o rooibos</string>
    <string name="producto_agua">Agua mineral</string>
    <string name="desc_agua">Botella de 50 cl</string>
    <string name="producto_napolitana">Napolitana de chocolate</string>
    <string name="desc_napolitana">Recién horneada, con chocolate fundido</string>
    <string name="producto_sandwich">Sándwich mixto</string>
    <string name="desc_sandwich">Jamón york y queso, a la plancha</string>
```

Añadir a `res/values-en/strings.xml`:

```xml
    <string name="categoria_todas">All</string>
    <string name="categoria_bebida">Drinks</string>
    <string name="categoria_comida">Food</string>
    <string name="categoria_bolleria">Pastries</string>

    <string name="producto_te">Tea</string>
    <string name="desc_te">Black, green or rooibos</string>
    <string name="producto_agua">Mineral water</string>
    <string name="desc_agua">50 cl bottle</string>
    <string name="producto_napolitana">Chocolate pastry</string>
    <string name="desc_napolitana">Freshly baked, with melted chocolate</string>
    <string name="producto_sandwich">Ham and cheese toastie</string>
    <string name="desc_sandwich">Ham and cheese, grilled</string>
```

### Ejercicio 3 — `ui/carta/FiltroCategorias.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.lazy.LazyRow
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.FilterChip
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.Categoria

@Composable
fun FiltroCategorias(
    seleccionada: Categoria?,
    onSeleccionar: (Categoria?) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyRow(
        modifier = modifier,
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
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
}
```

### Ejercicio 4 — Dependencia

En `gradle/libs.versions.toml`, sección `[libraries]`:

```
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleRuntimeKtx" }
```

En `app/build.gradle.kts`, dentro de `dependencies { }`:

```kotlin
implementation(libs.androidx.lifecycle.viewmodel.compose)
```

### Ejercicio 5 — `ui/carta/PedidoUiState.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import com.iesejemplo.cafeteria.data.Categoria

data class PedidoUiState(
    val cantidades: Map<Int, Int> = emptyMap(),   // id del producto -> cantidad
    val nombre: String = "",
    val paraLlevar: Boolean = false,
    val avisar: Boolean = true,
    val intentoConfirmar: Boolean = false,
    val mostrarConfirmacion: Boolean = false,
    val categoria: Categoria? = null              // null = todas
)
```

### Ejercicio 5 — `ui/carta/PedidoViewModel.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import android.util.Log
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel
import com.iesejemplo.cafeteria.data.Categoria
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.Producto

private const val TAG = "PedidoVM"

class PedidoViewModel : ViewModel() {

    val productos: List<Producto> = DatosCarta.productos

    var uiState by mutableStateOf(PedidoUiState())
        private set

    init {
        Log.d(TAG, "PedidoViewModel creado")
    }

    // --- Valores calculados ---
    val productosVisibles: List<Producto>
        get() = uiState.categoria?.let { cat -> productos.filter { it.categoria == cat } } ?: productos

    val unidades: Int
        get() = uiState.cantidades.values.sum()

    val total: Double
        get() = productos.sumOf { (uiState.cantidades[it.id] ?: 0) * it.precio }

    val errorNombre: Boolean
        get() = uiState.intentoConfirmar && uiState.nombre.isBlank()

    // --- Eventos ---
    fun cambiarCantidad(idProducto: Int, cantidad: Int) {
        uiState = uiState.copy(cantidades = uiState.cantidades + (idProducto to cantidad))
    }

    fun cambiarNombre(nombre: String) {
        uiState = uiState.copy(nombre = nombre)
    }

    fun cambiarParaLlevar(paraLlevar: Boolean) {
        uiState = uiState.copy(paraLlevar = paraLlevar)
    }

    fun cambiarAvisar(avisar: Boolean) {
        uiState = uiState.copy(avisar = avisar)
    }

    fun seleccionarCategoria(categoria: Categoria?) {
        uiState = uiState.copy(categoria = categoria)
    }

    fun vaciarPedido() {
        uiState = uiState.copy(cantidades = emptyMap())
    }

    fun confirmar() {
        uiState = uiState.copy(
            intentoConfirmar = true,
            mostrarConfirmacion = uiState.nombre.isNotBlank()
        )
    }

    fun cancelarConfirmacion() {
        uiState = uiState.copy(mostrarConfirmacion = false)
    }

    fun enviarPedido() {
        uiState = uiState.copy(
            cantidades = emptyMap(),
            mostrarConfirmacion = false,
            intentoConfirmar = false
        )
    }

    override fun onCleared() {
        super.onCleared()
        Log.d(TAG, "PedidoViewModel destruido (onCleared)")
    }
}
```

`productosVisibles` combina dos cosas de la UD2: `?.let { }` (si hay categoría, filtra) y el operador Elvis `?:` (si no la hay, devuelve todos).

### Ejercicios 2, 3 y 6 — `ui/carta/PantallaCarta.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import android.content.res.Configuration
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.Categoria
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme
import kotlinx.coroutines.launch

@Composable
fun PantallaCarta(
    modifier: Modifier = Modifier,
    pedidoViewModel: PedidoViewModel = viewModel()
) {
    val estado = pedidoViewModel.uiState

    // Estado puramente visual: se queda en la interfaz
    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()

    val mensajeVaciado = stringResource(R.string.pedido_vaciado)
    val mensajeEnviado = stringResource(R.string.pedido_enviado, estado.nombre)

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraSuperiorCarta(
                hayPedido = pedidoViewModel.unidades > 0,
                onVaciarPedido = {
                    pedidoViewModel.vaciarPedido()
                    scope.launch { snackbarHostState.showSnackbar(mensajeVaciado) }
                }
            )
        },
        bottomBar = {
            BarraPedido(
                total = pedidoViewModel.total,
                unidades = pedidoViewModel.unidades,
                onConfirmar = pedidoViewModel::confirmar
            )
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { innerPadding ->
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
    }

    if (estado.mostrarConfirmacion) {
        DialogoConfirmacion(
            nombre = estado.nombre,
            unidades = pedidoViewModel.unidades,
            total = pedidoViewModel.total,
            paraLlevar = estado.paraLlevar,
            avisar = estado.avisar,
            onEnviar = {
                pedidoViewModel.enviarPedido()
                scope.launch { snackbarHostState.showSnackbar(mensajeEnviado) }
            },
            onCancelar = pedidoViewModel::cancelarConfirmacion
        )
    }
}

@Composable
fun ContenidoCarta(
    productos: List<Producto>,
    estado: PedidoUiState,
    errorNombre: Boolean,
    onCantidadChange: (Int, Int) -> Unit,
    onCategoriaChange: (Categoria?) -> Unit,
    onNombreChange: (String) -> Unit,
    onParaLlevarChange: (Boolean) -> Unit,
    onAvisarChange: (Boolean) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyColumn(
        modifier = modifier
            .fillMaxSize()
            .imePadding(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        item {
            CabeceraCarta()
        }
        item {
            FiltroCategorias(
                seleccionada = estado.categoria,
                onSeleccionar = onCategoriaChange
            )
        }
        items(items = productos, key = { it.id }) { producto ->
            ProductoCard(
                producto = producto,
                cantidad = estado.cantidades[producto.id] ?: 0,
                onCantidadChange = { nueva -> onCantidadChange(producto.id, nueva) }
            )
        }
        item {
            DatosPedido(
                nombre = estado.nombre,
                onNombreChange = onNombreChange,
                mostrarError = errorNombre,
                paraLlevar = estado.paraLlevar,
                onParaLlevarChange = onParaLlevarChange,
                avisar = estado.avisar,
                onAvisarChange = onAvisarChange
            )
        }
    }
}

@Preview(name = "Español", showBackground = true)
@Preview(name = "Inglés", showBackground = true, locale = "en")
@Preview(name = "Oscuro", showBackground = true, uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
fun ContenidoCartaPreview() {
    CafeteriaTheme {
        Surface {
            ContenidoCarta(
                productos = DatosCarta.productos,
                estado = PedidoUiState(cantidades = mapOf(1 to 2, 3 to 1), nombre = "Ana"),
                errorNombre = false,
                onCantidadChange = { _, _ -> },
                onCategoriaChange = {},
                onNombreChange = {},
                onParaLlevarChange = {},
                onAvisarChange = {}
            )
        }
    }
}
```

En `ProductoCard`, `onCantidadChange` recibe solo la nueva cantidad; `ContenidoCarta` le añade el id del producto antes de pasárselo al `ViewModel`. En la preview, `{ _, _ -> }` es una lambda que recibe dos parámetros y no los usa.

### Ejercicio 6 — `MainActivity.kt` (solo cambia `setContent`)

```kotlin
setContent {
    CafeteriaTheme {
        PantallaCarta(modifier = Modifier.fillMaxSize())
    }
}
```

El import de `DatosCarta` ya no hace falta en `MainActivity`: ahora es el `ViewModel` quien sabe de dónde salen los productos.

### Ejercicio 7 — `PRUEBAS.md`, sección "Sesión 4" (resultados esperables)

| # | Prueba | Resultado esperado | Resultado obtenido |
|---|---|---|---|
| P4-1 | Desplazar la carta completa | Se llega a todo | OK en vertical y en horizontal |
| P4-2 | Abrir y girar dos veces | "PedidoViewModel creado" una sola vez | OK: `onCreate` → `onStart` → `onResume` (CicloVida) → "PedidoViewModel creado" (PedidoVM), porque el `ViewModel` se pide al componer la interfaz, que ocurre cuando la ventana ya es visible; en cada giro, `onPause` … `onDestroy` → `onCreate` … `onResume`, sin un nuevo "creado" ni "destruido" |
| P4-3 | Pedido + nombre + categoría y girar | Se conserva todo | OK: el `ViewModel` es el mismo objeto antes y después del giro |
| P4-4 | Desplazar a la mitad y girar | Misma posición | OK: el estado de la `LazyColumn` se guarda con `rememberSaveable` internamente |
| P4-5 | Croissant + filtro "Bebidas" | Oculto pero cuenta | OK: el total sigue incluyendo 1,50 € |
| P4-6 | *No conservar actividades*: pedido, Inicio y volver | Ver explicación | **El pedido se pierde.** Al pulsar Inicio: `onPause` → `onStop` → `onDestroy` → "PedidoViewModel destruido (onCleared)". Al volver: `onCreate (savedInstanceState trae datos guardados)` → "PedidoViewModel creado", con un estado nuevo vacío. El `ViewModel` sobrevive a los giros, pero no a que Android destruya la actividad sin que vaya a volver a crearse de inmediato. En la sesión 3, con `rememberSaveable`, el pedido sí se conservaba en esta misma prueba |

P4-6 es la prueba más importante de la sesión: demuestra con Logcat la fila del `ViewModel` en la tabla del apartado 4.9 de la teoría. No es un error de la práctica, es el comportamiento esperado, y se resuelve con el ejercicio opcional.

### Opcional — `SavedStateHandle`

```kotlin
import androidx.lifecycle.SavedStateHandle

class PedidoViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {

    val productos: List<Producto> = DatosCarta.productos

    // Al crearse, recupera lo que hubiera guardado (o empieza vacío)
    var uiState by mutableStateOf(
        PedidoUiState(
            cantidades = savedStateHandle.get<HashMap<Int, Int>>("cantidades") ?: emptyMap(),
            nombre = savedStateHandle.get<String>("nombre") ?: ""
        )
    )
        private set

    // Cada cambio de estado pasa por aquí y se guarda en el SavedStateHandle
    private fun actualizar(nuevo: PedidoUiState) {
        uiState = nuevo
        savedStateHandle["cantidades"] = HashMap(nuevo.cantidades)
        savedStateHandle["nombre"] = nuevo.nombre
    }

    fun cambiarCantidad(idProducto: Int, cantidad: Int) {
        actualizar(uiState.copy(cantidades = uiState.cantidades + (idProducto to cantidad)))
    }

    // ... el resto de eventos, igual pero llamando a actualizar(...) en lugar de asignar uiState
}
```

`viewModel()` detecta que el constructor pide un `SavedStateHandle` y se lo proporciona sin hacer nada más. Se convierte el mapa a `HashMap` porque el `SavedStateHandle` se guarda en un `Bundle`, que solo admite tipos concretos (y `HashMap` lo es). Con este cambio, la prueba P4-6 conserva el pedido y el nombre.

## Qué comprobar en cada elemento obligatorio

- **Modelo** → `Producto` tiene `id` como primera propiedad; los ids no se repiten (si se repiten, `LazyColumn` lanza una excepción por clave duplicada).
- **`LazyColumn`** → ningún `verticalScroll` en la pantalla; `items` con `key = { it.id }`; cabecera, filtro y datos del pedido como `item`.
- **Filtro** → con "Bebidas" seleccionado, el total no cambia: la lista visible se calcula en el `ViewModel`, pero el total se calcula siempre sobre **todos** los productos.
- **`ViewModel`** → `uiState` con `private set`; ningún composable asigna `uiState` directamente; `total`, `unidades`, `productosVisibles` y `errorNombre` son propiedades calculadas, no estado.
- **Separación** → `PantallaCarta` sin `rememberSaveable`; solo `remember` para el `SnackbarHostState`, que es estado visual. `ContenidoCarta` tiene preview funcionando.
- **Referencias a funciones** → `pedidoViewModel::confirmar`, `::cambiarCantidad`, `::cambiarNombre`, etc. (`onVaciarPedido` y `onEnviar` necesitan lambda porque además muestran un *Snackbar*).
- **`PRUEBAS.md`** → P4-2 con la secuencia de ambas etiquetas en Logcat y P4-6 explicada, no solo descrita.
