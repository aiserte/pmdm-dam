---
title: "Navegación entre pantallas con Navigation Compose"
unit: UD3
order: 5
duration: "2h"
---

## Objetivos

- Entender cómo se organiza una app Android de varias pantallas: una sola `Activity` y muchas pantallas composables.
- Conocer las tres piezas de *Navigation Compose*: el `NavController`, el `NavHost` y las rutas.
- Definir rutas con seguridad de tipos (`@Serializable`) y pasar argumentos entre pantallas.
- Comprender la **pila de pantallas** (*back stack*) y controlarla con `navigateUp`, `popBackStack`, `launchSingleTop` y `popUpTo`.
- Compartir un mismo `ViewModel` entre varias pantallas sin crear copias por error.
- Comprobar en el emulador cómo se comporta la navegación al girar la pantalla, al pulsar Atrás y cuando Android elimina la app.

<div class="callout-info">
Esta sesión trabaja los criterios <strong>a)</strong> (la app pasa a tener una estructura de pantallas, rutas y un punto central de navegación), <strong>b)</strong> (ventanas de la app y sus barras con botón de volver) y <strong>g)</strong> (pruebas de navegación en el emulador) del RA2.
</div>

## 5.1 De una pantalla a varias

Hasta ahora la cafetería lo hacía todo en una sola pantalla: la carta, los datos del cliente y la confirmación del pedido. Las apps reales reparten el trabajo en **varias pantallas**, cada una con una función clara. Hoy la cafetería tendrá cuatro:

| Pantalla | Qué muestra | Desde dónde se llega |
|---|---|---|
| **Carta** | Cabecera, filtro y lista de productos | Es la pantalla inicial |
| **Detalle** | Un producto en grande, con su descripción y el selector de cantidad | Tocando una tarjeta de la carta |
| **Resumen** | Las líneas del pedido, el total y los datos del cliente | Con el botón "Ver pedido" |
| **Pedido enviado** | Un mensaje de agradecimiento | Al enviar el pedido desde el resumen |

### Una sola `Activity`, muchas pantallas

En el Android clásico (el que visteis por encima en la UD1), cada pantalla era una `Activity` distinta y se pasaba de una a otra con un `Intent`. Hoy Google recomienda la arquitectura de **una sola `Activity`**: la `MainActivity` solo hace `setContent { ... }` y, dentro, una pieza de Compose decide qué pantalla se muestra en cada momento. Cambiar de pantalla es tan barato como recomponer: no hay que crear otra `Activity`, ni declararla en el manifiesto, ni pasar datos entre procesos.

La librería que hace ese trabajo es **Navigation Compose**, parte de Jetpack.

### La pila de pantallas (*back stack*)

Cuando el usuario va de la carta al detalle de un producto, la carta no desaparece: queda **debajo**, esperando. Android guarda las pantallas visitadas en una **pila** (*back stack*), como una pila de platos: la pantalla visible es la de arriba, y el botón Atrás quita la de arriba y deja ver la anterior.

```
Abrir la app     Tocar "Café"     Pulsar Atrás
                 ┌──────────┐
                 │ Detalle  │ ← visible
┌──────────┐     ├──────────┤     ┌──────────┐
│ Carta    │     │ Carta    │     │ Carta    │ ← visible
└──────────┘     └──────────┘     └──────────┘
```

Casi todo lo que hay que aprender de navegación consiste en decidir **qué entra y qué sale de esta pila** en cada momento.

## 5.2 Las tres piezas de Navigation Compose

| Pieza | Qué es | Analogía en la cafetería |
|---|---|---|
| **Ruta** | El "nombre" de una pantalla, con los datos que necesita (por ejemplo, el detalle **del producto 3**) | El nombre de cada zona: barra, mesas, caja |
| **`NavHost`** | El composable que muestra la pantalla de arriba de la pila y sabe qué composable corresponde a cada ruta | El plano del local: qué hay en cada zona |
| **`NavController`** | El objeto que gestiona la pila: añade pantallas (`navigate`), las quita (`navigateUp`, `popBackStack`) y recuerda dónde estamos | El encargado que acompaña al cliente de una zona a otra |

```kotlin
val navController = rememberNavController()

NavHost(navController = navController, startDestination = Carta) {
    composable<Carta> { /* pantalla de la carta */ }
    composable<Detalle> { /* pantalla de detalle */ }
    composable<Resumen> { /* pantalla de resumen */ }
}
```

- `rememberNavController()` crea el `NavController` y lo recuerda entre recomposiciones (y entre giros de pantalla, como `rememberSaveable`).
- `startDestination` es la pantalla con la que arranca la app: la primera de la pila.
- Cada `composable<Ruta> { }` asocia una ruta con el composable que la dibuja. Es parecido a un `when`: "si arriba de la pila está `Detalle`, dibuja esto".

## 5.3 Añadir las dependencias

Navigation Compose no viene en la plantilla. Además, las rutas con seguridad de tipos que vamos a usar necesitan la librería de **serialización** de Kotlin y su **plugin** de compilación. Es el segundo *just in time* de Gradle de la unidad, y esta vez aparece una sección nueva del catálogo: los *plugins*.

**Paso 1.** En `gradle/libs.versions.toml`:

```
[versions]
# ... las que ya hay
navigationCompose = "2.9.0"
kotlinxSerializationJson = "1.8.1"

[libraries]
# ... las que ya hay
androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigationCompose" }
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinxSerializationJson" }

[plugins]
# ... los que ya hay
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```

**Paso 2.** En el `build.gradle.kts` **del proyecto** (el de la raíz, no el de `app`), dentro de `plugins { }`:

```kotlin
alias(libs.plugins.kotlin.serialization) apply false
```

**Paso 3.** En `app/build.gradle.kts`, dentro de `plugins { }`:

```kotlin
alias(libs.plugins.kotlin.serialization)
```

y dentro de `dependencies { }`:

```kotlin
implementation(libs.androidx.navigation.compose)
implementation(libs.kotlinx.serialization.json)
```

**Paso 4.** *Sync Now*.

¿Por qué dos ficheros de Gradle para el plugin? El de la raíz **declara** qué plugins existen en el proyecto y en qué versión (`apply false`: "tenlo disponible, pero no lo apliques aquí"); el de `app` lo **aplica** al módulo que lo usa. Las librerías, en cambio, solo se añaden en `app`.

<div class="callout-warning">
El plugin de serialización debe ir con <strong>la misma versión que Kotlin</strong>; por eso usa <code>version.ref = "kotlin"</code>. Comprobad que en la sección <code>[versions]</code> de vuestro catálogo existe una clave <code>kotlin</code>; si se llama de otra forma, usad ese nombre. Si Android Studio sugiere versiones más recientes de <code>navigationCompose</code> o <code>kotlinxSerializationJson</code> (las subraya en amarillo), podéis aceptarlas.
</div>

## 5.4 Definir las rutas

Cada pantalla tiene una ruta, que es una clase de Kotlin marcada con `@Serializable`. Las ponemos todas juntas en un fichero `ui/navegacion/Rutas.kt`:

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

- Una pantalla que **no necesita datos** es un `data object`: solo existe una "carta", una "pantalla de resumen".
- Una pantalla que **necesita datos** es una `data class` cuyas propiedades son los **argumentos**: no hay "un detalle", hay "el detalle del producto 3".
- `@Serializable` permite a Navigation convertir la ruta (con sus argumentos) en datos que puede guardar y restaurar. Es lo que necesitaba el plugin del apartado anterior.

<div class="callout-info">
<strong>Rutas con texto.</strong> En tutoriales antiguos veréis rutas escritas como cadenas: <code>composable("detalle/{idProducto}")</code> y <code>navigate("detalle/3")</code>. Funcionan, pero un error al escribir la cadena solo se descubre al ejecutar. Con las rutas <code>@Serializable</code>, el compilador comprueba que pasáis un <code>Int</code> donde se espera un <code>Int</code>. Es la forma recomendada desde la versión 2.8 de Navigation Compose.
</div>

## 5.5 El `NavHost` y las acciones de navegar

### El composable raíz de la app

Toda la navegación vive en un composable nuevo, `AppCafeteria`, que pasa a ser lo que dibuja `MainActivity`:

```kotlin
@Composable
fun AppCafeteria() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = Carta) {
        composable<Carta> {
            PantallaCarta(
                onProductoClick = { id -> navController.navigate(Detalle(id)) },
                onVerPedido = { navController.navigate(Resumen) }
            )
        }
        composable<Detalle> { entrada ->
            val ruta: Detalle = entrada.toRoute()
            PantallaDetalle(
                idProducto = ruta.idProducto,
                onVolver = { navController.navigateUp() }
            )
        }
        // ... Resumen y PedidoEnviado
    }
}
```

```kotlin
// MainActivity
setContent {
    CafeteriaTheme {
        AppCafeteria()
    }
}
```

### Las pantallas no conocen al `NavController`

Fijaos en que `PantallaCarta` **no recibe el `navController`**: recibe dos lambdas, `onProductoClick` y `onVerPedido`. La carta solo avisa de que "el usuario ha tocado el producto 3" o "quiere ver el pedido"; **qué pantalla se abre** lo decide `AppCafeteria`.

Es el mismo *state hoisting* de la sesión 3 aplicado a la navegación: **los eventos suben**. Tiene tres ventajas:

1. **Las previews siguen funcionando**: en una preview no hay `NavController`, pero sí se pueden pasar lambdas vacías.
2. **Toda la navegación está en un solo fichero**: para saber cómo se mueve el usuario por la app, basta con leer `AppCafeteria`.
3. **Las pantallas se pueden reutilizar**: si mañana el detalle se abre también desde el resumen, la pantalla de detalle no cambia.

### Las acciones básicas

| Acción | Qué hace con la pila | Uso típico |
|---|---|---|
| `navController.navigate(Detalle(3))` | **Añade** la pantalla `Detalle(3)` arriba | Ir a otra pantalla |
| `navController.navigateUp()` | **Quita** la pantalla de arriba | La flecha ← de la barra superior |
| `navController.popBackStack()` | **Quita** la pantalla de arriba (devuelve `false` si no había nada que quitar) | Volver desde un botón propio |
| Botón Atrás del sistema | **Quita** la pantalla de arriba; si solo queda una, sale de la app | Lo gestiona `NavHost` solo, sin escribir nada |

`navigateUp()` y `popBackStack()` hacen lo mismo en una app como la nuestra. La diferencia aparece en casos raros (por ejemplo, cuando la app se abre desde un enlace externo): `navigateUp()` nunca saca al usuario de la app, y `popBackStack()` puede dejar la pila vacía. Por convención, la flecha ← de la barra usa `navigateUp()`.

### La flecha de volver en la barra superior

Las pantallas secundarias muestran una flecha en la `TopAppBar`, con el parámetro `navigationIcon`:

```kotlin
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
        }
    )
}
```

El icono `arrow_back` se añade como los anteriores, con *File › New › Vector Asset*. La barra recibe `onVolver` como lambda, igual que las pantallas: no sabe nada de navegación.

## 5.6 Pasar argumentos entre pantallas

Para abrir el detalle del producto 3 se crea la ruta con su argumento:

```kotlin
navController.navigate(Detalle(idProducto = 3))
```

Y en el destino se recupera con `toRoute()`:

```kotlin
composable<Detalle> { entrada ->
    val ruta: Detalle = entrada.toRoute()      // Detalle(idProducto = 3)
    PantallaDetalle(idProducto = ruta.idProducto, ...)
}
```

`entrada` es un `NavBackStackEntry`: la "ficha" de esa pantalla dentro de la pila. Guarda su ruta, sus argumentos y, como veremos en el apartado 5.8, puede tener sus propios `ViewModel`.

### Pasar el id, no el objeto

¿Por qué no pasar directamente el `Producto` entero, `Detalle(producto)`? Porque los argumentos se guardan en el mismo `Bundle` que usa `rememberSaveable` (el diccionario que custodia el sistema, explicado en el apartado 2.9 de la sesión 2), que solo admite datos pequeños y sencillos y tiene un límite de tamaño, y porque deben poder **sobrevivir a que Android elimine la app** (apartado 5.9). La regla es:

<div class="callout-warning">
<strong>Por la ruta solo viaja lo mínimo para identificar los datos</strong>, normalmente un id. La pantalla de destino busca los datos completos a partir de ese id (en nuestro caso, en el <code>ViewModel</code>; en la UD4, en la base de datos). Tipos válidos como argumento: <code>Int</code>, <code>Long</code>, <code>Float</code>, <code>Double</code>, <code>Boolean</code>, <code>String</code>, <code>enum</code> y listas de ellos.
</div>

En la cafetería, el `ViewModel` añade una función para buscar un producto por su id:

```kotlin
fun buscarProducto(id: Int): Producto? = productos.find { it.id == id }
```

Devuelve `Producto?` porque podría no existir ningún producto con ese id. La pantalla de detalle lo tiene en cuenta:

```kotlin
val producto = pedidoViewModel.buscarProducto(idProducto) ?: return
```

Si no lo encuentra, `return` sale del composable sin dibujar nada (en una app real se mostraría un mensaje de error).

## 5.7 Controlar la pila de pantallas

Con `navigate` y Atrás ya funciona casi todo. Pero hay dos situaciones en las que la pila no queda como queremos.

### Problema 1: pantallas repetidas → `launchSingleTop`

Si el usuario toca "Ver pedido" dos veces muy rápido, `navigate(Resumen)` se ejecuta dos veces y la pila queda así:

```
┌──────────┐
│ Resumen  │ ← visible
├──────────┤
│ Resumen  │ ← ¡repetida! Atrás lleva a otro resumen
├──────────┤
│ Carta    │
└──────────┘
```

`launchSingleTop = true` evita añadir una pantalla si **ya está arriba** de la pila:

```kotlin
onVerPedido = {
    navController.navigate(Resumen) { launchSingleTop = true }
}
```

El bloque `{ }` que sigue a `navigate` sirve para dar **opciones** de navegación; `launchSingleTop` es la primera que vemos.

### Problema 2: pantallas a las que no se debe volver → `popUpTo`

Al enviar el pedido se abre la pantalla "Pedido enviado". Si solo hiciéramos `navigate(PedidoEnviado("Ana"))`, la pila quedaría así:

```
┌───────────────┐
│ PedidoEnviado │ ← visible
├───────────────┤
│ Resumen       │ ← Atrás vuelve a un resumen de un pedido ya enviado
├───────────────┤
│ Carta         │
└───────────────┘
```

No tiene sentido volver al resumen de un pedido que ya se ha enviado. `popUpTo` **quita pantallas de la pila antes de añadir la nueva**, hasta llegar a la que se indica:

```kotlin
navController.navigate(PedidoEnviado(nombre)) {
    popUpTo<Carta>()          // quita todo lo que haya ENCIMA de Carta
}
```

```
Antes de navegar     popUpTo<Carta>     + PedidoEnviado
┌───────────┐                           ┌───────────────┐
│ Resumen   │                           │ PedidoEnviado │ ← visible
├───────────┤        ┌───────────┐      ├───────────────┤
│ Carta     │        │ Carta     │      │ Carta         │
└───────────┘        └───────────┘      └───────────────┘
```

Ahora, desde "Pedido enviado", Atrás lleva directamente a la carta.

`popUpTo<Carta> { inclusive = true }` quitaría **también** la carta. Se usa, por ejemplo, después de una pantalla de inicio de sesión: una vez dentro, Atrás no debe volver al login, sino salir de la app.

### Volver a una pantalla concreta

Para volver de golpe a una pantalla que está más abajo en la pila (sin añadir nada nuevo):

```kotlin
navController.popBackStack<Carta>(inclusive = false)
```

### Resumen de la navegación de la cafetería

```
Carta ── toca producto ──────► Detalle(id)    ← back: Carta
  │
  └── "Ver pedido" ──────────► Resumen        ← back: Carta
        (launchSingleTop)        │
                                 └─ "Enviar" ─► PedidoEnviado(nombre)
                                    (popUpTo<Carta>)   ← back: Carta
```

## 5.8 Compartir el `ViewModel` entre pantallas

La carta, el detalle y el resumen necesitan **el mismo pedido**: si en el detalle se añade un café, la carta y el resumen deben verlo. En la sesión 4, `PantallaCarta` obtenía el `ViewModel` con un valor por defecto:

```kotlin
fun PantallaCarta(pedidoViewModel: PedidoViewModel = viewModel(), ...)
```

Parece lógico hacer lo mismo en `PantallaDetalle` y `PantallaResumen`. **Es un error**, y muy habitual.

### La trampa: cada pantalla tendría su propio `ViewModel`

Recordad que `viewModel()` usa el almacén **más cercano al sitio donde se llama**. Fuera del `NavHost`, el más cercano es el de la `Activity` (el único que había en la sesión 4); dentro de un `composable<...>`, es el de esa entrada de la pila, porque **cada entrada de la pila tiene su propio almacén**. Si `viewModel()` se llama dentro de `composable<Detalle> { }`, el `ViewModel` se guarda en el almacén **de esa pantalla**: es un objeto nuevo, con el pedido vacío, y se destruye cuando se sale del detalle.

```
Si cada pantalla llama a viewModel():

  Carta ─────► PedidoViewModel A   (cantidades: café = 2)
  Detalle ───► PedidoViewModel B   (cantidades: vacío)    ← ¡otro objeto!
  Resumen ───► PedidoViewModel C   (cantidades: vacío)    ← ¡y otro!
```

Se nota enseguida: en el detalle del café aparece cantidad 0 aunque en la carta haya 2, y en Logcat sale "PedidoViewModel creado" **cada vez** que se abre una pantalla.

### La solución: crearlo una vez, por encima del `NavHost`

El `ViewModel` se pide **una sola vez** en `AppCafeteria`, fuera del `NavHost`, y se pasa a cada pantalla como parámetro:

```kotlin
@Composable
fun AppCafeteria() {
    val navController = rememberNavController()
    val pedidoViewModel: PedidoViewModel = viewModel()   // UNA vez, fuera del NavHost

    NavHost(navController = navController, startDestination = Carta) {
        composable<Carta> {
            PantallaCarta(pedidoViewModel = pedidoViewModel, ...)
        }
        composable<Detalle> { entrada ->
            PantallaDetalle(pedidoViewModel = pedidoViewModel, ...)
        }
        composable<Resumen> {
            PantallaResumen(pedidoViewModel = pedidoViewModel, ...)
        }
    }
}
```

Fuera del `NavHost`, `viewModel()` usa el almacén de la `Activity`, como en la sesión 4: hay **un único** `PedidoViewModel` para toda la app, que sobrevive a los giros y se comparte entre todas las pantallas.

Para que nadie caiga en la trampa, las pantallas reciben el `ViewModel` **sin valor por defecto**:

```kotlin
@Composable
fun PantallaDetalle(
    idProducto: Int,
    pedidoViewModel: PedidoViewModel,    // sin "= viewModel()": hay que pasarlo
    onVolver: () -> Unit,
    modifier: Modifier = Modifier
)
```

<div class="callout-info">
<strong>¿Y si cada pantalla necesitara un <code>ViewModel</code> propio?</strong> En apps grandes es lo normal: un <code>ViewModel</code> por pantalla, con su propio estado, y el estado compartido en una capa de datos común (un <em>repositorio</em>, que veremos en la UD4). Entonces sí se llama a <code>viewModel()</code> dentro de cada <code>composable&lt;...&gt;</code>, precisamente porque así cada pantalla tiene el suyo y se libera al salir de ella. En nuestra app, con un único pedido compartido, un solo <code>ViewModel</code> es la opción más sencilla.
</div>

## 5.9 Navegación, giros y muerte del proceso

| Situación | ¿Qué pantalla se ve después? | ¿Y el pedido? |
|---|---|---|
| Girar la pantalla en el detalle del café | El mismo detalle del café | Se conserva (`ViewModel`) |
| Pulsar Inicio y volver enseguida | La misma pantalla | Se conserva |
| *No conservar actividades*: pedido, abrir el detalle, Inicio y volver | **El mismo detalle del café** | **Se pierde** (cantidad 0) |

Las dos primeras filas no tienen misterio: `rememberNavController()` guarda la pila como `rememberSaveable`, y el `ViewModel` sobrevive al giro.

La tercera es la interesante, y la comprobaréis en la práctica. La **pila de pantallas, con sus argumentos**, se guarda en el `Bundle`, así que sobrevive a que Android destruya la actividad: la app vuelve al detalle del producto 3 porque `Detalle(idProducto = 3)` estaba guardado. El `PedidoViewModel`, en cambio, se destruye (apartado 4.9) y se crea uno nuevo, vacío. La pantalla puede volver a buscar el producto 3 porque los productos salen de `DatosCarta`, pero las cantidades del pedido ya no están. Es exactamente la razón de la regla del apartado 5.6: por la ruta viaja el id, que sí sobrevive; los datos se recuperan a partir de él.

<div class="callout-info">
<strong>Para saber más.</strong> Google ha publicado también <em>Navigation 3</em>, una librería nueva en la que la pila de pantallas es una lista que gestiona directamente la app. Las ideas de esta sesión (rutas, pila, argumentos mínimos, no pasar el controlador a las pantallas) son las mismas; usamos Navigation Compose porque es la más extendida en proyectos y documentación.
</div>

## Tabla resumen de la sesión

| Necesito... | Uso |
|---|---|
| Añadir la navegación al proyecto | `navigation-compose` + `kotlinx-serialization-json` + plugin `kotlin.plugin.serialization` |
| Definir una pantalla sin datos | `@Serializable data object Carta` |
| Definir una pantalla con argumentos | `@Serializable data class Detalle(val idProducto: Int)` |
| Crear el controlador | `val navController = rememberNavController()` |
| Declarar las pantallas | `NavHost(navController, startDestination = Carta) { composable<Carta> { ... } }` |
| Ir a otra pantalla | `navController.navigate(Detalle(3))` |
| Leer los argumentos en el destino | `val ruta: Detalle = entrada.toRoute()` |
| Volver atrás (flecha ←) | `navController.navigateUp()` |
| No repetir la pantalla de arriba | `navigate(Resumen) { launchSingleTop = true }` |
| Quitar pantallas antes de navegar | `navigate(PedidoEnviado(n)) { popUpTo<Carta>() }` |
| Volver a una pantalla concreta | `navController.popBackStack<Carta>(inclusive = false)` |
| Compartir el `ViewModel` | `viewModel()` una vez, fuera del `NavHost`, y pasarlo por parámetro |
| Flecha de volver en la barra | `TopAppBar(navigationIcon = { IconButton(onClick = onVolver) { ... } })` |

<div class="callout-practica">
<strong>Ejercicio de sesión</strong> (en un proyecto de pruebas, 20 minutos):

1. Crea dos rutas, <code>@Serializable data object Inicio</code> y <code>@Serializable data class Saludo(val nombre: String)</code>, y un <code>NavHost</code> con las dos pantallas. En <code>Inicio</code>, un <code>TextField</code> y un botón que navega a <code>Saludo</code> con el texto escrito; en <code>Saludo</code>, un texto "Hola, ..." y un botón que vuelve con <code>navigateUp()</code>.
2. Pulsa el botón de <code>Inicio</code> tres veces seguidas muy rápido y luego Atrás: ¿cuántas veces tienes que pulsar Atrás para volver? Añade <code>launchSingleTop = true</code> y repite.
3. Crea un <code>ContadorViewModel</code> con un <code>Log.d</code> en su <code>init</code>, y llama a <code>viewModel()</code> <strong>dentro</strong> de cada <code>composable&lt;...&gt;</code>. Cuenta en Logcat cuántas veces se crea al ir y volver. Muévelo fuera del <code>NavHost</code> y compara.
</div>

## Para la próxima sesión

La cafetería ya tiene varias pantallas, pero todo se olvida al cerrar la app: el nombre del cliente hay que volver a escribirlo en cada pedido. En la sesión 6 usaremos **DataStore** para que la app recuerde preferencias del usuario entre ejecuciones, como su nombre o si suele pedir para llevar.
