---
title: "Anatomía de una app Compose y recursos"
unit: UD3
order: 1
duration: "2h"
---

## Objetivos

- Identificar las piezas que forman una app Android con Jetpack Compose y el papel de cada una (manifiesto, `Activity`, funciones `@Composable`, tema, recursos).
- Organizar el código de una app en paquetes y ficheros con una estructura de clases razonable.
- Escribir funciones `@Composable` reutilizables siguiendo las convenciones de Compose (parámetro `modifier`, previsualizaciones con `@Preview`).
- Usar los recursos del proyecto (`strings.xml`, `drawable`) desde Compose con `stringResource` y `painterResource`.
- Preparar una app en varios idiomas y comprobar su funcionamiento en el emulador.

<div class="callout-info">
En el módulo de Desarrollo de Interfaces ya trabajáis el diseño de pantallas con Compose (<code>Column</code>, <code>Row</code>, <code>Box</code>, <code>Modifier</code>, Material 3…). Aquí lo damos por conocido y lo repasamos solo lo justo: el foco de esta sesión es <strong>cómo encaja todo eso dentro de una app Android</strong> y cómo se usan los recursos del proyecto.
</div>

## 1.1 De la consola a la app: qué cambia

En la UD2 vuestros programas empezaban en una función `main()` y terminaban cuando llegaban al final. Una app Android funciona de otra manera:

- **No hay `main()`.** Es el sistema operativo quien arranca la app cuando el usuario pulsa su icono, y lo hace creando un objeto de una clase vuestra: la `Activity`.
- **La app no "termina" cuando acaba el código.** Se queda esperando eventos (pulsaciones, giros de pantalla, llamadas entrantes…) y el sistema la puede pausar, destruir y volver a crear en cualquier momento.
- **La interfaz no se imprime: se declara.** En lugar de `println`, describís *qué* debe verse en pantalla con funciones `@Composable`, y Compose se encarga de dibujarlo y de redibujarlo cuando algo cambie.

Por eso, antes de escribir código, conviene saber qué ficheros intervienen y qué hace cada uno.

## 1.2 Las piezas de un proyecto Compose

Al crear un proyecto con la plantilla **Empty Activity**, Android Studio genera esta estructura (vista *Android* del panel de proyecto):

```
app/
├── manifests/
│   └── AndroidManifest.xml        ← "DNI" de la app
├── kotlin+java/
│   └── com.iesejemplo.cafeteria/
│       ├── MainActivity.kt        ← punto de entrada
│       └── ui/theme/
│           ├── Color.kt           ← colores del tema
│           ├── Theme.kt           ← tema Material 3 de la app
│           └── Type.kt            ← tipografías
└── res/
    ├── drawable/                  ← imágenes e iconos vectoriales
    ├── mipmap-*/                  ← icono del lanzador (varias densidades)
    └── values/
        ├── strings.xml            ← textos de la app
        ├── colors.xml
        └── themes.xml
Gradle Scripts/
└── build.gradle.kts (Module :app) ← dependencias y configuración
```

| Pieza | Qué hace | ¿La tocaremos a menudo? |
|---|---|---|
| `AndroidManifest.xml` | Declara la app ante el sistema: nombre, icono, actividades, permisos. | Poco (en UD5, para permisos). |
| `MainActivity.kt` | Clase que el sistema instancia al abrir la app. Aloja la interfaz. | Poco: solo llama a la pantalla principal. |
| Funciones `@Composable` | Describen la interfaz. Son donde está casi todo vuestro código de UI. | Constantemente. |
| `ui/theme/` | Colores, tipografías y formas de la app (Material 3). | Al principio, para personalizar. |
| `res/` | Recursos que no son código: textos, imágenes, iconos. | Constantemente. |
| `build.gradle.kts` | Dependencias (librerías) y versión mínima de Android. | Cuando añadamos librerías (Navigation, DataStore…). |

### El manifiesto

Este fragmento del `AndroidManifest.xml` es el que le dice a Android "cuando el usuario pulse el icono, crea un `MainActivity`":

```xml
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:theme="@style/Theme.Cafeteria">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

La pareja `MAIN` + `LAUNCHER` marca esta actividad como la que aparece en el cajón de aplicaciones. Una app Compose moderna suele tener **una única `Activity`**: las distintas pantallas no son actividades distintas, sino funciones `@Composable` entre las que navegaremos (sesión 5).

## 1.3 `MainActivity`, revisada con más detalle

Ya visteis una versión simplificada en la UD2. Esta es la que genera la plantilla actual:

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            CafeteriaTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Greeting(
                        name = "Android",
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }
}
```

Línea a línea:

- **`ComponentActivity`**: clase base de Android para actividades que usan Compose. Heredamos de ella (herencia de la UD2).
- **`onCreate`**: método del **ciclo de vida** que el sistema llama cuando crea la actividad. Ojo: no solo se llama al abrir la app; también cuando el sistema **destruye y vuelve a crear** la actividad, por ejemplo al girar la pantalla. Esto tendrá consecuencias en la sesión 2.
- **`enableEdgeToEdge()`**: hace que la app ocupe toda la pantalla, también detrás de la barra de estado y la de navegación del sistema.
- **`setContent { ... }`**: a partir de aquí, la interfaz se declara con Compose. Recibe una lambda (*trailing lambda*, UD2).
- **`CafeteriaTheme { ... }`**: función `@Composable` generada en `ui/theme/Theme.kt`. Aplica colores y tipografías de Material 3 a todo lo que va dentro. El nombre depende del nombre del proyecto.
- **`Scaffold`**: "andamio" de pantalla con huecos para barra superior, botón flotante, etc. (lo veremos a fondo en la sesión 3). Nos pasa `innerPadding`: el espacio que ocupan las barras del sistema, para que nuestro contenido no quede tapado por ellas.

<div class="callout-warning">
Si quitáis el <code>Modifier.padding(innerPadding)</code>, el contenido se dibuja por debajo de la barra de estado (hora, batería…) y parte del texto queda oculto. Es un error muy habitual al reorganizar el código de la plantilla: <strong>el <code>innerPadding</code> del <code>Scaffold</code> siempre tiene que aplicarse al contenido</strong>.
</div>

La idea que debéis llevaros: **la `Activity` es solo la puerta de entrada**. En cuanto la app crece, `MainActivity` se queda casi vacía y llama a una función `@Composable` de pantalla que vive en su propio fichero:

```kotlin
setContent {
    CafeteriaTheme {
        PantallaCarta()
    }
}
```

## 1.4 Funciones `@Composable`: reglas y convenciones

Una función `@Composable` es una función Kotlin normal marcada con la anotación `@Composable`. En lugar de devolver un valor, **emite** elementos de interfaz.

```kotlin
@Composable
fun Saludo(nombre: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hola, $nombre",
        modifier = modifier
    )
}
```

Reglas y convenciones que hay que respetar:

| Regla / convención | Por qué |
|---|---|
| El nombre empieza en **mayúscula** (`Saludo`, no `saludo`). | Convención de Compose: se leen como "componentes", no como acciones. Android Studio avisa si no se cumple. |
| Devuelve `Unit` (no se pone tipo de retorno). | No devuelve la interfaz: la emite. |
| Solo se puede llamar **desde otra función `@Composable`**. | Compose necesita saber en qué punto del árbol de interfaz se está. Llamarla desde una función normal da error de compilación. |
| Recibe por parámetro los datos que muestra. | Así es reutilizable: la misma `Saludo` sirve para cualquier nombre. |
| Tiene un parámetro `modifier: Modifier = Modifier`, el **primer parámetro opcional**, y lo aplica a su elemento raíz. | Permite que quien la llame decida márgenes, tamaño o posición sin tocar la función. |
| No debe tener efectos secundarios (escribir en ficheros, cambiar variables globales…). | Compose puede ejecutarla muchas veces, en cualquier orden. Lo veremos en la sesión 2 (recomposición). |

<div class="callout-warning">
El parámetro <code>modifier</code> se aplica <strong>solo al elemento raíz</strong> de la función, no a todos los hijos. Si lo pasáis a varios elementos internos, cualquier <code>padding</code> que ponga quien llama se aplicará varias veces y la interfaz se descuadrará.
</div>

### Organizar las funciones en ficheros y paquetes

Con dos o tres pantallas, meter todo en `MainActivity.kt` se vuelve inmanejable. Una estructura habitual, y la que usaremos en la app de la cafetería, es separar **datos** de **interfaz**:

```
com.iesejemplo.cafeteria/
├── MainActivity.kt
├── data/
│   ├── Producto.kt         ← data class del modelo
│   └── DatosCarta.kt       ← lista de productos de ejemplo
└── ui/
    ├── carta/
    │   ├── PantallaCarta.kt  ← composable de pantalla completa
    │   └── ProductoCard.kt   ← composable reutilizable
    └── theme/
```

Esto es precisamente lo que pide el criterio de evaluación **a)** ("generar la estructura de clases necesaria para la aplicación"): decidir qué clases y funciones necesita la app y dónde vive cada una. Para crear un paquete: clic derecho sobre el paquete principal → *New* → *Package*.

## 1.5 `@Preview`: ver sin ejecutar

`@Preview` permite ver una función `@Composable` en el panel *Design/Split* de Android Studio sin lanzar el emulador. Solo funciona sobre funciones **sin parámetros** (o con todos sus parámetros con valor por defecto), así que se suele escribir una función de preview aparte:

```kotlin
@Preview(showBackground = true)
@Composable
fun SaludoPreview() {
    CafeteriaTheme {
        Saludo(nombre = "Ana")
    }
}
```

Se pueden apilar varias `@Preview` sobre la misma función para ver distintas configuraciones a la vez:

```kotlin
@Preview(name = "Español", showBackground = true)
@Preview(name = "Inglés", showBackground = true, locale = "en")
@Preview(name = "Modo oscuro", showBackground = true, uiMode = Configuration.UI_MODE_NIGHT_YES)
@Preview(name = "Horizontal", showBackground = true, widthDp = 800, heightDp = 360)
@Composable
fun SaludoPreviews() {
    CafeteriaTheme {
        Saludo(nombre = "Ana")
    }
}
```

<div class="callout-info">
Envolved siempre la preview en el tema de la app (<code>CafeteriaTheme { }</code>). Si no, la preview usa los colores por defecto de Material y no se parece a lo que veréis en el emulador. <code>Configuration</code> se importa de <code>android.content.res.Configuration</code>.
</div>

La preview es una herramienta de **diseño**, no de **prueba**: no ejecuta la app real, no hay interacción completa ni ciclo de vida. Las pruebas de interacción se hacen en el emulador (apartado 1.9).

## 1.6 Repaso exprés: layouts y `Modifier`

Lo que ya conocéis de Desarrollo de Interfaces, en una tabla:

| Elemento | Para qué |
|---|---|
| `Column` | Apila hijos en vertical. `verticalArrangement` reparte el espacio; `horizontalAlignment` alinea. |
| `Row` | Coloca hijos en horizontal. `horizontalArrangement` / `verticalAlignment`. |
| `Box` | Superpone hijos (uno encima de otro). `contentAlignment`. |
| `Spacer` | Hueco vacío, normalmente con `Modifier.width()` o `.height()`. |
| `Modifier.weight(1f)` | Dentro de `Row`/`Column`, reparte el espacio sobrante. |

Solo un recordatorio importante, porque provoca muchos errores: **en `Modifier`, el orden importa**. Cada modificador se aplica sobre el resultado del anterior.

```kotlin
// Fondo que incluye el padding: caja coloreada más grande
Text("A", modifier = Modifier.background(Color.Yellow).padding(16.dp))

// Padding fuera del fondo: el color solo cubre el texto
Text("B", modifier = Modifier.padding(16.dp).background(Color.Yellow))
```

Lo mismo pasa con `clickable`: si va antes de `padding`, la zona pulsable incluye el margen; si va después, no.

## 1.7 Recursos: por qué no escribir textos "a pelo"

Hasta ahora escribíais los textos directamente en el código: `Text("Bienvenido")`. En una app real eso es un problema:

- Si queréis traducir la app, tendríais que buscar y cambiar cada texto en el código.
- Android no puede elegir automáticamente el idioma del usuario.
- Android Studio lo marca con un aviso (*Hardcoded string*).

La solución son los **recursos**: todo lo que no es lógica (textos, imágenes, colores, dimensiones) se guarda en la carpeta `res/`, y el código se refiere a ello por un identificador.

### `strings.xml` y `stringResource`

```xml
<!-- res/values/strings.xml -->
<resources>
    <string name="app_name">Cafetería</string>
    <string name="titulo_carta">Carta de hoy</string>
    <string name="saludo_usuario">Hola, %1$s</string>
    <string name="precio">%1$.2f €</string>
</resources>
```

```kotlin
Text(text = stringResource(R.string.titulo_carta))
Text(text = stringResource(R.string.saludo_usuario, "Ana"))   // "Hola, Ana"
Text(text = stringResource(R.string.precio, 1.5))             // "1,50 €"
```

- `R` es una clase que **genera Gradle automáticamente** a partir de la carpeta `res/`. Cada recurso tiene un identificador entero: `R.string.titulo_carta` es un `Int`, no un `String`.
- `stringResource(id)` busca ese identificador en el fichero del idioma activo y devuelve el texto.
- `%1$s`, `%1$d`, `%1$.2f` son **marcadores de formato** (como `String.format` en Java): el `1$` indica "primer argumento". Numerarlos permite que en otro idioma el orden de la frase sea distinto.
- El formato numérico respeta el idioma: en español, `1.5` se muestra como `1,50`.

<div class="callout-warning">
Dos errores típicos en <code>strings.xml</code>:
<ul>
<li>El <strong>apóstrofo</strong> hay que escaparlo: <code>&lt;string name="menu"&gt;Today\'s menu&lt;/string&gt;</code>. Sin la barra invertida, el proyecto no compila y el mensaje de error no siempre deja claro por qué.</li>
<li>Los nombres de recurso solo admiten <strong>minúsculas, números y guion bajo</strong> (<code>titulo_carta</code>, no <code>tituloCarta</code> ni <code>titulo-carta</code>).</li>
</ul>
</div>

`stringResource` solo puede usarse dentro de una función `@Composable`. Si alguna vez necesitáis un texto fuera de Compose (por ejemplo, en una clase normal), se usa `context.getString(R.string.titulo_carta)`; lo veremos cuando haga falta.

### Referencias a recursos en el modelo de datos

Si un objeto del modelo necesita un texto traducible, en vez de guardar el `String` guardamos **su identificador**. Las anotaciones `@StringRes` y `@DrawableRes` indican que ese `Int` no es un número cualquiera, sino un id de recurso, y Android Studio avisará si se pasa uno equivocado:

```kotlin
data class Producto(
    @StringRes val nombre: Int,
    @DrawableRes val imagen: Int,
    val precio: Double
)

val cafe = Producto(R.string.producto_cafe, R.drawable.ic_cafe, 1.20)
```

## 1.8 Imágenes: `drawable`, `painterResource` e `Image`

Las imágenes se guardan en `res/drawable/`. Pueden ser mapas de bits (PNG, JPG, WebP) o **vectores** (XML), que escalan sin perder calidad y ocupan muy poco.

La forma más rápida de conseguir iconos sin descargar nada es el **Vector Asset Studio**: clic derecho sobre `res/drawable` → *New* → *Vector Asset* → *Clip art*, y elegir entre los iconos de Material (café, bollería, pizza…). Android Studio genera el XML y lo nombra, por ejemplo, `baseline_local_cafe_24`. Para una imagen propia, basta con arrastrar el fichero a `res/drawable` (con un nombre válido: minúsculas y guion bajo, sin empezar por número).

```kotlin
Image(
    painter = painterResource(R.drawable.baseline_local_cafe_24),
    contentDescription = stringResource(R.string.producto_cafe),
    modifier = Modifier.size(64.dp),
    contentScale = ContentScale.Crop
)
```

- `painterResource(id)` carga el recurso y lo convierte en algo que Compose sabe dibujar.
- `contentDescription` es el texto que lee **TalkBack** (el lector de pantalla de Android) a personas con discapacidad visual. Es obligatorio pasarlo; si la imagen es puramente decorativa, se pasa `null` explícitamente.
- `contentScale` decide cómo encaja la imagen en su espacio: `Crop` (recorta para llenar), `Fit` (cabe entera), etc.

<div class="callout-info">
Para iconos de interfaz (los de las barras o los botones) se usa <code>Icon</code> en lugar de <code>Image</code>: funciona igual, pero se colorea automáticamente con el color del tema. <code>Image</code> se reserva para fotos e ilustraciones que tienen sus propios colores.
</div>

## 1.9 Varios idiomas y otras configuraciones

Android elige automáticamente qué fichero de recursos usar según la **configuración del dispositivo**. Esto se consigue con **calificadores** en el nombre de la carpeta:

```
res/
├── values/strings.xml       ← por defecto (español en nuestra app)
├── values-en/strings.xml    ← si el dispositivo está en inglés
└── values-ca/strings.xml    ← si el dispositivo está en catalán/valenciano
```

Mismo nombre de recurso en cada fichero, distinto texto:

```xml
<!-- res/values-en/strings.xml -->
<resources>
    <string name="titulo_carta">Today\'s menu</string>
    <string name="saludo_usuario">Hi, %1$s</string>
    <string name="precio">€%1$.2f</string>
</resources>
```

La forma más cómoda de crearlos es el **Translations Editor**: clic derecho sobre `strings.xml` → *Open Translations Editor* → icono del globo (*Add Locale*). Muestra una tabla con una columna por idioma y marca en rojo lo que falta por traducir.

Reglas importantes:

- La carpeta **sin calificador** (`values/`) es la de respaldo: debe contener **todos** los textos. Si falta una traducción en `values-en`, Android usa la de `values/`; si faltase en `values/` y el idioma del dispositivo no la tuviera, la app se cerraría al buscarla.
- `app_name` normalmente se marca como no traducible si es un nombre propio: `<string name="app_name" translatable="false">`.
- Los calificadores no son solo de idioma: `values-night/` (modo oscuro), `drawable-land/` (orientación horizontal)… El mecanismo es siempre el mismo: Android elige la carpeta más específica que encaja con el dispositivo.

## 1.10 Probar en el emulador (criterio g)

La preview no sustituye a una prueba real. El criterio de evaluación **g)** pide "pruebas de interacción usuario-aplicación a partir de emuladores". En esta sesión, la prueba consiste en comprobar que la app se comporta bien con distintas configuraciones:

| Qué probar | Cómo, en el emulador | Qué debe pasar |
|---|---|---|
| Idioma | *Ajustes* → *Sistema* → *Idiomas* → añadir y poner primero *English*. Volver a la app. | Todos los textos cambian; no queda ninguno en español. |
| Orientación | Botones de girar en la barra lateral del emulador (activad antes *Girar automáticamente* en los ajustes rápidos). | Nada se corta ni se solapa. |
| Modo oscuro | Ajustes rápidos → *Tema oscuro*. | Los textos siguen siendo legibles. |
| Tamaño de letra | *Ajustes* → *Pantalla* → *Tamaño de fuente* al máximo. | Los textos largos no se salen de su espacio. |
| Accesibilidad | Activar *TalkBack* un momento y tocar una imagen. | Lee su `contentDescription`. |

<div class="callout-warning">
Fijaos en lo que pasa al <strong>girar la pantalla</strong>: el sistema destruye la <code>Activity</code> y vuelve a crearla (se vuelve a ejecutar <code>onCreate</code>). Hoy no se nota porque nuestra app no guarda ningún dato que cambie. En la sesión 2, cuando la interfaz tenga estado (un contador, un texto escrito…), veremos que ese dato <strong>se pierde al girar</strong> y cómo evitarlo.
</div>

<div class="callout-info">
<strong>Colores del tema y Android 12+</strong>: la plantilla genera en <code>Theme.kt</code> un parámetro <code>dynamicColor: Boolean = true</code>. En dispositivos con Android 12 o superior, eso hace que la app tome los colores del fondo de pantalla del usuario e <strong>ignore los colores de <code>Color.kt</code></strong>. Si cambiáis los colores del tema y en el emulador no se ven, poned <code>dynamicColor = false</code>.
</div>

## Tabla resumen de la sesión

| Necesito... | Uso |
|---|---|
| Saber qué arranca la app | `AndroidManifest.xml` (`MAIN` + `LAUNCHER`) → `MainActivity.onCreate` |
| Declarar la interfaz | `setContent { Tema { PantallaPrincipal() } }` |
| Crear un componente reutilizable | `@Composable fun Nombre(datos, modifier: Modifier = Modifier)` |
| Ver un composable sin ejecutar | `@Preview` (una o varias, envuelto en el tema) |
| Mostrar un texto traducible | `stringResource(R.string.id)` / con argumentos `stringResource(R.string.id, valor)` |
| Mostrar una imagen del proyecto | `Image(painterResource(R.drawable.id), contentDescription = ...)` |
| Guardar un id de recurso en el modelo | `@StringRes val nombre: Int`, `@DrawableRes val imagen: Int` |
| Traducir la app | `res/values-xx/strings.xml` (Translations Editor) |

<div class="callout-practica">
<strong>Ejercicio de sesión</strong> (sobre el proyecto de la plantilla, sin mirar la práctica):

1. Mueve el texto <code>"Android"</code> y el saludo de <code>Greeting</code> a <code>strings.xml</code>, usando un marcador <code>%1$s</code> para el nombre.
2. Crea <code>values-en/strings.xml</code> con el Translations Editor y traduce el saludo.
3. Añade a la preview una segunda <code>@Preview</code> con <code>locale = "en"</code> y comprueba que cambia.
4. Ejecuta en el emulador, cambia el idioma del sistema a inglés y vuelve a la app.

15 minutos de trabajo autónomo; se corrige antes de empezar la práctica.
</div>

## Para la próxima sesión

Sesión 2: **ciclo de vida de la Activity y estado**. Empezaremos viendo, con **Logcat**, qué métodos del ciclo de vida (`onCreate`, `onStart`, `onResume`, `onPause`, `onStop`, `onDestroy`) ejecuta Android cuando abrimos la app, pulsamos Inicio, volvemos a ella o giramos la pantalla. Después haremos que nuestra carta, hasta ahora estática, reaccione a lo que hace el usuario (añadir productos al pedido) con `remember` y `mutableStateOf`, comprobaremos en Logcat por qué ese estado se pierde al girar la pantalla y cómo evitarlo con `rememberSaveable`.
