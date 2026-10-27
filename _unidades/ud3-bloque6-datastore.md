---
title: "Preferencias del usuario con DataStore"
unit: UD3
order: 6
duration: "1h"
---

## Objetivos

- Distinguir entre el estado que vive en memoria y los datos que se **guardan** en el dispositivo.
- Saber cuándo basta con guardar **preferencias** (pares clave-valor) y cuándo hace falta una base de datos.
- Guardar, leer y borrar preferencias con **Preferences DataStore**.
- Entender por qué DataStore es asíncrono y cómo se usa desde un `ViewModel` con `viewModelScope`.
- Comprobar en el emulador que los datos sobreviven a cerrar la app y localizar el fichero donde se guardan.

<div class="callout-info">
Esta sesión trabaja sobre todo el criterio <strong>a)</strong> del RA2: la app incorpora una capa de datos (un <em>repositorio</em> de preferencias) separada de la lógica y de la interfaz. Es también el primer contacto con la persistencia, que se trabaja a fondo en la UD4 con el criterio <strong>f)</strong>.
</div>

## 6.1 Lo que se pierde al cerrar la app

En la sesión 4 terminamos con esta tabla. La última fila se quedó pendiente:

| Mecanismo | Giro de pantalla | Android elimina la app en segundo plano | El usuario cierra la app |
|---|---|---|---|
| `remember` | No | No | No |
| `rememberSaveable` | Sí | Sí | No |
| `ViewModel` | Sí | No | No |
| **Persistencia** | Sí | Sí | **Sí** |

Todo lo que hemos usado hasta ahora vive en la **memoria** del proceso. Cuando el usuario cierra la app (la desliza fuera de la lista de apps recientes, la detiene desde Ajustes o reinicia el móvil), el proceso termina y la memoria se libera. Para que un dato siga ahí la próxima vez, hay que escribirlo en el **almacenamiento** del dispositivo. Eso es **persistir**.

En la cafetería, cada vez que alguien pide tiene que volver a escribir su nombre. Lo razonable es que la app **recuerde los datos del último pedido**: el nombre y si fue para llevar.

### Preferencias o base de datos

| Tipo de dato | Ejemplos | Herramienta |
|---|---|---|
| **Preferencias**: pocos valores sueltos, cada uno con un nombre | Nombre del cliente, tema oscuro sí/no, idioma elegido, última categoría vista | **DataStore** (esta sesión) |
| **Datos estructurados**: muchos registros con la misma forma, que se consultan y filtran | El historial de pedidos, la carta completa, los productos favoritos | **Base de datos** con Room (UD4) |

Una regla rápida: si los datos caben en una tabla de "nombre → valor" con unas pocas filas, son preferencias. Si piensas en ellos como "una lista de cosas", necesitas una base de datos.

<div class="callout-info">
<strong>¿Y <code>SharedPreferences</code>?</strong> Es la forma clásica de guardar preferencias en Android y la veréis en muchísimos tutoriales y proyectos antiguos. Funciona, pero lee y escribe el disco en el hilo principal (puede congelar la interfaz) y no avisa bien de los errores. <strong>DataStore</strong> es su sustituto oficial en Jetpack.
</div>

## 6.2 Cómo funciona Preferences DataStore

DataStore guarda los datos en **un fichero** dentro de la carpeta privada de la app. Cada valor se identifica con una **clave** que tiene tipo: la clave del nombre solo admite `String`; la de "para llevar", solo `Boolean`.

Lo que lo hace distinto de una variable normal es que **todas sus operaciones son asíncronas**:

- **Leer** devuelve un `Flow`: un flujo de valores que entrega los datos actuales y, después, un valor nuevo cada vez que cambian.
- **Escribir** es una función `suspend`: hay que llamarla desde una coroutine.

¿Por qué tanta complicación? Porque leer o escribir en el disco **tarda**. Si se hiciera en el hilo principal, el que dibuja la interfaz, la app se quedaría congelada mientras tanto; y si tarda más de unos segundos, Android muestra el diálogo *La aplicación no responde* (un **ANR**, *Application Not Responding*). Es exactamente el problema que resuelven las coroutines que visteis al final de la UD2: esperar sin bloquear.

### `Flow` en dos frases

Un `Flow` es como una suscripción: en lugar de devolver **un** valor, va entregando valores a lo largo del tiempo. Para leer DataStore nos basta con dos operaciones:

| Operación | Qué hace |
|---|---|
| `flow.map { ... }` | Transforma cada valor que llega (por ejemplo, de "todas las preferencias" a "solo el nombre") |
| `flow.first()` | Espera al **primer** valor, lo devuelve y deja de escuchar. Es `suspend` |

En la UD4, Room también devuelve `Flow` y lo aprovecharemos para que la interfaz se actualice sola cuando cambien los datos. Hoy solo necesitamos leer una vez, al arrancar.

## 6.3 Añadir la dependencia

En `gradle/libs.versions.toml`:

```
[versions]
datastore = "1.1.1"

[libraries]
androidx-datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }
```

En `app/build.gradle.kts`, dentro de `dependencies { }`:

```kotlin
implementation(libs.androidx.datastore.preferences)
```

Y *Sync Now*. Si Android Studio sugiere una versión más reciente, podéis aceptarla.

## 6.4 Crear el DataStore y sus claves

```kotlin
// Al principio del fichero, FUERA de cualquier clase
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "preferencias")
```

Esta línea tiene mucho dentro:

- `val Context.dataStore` es una **propiedad de extensión** (como las funciones de extensión de la UD2): a partir de ahora, cualquier `Context` (la `Activity`, la `Application`) tiene una propiedad `dataStore`.
- `by preferencesDataStore(...)` es una **delegación**, igual que el `by mutableStateOf` del estado: la librería se encarga de crear el DataStore la primera vez que se usa y de devolver **siempre el mismo** después.
- `name = "preferencias"` es el nombre del fichero que se creará.

<div class="callout-warning">
Debe existir <strong>un solo</strong> DataStore por fichero en toda la app. Si se crearan dos objetos sobre el mismo fichero, la app fallaría con una excepción. Por eso se declara una vez, en el nivel superior de un fichero, y nunca dentro de un composable o de una función.
</div>

Las claves se crean con una función por tipo:

```kotlin
val NOMBRE = stringPreferencesKey("nombre_cliente")
val PARA_LLEVAR = booleanPreferencesKey("para_llevar")
```

Hay una para cada tipo básico: `stringPreferencesKey`, `booleanPreferencesKey`, `intPreferencesKey`, `longPreferencesKey`, `floatPreferencesKey`, `doublePreferencesKey` y `stringSetPreferencesKey`. El texto entre comillas es el nombre con el que se guarda en el fichero; no debe cambiar una vez publicada la app, o se perderán los valores ya guardados.

## 6.5 Leer, escribir y borrar

**Leer.** `dataStore.data` es un `Flow<Preferences>`. Un objeto `Preferences` se consulta como un mapa: `prefs[CLAVE]` devuelve el valor o `null` si nunca se ha guardado.

```kotlin
val nombre: Flow<String> = dataStore.data.map { prefs ->
    prefs[NOMBRE] ?: ""          // si no hay nada guardado, cadena vacía
}
```

**Escribir.** `edit` recibe una lambda donde las preferencias se pueden modificar como un mapa mutable. Todo lo que se cambie dentro de una misma llamada a `edit` se guarda junto (si algo falla, no se guarda nada a medias):

```kotlin
suspend fun guardar(nombre: String, paraLlevar: Boolean) {
    dataStore.edit { prefs ->
        prefs[NOMBRE] = nombre
        prefs[PARA_LLEVAR] = paraLlevar
    }
}
```

**Borrar.** Una clave con `prefs.remove(NOMBRE)`; todas con `prefs.clear()`:

```kotlin
suspend fun borrar() {
    dataStore.edit { prefs -> prefs.clear() }
}
```

## 6.6 Un repositorio de preferencias

Podríamos usar el DataStore directamente desde el `ViewModel`, pero entonces el `ViewModel` tendría que saber qué claves existen, cómo se llaman en el fichero y qué valor usar si no hay nada guardado. Es mejor agrupar todo eso en una clase cuyo único trabajo sea **guardar y recuperar** las preferencias. A ese tipo de clase se le llama **repositorio**:

```kotlin
data class PreferenciasCliente(
    val nombre: String = "",
    val paraLlevar: Boolean = false
)

class PreferenciasRepository(private val dataStore: DataStore<Preferences>) {

    private object Claves {
        val NOMBRE = stringPreferencesKey("nombre_cliente")
        val PARA_LLEVAR = booleanPreferencesKey("para_llevar")
    }

    val preferencias: Flow<PreferenciasCliente> = dataStore.data.map { prefs ->
        PreferenciasCliente(
            nombre = prefs[Claves.NOMBRE] ?: "",
            paraLlevar = prefs[Claves.PARA_LLEVAR] ?: false
        )
    }

    suspend fun guardar(nombre: String, paraLlevar: Boolean) { /* edit, como en 6.5 */ }

    suspend fun borrar() { /* clear, como en 6.5 */ }
}
```

Fuera del repositorio nadie ve las claves ni el tipo `Preferences`: solo un `Flow<PreferenciasCliente>` y dos funciones. Si en la UD4 decidimos guardar estos datos en otro sitio, solo cambiará esta clase. Con esto la app queda en **tres capas**:

```
Interfaz (composables)  ──eventos──►  PedidoViewModel  ──►  PreferenciasRepository
        ◄──── estado ────                                       │
                                                                ▼
                                                    DataStore (fichero en disco)
```

## 6.7 Usar el repositorio desde el `ViewModel`

### El `ViewModel` necesita un `Context`

Para crear el repositorio hace falta el DataStore, y para obtener el DataStore hace falta un `Context`. Pero en la sesión 4 dijimos que el `ViewModel` **no debe guardar la `Activity`**, porque vive más que ella.

La solución más sencilla es `AndroidViewModel`: una variante de `ViewModel` que recibe en su constructor la **`Application`**, un `Context` que representa a la app entera y vive mientras vive el proceso. Como vive tanto como el `ViewModel` (o más), guardarla no provoca fugas de memoria.

```kotlin
class PedidoViewModel(application: Application) : AndroidViewModel(application) {

    private val preferenciasRepository = PreferenciasRepository(application.dataStore)

    // ... productos, uiState, etc.
}
```

En la interfaz no cambia nada: `viewModel()` sabe crear un `AndroidViewModel` y le pasa la `Application` automáticamente.

<div class="callout-info">
En apps profesionales el <code>ViewModel</code> no crea su repositorio: lo <strong>recibe</strong> en el constructor (<code>class PedidoViewModel(private val repo: PreferenciasRepository) : ViewModel()</code>), y una herramienta de <em>inyección de dependencias</em> como Hilt se encarga de construirlo. Es más flexible y más fácil de probar, pero añade bastante configuración. <code>AndroidViewModel</code> es la opción más directa para empezar.
</div>

### Coroutines en el `ViewModel`: `viewModelScope`

Las funciones del repositorio son `suspend`, así que hay que llamarlas dentro de una coroutine. En los composables usábamos `rememberCoroutineScope()`; en el `ViewModel` se usa **`viewModelScope`**, un ámbito de coroutines que el propio `ViewModel` trae de serie y que **cancela todas sus coroutines en `onCleared()`**. Así no queda ninguna tarea trabajando para una pantalla que ya no existe.

**Cargar al arrancar.** En el bloque `init`, se leen las preferencias una vez y se copian al estado:

```kotlin
init {
    viewModelScope.launch {
        val guardadas = preferenciasRepository.preferencias.first()
        uiState = uiState.copy(nombre = guardadas.nombre, paraLlevar = guardadas.paraLlevar)
    }
}
```

`first()` espera a que DataStore lea el fichero (unos milisegundos), devuelve los datos y deja de escuchar. Mientras tanto, la interfaz ya se ha dibujado con el estado inicial; cuando llegan los datos, el nombre aparece en el campo.

**Guardar al enviar el pedido.**

```kotlin
fun enviarPedido() {
    val nombre = uiState.nombre
    val paraLlevar = uiState.paraLlevar
    uiState = uiState.copy(cantidades = emptyMap(), mostrarConfirmacion = false, intentoConfirmar = false)
    viewModelScope.launch {
        preferenciasRepository.guardar(nombre, paraLlevar)
    }
}
```

El estado de la interfaz se actualiza **enseguida**; el guardado en disco se hace después, en segundo plano. El usuario no espera a que termine.

<div class="callout-warning">
<strong>Orden de las propiedades.</strong> En Kotlin, las propiedades y los bloques <code>init</code> se ejecutan en el orden en que están escritos. El <code>init</code> que carga las preferencias usa <code>preferenciasRepository</code> y <code>uiState</code>, así que las dos tienen que declararse <strong>antes</strong> que él. Si no, la app falla al arrancar con una <code>NullPointerException</code>.
</div>

¿Por qué guardar solo al enviar y no en cada tecla? Por diseño: queremos recordar los datos **del último pedido**, no lo que alguien escribió a medias y luego borró. Técnicamente, DataStore aguantaría guardar en cada cambio.

## 6.8 Comprobar dónde se guardan los datos

DataStore crea el fichero en la carpeta privada de la app:

```
/data/data/<paquete de la app>/files/datastore/preferencias.preferences_pb
```

Se puede ver desde Android Studio con *View › Tool Windows › Device Explorer*, navegando hasta esa ruta en el emulador. El fichero está en un formato binario (*Protocol Buffers*, de ahí la extensión `.pb`), pero al abrirlo se distinguen las claves y los valores guardados.

Esa carpeta es **privada**: ninguna otra app puede leerla. Y desaparece en dos casos:

- al **desinstalar** la app;
- con *Ajustes › Aplicaciones › Cafetería › Almacenamiento › Borrar datos*.

Con esto, la tabla de la sesión 4 queda completa:

| Mecanismo | Recomposición | Giro | Android elimina la app | Cerrar la app / reiniciar | Desinstalar |
|---|---|---|---|---|---|
| `remember` | Sí | No | No | No | No |
| `rememberSaveable` | Sí | Sí | Sí | No | No |
| `ViewModel` | Sí | Sí | No | No | No |
| `ViewModel` + `SavedStateHandle` | Sí | Sí | Sí | No | No |
| DataStore / base de datos | Sí | Sí | Sí | **Sí** | No |

## Tabla resumen de la sesión

| Necesito... | Uso |
|---|---|
| Añadir DataStore | `androidx.datastore:datastore-preferences` en el catálogo y en `app` |
| Crear el DataStore (una vez) | `val Context.dataStore by preferencesDataStore(name = "...")` en el nivel superior |
| Crear una clave | `stringPreferencesKey("...")`, `booleanPreferencesKey("...")`, `intPreferencesKey("...")`… |
| Leer como flujo | `dataStore.data.map { prefs -> prefs[CLAVE] ?: valorPorDefecto }` |
| Leer una sola vez | `flow.first()` (dentro de una coroutine) |
| Escribir | `dataStore.edit { prefs -> prefs[CLAVE] = valor }` (función `suspend`) |
| Borrar | `prefs.remove(CLAVE)` o `prefs.clear()` dentro de `edit` |
| Un `ViewModel` con acceso a `Context` | `class XxxViewModel(application: Application) : AndroidViewModel(application)` |
| Lanzar coroutines desde el `ViewModel` | `viewModelScope.launch { ... }` |
| Ver el fichero | *Device Explorer* › `/data/data/<paquete>/files/datastore/` |

## Cierre de la unidad

Con esta sesión termina la UD3. La **Cafetería del instituto** ha pasado de ser una carta estática a una app de varias pantallas con una estructura de clases por capas:

| Capa | Clases | Sesión |
|---|---|---|
| Modelo | `Producto`, `Categoria`, `DatosCarta` | 1 y 4 |
| Datos | `PreferenciasRepository` + DataStore | 6 |
| Lógica y estado | `PedidoViewModel`, `PedidoUiState`, `LineaPedido` | 4 y 5 |
| Navegación | Rutas `@Serializable`, `AppCafeteria` | 5 |
| Interfaz | Pantallas con `Scaffold`, componentes stateless, recursos en dos idiomas | 1, 2, 3 y 5 |

En la UD4 (*Persistencia y almacenamiento*) pasaremos de las preferencias a las **bases de datos**, para guardar datos estructurados como la carta o el historial de pedidos.
