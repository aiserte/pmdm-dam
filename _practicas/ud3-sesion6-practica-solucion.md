---
title: "Cafetería del instituto: recordar al cliente — Solución (Sesión 6)"
unidad: UD3
sesion: 6
orden: 6
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Solo se muestran los ficheros nuevos o que cambian respecto a la solución de la sesión 5. Las pantallas de detalle, resumen y pedido enviado, la navegación y el resto de componentes no cambian: reciben el mismo <code>PedidoViewModel</code> y no saben que ahora guarda datos.
</div>

### Estructura final

```
com.iesejemplo.cafeteria/
├── data/
│   ├── Producto.kt
│   ├── DatosCarta.kt
│   └── PreferenciasRepository.kt   ← nuevo: DataStore + repositorio
└── ui/
    ├── navegacion/                 (sin cambios)
    └── carta/
        ├── PedidoViewModel.kt      ← AndroidViewModel, carga y guarda
        ├── BarraSuperiorCarta.kt   ← + "Olvidar mis datos"
        ├── PantallaCarta.kt        ← conecta la nueva opción
        └── (resto sin cambios)
```

### Ejercicio 1 — Dependencia y recursos

`gradle/libs.versions.toml`:

```
[versions]
datastore = "1.1.1"

[libraries]
androidx-datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }
```

`app/build.gradle.kts`, dentro de `dependencies { }`:

```kotlin
implementation(libs.androidx.datastore.preferences)
```

`res/values/strings.xml`:

```xml
    <string name="olvidar_datos">Olvidar mis datos</string>
    <string name="datos_olvidados">Se han borrado tus datos</string>
```

`res/values-en/strings.xml`:

```xml
    <string name="olvidar_datos">Forget my details</string>
    <string name="datos_olvidados">Your details have been deleted</string>
```

### Ejercicio 2 — `data/PreferenciasRepository.kt`

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

// Un único DataStore para toda la app, en el fichero "preferencias"
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

`Claves` es un `object` privado dentro de la clase: agrupa las claves con un nombre que se lee bien (`Claves.NOMBRE`) y nadie de fuera puede usarlas. `PreferenciasCliente` tiene valores por defecto, igual que `PedidoUiState`: describe lo que hay "si no se ha guardado nada".

### Ejercicio 3 — `ui/carta/PedidoViewModel.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import android.app.Application
import android.util.Log
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import com.iesejemplo.cafeteria.data.Categoria
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.PreferenciasRepository
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.data.dataStore
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.launch

private const val TAG = "PedidoVM"

class PedidoViewModel(application: Application) : AndroidViewModel(application) {

    // 1. Primero, todo lo que usa el init
    private val preferenciasRepository = PreferenciasRepository(application.dataStore)

    val productos: List<Producto> = DatosCarta.productos

    var uiState by mutableStateOf(PedidoUiState())
        private set

    // 2. Después, el init
    init {
        Log.d(TAG, "PedidoViewModel creado")
        viewModelScope.launch {
            val guardadas = preferenciasRepository.preferencias.first()
            uiState = uiState.copy(nombre = guardadas.nombre, paraLlevar = guardadas.paraLlevar)
            Log.d(TAG, "Preferencias cargadas: $guardadas")
        }
    }

    // --- Valores calculados (sin cambios) ---
    val productosVisibles: List<Producto>
        get() = uiState.categoria?.let { cat -> productos.filter { it.categoria == cat } } ?: productos

    val unidades: Int
        get() = uiState.cantidades.values.sum()

    val total: Double
        get() = productos.sumOf { (uiState.cantidades[it.id] ?: 0) * it.precio }

    val errorNombre: Boolean
        get() = uiState.intentoConfirmar && uiState.nombre.isBlank()

    val lineasPedido: List<LineaPedido>
        get() = productos
            .filter { (uiState.cantidades[it.id] ?: 0) > 0 }
            .map { LineaPedido(it, uiState.cantidades.getValue(it.id)) }

    fun buscarProducto(id: Int): Producto? = productos.find { it.id == id }

    // --- Eventos ---
    // cambiarCantidad, cambiarNombre, cambiarParaLlevar, cambiarAvisar,
    // seleccionarCategoria, vaciarPedido, confirmar y cancelarConfirmacion: sin cambios

    fun enviarPedido() {
        val nombre = uiState.nombre
        val paraLlevar = uiState.paraLlevar
        uiState = uiState.copy(
            cantidades = emptyMap(),
            mostrarConfirmacion = false,
            intentoConfirmar = false
        )
        viewModelScope.launch {
            preferenciasRepository.guardar(nombre, paraLlevar)
            Log.d(TAG, "Preferencias guardadas: nombre=$nombre, paraLlevar=$paraLlevar")
        }
    }

    fun olvidarDatos() {
        uiState = uiState.copy(nombre = "", paraLlevar = false)
        viewModelScope.launch {
            preferenciasRepository.borrar()
            Log.d(TAG, "Preferencias borradas")
        }
    }

    override fun onCleared() {
        super.onCleared()
        Log.d(TAG, "PedidoViewModel destruido (onCleared)")
    }
}
```

Puntos a revisar:

- **El constructor**: `application` llega como parámetro y se pasa a `AndroidViewModel`. Se usa una sola vez, para obtener `application.dataStore`; la `Activity` no aparece en ningún sitio.
- **El orden**: `preferenciasRepository` y `uiState` se declaran antes que el `init`. Si se mueve el `init` arriba del todo, la app falla al arrancar.
- **`first()`**: lee el fichero una vez. Si se usara `collect` en su lugar, el `init` se quedaría escuchando cambios para siempre y cada `guardar` volvería a copiar el nombre al estado (aquí no pasaría nada grave, pero no es lo que queremos).
- **`enviarPedido()`**: guarda el nombre y "para llevar" en variables antes de cambiar el estado, igual que en la sesión 5, y no los borra del estado: el siguiente pedido ya los tiene puestos.
- `AppCafeteria` no cambia: `viewModel()` crea el `AndroidViewModel` y le pasa la `Application` sin que haya que hacer nada.

### Ejercicio 4 — `ui/carta/BarraSuperiorCarta.kt`

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraSuperiorCarta(
    hayPedido: Boolean,
    onVaciarPedido: () -> Unit,
    onOlvidarDatos: () -> Unit,
    modifier: Modifier = Modifier
) {
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
                    DropdownMenuItem(
                        text = { Text(stringResource(R.string.olvidar_datos)) },
                        onClick = {
                            menuAbierto = false
                            onOlvidarDatos()
                        }
                    )
                }
            }
        }
    )
}
```

### Ejercicio 4 — `ui/carta/PantallaCarta.kt` (solo cambia la parte de arriba)

```kotlin
    val mensajeVaciado = stringResource(R.string.pedido_vaciado)
    val mensajeOlvidados = stringResource(R.string.datos_olvidados)

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraSuperiorCarta(
                hayPedido = pedidoViewModel.unidades > 0,
                onVaciarPedido = {
                    pedidoViewModel.vaciarPedido()
                    scope.launch { snackbarHostState.showSnackbar(mensajeVaciado) }
                },
                onOlvidarDatos = {
                    pedidoViewModel.olvidarDatos()
                    scope.launch { snackbarHostState.showSnackbar(mensajeOlvidados) }
                }
            )
        },
        // ... bottomBar, snackbarHost y contenido sin cambios
```

Es el mismo patrón que "Vaciar pedido": la lógica en el `ViewModel`, el *Snackbar* en la interfaz.

### Ejercicio 5 — `PRUEBAS.md`, sección "Sesión 6" (resultados esperables)

| # | Prueba | Resultado obtenido |
|---|---|---|
| P6-1 | Primera ejecución | OK: nombre vacío y "Para llevar" desactivado. Logcat: `Preferencias cargadas: PreferenciasCliente(nombre=, paraLlevar=false)` (los valores por defecto del repositorio, porque el fichero aún no existe) |
| P6-2 | Enviar como "Ana", para llevar; cerrar y abrir | OK. Logcat al enviar: `Preferencias guardadas: nombre=Ana, paraLlevar=true`. Al volver a abrir: `PedidoViewModel creado` y `Preferencias cargadas: PreferenciasCliente(nombre=Ana, paraLlevar=true)`. Al cerrar la app desde recientes se destruyó todo el proceso, incluido el `ViewModel`; los datos vienen del fichero |
| P6-3 | Cambiar el nombre sin enviar; cerrar y abrir | OK: vuelve "Ana". El nombre escrito solo estaba en `uiState` (memoria); al fichero solo se escribe en `enviarPedido()` |
| P6-4 | *Device Explorer* | `/data/data/com.iesejemplo.cafeteria/files/datastore/preferencias.preferences_pb`, unas decenas de bytes |
| P6-5 | "Olvidar mis datos"; cerrar y abrir | OK: *Snackbar* "Se han borrado tus datos" y, al volver, nombre vacío. El fichero **sigue existiendo**, pero ocupa 0 bytes (o casi): `clear()` vacía las preferencias, no borra el fichero |
| P6-6 | *Borrar datos* desde Ajustes | Se pierde el nombre guardado: esa opción borra toda la carpeta privada de la app (`/data/data/<paquete>/`), incluido el fichero de DataStore. Lo mismo ocurre al desinstalar |

P6-2 y P6-3 juntas muestran la diferencia entre **estado** (lo que hay en memoria ahora) y **datos persistidos** (lo que se ha decidido guardar). Es la idea con la que arranca la UD4.

### Opcional — Recordar el filtro de la carta

En `PreferenciasRepository`:

```kotlin
    private object Claves {
        val NOMBRE = stringPreferencesKey("nombre_cliente")
        val PARA_LLEVAR = booleanPreferencesKey("para_llevar")
        val CATEGORIA = stringPreferencesKey("categoria")
    }

    val categoria: Flow<Categoria?> = dataStore.data.map { prefs ->
        prefs[Claves.CATEGORIA]?.let { nombre -> Categoria.valueOf(nombre) }
    }

    suspend fun guardarCategoria(categoria: Categoria?) {
        dataStore.edit { prefs ->
            if (categoria == null) prefs.remove(Claves.CATEGORIA)
            else prefs[Claves.CATEGORIA] = categoria.name
        }
    }
```

En `PedidoViewModel`:

```kotlin
    init {
        // ... lo de antes, y dentro del mismo launch:
        //     val categoria = preferenciasRepository.categoria.first()
        //     uiState = uiState.copy(categoria = categoria)
    }

    fun seleccionarCategoria(categoria: Categoria?) {
        uiState = uiState.copy(categoria = categoria)
        viewModelScope.launch { preferenciasRepository.guardarCategoria(categoria) }
    }
```

"Todo" (`null`) se representa **quitando** la clave: así "no hay nada guardado" y "el usuario eligió Todo" significan lo mismo. Si en una versión futura de la app se renombrara un valor del `enum`, `Categoria.valueOf` lanzaría una excepción con el nombre antiguo; una versión más robusta usaría `Categoria.entries.find { it.name == nombre }`, que devuelve `null` en ese caso.

## Qué comprobar en cada elemento obligatorio

- **DataStore único** → `preferencesDataStore` aparece una sola vez en todo el proyecto, fuera de cualquier clase o función.
- **Repositorio** → las claves son privadas; fuera del repositorio no aparece `Preferences` ni ninguna clave. Buscad `dataStore` en el proyecto: solo debe salir en `PreferenciasRepository.kt` y en la línea del `ViewModel` que crea el repositorio.
- **`ViewModel`** → hereda de `AndroidViewModel`; no hay ninguna referencia a `MainActivity` ni a `LocalContext`. El `init` va después de las propiedades que usa.
- **Coroutines** → `viewModelScope.launch` en `init`, `enviarPedido` y `olvidarDatos`; ningún `runBlocking`.
- **`PRUEBAS.md`** → P6-2 con los dos mensajes de Logcat y P6-3 explicada con la diferencia entre estado y datos guardados.
