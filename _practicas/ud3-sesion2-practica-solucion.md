---
title: "Cafetería del instituto: el pedido — Solución (Sesión 2)"
unidad: UD3
sesion: 2
orden: 2
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Solo se muestran los ficheros que cambian respecto a la solución de la sesión 1; el resto (<code>Producto.kt</code>, <code>DatosCarta.kt</code>, <code>CabeceraCarta.kt</code>, tema) sigue igual. Los nombres de los iconos (<code>baseline_add_24</code>, <code>baseline_remove_24</code>) son los que Android Studio pone por defecto.
</div>

### Ejercicio 1 — `MainActivity.kt`

```kotlin
package com.iesejemplo.cafeteria

import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Scaffold
import androidx.compose.ui.Modifier
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.ui.carta.PantallaCarta
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme

private const val TAG = "CicloVida"

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val origen = if (savedInstanceState == null) "es null" else "trae datos guardados"
        Log.d(TAG, "onCreate (savedInstanceState $origen)")
        enableEdgeToEdge()
        setContent {
            CafeteriaTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    PantallaCarta(
                        productos = DatosCarta.productos,
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }

    override fun onStart() {
        super.onStart()
        Log.d(TAG, "onStart")
    }

    override fun onResume() {
        super.onResume()
        Log.d(TAG, "onResume")
    }

    override fun onPause() {
        super.onPause()
        Log.d(TAG, "onPause")
    }

    override fun onStop() {
        super.onStop()
        Log.d(TAG, "onStop")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(TAG, "onDestroy")
    }
}
```

### Ejercicio 3 — Recursos nuevos

Añadir a `res/values/strings.xml`:

```xml
    <string name="anadir">Añadir una unidad</string>
    <string name="quitar">Quitar una unidad</string>
```

Añadir a `res/values-en/strings.xml`:

```xml
    <string name="anadir">Add one</string>
    <string name="quitar">Remove one</string>
```

### Ejercicios 4 y 7 — `ui/carta/ProductoCard.kt` (versión final, con `rememberSaveable`)

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.*
import androidx.compose.material3.Card
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableIntStateOf
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme

@Composable
fun ProductoCard(producto: Producto, modifier: Modifier = Modifier) {
    // Estado local de la tarjeta: sobrevive a recomposiciones (remember)
    // y también a giros de pantalla y a la destrucción de la actividad (Saveable).
    var cantidad by rememberSaveable { mutableIntStateOf(0) }

    Card(modifier = modifier.fillMaxWidth()) {
        Column(modifier = Modifier.padding(16.dp)) {
            // Fila de la sesión 1: icono, nombre y descripción, precio
            Row(verticalAlignment = Alignment.CenterVertically) {
                Icon(
                    painter = painterResource(producto.icono),
                    contentDescription = null,
                    tint = MaterialTheme.colorScheme.primary,
                    modifier = Modifier.size(40.dp)
                )
                Spacer(modifier = Modifier.width(16.dp))
                Column(modifier = Modifier.weight(1f)) {
                    Text(
                        text = stringResource(producto.nombre),
                        style = MaterialTheme.typography.titleMedium
                    )
                    Text(
                        text = stringResource(producto.descripcion),
                        style = MaterialTheme.typography.bodySmall
                    )
                }
                Spacer(modifier = Modifier.width(8.dp))
                Text(
                    text = stringResource(R.string.precio, producto.precio),
                    style = MaterialTheme.typography.titleMedium,
                    color = MaterialTheme.colorScheme.primary
                )
            }
            // Fila nueva: selector de cantidad
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.End,
                verticalAlignment = Alignment.CenterVertically
            ) {
                IconButton(onClick = { cantidad-- }, enabled = cantidad > 0) {
                    Icon(
                        painter = painterResource(R.drawable.baseline_remove_24),
                        contentDescription = stringResource(R.string.quitar)
                    )
                }
                Text(
                    text = cantidad.toString(),
                    style = MaterialTheme.typography.titleMedium
                )
                IconButton(onClick = { cantidad++ }) {
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
        ProductoCard(producto = DatosCarta.productos.first())
    }
}
```

En la versión intermedia del ejercicio 4, la única diferencia es la línea del estado:

```kotlin
var cantidad by remember { mutableIntStateOf(0) }   // import androidx.compose.runtime.remember
```

Y en el ejercicio 5, la línea temporal (justo después de declarar `cantidad`, eliminada en la versión final):

```kotlin
Log.d("Recomposicion", "ProductoCard ${producto.nombre}: cantidad = $cantidad")
```

### Ejercicio 8 — `ui/carta/PantallaCarta.kt` (solo cambia la función principal)

```kotlin
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll

@Composable
fun PantallaCarta(productos: List<Producto>, modifier: Modifier = Modifier) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .verticalScroll(rememberScrollState())
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        CabeceraCarta()
        productos.forEach { producto ->
            ProductoCard(producto = producto)
        }
    }
}
```

El orden importa: `verticalScroll` va **antes** de `padding` para que el margen se desplace junto con el contenido.

### `PRUEBAS.md` — resultados esperables (emulador con Android 14)

**Ciclo de vida** (versión de Android del emulador: 14, API 34)

| # | Acción | Secuencia observada en Logcat |
|---|---|---|
| CV1 | Abrir la app | `onCreate (savedInstanceState es null)` → `onStart` → `onResume` |
| CV2 | Pulsar Inicio | `onPause` → `onStop` |
| CV3 | Volver desde apps recientes | `onStart` → `onResume` |
| CV4 | Girar a horizontal | `onPause` → `onStop` → `onDestroy` → `onCreate (savedInstanceState trae datos guardados)` → `onStart` → `onResume` |
| CV5 | Pulsar Atrás | `onPause` → `onStop` (en Android 11 o inferior, además `onDestroy`) |
| CV6 | Abrir de nuevo desde el icono | `onStart` → `onResume` (en Android 11 o inferior: `onCreate` → `onStart` → `onResume`) |

**Ejercicio 5 — Recomposición.** Al pulsar "+" en una tarjeta solo aparece en Logcat el mensaje de **esa** tarjeta. Cada tarjeta tiene su propio estado `cantidad` y solo ella lo lee, así que Compose solo vuelve a ejecutar la función de la tarjeta cuyo estado ha cambiado.

**Ejercicio 6 — Giro con `remember`.** Todas las cantidades vuelven a 0. En Logcat se ve que al girar se ejecuta `onDestroy` y después un nuevo `onCreate`: la actividad se destruye y se crea otra, con una interfaz nueva. `remember` solo guarda el valor mientras la interfaz existe, así que en la actividad nueva vuelve a ejecutar `mutableIntStateOf(0)`.

**Ejercicio 7 — Giro con `rememberSaveable`.** Las cantidades se conservan. `rememberSaveable` guarda el valor en el `Bundle` antes de destruir la actividad; por eso `savedInstanceState` "trae datos guardados" en el nuevo `onCreate` y la tarjeta recupera su cantidad.

**Ejercicio 8 — Desplazamiento.** La prueba 5 de la sesión 1 pasa: en horizontal se puede desplazar la carta hasta el último producto. Además, la posición del desplazamiento se conserva al girar: `rememberScrollState()` está construido internamente con `rememberSaveable`, igual que nuestra cantidad.

**Ejercicio 9 — "No conservar actividades".** Al pulsar Inicio aparece `onPause` → `onStop` → **`onDestroy`**. Al volver: `onCreate (savedInstanceState trae datos guardados)` → `onStart` → `onResume`. Las cantidades se conservan gracias a `rememberSaveable`. Con `remember` se habrían perdido: es lo que le pasaría a un usuario real si el sistema eliminase la app en segundo plano por falta de memoria.

**Opcional — Error provocado.** Sin `super.onStop()`, al pulsar Inicio la app se cierra y Logcat muestra `FATAL EXCEPTION: main` con la excepción `android.util.SuperNotCalledException: Activity {com.iesejemplo.cafeteria/com.iesejemplo.cafeteria.MainActivity} did not call through to super.onStop()`.

## Qué comprobar en cada elemento obligatorio

- **Métodos del ciclo de vida** → los seis con `super` como primera línea; si falta alguno, la app se cierra con `SuperNotCalledException` (es justo el ejercicio opcional).
- **`TAG`** → constante `private const val` fuera de la clase; todos los `Log.d` la usan, de modo que el filtro `tag:CicloVida` muestra exactamente las secuencias.
- **`PRUEBAS.md`** → secuencias CV1-CV6 coherentes con la versión de Android anotada; explicación del ejercicio 6 que relacione `onDestroy`/`onCreate` con la pérdida del estado (no basta con decir "se pierde").
- **Selector** → `enabled = cantidad > 0` en el botón de quitar: nunca se llega a cantidades negativas y el botón aparece atenuado en 0.
- **`rememberSaveable` + `by`** → imports de `getValue`, `setValue` y `rememberSaveable`; nada de `.value`.
- **Recursos** → `contentDescription` de los botones con `stringResource`, traducidos; ningún texto nuevo escrito en el código.
- **Desplazamiento** → `verticalScroll(rememberScrollState())` antes de `padding`.
- **Sin `Log.d` de recomposición** en la entrega final: era solo para observar.
- **Lo que todavía no se pide**: el total del pedido. Si alguien lo ha intentado, habrá comprobado que desde `PantallaCarta` no puede leer la cantidad de cada tarjeta: es la pregunta con la que empieza la sesión 3.
