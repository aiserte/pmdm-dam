---
title: "Cafetería del instituto: confirmar el pedido — Solución (Sesión 3)"
unidad: UD3
sesion: 3
orden: 3
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Solo se muestran los ficheros nuevos o que cambian respecto a la solución de la sesión 2; <code>Producto.kt</code>, <code>DatosCarta.kt</code>, <code>CabeceraCarta.kt</code> y el tema siguen igual. El nombre del icono (<code>baseline_more_vert_24</code>) es el que pone Android Studio por defecto.
</div>

### Estructura final del paquete `ui/carta`

```
ui/carta/
├── PantallaCarta.kt         ← guarda TODO el estado del pedido
├── CabeceraCarta.kt         (sin cambios)
├── ProductoCard.kt          ← ahora stateless
├── BarraSuperiorCarta.kt    ← nuevo: TopAppBar + menú
├── BarraPedido.kt           ← nuevo: BottomAppBar con el total
├── DatosPedido.kt           ← nuevo: nombre, para llevar, avisar
└── DialogoConfirmacion.kt   ← nuevo: AlertDialog
```

### Ejercicio 1 — Recursos

Añadir a `res/values/strings.xml`:

```xml
    <string name="mas_opciones">Más opciones</string>
    <string name="vaciar_pedido">Vaciar pedido</string>
    <string name="pedido_vaciado">Pedido vaciado</string>
    <string name="total_pedido">Total: %1$.2f €</string>
    <string name="unidades_pedido">Unidades: %1$d</string>
    <string name="confirmar">Confirmar</string>
    <string name="nombre_pedido">¿A nombre de quién?</string>
    <string name="error_nombre">Escribe un nombre para el pedido</string>
    <string name="para_llevar">Para llevar</string>
    <string name="avisar_listo">Avisarme cuando esté listo</string>
    <string name="titulo_confirmacion">¿Enviar el pedido?</string>
    <string name="resumen_pedido">A nombre de %1$s · %2$d unidades · %3$.2f €</string>
    <string name="resumen_aqui">Para tomar aquí</string>
    <string name="resumen_aviso">Te avisaremos cuando esté listo</string>
    <string name="enviar">Enviar</string>
    <string name="cancelar">Cancelar</string>
    <string name="pedido_enviado">Pedido de %1$s enviado</string>
    <string name="deshacer">Deshacer</string>
```

Añadir a `res/values-en/strings.xml`:

```xml
    <string name="mas_opciones">More options</string>
    <string name="vaciar_pedido">Clear order</string>
    <string name="pedido_vaciado">Order cleared</string>
    <string name="total_pedido">Total: €%1$.2f</string>
    <string name="unidades_pedido">Items: %1$d</string>
    <string name="confirmar">Confirm</string>
    <string name="nombre_pedido">Name for the order</string>
    <string name="error_nombre">Please enter a name for the order</string>
    <string name="para_llevar">To take away</string>
    <string name="avisar_listo">Notify me when it\'s ready</string>
    <string name="titulo_confirmacion">Send the order?</string>
    <string name="resumen_pedido">For %1$s · %2$d items · €%3$.2f</string>
    <string name="resumen_aqui">To eat in</string>
    <string name="resumen_aviso">We\'ll let you know when it\'s ready</string>
    <string name="enviar">Send</string>
    <string name="cancelar">Cancel</string>
    <string name="pedido_enviado">Order for %1$s sent</string>
    <string name="deshacer">Undo</string>
```

### Ejercicio 2 — `ProductoCard.kt` (stateless)

Solo cambian la firma, los botones y la preview; el resto de la tarjeta es igual que en la sesión 2.

```kotlin
@Composable
fun ProductoCard(
    producto: Producto,
    cantidad: Int,
    onCantidadChange: (Int) -> Unit,
    modifier: Modifier = Modifier
) {
    Card(modifier = modifier.fillMaxWidth()) {
        Column(modifier = Modifier.padding(16.dp)) {
            Row(verticalAlignment = Alignment.CenterVertically) {
                // ... icono, nombre y descripción, precio (igual que en la sesión 2)
            }
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.End,
                verticalAlignment = Alignment.CenterVertically
            ) {
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
    }
}

@Preview(showBackground = true)
@Composable
fun ProductoCardPreview() {
    CafeteriaTheme {
        ProductoCard(producto = DatosCarta.productos.first(), cantidad = 2, onCantidadChange = {})
    }
}
```

Desaparecen los imports de `getValue`, `setValue`, `mutableIntStateOf` y `rememberSaveable`: la tarjeta ya no guarda nada. Fíjate en la preview: ahora se puede mostrar la tarjeta con cualquier cantidad, algo imposible cuando el estado estaba dentro.

### Ejercicio 3 — `MainActivity.kt` (solo cambia `onCreate`)

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val origen = if (savedInstanceState == null) "es null" else "trae datos guardados"
    Log.d(TAG, "onCreate (savedInstanceState $origen)")
    enableEdgeToEdge()
    setContent {
        CafeteriaTheme {
            PantallaCarta(
                productos = DatosCarta.productos,
                modifier = Modifier.fillMaxSize()
            )
        }
    }
}
// onStart, onResume, onPause, onStop y onDestroy: sin cambios
```

### Ejercicio 4 — `ui/carta/BarraSuperiorCarta.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.Box
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import com.iesejemplo.cafeteria.R

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraSuperiorCarta(
    hayPedido: Boolean,
    onVaciarPedido: () -> Unit,
    modifier: Modifier = Modifier
) {
    // Estado que solo necesita este componente: puede quedarse aquí
    var menuAbierto by remember { mutableStateOf(false) }

    TopAppBar(
        title = { Text(stringResource(R.string.app_name)) },
        modifier = modifier,
        colors = TopAppBarDefaults.topAppBarColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer,
            titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
        ),
        actions = {
            Box {
                IconButton(onClick = { menuAbierto = true }) {
                    Icon(
                        painter = painterResource(R.drawable.baseline_more_vert_24),
                        contentDescription = stringResource(R.string.mas_opciones)
                    )
                }
                DropdownMenu(
                    expanded = menuAbierto,
                    onDismissRequest = { menuAbierto = false }
                ) {
                    DropdownMenuItem(
                        text = { Text(stringResource(R.string.vaciar_pedido)) },
                        enabled = hayPedido,
                        onClick = {
                            menuAbierto = false
                            onVaciarPedido()
                        }
                    )
                }
            }
        }
    )
}
```

`menuAbierto` usa `remember` (no `rememberSaveable`): que el menú se cierre al girar la pantalla es el comportamiento esperado.

### Ejercicio 5 — `ui/carta/BarraPedido.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

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
    onConfirmar: () -> Unit,
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
            onClick = onConfirmar,
            enabled = unidades > 0,
            modifier = Modifier.padding(end = 16.dp)
        ) {
            Text(stringResource(R.string.confirmar))
        }
    }
}
```

### Ejercicio 6 — `ui/carta/DatosPedido.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.text.input.ImeAction
import androidx.compose.ui.text.input.KeyboardCapitalization
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R

@Composable
fun DatosPedido(
    nombre: String,
    onNombreChange: (String) -> Unit,
    mostrarError: Boolean,
    paraLlevar: Boolean,
    onParaLlevarChange: (Boolean) -> Unit,
    avisar: Boolean,
    onAvisarChange: (Boolean) -> Unit,
    modifier: Modifier = Modifier
) {
    Card(modifier = modifier.fillMaxWidth()) {
        Column(modifier = Modifier.padding(16.dp)) {
            OutlinedTextField(
                value = nombre,
                onValueChange = onNombreChange,
                label = { Text(stringResource(R.string.nombre_pedido)) },
                singleLine = true,
                isError = mostrarError,
                supportingText = {
                    if (mostrarError) Text(stringResource(R.string.error_nombre))
                },
                keyboardOptions = KeyboardOptions(
                    capitalization = KeyboardCapitalization.Words,
                    imeAction = ImeAction.Done
                ),
                modifier = Modifier.fillMaxWidth()
            )
            Row(verticalAlignment = Alignment.CenterVertically) {
                Text(
                    text = stringResource(R.string.para_llevar),
                    modifier = Modifier.weight(1f)
                )
                Switch(checked = paraLlevar, onCheckedChange = onParaLlevarChange)
            }
            Row(verticalAlignment = Alignment.CenterVertically) {
                Checkbox(checked = avisar, onCheckedChange = onAvisarChange)
                Text(text = stringResource(R.string.avisar_listo))
            }
        }
    }
}
```

### Ejercicio 7 — `ui/carta/DialogoConfirmacion.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.Column
import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.runtime.Composable
import androidx.compose.ui.res.stringResource
import com.iesejemplo.cafeteria.R

@Composable
fun DialogoConfirmacion(
    nombre: String,
    unidades: Int,
    total: Double,
    paraLlevar: Boolean,
    avisar: Boolean,
    onEnviar: () -> Unit,
    onCancelar: () -> Unit
) {
    AlertDialog(
        onDismissRequest = onCancelar,
        title = { Text(stringResource(R.string.titulo_confirmacion)) },
        text = {
            Column {
                Text(stringResource(R.string.resumen_pedido, nombre, unidades, total))
                Text(stringResource(if (paraLlevar) R.string.para_llevar else R.string.resumen_aqui))
                if (avisar) Text(stringResource(R.string.resumen_aviso))
            }
        },
        confirmButton = {
            TextButton(onClick = onEnviar) { Text(stringResource(R.string.enviar)) }
        },
        dismissButton = {
            TextButton(onClick = onCancelar) { Text(stringResource(R.string.cancelar)) }
        }
    )
}
```

### Ejercicios 2 a 7 — `ui/carta/PantallaCarta.kt` (el componente que guarda el estado)

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import android.content.res.Configuration
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme
import kotlinx.coroutines.launch

@Composable
fun PantallaCarta(productos: List<Producto>, modifier: Modifier = Modifier) {
    // --- Estado del pedido (elevado aquí desde los componentes) ---
    var cantidades by rememberSaveable { mutableStateOf(mapOf<Int, Int>()) }
    var nombre by rememberSaveable { mutableStateOf("") }
    var paraLlevar by rememberSaveable { mutableStateOf(false) }
    var avisar by rememberSaveable { mutableStateOf(true) }
    var intentoConfirmar by rememberSaveable { mutableStateOf(false) }
    var mostrarConfirmacion by rememberSaveable { mutableStateOf(false) }

    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()

    // --- Valores calculados a partir del estado (no son estado) ---
    val unidades = cantidades.values.sum()
    val total = productos.sumOf { (cantidades[it.nombre] ?: 0) * it.precio }
    val errorNombre = intentoConfirmar && nombre.isBlank()

    // --- Textos que se usan dentro de lambdas ---
    val mensajeVaciado = stringResource(R.string.pedido_vaciado)
    val mensajeEnviado = stringResource(R.string.pedido_enviado, nombre)

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraSuperiorCarta(
                hayPedido = unidades > 0,
                onVaciarPedido = {
                    cantidades = emptyMap()
                    scope.launch { snackbarHostState.showSnackbar(mensajeVaciado) }
                }
            )
        },
        bottomBar = {
            BarraPedido(
                total = total,
                unidades = unidades,
                onConfirmar = {
                    intentoConfirmar = true
                    if (nombre.isNotBlank()) mostrarConfirmacion = true
                }
            )
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { innerPadding ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding)
                .imePadding()
                .verticalScroll(rememberScrollState())
                .padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            CabeceraCarta()
            productos.forEach { producto ->
                ProductoCard(
                    producto = producto,
                    cantidad = cantidades[producto.nombre] ?: 0,
                    onCantidadChange = { nueva ->
                        cantidades = cantidades + (producto.nombre to nueva)
                    }
                )
            }
            DatosPedido(
                nombre = nombre,
                onNombreChange = { nombre = it },
                mostrarError = errorNombre,
                paraLlevar = paraLlevar,
                onParaLlevarChange = { paraLlevar = it },
                avisar = avisar,
                onAvisarChange = { avisar = it }
            )
        }
    }

    if (mostrarConfirmacion) {
        DialogoConfirmacion(
            nombre = nombre,
            unidades = unidades,
            total = total,
            paraLlevar = paraLlevar,
            avisar = avisar,
            onEnviar = {
                mostrarConfirmacion = false
                cantidades = emptyMap()
                intentoConfirmar = false
                scope.launch { snackbarHostState.showSnackbar(mensajeEnviado) }
            },
            onCancelar = { mostrarConfirmacion = false }
        )
    }
}

@Preview(name = "Español", showBackground = true)
@Preview(name = "Inglés", showBackground = true, locale = "en")
@Preview(name = "Oscuro", showBackground = true, uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
fun PantallaCartaPreview() {
    CafeteriaTheme {
        PantallaCarta(productos = DatosCarta.productos)
    }
}
```

La preview ya no necesita el `Surface` de la sesión 1: el `Scaffold` pinta su propio fondo con el color del tema.

### Opcional — "Deshacer" al vaciar

Solo cambia la lambda `onVaciarPedido` de `PantallaCarta` (y se añaden los imports de `SnackbarDuration` y `SnackbarResult`, incluidos en `androidx.compose.material3.*`):

```kotlin
val textoDeshacer = stringResource(R.string.deshacer)   // en el cuerpo, junto a los otros textos

// ...
onVaciarPedido = {
    val copia = cantidades          // guardamos el pedido antes de vaciarlo
    cantidades = emptyMap()
    scope.launch {
        val resultado = snackbarHostState.showSnackbar(
            message = mensajeVaciado,
            actionLabel = textoDeshacer,
            duration = SnackbarDuration.Short
        )
        if (resultado == SnackbarResult.ActionPerformed) {
            cantidades = copia
        }
    }
}
```

Funciona porque los `Map` son inmutables: `copia` apunta al mapa anterior, que nadie ha modificado; vaciar el pedido solo hace que `cantidades` apunte a otro mapa.

### Ejercicio 8 — `PRUEBAS.md`, sección "Sesión 3" (resultados esperables)

| # | Prueba | Resultado esperado | Resultado obtenido |
|---|---|---|---|
| P3-1 | Añadir productos y comprobar la barra inferior | El total y las unidades cambian al momento | OK: 2 cafés + 1 croissant → "Total: 3,90 €", "Unidades: 3" |
| P3-2 | Escribir un nombre y girar | Se conservan nombre, `Switch` y `Checkbox` | OK (los tres con `rememberSaveable`) |
| P3-3 | Tocar el campo del nombre abajo del todo | El teclado no tapa el campo | OK con `imePadding()`; sin él, el campo queda oculto tras el teclado |
| P3-4 | Confirmar sin nombre | Error en el campo, sin diálogo | OK: borde rojo y "Escribe un nombre para el pedido" |
| P3-5 | Abrir el diálogo y girar | El diálogo sigue abierto | OK (`mostrarConfirmacion` con `rememberSaveable`) |
| P3-6 | Abrir el diálogo con `tag:CicloVida` | Ningún método del ciclo de vida | OK: el diálogo forma parte de la misma actividad, que sigue en primer plano |
| P3-7 | TalkBack sobre los tres puntos | Lee "Más opciones" | OK |
| P3-8 | Flujo completo en inglés | Textos nuevos traducidos | OK: "Total: €3.90", "Send the order?", "Order for Ana sent" |

## Qué comprobar en cada elemento obligatorio

- **Componentes stateless** → `ProductoCard`, `DatosPedido`, `BarraPedido` y `DialogoConfirmacion` no tienen ningún `remember`; todo lo reciben por parámetro. Señal clara: sus previews se pueden escribir con valores fijos.
- **Estado en `PantallaCarta`** → seis variables con `rememberSaveable`; el mapa se reemplaza con `cantidades + (clave to valor)` y `emptyMap()`, nunca se modifica por dentro.
- **Valores calculados** → `total`, `unidades` y `errorNombre` son `val` normales, recalculados en cada recomposición. Guardarlos como estado aparte obligaría a mantenerlos sincronizados a mano (error típico).
- **`stringResource` fuera de lambdas** → `mensajeVaciado` y `mensajeEnviado` se obtienen en el cuerpo del composable.
- **Validación** → el error solo aparece después de intentar confirmar (`intentoConfirmar`), no al abrir la app con el campo vacío.
- **Diálogo** → `onDismissRequest` y Cancelar cierran sin enviar; Enviar vacía las cantidades pero conserva el nombre y las opciones (el mismo cliente puede hacer otro pedido).
- **`Scaffold`** → `innerPadding` aplicado; `imePadding()` presente; en `MainActivity` ya no hay `Scaffold`, pero siguen los seis métodos del ciclo de vida.
- **Lo que viene en la sesión 4**: `PantallaCarta` concentra ahora mucho estado y lógica (calcular el total, vaciar, enviar). Es exactamente lo que trasladaremos a un `ViewModel`.
