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

Para ampliar el concepto de `Flow`, visita [Flujos de Kotlin en Android](https://developer.android.com/kotlin/flow?hl=es-419).

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

`Context` es una clase abstracta de Android de la que heredan varias clases; las que usamos son `Activity` y `Application`. A tener en cuenta:

- El `Context` es la puerta de la app al sistema: carpetas, recursos, servicios.
- La `Activity` y la `Application` son `Context`. La primera muere al girar y la segunda vive toda la ejecución.
- DataStore necesita un `Context` para saber dónde guardar el fichero, y le pasamos la `Application` porque el `ViewModel` no puede guardar la `Activity`.

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

En el apartado anterior hemos visto las tres operaciones de DataStore (leer, escribir y borrar) sueltas. Ahora toca decidir **dónde** se escriben en nuestra app. La respuesta es una clase nueva, el **repositorio**. Vamos a verlo despacio, porque es la primera vez que la app tiene una capa de datos propia.

### El problema: un `ViewModel` que sabe demasiado

Podríamos poner el código de DataStore directamente en el `PedidoViewModel`. Quedaría algo así:

```kotlin
class PedidoViewModel(application: Application) : AndroidViewModel(application) {

    private val dataStore = application.dataStore
    private val NOMBRE = stringPreferencesKey("nombre_cliente")
    private val PARA_LLEVAR = booleanPreferencesKey("para_llevar")

    fun enviarPedido() {
        // ... vaciar el pedido ...
        viewModelScope.launch {
            dataStore.edit { prefs ->
                prefs[NOMBRE] = uiState.nombre
                prefs[PARA_LLEVAR] = uiState.paraLlevar
            }
        }
    }
    // ... y lo mismo para leer al arrancar y para borrar
}
```

Funciona, pero el `ViewModel`, que ya se encarga del pedido, ahora también tiene que saber:

1. **Qué claves existen** y cómo se llaman dentro del fichero (`"nombre_cliente"`, `"para_llevar"`).
2. **Qué valor usar si no hay nada guardado** (`""`, `false`).
3. **Cómo se guarda** físicamente la información (DataStore, `edit`, `Preferences`…).

Es el mismo problema que tenía `PantallaCarta` en la sesión 4, cuando dibujaba, guardaba y decidía a la vez: una clase con demasiados trabajos. Y tiene una consecuencia práctica: si en la UD4 decidimos guardar estos datos en una base de datos, habría que reescribir el `ViewModel`.

<div class="callout-info">
<strong>La comparación con la cafetería de verdad.</strong> El encargado de caja (el <code>ViewModel</code>) no baja al almacén a buscar entre las estanterías: le pide al responsable del almacén (el <strong>repositorio</strong>) «dame los datos del cliente» o «guarda esto». Cómo están ordenadas las estanterías solo lo sabe el responsable del almacén. Si mañana cambian el almacén de sitio, la caja ni se entera.
</div>

### La idea: una clase que solo guarda y recupera

Un **repositorio** es una clase cuyo único trabajo es **guardar y recuperar datos**, escondiendo dónde y cómo se guardan. Ofrece al resto de la app unas pocas operaciones con nombres claros, y nada más. Nuestro repositorio de preferencias ofrecerá exactamente tres cosas:

| Lo que ofrece | Para qué |
|---|---|
| `preferencias` | Leer los datos guardados del cliente |
| `guardar(nombre, paraLlevar)` | Guardar los datos del último pedido |
| `borrar()` | Olvidar los datos guardados |

Lo construimos en cinco pasos, en un fichero nuevo `data/PreferenciasRepository.kt` (el mismo en el que va la línea `val Context.dataStore ...` del apartado 6.4).

### Paso 1: qué datos devuelve, la clase `PreferenciasCliente`

```kotlin
data class PreferenciasCliente(
    val nombre: String = "",
    val paraLlevar: Boolean = false
)
```

En lugar de devolver dos valores sueltos, el repositorio los entrega **juntos** en una `data class`, igual que `PedidoUiState` agrupaba el estado del pedido. Los valores por defecto describen la situación "no hay nada guardado": nombre vacío y para tomar aquí.

### Paso 2: la clase y lo que recibe

```kotlin
class PreferenciasRepository(private val dataStore: DataStore<Preferences>) {
    // ...
}
```

- El repositorio **no crea** el DataStore: lo **recibe** en el constructor. Quien lo crea le pasará el que ya existe; en nuestro caso será el `ViewModel`, con `PreferenciasRepository(application.dataStore)` (lo veremos en el apartado 6.7).
- `private val` hace dos cosas a la vez: guarda el parámetro como propiedad de la clase y la deja **privada**. Nadie de fuera puede escribir `repositorio.dataStore`.
- `DataStore<Preferences>` es el tipo del objeto: un DataStore que guarda **preferencias** (pares clave-valor).

### Paso 3: las claves, escondidas dentro

```kotlin
    private object Claves {
        val NOMBRE = stringPreferencesKey("nombre_cliente")
        val PARA_LLEVAR = booleanPreferencesKey("para_llevar")
    }
```

- `object` crea un objeto único (no se instancia con `Claves()`): es una forma cómoda de **agrupar constantes** con un nombre que se lee bien, `Claves.NOMBRE`.
- `private` hace que solo el repositorio pueda usarlas. Así nadie más en la app puede escribir mal el nombre de una clave (por ejemplo, `"nombre"` en lugar de `"nombre_cliente"`) y acabar leyendo un valor que no existe.

### Paso 4: leer, la propiedad `preferencias`

Es la parte que más cuesta, así que vamos línea a línea:

```kotlin
    val preferencias: Flow<PreferenciasCliente> = dataStore.data.map { prefs ->
        PreferenciasCliente(
            nombre = prefs[Claves.NOMBRE] ?: "",
            paraLlevar = prefs[Claves.PARA_LLEVAR] ?: false
        )
    }
```

1. **`dataStore.data`** es un `Flow<Preferences>`: cada vez que se lee el fichero (y cada vez que cambia), entrega **todas** las preferencias guardadas, como un mapa.
2. **`.map { prefs -> ... }`** transforma cada uno de esos mapas en otra cosa. Dentro de la lambda, `prefs` es el mapa con todo lo guardado.
3. **`prefs[Claves.NOMBRE]`** busca el nombre en el mapa. Si nunca se ha guardado, devuelve `null`, y **`?: ""`** (el operador Elvis de la UD2) pone en su lugar una cadena vacía. Lo mismo con `paraLlevar` y `false`.
4. El resultado es un **`Flow<PreferenciasCliente>`**: quien lo use ya no ve claves ni mapas, solo objetos `PreferenciasCliente`.

Esta tabla muestra qué entrega el `Flow` según lo que haya en el fichero:

| Lo que hay en el fichero | Lo que entrega `preferencias` |
|---|---|
| Nada (primera vez que se abre la app) | `PreferenciasCliente(nombre = "", paraLlevar = false)` |
| Después de `guardar("Ana", true)` | `PreferenciasCliente(nombre = "Ana", paraLlevar = true)` |
| Después de `borrar()` | `PreferenciasCliente(nombre = "", paraLlevar = false)` |

<div class="callout-warning">
<strong>Declarar <code>preferencias</code> no lee el fichero.</strong> Esta línea es una <em>receta</em>: dice cómo convertir lo que haya en el fichero en un <code>PreferenciasCliente</code>. El fichero se lee de verdad cuando alguien pide un valor al <code>Flow</code>, por ejemplo con <code>first()</code> desde el <code>ViewModel</code> (apartado 6.7).
</div>

### Paso 5: escribir y borrar

Son las mismas funciones del apartado 6.5, ahora dentro del repositorio y usando sus claves:

```kotlin
    suspend fun guardar(nombre: String, paraLlevar: Boolean) {
        dataStore.edit { prefs ->
            prefs[Claves.NOMBRE] = nombre
            prefs[Claves.PARA_LLEVAR] = paraLlevar
        }
    }

    suspend fun borrar() {
        dataStore.edit { prefs -> prefs.clear() }
    }
```

Son `suspend` porque escribir en el disco tarda (apartado 6.2): quien las llame tendrá que hacerlo desde una coroutine.

### El fichero completo

```kotlin
package com.iesejemplo.cafeteria.data

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

// Un único DataStore para toda la app (apartado 6.4)
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "preferencias")

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

    suspend fun guardar(nombre: String, paraLlevar: Boolean) {
        dataStore.edit { prefs ->
            prefs[Claves.NOMBRE] = nombre
            prefs[Claves.PARA_LLEVAR] = paraLlevar
        }
    }

    suspend fun borrar() {
        dataStore.edit { prefs -> prefs.clear() }
    }
}
```

### Qué sabe cada capa

Con el repositorio, la app queda organizada en **tres capas**, y cada una solo conoce a la de al lado:

```
Interfaz (composables)  ──eventos──►  PedidoViewModel  ──►  PreferenciasRepository
        ◄──── estado ────                                       │
                                                                ▼
                                                    DataStore (fichero en disco)
```

| Capa | Qué sabe | Qué **no** sabe |
|---|---|---|
| Interfaz (composables) | Qué mostrar (`uiState`) y a qué función del `ViewModel` avisar | Que existe un repositorio o un fichero |
| `PedidoViewModel` | Que el repositorio ofrece `preferencias`, `guardar` y `borrar` | Las claves, los valores por defecto, que se usa DataStore |
| `PreferenciasRepository` | Las claves, los valores por defecto y cómo usar DataStore | Qué pantallas hay o cómo se dibujan |

### Dos recorridos completos

**Al enviar el pedido de Ana, para llevar:**

1. El usuario pulsa "Enviar" en el diálogo: la interfaz llama a `pedidoViewModel.enviarPedido()`.
2. El `ViewModel` vacía el pedido y llama a `preferenciasRepository.guardar("Ana", true)`.
3. El repositorio traduce eso a claves (`nombre_cliente`, `para_llevar`) y lo escribe con `dataStore.edit`.
4. DataStore guarda el fichero en el disco.

**Al volver a abrir la app al día siguiente:**

1. Se crea el `PedidoViewModel` y su bloque `init` pide `preferenciasRepository.preferencias.first()`.
2. DataStore lee el fichero; el repositorio convierte su contenido en `PreferenciasCliente(nombre = "Ana", paraLlevar = true)`.
3. El `ViewModel` copia esos dos valores en su `uiState`.
4. La interfaz se recompone y el campo del nombre ya muestra "Ana".

<div class="callout-info">
<strong>Si te pierdes, quédate con esta frase: el <code>ViewModel</code> pide, el repositorio sabe cómo.</strong> El patrón repositorio es la forma recomendada por Google de organizar los datos de una app Android. En la UD4 crearemos otro repositorio con la misma idea, esta vez para una base de datos.
</div>

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
