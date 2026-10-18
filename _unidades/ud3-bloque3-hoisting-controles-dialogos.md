---
title: "State hoisting, controles de entrada, diálogos y menús"
unit: UD3
order: 3
duration: "2h"
---

## Objetivos

- Distinguir entre composables **con estado** (*stateful*) y **sin estado** (*stateless*), y aplicar el patrón **state hoisting** para compartir el estado entre varios componentes.
- Usar los controles de entrada de Compose: campos de texto (`TextField`, `OutlinedTextField`), `Checkbox` y `Switch`.
- Construir la estructura completa de una pantalla con `Scaffold`: barra superior (`TopAppBar`), barra inferior y mensajes emergentes (`Snackbar`).
- Mostrar menús desplegables (`DropdownMenu`) y diálogos de alerta (`AlertDialog`) controlados por el estado.
- Probar en el emulador la interacción con teclado, diálogos y menús.

<div class="callout-info">
Esta sesión cubre de lleno el criterio <strong>b)</strong> del RA2: "clases que modelan ventanas, menús, alertas y controles". En Compose, esas "clases" son funciones <code>@Composable</code>: <code>Scaffold</code> (la ventana), <code>DropdownMenu</code> (los menús), <code>AlertDialog</code> y <code>Snackbar</code> (las alertas) y <code>TextField</code>, <code>Checkbox</code>, <code>Switch</code> o <code>Button</code> (los controles).
</div>

## 3.1 El problema: el estado "encerrado" en la tarjeta

Al final de la sesión 2 cada `ProductoCard` guardaba su propia cantidad:

```kotlin
@Composable
fun ProductoCard(producto: Producto, modifier: Modifier = Modifier) {
    var cantidad by rememberSaveable { mutableIntStateOf(0) }
    // ... botones que modifican "cantidad"
}
```

Funciona, pero `cantidad` es una variable **local** de la función: nadie fuera de la tarjeta puede leerla. `PantallaCarta` no tiene forma de saber cuántos cafés y cuántos croissants se han pedido, así que tampoco puede calcular el total ni vaciar el pedido.

Un composable que guarda su propio estado con `remember` se llama **stateful** (con estado). Es cómodo, pero ese estado no se puede compartir ni controlar desde fuera.

## 3.2 State hoisting: elevar el estado

La solución es **sacar el estado de la tarjeta y subirlo** a quien lo necesita (la pantalla). La tarjeta pasa a ser **stateless** (sin estado): no guarda nada, solo recibe dos cosas por parámetro:

- **el valor** que tiene que mostrar (`cantidad: Int`);
- **una función** para avisar cuando el usuario quiere cambiarlo (`onCantidadChange: (Int) -> Unit`).

```kotlin
@Composable
fun ProductoCard(
    producto: Producto,
    cantidad: Int,
    onCantidadChange: (Int) -> Unit,
    modifier: Modifier = Modifier
) {
    // ...
    IconButton(onClick = { onCantidadChange(cantidad - 1) }, enabled = cantidad > 0) { /* - */ }
    Text(text = cantidad.toString())
    IconButton(onClick = { onCantidadChange(cantidad + 1) }) { /* + */ }
}
```

La tarjeta ya no decide nada: cuando el usuario pulsa "+", **pide** que la cantidad pase a ser `cantidad + 1`. Es la pantalla quien guarda el valor y decide:

```kotlin
@Composable
fun PantallaCarta(productos: List<Producto>, modifier: Modifier = Modifier) {
    // Cantidad de cada producto, identificado por el id de su nombre
    var cantidades by rememberSaveable { mutableStateOf(mapOf<Int, Int>()) }

    Column(modifier = modifier) {
        productos.forEach { producto ->
            ProductoCard(
                producto = producto,
                cantidad = cantidades[producto.nombre] ?: 0,
                onCantidadChange = { nueva -> cantidades = cantidades + (producto.nombre to nueva) }
            )
        }
    }
}
```

Ahora el estado vive en `PantallaCarta` y calcular el total es inmediato con lo que ya sabéis de la UD2 (`sumOf` y el operador Elvis):

```kotlin
val total = productos.sumOf { (cantidades[it.nombre] ?: 0) * it.precio }
```

### Flujo unidireccional de datos

El patrón se resume en una frase: **el estado baja, los eventos suben**.

```
                  PantallaCarta  (guarda "cantidades")
                    │                         ▲
   el estado baja:  │ cantidad = 2            │  los eventos suben:
                    ▼                         │  onCantidadChange(3)
                  ProductoCard  (solo muestra y avisa)
```

| | Stateful (sesión 2) | Stateless (state hoisting) |
|---|---|---|
| ¿Dónde vive el estado? | Dentro del composable (`remember`) | En quien lo llama |
| ¿Se puede leer desde fuera? | No | Sí |
| ¿Se puede reutilizar con otros datos o en una preview? | Difícil | Fácil: basta con pasarle valores |
| Parámetros típicos | Solo los datos | `valor` + `onValorChange` |

<div class="callout-info">
<strong>¿Hasta dónde se sube el estado?</strong> Hasta el composable más bajo que lo necesite. La cantidad la necesitan la tarjeta (para mostrarla) y la pantalla (para el total), así que sube a <code>PantallaCarta</code>. Si un dato solo lo usa un componente (por ejemplo, si un menú está abierto o cerrado), puede quedarse dentro de él. En la sesión 4 daremos un paso más: sacar el estado de la pantalla a un <code>ViewModel</code>.
</div>

<div class="callout-warning">
Los <code>Map</code> y las <code>List</code> de Kotlin que se crean con <code>mapOf</code> o <code>listOf</code> son <strong>inmutables</strong>: no se modifican, se reemplazan por uno nuevo. Por eso escribimos <code>cantidades = cantidades + (clave to valor)</code>: el operador <code>+</code> crea un mapa nuevo, la asignación cambia el estado y Compose recompone. Si usáramos un <code>MutableMap</code> y lo modificáramos por dentro (<code>mapa[clave] = valor</code>), el estado <strong>no</strong> cambiaría de objeto y Compose no se enteraría.
</div>

## 3.3 Campos de texto: `TextField` y `OutlinedTextField`

Un campo de texto es el ejemplo más claro de state hoisting: Compose **no guarda** lo que se escribe. Le pasáis el texto actual (`value`) y él os avisa de cada cambio (`onValueChange`); si no actualizáis vuestro estado, el campo no cambia aunque el usuario teclee.

```kotlin
var nombre by rememberSaveable { mutableStateOf("") }

OutlinedTextField(
    value = nombre,
    onValueChange = { nombre = it },
    label = { Text(stringResource(R.string.nombre_pedido)) },
    singleLine = true,
    modifier = Modifier.fillMaxWidth()
)
```

`TextField` (relleno) y `OutlinedTextField` (con borde) funcionan igual; solo cambia el aspecto. Parámetros útiles:

| Parámetro | Para qué |
|---|---|
| `label` | Etiqueta que flota sobre el campo al escribir. |
| `placeholder` | Texto de ejemplo cuando está vacío. |
| `singleLine = true` | Una sola línea (Intro no añade saltos). |
| `isError` + `supportingText` | Marca el campo en rojo y muestra un mensaje debajo. |
| `keyboardOptions` | Tipo de teclado y comportamiento: `KeyboardOptions(keyboardType = KeyboardType.Number)`, `capitalization = KeyboardCapitalization.Words`, `imeAction = ImeAction.Done`… |
| `enabled` / `readOnly` | Desactivar o impedir la edición. |

Validar es solo cuestión de estado: el error es una condición calculada a partir de lo que hay escrito.

```kotlin
val nombreVacio = nombre.isBlank()

OutlinedTextField(
    value = nombre,
    onValueChange = { nombre = it },
    label = { Text(stringResource(R.string.nombre_pedido)) },
    isError = nombreVacio,
    supportingText = { if (nombreVacio) Text(stringResource(R.string.error_nombre)) }
)
```

<div class="callout-warning">
<strong>El teclado puede tapar el campo.</strong> Con <code>enableEdgeToEdge()</code> (sesión 1), la app ocupa toda la pantalla y el teclado se dibuja <em>encima</em> del contenido. Si el campo está en la parte de abajo, al tocarlo queda oculto. La solución es añadir <code>Modifier.imePadding()</code> al contenedor desplazable: deja un margen inferior del tamaño del teclado mientras está abierto. (<em>IME</em>, <em>Input Method Editor</em>, es el nombre técnico del teclado en pantalla).
</div>

## 3.4 `Checkbox` y `Switch`

Mismo patrón, con un `Boolean`: `checked` es el valor y `onCheckedChange` avisa del cambio.

```kotlin
var paraLlevar by rememberSaveable { mutableStateOf(false) }
var avisar by rememberSaveable { mutableStateOf(true) }

Row(verticalAlignment = Alignment.CenterVertically) {
    Text(stringResource(R.string.para_llevar), modifier = Modifier.weight(1f))
    Switch(checked = paraLlevar, onCheckedChange = { paraLlevar = it })
}
Row(verticalAlignment = Alignment.CenterVertically) {
    Checkbox(checked = avisar, onCheckedChange = { avisar = it })
    Text(stringResource(R.string.avisar_listo))
}
```

¿Cuándo usar cada uno? Por convención de Material Design, el **`Switch`** activa o desactiva algo con efecto inmediato (un ajuste), y el **`Checkbox`** marca opciones que se confirman después (como en un formulario). Para elegir una sola opción entre varias existe `RadioButton`, con `selected` y `onClick`.

## 3.5 La ventana completa: `Scaffold`

En las sesiones anteriores usábamos `Scaffold` solo por su `innerPadding`. En realidad es la **estructura de una pantalla Material**, con huecos para cada elemento:

```
┌────────────────────────────────────┐
│ topBar      (TopAppBar)       ⋮    │
├────────────────────────────────────┤
│                                    │
│ content     (el contenido,         │
│              con innerPadding)     │
│                                    │
│ snackbarHost (mensajes temporales) │
├────────────────────────────────────┤
│ bottomBar   (BottomAppBar)         │
└────────────────────────────────────┘
       floatingActionButton (FAB) ↗
```

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PantallaCarta(productos: List<Producto>, modifier: Modifier = Modifier) {
    Scaffold(
        modifier = modifier,
        topBar = {
            TopAppBar(
                title = { Text(stringResource(R.string.app_name)) },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer
                )
            )
        },
        bottomBar = {
            BottomAppBar {
                Text(text = stringResource(R.string.total_pedido, total), modifier = Modifier.weight(1f))
                Button(onClick = { /* ... */ }) { Text(stringResource(R.string.confirmar)) }
            }
        }
    ) { innerPadding ->
        Column(modifier = Modifier.padding(innerPadding)) {
            // carta
        }
    }
}
```

- Cada hueco recibe una **lambda** con su contenido (*trailing lambdas* y parámetros con nombre, como en la UD2).
- El `innerPadding` ahora incluye también la altura de las barras: sin él, el contenido quedaría debajo de ellas.
- Como la pantalla tiene sus propias barras, el `Scaffold` pasa a vivir **dentro de `PantallaCarta`**, y `MainActivity` solo llama a `PantallaCarta(...)` dentro del tema.

<div class="callout-warning">
Si Android Studio subraya <code>TopAppBar</code> con un aviso de API <strong>experimental</strong>, añadid <code>@OptIn(ExperimentalMaterial3Api::class)</code> encima de la función que lo usa (<code>Alt+Enter</code> lo propone). No es un error: indica que Google aún puede cambiar detalles de esa API en futuras versiones.
</div>

## 3.6 Menús: `DropdownMenu`

Un menú desplegable también se controla con estado: un `Boolean` que dice si está abierto. Se coloca en un `Box` junto al botón que lo abre, para que aparezca anclado a él. Lo habitual es el botón de tres puntos (icono `more_vert`) en las acciones de la barra superior:

```kotlin
var menuAbierto by remember { mutableStateOf(false) }

TopAppBar(
    title = { Text(stringResource(R.string.app_name)) },
    actions = {
        Box {
            IconButton(onClick = { menuAbierto = true }) {
                Icon(
                    painter = painterResource(R.drawable.baseline_more_vert_24),
                    contentDescription = stringResource(R.string.mas_opciones)
                )
            }
            DropdownMenu(expanded = menuAbierto, onDismissRequest = { menuAbierto = false }) {
                DropdownMenuItem(
                    text = { Text(stringResource(R.string.vaciar_pedido)) },
                    onClick = {
                        menuAbierto = false
                        // acción
                    }
                )
            }
        }
    }
)
```

- `expanded`: si se muestra o no (el estado).
- `onDismissRequest`: el usuario ha tocado fuera del menú o ha pulsado Atrás; lo normal es cerrarlo.
- Cada `DropdownMenuItem` debe **cerrar el menú** en su `onClick`, además de hacer su acción. Si se os olvida, el menú se queda abierto.

## 3.7 Alertas: `AlertDialog`

En Compose, un diálogo no se "abre" con una llamada: se muestra **mientras el estado lo diga**. Es una función `@Composable` más, dentro de un `if`:

```kotlin
var mostrarConfirmacion by rememberSaveable { mutableStateOf(false) }

if (mostrarConfirmacion) {
    AlertDialog(
        onDismissRequest = { mostrarConfirmacion = false },
        title = { Text(stringResource(R.string.titulo_confirmacion)) },
        text = { Text(resumen) },
        confirmButton = {
            TextButton(onClick = {
                mostrarConfirmacion = false
                // enviar el pedido
            }) { Text(stringResource(R.string.enviar)) }
        },
        dismissButton = {
            TextButton(onClick = { mostrarConfirmacion = false }) {
                Text(stringResource(R.string.cancelar))
            }
        }
    )
}
```

| Parámetro | Qué es |
|---|---|
| `onDismissRequest` | El usuario toca fuera del diálogo o pulsa Atrás. |
| `title`, `text` | Título y mensaje (lambdas con composables). |
| `confirmButton` | Botón de la acción principal. |
| `dismissButton` | Botón de cancelar (opcional). |
| `icon` | Icono sobre el título (opcional). |

Como la visibilidad es estado, si lo guardáis con `rememberSaveable` el diálogo **sigue abierto tras girar la pantalla**, algo que con las APIs clásicas de Android había que programar a mano.

## 3.8 Mensajes breves: `Snackbar`

Un **Snackbar** es un aviso que aparece unos segundos en la parte inferior ("Pedido enviado") y puede llevar una acción ("Deshacer"). Necesita tres piezas:

```kotlin
val snackbarHostState = remember { SnackbarHostState() }   // 1. quién gestiona los mensajes
val scope = rememberCoroutineScope()                        // 2. un ámbito para lanzar coroutines

Scaffold(
    snackbarHost = { SnackbarHost(snackbarHostState) }      // 3. dónde se dibujan
) { /* ... */ }

// Para mostrar un mensaje, por ejemplo en un onClick:
scope.launch {
    snackbarHostState.showSnackbar(mensaje)
}
```

¿Por qué una coroutine? Porque `showSnackbar` es una función **`suspend`**: se queda esperando a que el mensaje desaparezca (o a que el usuario pulse su acción) sin bloquear la interfaz. Es justo lo que visteis en la introducción a las coroutines de la UD2: `launch` arranca la tarea y `suspend` permite esperar sin congelar la app. `rememberCoroutineScope()` da un ámbito ligado a la pantalla: si la pantalla desaparece, sus coroutines se cancelan solas.

Con acción, `showSnackbar` devuelve qué ha pasado:

```kotlin
scope.launch {
    val resultado = snackbarHostState.showSnackbar(
        message = mensajeVaciado,
        actionLabel = textoDeshacer,
        duration = SnackbarDuration.Short
    )
    if (resultado == SnackbarResult.ActionPerformed) {
        cantidades = copiaAnterior   // deshacer
    }
}
```

<div class="callout-warning">
<code>stringResource</code> solo funciona en el cuerpo de un composable, <strong>no dentro de un <code>onClick</code></strong> ni de un <code>launch</code> (son lambdas normales, no composables). Obtened los textos antes, en el cuerpo de la función, y usad la variable dentro de la lambda:
<ul>
<li><code>val mensajeVaciado = stringResource(R.string.pedido_vaciado)</code> en el cuerpo del composable;</li>
<li><code>scope.launch { snackbarHostState.showSnackbar(mensajeVaciado) }</code> dentro del <code>onClick</code>.</li>
</ul>
</div>

## 3.9 Equivalencias con el Android clásico (criterio b)

Si consultáis documentación o código antiguo, veréis estas clases. Su equivalente en Compose:

| Elemento | Android clásico (Views + XML) | Jetpack Compose |
|---|---|---|
| Ventana / estructura de pantalla | `Activity` + layout XML, `Toolbar` | `Scaffold` + `TopAppBar` |
| Menú de opciones | `onCreateOptionsMenu`, `menu.xml` | `actions` de `TopAppBar` + `DropdownMenu` |
| Alerta | `AlertDialog.Builder(...).show()` | `AlertDialog` controlado por estado |
| Aviso breve | `Toast`, `Snackbar.make(...)` | `SnackbarHost` + `showSnackbar` |
| Campo de texto | `EditText` | `TextField` / `OutlinedTextField` |
| Casillas | `CheckBox`, `Switch` | `Checkbox`, `Switch` |

La diferencia de fondo es siempre la misma: en el modelo clásico se **ordena** a cada elemento que cambie (`dialog.show()`, `editText.setText(...)`); en Compose se **cambia el estado** y la interfaz se redibuja de acuerdo con él.

## 3.10 Probar la interacción en el emulador (criterio g)

Con controles de entrada y diálogos aparecen pruebas nuevas:

| Qué probar | Qué debe pasar |
|---|---|
| Escribir en un campo y girar la pantalla | El texto se conserva (si su estado usa `rememberSaveable`). |
| Tocar un campo situado abajo | El teclado no lo tapa (`imePadding`). |
| Abrir un diálogo y girar | El diálogo sigue abierto. |
| Abrir un diálogo con Logcat filtrado por `tag:CicloVida` | No aparece `onPause`: un diálogo de la propia app no pausa la actividad. |
| Pulsar Atrás con el menú o el diálogo abierto | Se cierra el menú o el diálogo (`onDismissRequest`), no la app. |
| Acción no permitida (confirmar sin nombre) | Se muestra el error en el campo y no se continúa. |
| TalkBack sobre el botón de tres puntos | Lee su `contentDescription` ("Más opciones"). |

<div class="callout-info">
Para escribir en el emulador podéis usar el teclado del ordenador directamente. Si no aparece el teclado en pantalla al tocar un campo, es porque el emulador detecta el teclado físico; se puede cambiar en <em>Ajustes → Sistema → Teclado → Teclado físico → Mostrar teclado en pantalla</em>.
</div>

## Tabla resumen de la sesión

| Necesito... | Uso |
|---|---|
| Que varios componentes compartan un dato | State hoisting: el estado sube al padre; el hijo recibe `valor` + `onValorChange` |
| Un campo de texto | `OutlinedTextField(value, onValueChange, label, singleLine, isError, supportingText, keyboardOptions)` |
| Que el teclado no tape el contenido | `Modifier.imePadding()` en el contenedor desplazable |
| Una opción sí/no | `Switch(checked, onCheckedChange)` o `Checkbox(checked, onCheckedChange)` |
| Barras superior e inferior | `Scaffold(topBar = { TopAppBar(...) }, bottomBar = { BottomAppBar { ... } })` |
| Un menú de opciones | `Box { IconButton(...); DropdownMenu(expanded, onDismissRequest) { DropdownMenuItem(...) } }` |
| Pedir confirmación | `if (mostrar) AlertDialog(onDismissRequest, title, text, confirmButton, dismissButton)` |
| Un aviso breve, con o sin "Deshacer" | `SnackbarHostState` + `rememberCoroutineScope()` + `scope.launch { showSnackbar(...) }` |

<div class="callout-practica">
<strong>Ejercicio de sesión</strong> (en un proyecto de pruebas, 15 minutos):

1. Crea un composable stateless <code>CampoEdad(edad: String, onEdadChange: (String) -> Unit)</code> con un <code>OutlinedTextField</code> de teclado numérico.
2. Desde la pantalla, guarda la edad con <code>rememberSaveable</code> y muestra debajo "Eres mayor de edad" o "Eres menor de edad" (pista: <code>toIntOrNull()</code>).
3. Marca el campo con error si lo escrito no es un número.
4. Gira el emulador y comprueba que la edad escrita se conserva.
</div>

## Para la próxima sesión

Nuestra `PantallaCarta` ya guarda mucho estado: cantidades, nombre, opciones del pedido, diálogos… y la carta sigue siendo una `Column` que crea **todas** las tarjetas aunque no se vean. En la sesión 4 veremos dos mejoras que se usan en cualquier app real: listas eficientes con **`LazyColumn`** y un **`ViewModel`** que saque el estado y la lógica del pedido fuera de la interfaz.
