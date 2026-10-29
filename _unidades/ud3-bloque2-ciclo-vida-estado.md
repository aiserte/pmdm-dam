---
title: "Ciclo de vida de la Activity, Logcat y estado"
unit: UD3
order: 2
duration: "2h"
---

## Objetivos

- Conocer los estados por los que pasa una `Activity` y los métodos del ciclo de vida que Android ejecuta en cada cambio (`onCreate`, `onStart`, `onResume`, `onPause`, `onStop`, `onDestroy`).
- Usar **Logcat** para registrar mensajes desde la app, filtrarlos y leer el error de una app que se cierra.
- Entender qué es el **estado** en Compose y cómo provoca la **recomposición** de la interfaz.
- Guardar estado con `remember` y `mutableStateOf`, y responder a eventos (`onClick`).
- Explicar por qué el estado se pierde al girar la pantalla y conservarlo con `rememberSaveable`.

<div class="callout-info">
En la UD1 visteis, a nivel teórico, el modelo de estados de una app (activa, en pausa, destruida). Hoy lo vais a <strong>ver ocurrir</strong> en vuestra propia app, paso a paso, en Logcat. Y lo vais a necesitar enseguida: es la explicación de por qué la app "olvida" datos al girar la pantalla.
</div>

## 2.1 Por qué existe un ciclo de vida

En un ordenador, un programa abierto sigue ahí hasta que lo cerráis. En un móvil no: la pantalla es pequeña, la memoria y la batería son limitadas, y el usuario salta de una app a otra continuamente (le llega un mensaje, recibe una llamada, gira el móvil, vuelve a la pantalla de inicio…).

Por eso **es Android, no vuestra app, quien decide cuándo una `Activity` se muestra, se queda en segundo plano o se destruye**. Cada vez que cambia su situación, Android avisa a la `Activity` llamando a un método concreto: son los **métodos del ciclo de vida** (*lifecycle callbacks*). Vosotros no los llamáis nunca; los sobrescribís para enteraros de lo que pasa y reaccionar.

## 2.2 Estados y métodos del ciclo de vida

```
          onCreate()
              │
          onStart()  ◄────────────── onRestart()
              │                           ▲
          onResume()                      │
              │                           │
   [ ACTIVA: el usuario interactúa ]      │
              │                           │
          onPause()                       │
              │                           │
          onStop()  ── el usuario vuelve ─┘
              │
          onDestroy()
```

| Método | Cuándo lo llama Android | Estado en que queda la `Activity` |
|---|---|---|
| `onCreate` | Al crear la actividad (abrir la app, o recrearla tras un giro). Aquí se llama a `setContent`. | Creada |
| `onStart` | Cuando pasa a ser **visible**. | Iniciada (visible) |
| `onResume` | Cuando pasa a **primer plano** y el usuario puede interactuar. | Reanudada → **activa** |
| `onPause` | Cuando pierde el primer plano (sale un diálogo del sistema, el usuario pulsa Inicio…). Es lo primero que se ejecuta al salir. | En pausa |
| `onStop` | Cuando **deja de verse** por completo. | Detenida |
| `onRestart` | Antes de `onStart`, cuando una actividad detenida vuelve a mostrarse. | — |
| `onDestroy` | Antes de destruir la actividad (el usuario la cierra, se gira la pantalla, el sistema necesita memoria…). | Destruida |

Los métodos van **por parejas**, y conviene recordarlos así: `onCreate`/`onDestroy` (existe o no existe), `onStart`/`onStop` (visible o no visible), `onResume`/`onPause` (en primer plano o no).

<div class="callout-warning">
<strong><code>onDestroy</code> no está garantizado.</strong> Si el sistema necesita memoria con la app en segundo plano, o el usuario la cierra deslizándola en la pantalla de apps recientes, Android puede matar el proceso sin llamar a <code>onDestroy</code>. Nunca dejéis para <code>onDestroy</code> algo importante, como guardar datos: el último método con el que se puede contar es <code>onStop</code>.
</div>

## 2.3 Observar el ciclo de vida: sobrescribir los métodos

Para ver el ciclo de vida en acción, sobrescribimos los seis métodos en `MainActivity` y en cada uno escribimos un mensaje de registro con `Log.d`:

```kotlin
import android.util.Log

private const val TAG = "CicloVida"

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d(TAG, "onCreate (savedInstanceState = $savedInstanceState)")
        enableEdgeToEdge()
        setContent {
            // ... igual que en la sesión 1
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

- La llamada a `super.onXxx()` es **obligatoria**: la clase padre (`ComponentActivity`) también tiene trabajo que hacer en cada paso. Si la quitáis, la app se cierra con un error.
- `TAG` es una etiqueta que identifica vuestros mensajes. Por convención es una constante `private const val` declarada fuera de la clase, al principio del fichero.
- Fijaos en `savedInstanceState`: la primera vez que se abre la app vale `null`; cuando la actividad se **recrea** (por ejemplo, tras girar la pantalla) trae datos guardados. Volveremos a él en el apartado 2.9.

<div class="callout-info">
Truco: en Android Studio, escribid <code>onSt</code> dentro de la clase y el autocompletado ofrece <code>override fun onStart()</code> con el <code>super</code> ya puesto. También podéis usar <em>Code → Override Methods…</em> (<code>Ctrl+O</code>).
</div>

## 2.4 Logcat: la consola de Android

Una app Android no tiene consola donde ver un `println`. Los mensajes de todas las apps y del propio sistema van a un registro común: **Logcat**. En Android Studio se abre desde la pestaña *Logcat* de la parte inferior (o *View → Tool Windows → Logcat*) con la app ejecutándose en el emulador.

### Escribir en Logcat

La clase `android.util.Log` tiene un método por cada **nivel de importancia**:

| Método | Nivel | Para qué |
|---|---|---|
| `Log.v(TAG, msg)` | *Verbose* | Detalle muy fino; casi nunca se usa. |
| `Log.d(TAG, msg)` | *Debug* | Mensajes de depuración mientras desarrolláis. **El más habitual.** |
| `Log.i(TAG, msg)` | *Info* | Información general del funcionamiento normal. |
| `Log.w(TAG, msg)` | *Warning* | Algo raro que no impide seguir. |
| `Log.e(TAG, msg, excepción)` | *Error* | Un error. Admite la excepción como tercer parámetro para registrar su traza. |

<div class="callout-warning">
<code>println()</code> también funciona (sus mensajes salen en Logcat con la etiqueta <code>System.out</code>), pero no tiene nivel ni etiqueta propia y se pierde entre miles de líneas. En Android se usa siempre <code>Log</code>.
</div>

### Filtrar: sin filtro no se ve nada

Logcat muestra **todo** lo que ocurre en el dispositivo: cientos de líneas por segundo del sistema y de otras apps. Hay que filtrar siempre, escribiendo en el cuadro de búsqueda de Logcat:

| Filtro | Muestra |
|---|---|
| `package:mine` | Solo los mensajes de vuestra app (es el filtro por defecto). |
| `tag:CicloVida` | Solo los mensajes con esa etiqueta. |
| `level:error` | Solo errores (y niveles superiores). |
| `package:mine tag:CicloVida` | Combinación: se pueden juntar varios filtros. |
| `-tag:CicloVida` | El guion **excluye**: todo menos esa etiqueta. |

Cada línea de Logcat tiene este aspecto (fecha y hora, proceso, etiqueta, paquete, nivel y mensaje):

```
2026-10-05 10:15:02.123  4567-4567  CicloVida  com.iesejemplo.cafeteria  D  onCreate (savedInstanceState = null)
2026-10-05 10:15:02.180  4567-4567  CicloVida  com.iesejemplo.cafeteria  D  onStart
2026-10-05 10:15:02.185  4567-4567  CicloVida  com.iesejemplo.cafeteria  D  onResume
```

### Leer el error de una app que se cierra

Cuando una app se cierra de golpe ("La app se ha detenido"), la explicación está en Logcat. Filtrad por `package:mine level:error` y buscad una línea **`FATAL EXCEPTION: main`**. Debajo aparece la excepción (por ejemplo, `java.lang.NullPointerException`) y la **traza**: la lista de llamadas que llevaron al error. Las líneas que pertenecen a vuestro código aparecen como **enlaces azules** (`MainActivity.kt:42`): al pulsarlas, Android Studio os lleva a la línea exacta.

<div class="callout-practica">
<strong>Ejercicio de sesión (Logcat, 5 minutos):</strong> provocad un error a propósito. Borrad la línea <code>super.onStart()</code> en vuestro <code>onStart</code>, ejecutad, y buscad en Logcat el <code>FATAL EXCEPTION</code>. ¿Qué excepción aparece y qué dice su mensaje? Volved a poner la línea.
</div>

## 2.5 Experimentos con el ciclo de vida

Con el filtro `tag:CicloVida` activo, cada acción del usuario produce una secuencia concreta de métodos:

| Acción en el emulador | Secuencia en Logcat |
|---|---|
| Abrir la app | `onCreate` → `onStart` → `onResume` |
| Pulsar **Inicio** | `onPause` → `onStop` |
| Volver a la app (desde recientes o su icono) | `onStart` → `onResume` (antes se ejecuta `onRestart`) |
| **Girar** la pantalla | `onPause` → `onStop` → `onDestroy` → `onCreate` → `onStart` → `onResume` |
| Pulsar **Atrás** en la pantalla principal | Android 12 o superior: `onPause` → `onStop` (igual que Inicio). Android 11 o inferior: además `onDestroy`. |
| Cerrar la app deslizándola en recientes | Puede que no aparezca nada más: el proceso se elimina sin avisar. |

<div class="callout-warning">
Lo más llamativo de la tabla es el <strong>giro de pantalla</strong>: Android <strong>destruye la actividad y crea otra nueva</strong>. Lo hace porque la nueva orientación puede necesitar otros recursos (recordad los calificadores de la sesión 1, como <code>drawable-land</code>). Lo mismo pasa con otros <em>cambios de configuración</em>: cambiar el idioma del sistema, activar el modo oscuro… Y si la actividad se destruye, con ella se destruye su interfaz y todo lo que guardaba.
</div>

<div class="callout-info">
Sobre el botón Atrás: desde Android 12, pulsar Atrás en la primera pantalla de una app ya no la destruye; Android la envía a segundo plano, igual que Inicio, para que volver a abrirla sea más rápido. Si en vuestro emulador veis <code>onDestroy</code> al pulsar Atrás, comprobad su versión de Android.
</div>

### ¿Para qué se usan estos métodos en una app real?

En una app Compose, casi todo el código de interfaz vive en funciones `@Composable` y no en la `Activity`. Pero el ciclo de vida sigue importando para lo que **consume recursos**:

- `onResume` / `onPause`: activar y desactivar lo que solo tiene sentido con la app en primer plano: la cámara, los sensores, las actualizaciones de ubicación (lo veremos en la UD5).
- `onStop`: último momento fiable para guardar lo que el usuario no debe perder.

## 2.6 Estado en Compose: la interfaz como función de los datos

Hasta ahora nuestra carta era **estática**: siempre mostraba lo mismo. Una interfaz útil cambia: un contador, un texto que se escribe, un elemento seleccionado. Ese dato que puede cambiar con el tiempo y que la interfaz muestra es el **estado**.

La idea central de Compose es:

```
interfaz = f(estado)
```

No modificáis la interfaz directamente (no hay un `texto.setText(...)` como en otras tecnologías). Lo que modificáis es el **estado**, y Compose **vuelve a ejecutar** las funciones `@Composable` que lo leen para que la pantalla refleje el nuevo valor. Ese volver a ejecutar se llama **recomposición**.

### Primer intento: una variable normal (no funciona)

```kotlin
@Composable
fun Contador() {
    var veces = 0
    Button(onClick = { veces++ }) {
        Text("Pulsado $veces veces")
    }
}
```

Al pulsar el botón, `veces` sí se incrementa, pero la pantalla sigue mostrando 0. Compose **no sabe** que esa variable ha cambiado, así que no recompone. Y aunque recompusiera por otro motivo, la primera línea volvería a poner `veces = 0`.

### Segundo intento: `mutableStateOf` (casi)

```kotlin
var veces by mutableStateOf(0)   // Android Studio lo marca como error
```

`mutableStateOf` crea un **estado observable**: un contenedor de un valor que **avisa a Compose cuando cambia**. Ahora, al pulsar, Compose recompone… y al recomponer vuelve a ejecutar esta línea, que crea un estado **nuevo** con valor 0. Android Studio lo detecta y lo marca: *Creating a state object during composition without using remember*.

### Solución: `remember` + `mutableStateOf`

```kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue

@Composable
fun Contador(modifier: Modifier = Modifier) {
    var veces by remember { mutableStateOf(0) }
    Button(onClick = { veces++ }, modifier = modifier) {
        Text("Pulsado $veces veces")
    }
}
```

Las tres piezas trabajan juntas:

| Pieza | Qué aporta |
|---|---|
| `mutableStateOf(0)` | Un estado observable: cuando cambia su valor, Compose recompone a quien lo lee. |
| `remember { ... }` | Ejecuta el bloque **solo la primera vez** y, en las recomposiciones siguientes, devuelve el mismo objeto guardado. |
| `by` | Delegación de propiedades: permite escribir `veces` y `veces++` en lugar de `veces.value` y `veces.value++`. Necesita los imports de `getValue` y `setValue`. |

Sin `by`, el mismo código se escribe así (es equivalente):

```kotlin
val veces = remember { mutableStateOf(0) }
Button(onClick = { veces.value++ }) {
    Text("Pulsado ${veces.value} veces")
}
```

<div class="callout-warning">
Si Android Studio subraya <code>by</code> en rojo con un error sobre <code>getValue</code> o <code>setValue</code>, faltan los imports <code>androidx.compose.runtime.getValue</code> y <code>androidx.compose.runtime.setValue</code>. <code>Alt+Enter</code> sobre el error los añade. Android Studio también puede sugerir cambiar <code>mutableStateOf(0)</code> por <code>mutableIntStateOf(0)</code>: es una versión optimizada para enteros que funciona igual; podéis aceptar la sugerencia.
</div>

## 2.7 Eventos: el ciclo evento → estado → recomposición

Los componentes interactivos de Compose reciben **lambdas** para avisar de lo que hace el usuario: `onClick` en `Button`, `IconButton` o `Modifier.clickable`, `onValueChange` en un campo de texto (sesión 3)… Dentro de esas lambdas se **cambia el estado**, y Compose se encarga del resto:

```
  el usuario pulsa  ──►  onClick { veces++ }  ──►  el estado cambia
          ▲                                               │
          │                                               ▼
   nueva interfaz  ◄──  Compose recompone las funciones que leen "veces"
```

Un ejemplo más cercano a nuestra app: un selector de cantidad con botones de quitar y añadir.

```kotlin
@Composable
fun SelectorCantidad(modifier: Modifier = Modifier) {
    var cantidad by remember { mutableStateOf(0) }
    Row(modifier = modifier, verticalAlignment = Alignment.CenterVertically) {
        IconButton(onClick = { cantidad-- }, enabled = cantidad > 0) {
            Icon(painterResource(R.drawable.baseline_remove_24), contentDescription = stringResource(R.string.quitar))
        }
        Text(text = "$cantidad", style = MaterialTheme.typography.titleMedium)
        IconButton(onClick = { cantidad++ }) {
            Icon(painterResource(R.drawable.baseline_add_24), contentDescription = stringResource(R.string.anadir))
        }
    }
}
```

`enabled = cantidad > 0` también depende del estado: cuando la cantidad llega a 0, la recomposición desactiva el botón de quitar automáticamente, sin ningún `if` que lo haga a mano.

### Ver la recomposición en Logcat

Podemos comprobar cuándo se recompone una función poniendo un `Log.d` dentro de ella:

```kotlin
@Composable
fun SelectorCantidad(modifier: Modifier = Modifier) {
    var cantidad by remember { mutableStateOf(0) }
    Log.d("Recomposicion", "SelectorCantidad: cantidad = $cantidad")
    // ...
}
```

Si hay varios selectores en pantalla y pulsáis "+" en uno, en Logcat solo aparece el mensaje de **ese** selector: Compose solo vuelve a ejecutar lo que lee el estado que ha cambiado.

<div class="callout-warning">
Un <code>Log.d</code> dentro de un composable es un <strong>efecto secundario</strong>, justo lo que la sesión 1 decía que había que evitar. Aquí lo usamos <strong>solo para aprender</strong> cómo funciona la recomposición; quitadlo cuando lo hayáis visto. Además, ese mensaje puede aparecer más veces de las que esperáis: Compose puede recomponer cuando lo necesita, y vuestro código nunca debe depender de cuántas veces se ejecuta una función <code>@Composable</code>.
</div>

## 2.8 El problema: el estado se pierde al girar

Juntemos los dos bloques de la sesión. Con el selector funcionando, poned una cantidad de 3 y girad el emulador: **la cantidad vuelve a 0**.

Logcat nos da la explicación:

```
CicloVida   D  onPause
CicloVida   D  onStop
CicloVida   D  onDestroy
CicloVida   D  onCreate (savedInstanceState = Bundle[...])
CicloVida   D  onStart
CicloVida   D  onResume
```

`remember` guarda el valor **mientras la interfaz existe**. Pero al girar, la actividad se destruye y, con ella, toda su interfaz y todo lo que había en `remember`. La nueva actividad ejecuta `setContent` desde cero, y `remember` vuelve a ejecutar su bloque: `mutableStateOf(0)`.

## 2.9 La solución: `rememberSaveable`

```kotlin
import androidx.compose.runtime.saveable.rememberSaveable

var cantidad by rememberSaveable { mutableStateOf(0) }
```

`rememberSaveable` funciona igual que `remember`, pero además **guarda el valor** antes de que la actividad se destruya y lo **restaura** en la actividad nueva. ¿Dónde lo guarda? En el `Bundle` que Android pasa a `onCreate(savedInstanceState: Bundle?)`: por eso en Logcat ese parámetro valía `null` al abrir la app y trae datos al girar.

<div class="callout-info">
<strong>¿Qué es el <code>Bundle</code>?</strong> Es un pequeño <strong>diccionario clave-valor</strong> (parecido a un <code>Map&lt;String, …&gt;</code>) que <strong>guarda el sistema</strong>, no la app. Antes de destruir la actividad, Android pide a la app que meta en él lo que quiere conservar; al crear la actividad nueva, se lo devuelve en <code>onCreate(savedInstanceState)</code>. Como lo custodia el sistema, fuera de la memoria de la app, sobrevive a los giros y también a que Android elimine la app en segundo plano, pero no a que el usuario la cierre. Solo admite tipos sencillos (números, textos, booleanos y listas de ellos) y tiene un <strong>límite de tamaño</strong>: lo recomendable es que no pase de unas decenas de KB, y si se supera el límite real (alrededor de 1 MB), la app se cierra con una <code>TransactionTooLargeException</code>. Por eso sirve para un número, un texto o un id, nunca para una lista de productos o una imagen. Volverá a aparecer en las sesiones 4 y 5.
</div>

| Situación | Variable normal | `remember` | `rememberSaveable` |
|---|---|---|---|
| Sobrevive a una recomposición | No | **Sí** | **Sí** |
| Sobrevive a un giro de pantalla u otro cambio de configuración | No | No | **Sí** |
| Sobrevive a que el sistema mate la app en segundo plano | No | No | **Sí** |
| Sobrevive a cerrar la app (o a reiniciar el móvil) | No | No | No (para eso, persistencia: DataStore en la sesión 6 y bases de datos en la UD4) |

Entonces, ¿por qué no usar siempre `rememberSaveable`? Porque el `Bundle` está pensado para **datos pequeños y sencillos**: números, textos, booleanos (para guardar objetos propios hace falta trabajo extra). Para los datos de una pantalla completa, como una lista de productos o un pedido, en la sesión 4 veremos la herramienta adecuada: el **`ViewModel`**.

<div class="callout-info">
<strong>Prueba avanzada:</strong> en <em>Ajustes → Sistema → Opciones de desarrollador</em> del emulador existe la opción <strong>No conservar actividades</strong>. Con ella activada, Android destruye cada actividad en cuanto deja de verse, como si le faltara memoria. Pulsad Inicio y volved: en Logcat veréis <code>onDestroy</code> y un nuevo <code>onCreate</code>. Es la forma de comprobar que vuestra app sobrevive a que el sistema la elimine. Desactivadla al terminar. (Si no veis las opciones de desarrollador, se activan en <em>Acerca del teléfono</em>, pulsando 7 veces sobre <em>Número de compilación</em>.)
</div>

## Tabla resumen de la sesión

| Necesito... | Uso |
|---|---|
| Enterarme de que la actividad se crea, se muestra, se oculta o se destruye | Sobrescribir `onCreate`, `onStart`, `onResume`, `onPause`, `onStop`, `onDestroy` (llamando siempre a `super`) |
| Registrar un mensaje de depuración | `Log.d(TAG, "mensaje")`, con `private const val TAG = "..."` |
| Ver solo mis mensajes | Logcat: `package:mine`, `tag:MiEtiqueta`, `level:error` |
| Averiguar por qué se ha cerrado la app | Logcat: `level:error` → `FATAL EXCEPTION` → enlace a la línea |
| Un dato que cambia y que la interfaz muestra | `var x by remember { mutableStateOf(valorInicial) }` |
| Reaccionar a un toque | Lambda `onClick = { ... }` que modifica el estado |
| Que el dato sobreviva al giro de pantalla | `rememberSaveable` en lugar de `remember` |

<div class="callout-practica">
<strong>Ejercicio de sesión</strong> (en un proyecto de pruebas o en la propia cafetería, 15 minutos):

1. Añade a la pantalla el composable <code>Contador</code> del apartado 2.6 en su versión con variable normal y comprueba que no funciona. Pásalo a <code>remember { mutableStateOf(0) }</code>.
2. Pon el contador a 5 y gira el emulador. Mira en Logcat (<code>tag:CicloVida</code>) qué ha ocurrido.
3. Cambia <code>remember</code> por <code>rememberSaveable</code> y repite la prueba.
</div>

## Para la próxima sesión

Ahora cada tarjeta de producto puede guardar su propia cantidad, pero intentad responder a esta pregunta: **¿cómo mostraríais, debajo de la carta, el total del pedido?** La cantidad de cada producto está "encerrada" dentro de su tarjeta y la pantalla no puede leerla. En la sesión 3 resolveremos este problema con el **state hoisting** (elevar el estado) y lo aplicaremos a controles de entrada de texto, diálogos y menús.
