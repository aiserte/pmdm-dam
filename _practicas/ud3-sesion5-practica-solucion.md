---
title: "Cafetería del instituto: varias pantallas — Solución (Sesión 5)"
unidad: UD3
sesion: 5
orden: 5
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Solo se muestran los ficheros nuevos o que cambian respecto a la solución de la sesión 4. No cambian: <code>data/</code>, <code>CabeceraCarta.kt</code>, <code>FiltroCategorias.kt</code>, <code>BarraSuperiorCarta.kt</code>, <code>DatosPedido.kt</code>, <code>DialogoConfirmacion.kt</code> y el tema.
</div>

### Estructura final

```
com.iesejemplo.cafeteria/
├── MainActivity.kt                 ← dibuja AppCafeteria()
├── data/                           (sin cambios)
└── ui/
    ├── navegacion/
    │   ├── Rutas.kt                ← nuevo: las cuatro rutas
    │   └── AppCafeteria.kt         ← nuevo: NavHost + ViewModel compartido
    └── carta/
        ├── PedidoUiState.kt        ← + LineaPedido
        ├── PedidoViewModel.kt      ← + buscarProducto, + lineasPedido
        ├── PantallaCarta.kt        ← sin datos del pedido ni diálogo
        ├── PantallaDetalle.kt      ← nuevo
        ├── PantallaResumen.kt      ← nuevo (incluye FilaPedido)
        ├── PantallaPedidoEnviado.kt← nuevo
        ├── SelectorCantidad.kt     ← nuevo (sale de ProductoCard)
        ├── BarraConVolver.kt       ← nuevo
        ├── ProductoCard.kt         ← + onClick, usa SelectorCantidad
        ├── BarraPedido.kt          ← botón configurable
        └── (resto sin cambios)
```

### Ejercicio 1 — Gradle

`gradle/libs.versions.toml`:

```
[versions]
navigationCompose = "2.9.0"
kotlinxSerializationJson = "1.8.1"

[libraries]
androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigationCompose" }
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinxSerializationJson" }

[plugins]
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```

`build.gradle.kts` (raíz del proyecto), dentro de `plugins { }`:

```kotlin
alias(libs.plugins.kotlin.serialization) apply false
```

`app/build.gradle.kts`:

```kotlin
plugins {
    // ... los que ya había
    alias(libs.plugins.kotlin.serialization)
}

dependencies {
    // ... las que ya había
    implementation(libs.androidx.navigation.compose)
    implementation(libs.kotlinx.serialization.json)
}
```

### Ejercicio 1 — Recursos

Iconos: `baseline_arrow_back_24` y `baseline_check_circle_24` (*Vector Asset*, *Clip art*).

`res/values/strings.xml`:

```xml
    <string name="ver_pedido">Ver pedido</string>
    <string name="volver">Volver</string>
    <string name="titulo_resumen">Tu pedido</string>
    <string name="pedido_vacio">Todavía no has añadido nada al pedido.</string>
    <string name="linea_pedido">%1$d × %2$s</string>
    <string name="titulo_enviado">¡Pedido enviado!</string>
    <string name="mensaje_enviado">Gracias, %1$s. Ya estamos preparando tu pedido.</string>
    <string name="volver_carta">Volver a la carta</string>
```

`res/values-en/strings.xml`:

```xml
    <string name="ver_pedido">View order</string>
    <string name="volver">Back</string>
    <string name="titulo_resumen">Your order</string>
    <string name="pedido_vacio">You haven\'t added anything to your order yet.</string>
    <string name="linea_pedido">%1$d × %2$s</string>
    <string name="titulo_enviado">Order sent!</string>
    <string name="mensaje_enviado">Thank you, %1$s. We\'re already preparing your order.</string>
    <string name="volver_carta">Back to the menu</string>
```

### Ejercicio 2 — `ui/navegacion/Rutas.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.navegacion

import kotlinx.serialization.Serializable

@Serializable
data object Carta

@Serializable
data class Detalle(val idProducto: Int)

@Serializable
data object Resumen

@Serializable
data class PedidoEnviado(val nombre: String)
```

### Ejercicio 3 — `ui/carta/SelectorCantidad.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.Row
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import com.iesejemplo.cafeteria.R

@Composable
fun SelectorCantidad(
    cantidad: Int,
    onCantidadChange: (Int) -> Unit,
    modifier: Modifier = Modifier
) {
    Row(modifier = modifier, verticalAlignment = Alignment.CenterVertically) {
        IconButton(onClick = { onCantidadChange(cantidad - 1) }, enabled = cantidad > 0) {
            Icon(
                painter = painterResource(R.drawable.baseline_remove_24),
                contentDescription = stringResource(R.string.quitar)
            )
        }
        Text(text = cantidad.toString(), style = MaterialTheme.typography.titleMedium)
        IconButton(onClick = { onCantidadChange(cantidad + 1) }) {
            Icon(
                painter = painterResource(R.drawable.baseline_add_24),
                contentDescription = stringResource(R.string.anadir)
            )
        }
    }
}
```

### Ejercicio 3 — `ui/carta/ProductoCard.kt`

```kotlin
@Composable
fun ProductoCard(
    producto: Producto,
    cantidad: Int,
    onCantidadChange: (Int) -> Unit,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Card(onClick = onClick, modifier = modifier.fillMaxWidth()) {
        Column(modifier = Modifier.padding(16.dp)) {
            Row(verticalAlignment = Alignment.CenterVertically) {
                // ... icono, nombre y descripción, precio (igual que en la sesión 4)
            }
            SelectorCantidad(
                cantidad = cantidad,
                onCantidadChange = onCantidadChange,
                modifier = Modifier.align(Alignment.End)
            )
        }
    }
}

@Preview(showBackground = true)
@Composable
fun ProductoCardPreview() {
    CafeteriaTheme {
        ProductoCard(
            producto = DatosCarta.productos.first(),
            cantidad = 2,
            onCantidadChange = {},
            onClick = {}
        )
    }
}
```

`Card(onClick = ...)` es una versión de `Card` que ya se comporta como un botón: muestra el efecto al pulsarla y es accesible con TalkBack. Los botones − y + siguen funcionando por separado: un toque en ellos no abre el detalle.

### Ejercicio 3 — `ui/carta/BarraPedido.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.annotation.StringRes
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R

@Composable
fun BarraPedido(
    total: Double,
    unidades: Int,
    @StringRes textoBoton: Int,
    onBotonClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    BottomAppBar(modifier = modifier) {
        Column(modifier = Modifier.weight(1f).padding(horizontal = 16.dp)) {
            Text(
                text = stringResource(R.string.total_pedido, total),
                style = MaterialTheme.typography.titleMedium
            )
            Text(
                text = stringResource(R.string.unidades_pedido, unidades),
                style = MaterialTheme.typography.bodySmall
            )
        }
        Button(
            onClick = onBotonClick,
            enabled = unidades > 0,
            modifier = Modifier.padding(end = 16.dp)
        ) {
            Text(stringResource(textoBoton))
        }
    }
}
```

### Ejercicio 3 — `ui/carta/BarraConVolver.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import com.iesejemplo.cafeteria.R

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraConVolver(
    titulo: String,
    onVolver: () -> Unit,
    modifier: Modifier = Modifier
) {
    TopAppBar(
        title = { Text(titulo) },
        modifier = modifier,
        navigationIcon = {
            IconButton(onClick = onVolver) {
                Icon(
                    painter = painterResource(R.drawable.baseline_arrow_back_24),
                    contentDescription = stringResource(R.string.volver)
                )
            }
        },
        colors = TopAppBarDefaults.topAppBarColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer,
            titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
        )
    )
}
```

### Ejercicio 4 — `ui/carta/PedidoUiState.kt` (se añade al final)

```kotlin
data class LineaPedido(
    val producto: Producto,
    val cantidad: Int
)
```

(con `import com.iesejemplo.cafeteria.data.Producto`).

### Ejercicio 4 — `ui/carta/PedidoViewModel.kt` (se añade)

```kotlin
    fun buscarProducto(id: Int): Producto? = productos.find { it.id == id }

    val lineasPedido: List<LineaPedido>
        get() = productos
            .filter { (uiState.cantidades[it.id] ?: 0) > 0 }
            .map { LineaPedido(it, uiState.cantidades.getValue(it.id)) }
```

`lineasPedido` recorre `productos` (no las claves del mapa) para que el resumen salga siempre en el orden de la carta. El `filter` descarta los productos que se añadieron y luego se quitaron hasta 0, que siguen en el mapa con cantidad 0. Después del filtro, `getValue` es seguro: la clave existe.

### Ejercicio 5 — `ui/carta/PantallaCarta.kt`

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
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.Categoria
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme
import kotlinx.coroutines.launch

@Composable
fun PantallaCarta(
    pedidoViewModel: PedidoViewModel,
    onProductoClick: (Int) -> Unit,
    onVerPedido: () -> Unit,
    modifier: Modifier = Modifier
) {
    val estado = pedidoViewModel.uiState
    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()
    val mensajeVaciado = stringResource(R.string.pedido_vaciado)

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
                textoBoton = R.string.ver_pedido,
                onBotonClick = onVerPedido
            )
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { innerPadding ->
        ContenidoCarta(
            productos = pedidoViewModel.productosVisibles,
            cantidades = estado.cantidades,
            categoria = estado.categoria,
            onCantidadChange = pedidoViewModel::cambiarCantidad,
            onCategoriaChange = pedidoViewModel::seleccionarCategoria,
            onProductoClick = onProductoClick,
            modifier = Modifier.padding(innerPadding)
        )
    }
}

@Composable
fun ContenidoCarta(
    productos: List<Producto>,
    cantidades: Map<Int, Int>,
    categoria: Categoria?,
    onCantidadChange: (Int, Int) -> Unit,
    onCategoriaChange: (Categoria?) -> Unit,
    onProductoClick: (Int) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyColumn(
        modifier = modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        item {
            CabeceraCarta()
        }
        item {
            FiltroCategorias(seleccionada = categoria, onSeleccionar = onCategoriaChange)
        }
        items(items = productos, key = { it.id }) { producto ->
            ProductoCard(
                producto = producto,
                cantidad = cantidades[producto.id] ?: 0,
                onCantidadChange = { nueva -> onCantidadChange(producto.id, nueva) },
                onClick = { onProductoClick(producto.id) }
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
                cantidades = mapOf(1 to 2, 3 to 1),
                categoria = null,
                onCantidadChange = { _, _ -> },
                onCategoriaChange = {},
                onProductoClick = {}
            )
        }
    }
}
```

Desaparecen de esta pantalla `DatosPedido`, `DialogoConfirmacion`, el `imePadding` (ya no hay campo de texto) y el mensaje `pedido_enviado`. La pantalla **no sabe** qué pasa al tocar un producto o "Ver pedido": solo llama a las lambdas.

### Ejercicio 6 — `ui/carta/PantallaDetalle.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme

@Composable
fun PantallaDetalle(
    idProducto: Int,
    pedidoViewModel: PedidoViewModel,
    onVolver: () -> Unit,
    modifier: Modifier = Modifier
) {
    val producto = pedidoViewModel.buscarProducto(idProducto) ?: return
    val cantidad = pedidoViewModel.uiState.cantidades[producto.id] ?: 0

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraConVolver(titulo = stringResource(producto.nombre), onVolver = onVolver)
        }
    ) { innerPadding ->
        ContenidoDetalle(
            producto = producto,
            cantidad = cantidad,
            onCantidadChange = { nueva -> pedidoViewModel.cambiarCantidad(producto.id, nueva) },
            modifier = Modifier.padding(innerPadding)
        )
    }
}

@Composable
fun ContenidoDetalle(
    producto: Producto,
    cantidad: Int,
    onCantidadChange: (Int) -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .verticalScroll(rememberScrollState())
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Icon(
            painter = painterResource(producto.icono),
            contentDescription = null,
            tint = MaterialTheme.colorScheme.primary,
            modifier = Modifier.size(120.dp)
        )
        Text(
            text = stringResource(producto.nombre),
            style = MaterialTheme.typography.headlineMedium
        )
        Text(
            text = stringResource(producto.categoria.nombre),
            style = MaterialTheme.typography.labelLarge,
            color = MaterialTheme.colorScheme.secondary
        )
        Text(
            text = stringResource(producto.descripcion),
            style = MaterialTheme.typography.bodyLarge,
            textAlign = TextAlign.Center
        )
        Text(
            text = stringResource(R.string.precio, producto.precio),
            style = MaterialTheme.typography.titleLarge,
            color = MaterialTheme.colorScheme.primary
        )
        SelectorCantidad(cantidad = cantidad, onCantidadChange = onCantidadChange)
    }
}

@Preview(showBackground = true)
@Composable
fun ContenidoDetallePreview() {
    CafeteriaTheme {
        Surface {
            ContenidoDetalle(producto = DatosCarta.productos.first(), cantidad = 1, onCantidadChange = {})
        }
    }
}
```

`?: return` sale de la función sin dibujar nada si el id no existe: con los datos de `DatosCarta` no debería pasar nunca, pero el compilador obliga a decidir qué hacer con el `Producto?`. Aquí no sirve una referencia a función para `onCantidadChange`: el selector solo da la nueva cantidad y hay que añadirle el id.

### Ejercicio 7 — `ui/carta/PantallaResumen.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import android.content.res.Configuration
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme

@Composable
fun PantallaResumen(
    pedidoViewModel: PedidoViewModel,
    onVolver: () -> Unit,
    onPedidoEnviado: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    val estado = pedidoViewModel.uiState

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraConVolver(titulo = stringResource(R.string.titulo_resumen), onVolver = onVolver)
        },
        bottomBar = {
            BarraPedido(
                total = pedidoViewModel.total,
                unidades = pedidoViewModel.unidades,
                textoBoton = R.string.confirmar,
                onBotonClick = pedidoViewModel::confirmar
            )
        }
    ) { innerPadding ->
        ContenidoResumen(
            lineas = pedidoViewModel.lineasPedido,
            estado = estado,
            errorNombre = pedidoViewModel.errorNombre,
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
                val nombre = estado.nombre
                pedidoViewModel.enviarPedido()
                onPedidoEnviado(nombre)
            },
            onCancelar = pedidoViewModel::cancelarConfirmacion
        )
    }
}

@Composable
fun ContenidoResumen(
    lineas: List<LineaPedido>,
    estado: PedidoUiState,
    errorNombre: Boolean,
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
        if (lineas.isEmpty()) {
            item {
                Text(
                    text = stringResource(R.string.pedido_vacio),
                    style = MaterialTheme.typography.bodyLarge
                )
            }
        }
        items(items = lineas, key = { it.producto.id }) { linea ->
            FilaPedido(linea = linea)
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

@Composable
fun FilaPedido(linea: LineaPedido, modifier: Modifier = Modifier) {
    Row(modifier = modifier.fillMaxWidth(), verticalAlignment = Alignment.CenterVertically) {
        Text(
            text = stringResource(
                R.string.linea_pedido,
                linea.cantidad,
                stringResource(linea.producto.nombre)
            ),
            style = MaterialTheme.typography.bodyLarge,
            modifier = Modifier.weight(1f)
        )
        Text(
            text = stringResource(R.string.precio, linea.cantidad * linea.producto.precio),
            style = MaterialTheme.typography.bodyLarge
        )
    }
}

@Preview(name = "Con pedido", showBackground = true)
@Preview(name = "Oscuro", showBackground = true, uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
fun ContenidoResumenPreview() {
    CafeteriaTheme {
        Surface {
            ContenidoResumen(
                lineas = listOf(
                    LineaPedido(DatosCarta.productos[0], 2),
                    LineaPedido(DatosCarta.productos[2], 1)
                ),
                estado = PedidoUiState(nombre = "Ana"),
                errorNombre = false,
                onNombreChange = {},
                onParaLlevarChange = {},
                onAvisarChange = {}
            )
        }
    }
}
```

Dentro de una `LazyColumn` se puede usar `if` para decidir qué `item` se añaden: es código Kotlin normal que se ejecuta al construir la lista. En `onEnviar`, el nombre se guarda en una variable **antes** de enviar: así el código no depende de lo que haga `enviarPedido()` con el estado.

### Ejercicio 8 — `ui/carta/PantallaPedidoEnviado.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme

@Composable
fun PantallaPedidoEnviado(
    nombre: String,
    onVolverCarta: () -> Unit,
    modifier: Modifier = Modifier
) {
    Scaffold(modifier = modifier) { innerPadding ->
        Column(
            modifier = Modifier
                .padding(innerPadding)
                .fillMaxSize()
                .padding(24.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.Center
        ) {
            Icon(
                painter = painterResource(R.drawable.baseline_check_circle_24),
                contentDescription = null,
                tint = MaterialTheme.colorScheme.primary,
                modifier = Modifier.size(96.dp)
            )
            Spacer(modifier = Modifier.height(16.dp))
            Text(
                text = stringResource(R.string.titulo_enviado),
                style = MaterialTheme.typography.headlineMedium
            )
            Spacer(modifier = Modifier.height(8.dp))
            Text(
                text = stringResource(R.string.mensaje_enviado, nombre),
                style = MaterialTheme.typography.bodyLarge,
                textAlign = TextAlign.Center
            )
            Spacer(modifier = Modifier.height(24.dp))
            Button(onClick = onVolverCarta) {
                Text(stringResource(R.string.volver_carta))
            }
        }
    }
}

@Preview(showBackground = true)
@Composable
fun PantallaPedidoEnviadoPreview() {
    CafeteriaTheme {
        PantallaPedidoEnviado(nombre = "Ana", onVolverCarta = {})
    }
}
```

Esta pantalla sí tiene preview directamente: no usa el `ViewModel`, solo recibe un texto y una lambda.

### Ejercicio 9 — `ui/navegacion/AppCafeteria.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.navegacion

import androidx.compose.runtime.Composable
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.toRoute
import com.iesejemplo.cafeteria.ui.carta.PantallaCarta
import com.iesejemplo.cafeteria.ui.carta.PantallaDetalle
import com.iesejemplo.cafeteria.ui.carta.PantallaPedidoEnviado
import com.iesejemplo.cafeteria.ui.carta.PantallaResumen
import com.iesejemplo.cafeteria.ui.carta.PedidoViewModel

@Composable
fun AppCafeteria() {
    val navController = rememberNavController()
    // Un único ViewModel para toda la app: se pide FUERA del NavHost
    val pedidoViewModel: PedidoViewModel = viewModel()

    NavHost(navController = navController, startDestination = Carta) {

        composable<Carta> {
            PantallaCarta(
                pedidoViewModel = pedidoViewModel,
                onProductoClick = { id -> navController.navigate(Detalle(id)) },
                onVerPedido = {
                    navController.navigate(Resumen) { launchSingleTop = true }
                }
            )
        }

        composable<Detalle> { entrada ->
            val ruta: Detalle = entrada.toRoute()
            PantallaDetalle(
                idProducto = ruta.idProducto,
                pedidoViewModel = pedidoViewModel,
                onVolver = { navController.navigateUp() }
            )
        }

        composable<Resumen> {
            PantallaResumen(
                pedidoViewModel = pedidoViewModel,
                onVolver = { navController.navigateUp() },
                onPedidoEnviado = { nombre ->
                    navController.navigate(PedidoEnviado(nombre)) {
                        popUpTo<Carta>()
                    }
                }
            )
        }

        composable<PedidoEnviado> { entrada ->
            val ruta: PedidoEnviado = entrada.toRoute()
            PantallaPedidoEnviado(
                nombre = ruta.nombre,
                onVolverCarta = { navController.popBackStack() }
            )
        }
    }
}
```

### Ejercicio 9 — `MainActivity.kt` (solo cambia `setContent`)

```kotlin
setContent {
    CafeteriaTheme {
        AppCafeteria()
    }
}
```

(con `import com.iesejemplo.cafeteria.ui.navegacion.AppCafeteria`). Ya no hace falta el `Modifier.fillMaxSize()`: cada pantalla tiene su propio `Scaffold`, que ocupa toda la ventana.

### Ejercicio 10 — `PRUEBAS.md`, sección "Sesión 5" (resultados esperables)

| # | Prueba | Resultado obtenido |
|---|---|---|
| P5-1 | Detalle del café, +2, volver | OK. Pila: `Carta` → `Carta, Detalle(1)` → `Carta`. La carta muestra 2 cafés y "Total: 2,40 €": las dos pantallas leen el mismo `ViewModel` |
| P5-2 | "Ver pedido" dos veces rápido | Con `launchSingleTop`: pila `Carta, Resumen`; un Atrás vuelve a la carta. Sin él: pila `Carta, Resumen, Resumen`; hacen falta dos Atrás, y el primero lleva a un resumen idéntico, lo que parece que "no hace nada" |
| P5-3 | Confirmar sin nombre | OK: error bajo el campo y no se abre el diálogo (la regla sigue en `confirmar()` del `ViewModel`, no ha habido que tocarla) |
| P5-4 | Enviar y pulsar Atrás | OK. Pila antes de enviar: `Carta, Resumen`; después: `Carta, PedidoEnviado("Ana")` gracias a `popUpTo<Carta>()`. Atrás → `Carta`, con el pedido vacío |
| P5-5 | Resumen con pedido vacío | OK: mensaje `pedido_vacio`; "Confirmar" desactivado porque `BarraPedido` lo habilita solo si `unidades > 0` |
| P5-6 | Girar en el detalle | OK: misma pantalla y misma cantidad. La pila (con `Detalle(id)`) la guarda `rememberNavController`; la cantidad, el `ViewModel` |
| P5-7 | Recorrer las cuatro pantallas con `tag:PedidoVM` | OK: "PedidoViewModel creado" una sola vez, al abrir la app |
| P5-8 | `viewModel()` dentro de `composable<Detalle>` | El detalle muestra **cantidad 0** aunque en la carta haya unidades. Logcat: "PedidoViewModel creado" al entrar en el detalle y "PedidoViewModel destruido (onCleared)" al salir. Ese `ViewModel` pertenece a la entrada `Detalle` de la pila: nace con ella y muere cuando sale de la pila. Además, lo que se añade en ese detalle no llega a la carta |
| P5-9 | *No conservar actividades* en el detalle | Vuelve al **mismo detalle** (la pila y el argumento `idProducto` se guardan en el `Bundle`), pero con **cantidad 0**: el `ViewModel` se destruyó con la actividad y el nuevo empieza vacío. El producto se puede mostrar porque se busca por id en `DatosCarta` |

P5-8 y P5-9 son las pruebas clave de la sesión. P5-8 demuestra por qué el `ViewModel` se pide fuera del `NavHost`; P5-9, por qué por la ruta solo viaja el id: es lo único que sobrevive con seguridad.

### Opcional — Transiciones animadas

```kotlin
import androidx.compose.animation.AnimatedContentTransitionScope.SlideDirection

NavHost(
    navController = navController,
    startDestination = Carta,
    enterTransition = { slideIntoContainer(SlideDirection.Start) },
    exitTransition = { slideOutOfContainer(SlideDirection.Start) },
    popEnterTransition = { slideIntoContainer(SlideDirection.End) },
    popExitTransition = { slideOutOfContainer(SlideDirection.End) }
) {
    // ... los mismos composable<...> de antes
}
```

`enter`/`exit` se aplican al avanzar (`navigate`) y `popEnter`/`popExit` al volver. `Start` y `End` respetan los idiomas que se escriben de derecha a izquierda: en árabe o hebreo, el deslizamiento se invierte solo.

## Qué comprobar en cada elemento obligatorio

- **Gradle** → el plugin aparece en los dos `build.gradle.kts` (raíz con `apply false`, `app` sin él). Si falta en `app`, el error es *Serializer for class 'Detalle' is not found*.
- **Rutas** → `data object` para las pantallas sin argumentos y `data class` para las que los tienen, todas con `@Serializable`. `Detalle` lleva un `Int`, no un `Producto`.
- **`AppCafeteria`** → `viewModel()` aparece **una sola vez**, antes del `NavHost`. Ninguna pantalla recibe `navController`: buscad "navController" en el proyecto, solo debe salir en `AppCafeteria.kt`.
- **Pantallas** → `PedidoViewModel` como parámetro **sin** `= viewModel()`.
- **Pila** → `launchSingleTop` en "Ver pedido" y `popUpTo<Carta>()` al enviar; las pruebas P5-2 y P5-4 con las pilas dibujadas.
- **Componentes** → `ProductoCard` y `ContenidoDetalle` usan el mismo `SelectorCantidad`; `BarraPedido` se usa en la carta ("Ver pedido") y en el resumen ("Confirmar").
- **`PRUEBAS.md`** → P5-8 y P5-9 explicadas con los apartados 5.8 y 5.9, no solo descritas.
