---
title: "Cafetería del instituto: la carta — Solución (Sesión 1)"
unidad: UD3
sesion: 1
orden: 1
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Los nombres de los iconos (<code>baseline_local_cafe_24</code>, etc.) son los que Android Studio pone por defecto; si en tu proyecto se llaman distinto, usa los tuyos. El paquete <code>com.iesejemplo.cafeteria</code> también será distinto en tu proyecto.
</div>

### Ejercicio 0 — `Theme.kt` (solo la línea que cambia)

```kotlin
@Composable
fun CafeteriaTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = false,   // antes: true
    content: @Composable () -> Unit
) {
    // ... resto sin cambios
}
```

### Ejercicio 2 — Recursos de texto

`res/values/strings.xml`:

```xml
<resources>
    <string name="app_name" translatable="false">Cafetería IES</string>
    <string name="titulo_carta">Carta de hoy</string>
    <string name="subtitulo_carta">Cafetería del instituto · Abierto de %1$s a %2$s</string>
    <string name="precio">%1$.2f €</string>
    <string name="logo_descripcion">Logotipo de la cafetería</string>

    <string name="producto_cafe">Café con leche</string>
    <string name="desc_cafe">Recién hecho, con leche entera o de avena</string>
    <string name="producto_bocadillo">Bocadillo de tortilla</string>
    <string name="desc_bocadillo">Pan del día y tortilla de patata casera</string>
    <string name="producto_croissant">Croissant</string>
    <string name="desc_croissant">Hojaldre de mantequilla, recién horneado</string>
    <string name="producto_zumo">Zumo de naranja</string>
    <string name="desc_zumo">Naranjas valencianas exprimidas al momento</string>
</resources>
```

`res/values-en/strings.xml`:

```xml
<resources>
    <string name="titulo_carta">Today\'s menu</string>
    <string name="subtitulo_carta">School cafeteria · Open from %1$s to %2$s</string>
    <string name="precio">€%1$.2f</string>
    <string name="logo_descripcion">Cafeteria logo</string>

    <string name="producto_cafe">Coffee with milk</string>
    <string name="desc_cafe">Freshly brewed, with whole or oat milk</string>
    <string name="producto_bocadillo">Spanish omelette sandwich</string>
    <string name="desc_bocadillo">Fresh bread and homemade potato omelette</string>
    <string name="producto_croissant">Croissant</string>
    <string name="desc_croissant">Butter puff pastry, freshly baked</string>
    <string name="producto_zumo">Orange juice</string>
    <string name="desc_zumo">Valencian oranges, freshly squeezed</string>
</resources>
```

### Ejercicio 4 — El modelo

`data/Producto.kt`:

```kotlin
package com.iesejemplo.cafeteria.data

import androidx.annotation.DrawableRes
import androidx.annotation.StringRes

data class Producto(
    @StringRes val nombre: Int,
    @StringRes val descripcion: Int,
    @DrawableRes val icono: Int,
    val precio: Double
)
```

`data/DatosCarta.kt`:

```kotlin
package com.iesejemplo.cafeteria.data

import com.iesejemplo.cafeteria.R

object DatosCarta {
    val productos = listOf(
        Producto(R.string.producto_cafe, R.string.desc_cafe, R.drawable.baseline_local_cafe_24, 1.20),
        Producto(R.string.producto_bocadillo, R.string.desc_bocadillo, R.drawable.baseline_lunch_dining_24, 2.80),
        Producto(R.string.producto_croissant, R.string.desc_croissant, R.drawable.baseline_bakery_dining_24, 1.50),
        Producto(R.string.producto_zumo, R.string.desc_zumo, R.drawable.baseline_local_drink_24, 2.00)
    )
}
```

`object` declara una clase con **una única instancia**, creada automáticamente: se usa directamente como `DatosCarta.productos`, sin `DatosCarta()`. Es la forma habitual en Kotlin de agrupar datos o utilidades que no necesitan varias copias.

### Ejercicio 5 — `ui/carta/ProductoCard.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.*
import androidx.compose.material3.Card
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
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
    Card(modifier = modifier.fillMaxWidth()) {
        Row(
            modifier = Modifier.padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Icon(
                painter = painterResource(producto.icono),
                contentDescription = null, // decorativo: el nombre del producto está justo al lado
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

### Ejercicio 6 — `ui/carta/CabeceraCarta.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R

@Composable
fun CabeceraCarta(modifier: Modifier = Modifier) {
    Column(
        modifier = modifier.fillMaxWidth(),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Box(
            modifier = Modifier
                .size(96.dp)
                .clip(CircleShape)
        ) {
            Image(
                painter = painterResource(R.drawable.ic_launcher_background),
                contentDescription = null, // fondo decorativo
                modifier = Modifier.fillMaxSize()
            )
            Image(
                painter = painterResource(R.drawable.ic_launcher_foreground),
                contentDescription = stringResource(R.string.logo_descripcion),
                modifier = Modifier.fillMaxSize()
            )
        }
        Spacer(modifier = Modifier.height(8.dp))
        Text(
            text = stringResource(R.string.titulo_carta),
            style = MaterialTheme.typography.headlineMedium
        )
        Text(
            text = stringResource(R.string.subtitulo_carta, "8:00", "13:30"),
            style = MaterialTheme.typography.bodyMedium
        )
    }
}
```

### Ejercicios 7 y 8 — `ui/carta/PantallaCarta.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import android.content.res.Configuration
import androidx.compose.foundation.layout.*
import androidx.compose.material3.Surface
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme

@Composable
fun PantallaCarta(productos: List<Producto>, modifier: Modifier = Modifier) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        CabeceraCarta()
        productos.forEach { producto ->
            ProductoCard(producto = producto)
        }
    }
}

@Preview(name = "Español", showBackground = true)
@Preview(name = "Inglés", showBackground = true, locale = "en")
@Preview(name = "Oscuro", showBackground = true, uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
fun PantallaCartaPreview() {
    CafeteriaTheme {
        Surface {
            PantallaCarta(productos = DatosCarta.productos)
        }
    }
}
```

### Ejercicio 7 — `MainActivity.kt`

```kotlin
package com.iesejemplo.cafeteria

import android.os.Bundle
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

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
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
}
```

### Ejercicio 9 — `PRUEBAS.md` (resultado esperable)

| # | Prueba | Resultado esperado | Resultado obtenido |
|---|---|---|---|
| 1 | Idioma del sistema en inglés | Todos los textos en inglés, precio con "€" delante | OK: "Today's menu", "€1.20" |
| 2 | Volver a español | Todos los textos en español, precio con coma decimal | OK: "1,20 €" |
| 3 | Modo oscuro | Todo legible | OK: fondo oscuro, texto claro |
| 4 | Tamaño de fuente máximo | Ningún texto se sale de su tarjeta | OK: las descripciones pasan a dos líneas; el precio se mantiene a la derecha |
| 5 | Orientación horizontal | Se ve toda la carta | **Falla**: los últimos productos quedan por debajo del borde de la pantalla y no se puede desplazar |
| 6 | TalkBack sobre el logotipo | Lee la descripción del logotipo | OK: "Logotipo de la cafetería" |

La prueba 5 **debe fallar** con lo visto hasta ahora: una `Column` no permite desplazamiento por sí sola. Detectarlo y documentarlo es precisamente el objetivo del criterio g). Solución rápida (ampliación): añadir `.verticalScroll(rememberScrollState())` al `Modifier` de la `Column` de `PantallaCarta`. La solución adecuada para listas, `LazyColumn`, llega en la sesión 4.

## Qué comprobar en cada elemento obligatorio

- **Estructura de paquetes** → `data/` solo contiene Kotlin sin nada de Compose; `ui/carta/` contiene un fichero por composable; `MainActivity` no contiene ningún componente de interfaz propio.
- **Sin textos en el código** → Android Studio no muestra ningún aviso *Hardcoded string*. Único texto literal permitido: las horas `"8:00"` y `"13:30"`, que son datos, no textos de interfaz.
- **Dos idiomas** → el Translations Editor no muestra ninguna celda en rojo en la columna de inglés.
- **Argumentos numerados** → `subtitulo_carta` usa `%1$s` y `%2$s`; `precio` usa `%1$.2f`, y en inglés el símbolo cambia de posición sin tocar el código.
- **`@StringRes` / `@DrawableRes`** → si se intercambian por error (pasar `R.drawable.x` como nombre), Android Studio lo subraya.
- **Parámetro `modifier`** → se aplica una sola vez, al elemento raíz (`Card` en `ProductoCard`, `Column` en `CabeceraCarta` y `PantallaCarta`). Los hijos usan `Modifier` (con mayúscula), no `modifier`.
- **`contentDescription`** → `null` en el icono del producto (su nombre ya está al lado) y en el fondo del logo; texto real en el logotipo.
- **Previews** → el `Surface` hace que la preview oscura tenga fondo oscuro; sin él, `showBackground` pinta fondo blanco y el texto claro del modo oscuro no se lee.
- **`PRUEBAS.md`** → las seis pruebas con resultado; la 5 identificada como fallo con descripción del problema.
