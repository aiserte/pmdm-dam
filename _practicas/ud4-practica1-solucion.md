---
title: "Cafetería del instituto: el último ticket y el historial — Solución (Sesiones 1 y 2)"
unidad: UD4
sesion: 1
orden: 1
tipo: solucion
disponible: false
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Solo se muestran los ficheros nuevos o que cambian respecto a la última solución de la UD3. No cambian: la navegación, <code>PreferenciasRepository</code>, las pantallas de detalle, resumen y pedido enviado, y el resto de componentes.
</div>

### Estructura final

```
com.iesejemplo.cafeteria/
├── CafeteriaApp.kt                 ← nuevo: Application con la base de datos
├── MainActivity.kt
├── data/
│   ├── Producto.kt
│   ├── DatosCarta.kt
│   ├── PreferenciasRepository.kt
│   ├── Ticket.kt                   ← nuevo
│   ├── TicketRepository.kt         ← nuevo: fichero JSON
│   ├── HistorialRepository.kt      ← nuevo: historial en Room
│   └── local/
│       ├── PedidoEntity.kt         ← nuevo
│       ├── PedidoDao.kt            ← nuevo
│       └── CafeteriaDatabase.kt    ← nuevo
└── ui/
    ├── navegacion/                 (sin cambios)
    └── carta/
        ├── PedidoUiState.kt        ← + ultimoTicket, mostrarTicket,
        │                             pedidosCliente
        ├── PedidoViewModel.kt      ← usa los tres repositorios
        ├── DialogoTicket.kt        ← nuevo
        ├── BarraSuperiorCarta.kt   ← + "Último pedido"
        ├── PantallaCarta.kt        ← conecta la opción y el diálogo
        └── (resto sin cambios)
```

## Parte A · El último ticket en un fichero JSON

### Ejercicio 1 — Recursos

`res/values/strings.xml`:

```xml
    <string name="ultimo_pedido">Último pedido</string>
    <string name="titulo_ticket">Tu último pedido</string>
    <string name="ticket_para_llevar">Para llevar</string>
    <string name="ticket_para_tomar">Para tomar aquí</string>
    <string name="producto_desconocido">Producto %1$d</string>
    <string name="cerrar">Cerrar</string>
    <string name="pedidos_cliente">Pedidos de %1$s: %2$d</string>
```

`res/values-en/strings.xml`:

```xml
    <string name="ultimo_pedido">Last order</string>
    <string name="titulo_ticket">Your last order</string>
    <string name="ticket_para_llevar">Takeaway</string>
    <string name="ticket_para_tomar">Eat in</string>
    <string name="producto_desconocido">Product %1$d</string>
    <string name="cerrar">Close</string>
    <string name="pedidos_cliente">Orders by %1$s: %2$d</string>
```

### Ejercicio 2 — `data/Ticket.kt`

```kotlin
package com.iesejemplo.cafeteria.data

import kotlinx.serialization.Serializable

@Serializable
data class LineaTicket(
    val idProducto: Int,
    val cantidad: Int,
    val precioUnitario: Double      // precio en el momento del pedido
)

@Serializable
data class Ticket(
    val nombre: String,
    val paraLlevar: Boolean,
    val fechaHora: Long,            // System.currentTimeMillis()
    val lineas: List<LineaTicket>,
    val total: Double
)
```

`Ticket` está en el paquete `data` porque es un dato que se guarda, no algo de la interfaz. Su lista de líneas es de otra clase `@Serializable`, así que la librería sabe convertirla.

### Ejercicio 3 — `data/TicketRepository.kt`

```kotlin
package com.iesejemplo.cafeteria.data

import android.util.Log
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import kotlinx.serialization.SerializationException
import kotlinx.serialization.encodeToString
import kotlinx.serialization.json.Json
import java.io.File
import java.io.IOException

private const val TAG = "TicketRepo"

class TicketRepository(directorio: File) {

    private val fichero = File(directorio, "ultimo_ticket.json")

    private val json = Json {
        prettyPrint = true
        ignoreUnknownKeys = true
    }

    suspend fun guardar(ticket: Ticket) {
        withContext(Dispatchers.IO) {
            fichero.writeText(json.encodeToString(ticket))
        }
    }

    suspend fun cargar(): Ticket? = withContext(Dispatchers.IO) {
        if (!fichero.exists()) return@withContext null
        try {
            json.decodeFromString<Ticket>(fichero.readText())
        } catch (e: SerializationException) {
            Log.w(TAG, "El ticket guardado está dañado; se ignora", e)
            null
        } catch (e: IOException) {
            Log.w(TAG, "No se ha podido leer el ticket", e)
            null
        }
    }

    suspend fun borrar() {
        withContext(Dispatchers.IO) {
            fichero.delete()
        }
    }
}
```

- Si Android Studio marca en gris el `import kotlinx.serialization.encodeToString` (según la versión de la librería, puede no necesitarse), se puede borrar.
- `guardar` y `borrar` usan un cuerpo con llaves en lugar de `= withContext(...)`: así devuelven `Unit` y no el valor de la última línea (`delete()` devuelve un `Boolean` que no nos interesa).
- El `Json` se crea **una vez** por repositorio y se reutiliza.

### Ejercicio 4 — `ui/carta/PedidoUiState.kt`

Se muestra ya con el campo de la parte B:

```kotlin
data class PedidoUiState(
    val cantidades: Map<Int, Int> = emptyMap(),
    val nombre: String = "",
    val paraLlevar: Boolean = false,
    val avisar: Boolean = true,
    val intentoConfirmar: Boolean = false,
    val mostrarConfirmacion: Boolean = false,
    val categoria: Categoria? = null,
    val ultimoTicket: Ticket? = null,       // parte A
    val mostrarTicket: Boolean = false,     // parte A
    val pedidosCliente: Int? = null         // parte B
)
```

(con `import com.iesejemplo.cafeteria.data.Ticket`). `LineaPedido` sigue igual al final del fichero.

### Ejercicio 5 — `ui/carta/DialogoTicket.kt`

Se muestra ya con el parámetro `pedidosCliente` de la parte B:

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.LineaTicket
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.data.Ticket
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme
import java.text.DateFormat
import java.util.Date

@Composable
fun DialogoTicket(
    ticket: Ticket,
    pedidosCliente: Int?,
    buscarProducto: (Int) -> Producto?,
    onCerrar: () -> Unit,
    modifier: Modifier = Modifier
) {
    val fecha = remember(ticket.fechaHora) {
        DateFormat.getDateTimeInstance(DateFormat.SHORT, DateFormat.SHORT)
            .format(Date(ticket.fechaHora))
    }

    AlertDialog(
        onDismissRequest = onCerrar,
        modifier = modifier,
        title = { Text(stringResource(R.string.titulo_ticket)) },
        text = {
            Column(verticalArrangement = Arrangement.spacedBy(8.dp)) {
                Text(text = fecha, style = MaterialTheme.typography.labelLarge)
                Text(
                    text = ticket.nombre + " · " + stringResource(
                        if (ticket.paraLlevar) R.string.ticket_para_llevar
                        else R.string.ticket_para_tomar
                    )
                )
                HorizontalDivider()
                ticket.lineas.forEach { linea ->
                    FilaTicket(linea, buscarProducto(linea.idProducto))
                }
                HorizontalDivider()
                Text(
                    text = stringResource(R.string.total_pedido, ticket.total),
                    style = MaterialTheme.typography.titleMedium
                )
                if (pedidosCliente != null) {
                    Text(
                        text = stringResource(R.string.pedidos_cliente,
                            ticket.nombre, pedidosCliente),
                        style = MaterialTheme.typography.bodySmall
                    )
                }
            }
        },
        confirmButton = {
            TextButton(onClick = onCerrar) {
                Text(stringResource(R.string.cerrar))
            }
        }
    )
}

@Composable
private fun FilaTicket(linea: LineaTicket, producto: Producto?) {
    val nombre = if (producto != null) {
        stringResource(producto.nombre)
    } else {
        stringResource(R.string.producto_desconocido, linea.idProducto)
    }
    Row(modifier = Modifier.fillMaxWidth()) {
        Text(
            text = stringResource(R.string.linea_pedido,
                linea.cantidad, nombre),
            modifier = Modifier.weight(1f)
        )
        val importe = linea.cantidad * linea.precioUnitario
        Text(text = stringResource(R.string.precio, importe))
    }
}

@Preview
@Composable
fun DialogoTicketPreview() {
    CafeteriaTheme {
        DialogoTicket(
            ticket = Ticket(
                nombre = "Ana",
                paraLlevar = true,
                fechaHora = System.currentTimeMillis(),
                lineas = listOf(
                    LineaTicket(1, 2, 1.20),
                    LineaTicket(3, 1, 1.50),
                    LineaTicket(99, 1, 2.00)   // producto inexistente
                ),
                total = 5.90
            ),
            pedidosCliente = 3,
            buscarProducto = { id ->
                DatosCarta.productos.find { it.id == id }
            },
            onCerrar = {}
        )
    }
}
```

- El nombre del producto se resuelve **al dibujar**, con `stringResource`: por eso, si el móvil cambia de idioma, el ticket aparece traducido (prueba P1-5).
- El importe de cada línea usa `precioUnitario` del ticket, **no** el precio actual del producto.
- `remember(ticket.fechaHora) { ... }` es un `remember` **con clave**, una forma que no habíamos usado. Igual que el `remember` de la UD3, guarda el resultado entre recomposiciones; la diferencia es que, si el valor entre paréntesis cambia (otro ticket, otra fecha), descarta lo guardado y vuelve a ejecutar el bloque. Así la fecha no se formatea en cada recomposición, pero nunca se queda la de un ticket anterior.
- `HorizontalDivider()` es el componente de Material 3 que dibuja una línea fina de separación.
- El diálogo no conoce el `ViewModel`: recibe `buscarProducto` como lambda.

### Ejercicio 5 — `ui/carta/BarraSuperiorCarta.kt`

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraSuperiorCarta(
    hayPedido: Boolean,
    hayTicket: Boolean,
    onVaciarPedido: () -> Unit,
    onVerTicket: () -> Unit,
    onOlvidarDatos: () -> Unit,
    modifier: Modifier = Modifier
) {
    var menuAbierto by remember { mutableStateOf(false) }

    TopAppBar(
        // ... title, modifier y colors sin cambios
        actions = {
            Box {
                IconButton(onClick = { menuAbierto = true }) {
                    // ... icono de tres puntos sin cambios
                }
                DropdownMenu(
                    expanded = menuAbierto,
                    onDismissRequest = { menuAbierto = false }
                ) {
                    DropdownMenuItem(
                        text = { Text(stringResource(R.string.ultimo_pedido)) },
                        enabled = hayTicket,
                        onClick = {
                            menuAbierto = false
                            onVerTicket()
                        }
                    )
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

### Ejercicio 5 — `ui/carta/PantallaCarta.kt` (solo cambia la parte de arriba y el final)

```kotlin
    val estado = pedidoViewModel.uiState
    // ... snackbarHostState, scope y mensajes sin cambios

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraSuperiorCarta(
                hayPedido = pedidoViewModel.unidades > 0,
                hayTicket = estado.ultimoTicket != null,
                onVaciarPedido = { /* sin cambios */ },
                onVerTicket = pedidoViewModel::verTicket,
                onOlvidarDatos = { /* sin cambios */ }
            )
        },
        // ... bottomBar, snackbarHost y contenido sin cambios
    )

    val ticket = estado.ultimoTicket
    if (estado.mostrarTicket && ticket != null) {
        DialogoTicket(
            ticket = ticket,
            pedidosCliente = estado.pedidosCliente,
            buscarProducto = pedidoViewModel::buscarProducto,
            onCerrar = pedidoViewModel::cerrarTicket
        )
    }
```

Se copia `estado.ultimoTicket` a una variable local para que, después de comprobar `ticket != null`, el compilador sepa que no es nulo y se pueda pasar a `DialogoTicket` sin `!!`.

## Parte B · El historial de pedidos con Room

### Ejercicio 6 — Gradle

`gradle/libs.versions.toml`:

```
[versions]
room = "2.8.5"
ksp = "2.3.12"

[libraries]
androidx-room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
androidx-room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }

[plugins]
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

`build.gradle.kts` (raíz del proyecto), dentro de `plugins { }`:

```kotlin
alias(libs.plugins.ksp) apply false
```

`app/build.gradle.kts`:

```kotlin
plugins {
    // ... los que ya había
    alias(libs.plugins.ksp)
}

dependencies {
    // ... las que ya había
    implementation(libs.androidx.room.runtime)
    ksp(libs.androidx.room.compiler)
}
```

### Ejercicio 7 — `data/local/PedidoEntity.kt`

```kotlin
package com.iesejemplo.cafeteria.data.local

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "pedidos")
data class PedidoEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val nombre: String,
    @ColumnInfo(name = "para_llevar")
    val paraLlevar: Boolean,
    @ColumnInfo(name = "fecha_hora")
    val fechaHora: Long,
    val unidades: Int,
    val total: Double
)
```

### Ejercicio 7 — `data/local/PedidoDao.kt`

```kotlin
package com.iesejemplo.cafeteria.data.local

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.Query

@Dao
interface PedidoDao {

    @Insert
    suspend fun insertar(pedido: PedidoEntity): Long

    @Query("SELECT COUNT(*) FROM pedidos WHERE nombre = :nombre")
    suspend fun contarPorNombre(nombre: String): Int

    @Query("SELECT COUNT(*) FROM pedidos")
    suspend fun contarTodos(): Int

    @Query("DELETE FROM pedidos")
    suspend fun borrarTodos()
}
```

### Ejercicio 7 — `data/local/CafeteriaDatabase.kt`

```kotlin
package com.iesejemplo.cafeteria.data.local

import androidx.room.Database
import androidx.room.RoomDatabase

@Database(entities = [PedidoEntity::class], version = 1, exportSchema = false)
abstract class CafeteriaDatabase : RoomDatabase() {
    abstract fun pedidoDao(): PedidoDao
}
```

Tras compilar, en la vista *Project* de Android Studio se pueden ver las clases que ha generado Room: `app/build/generated/ksp/debug/kotlin/com/iesejemplo/cafeteria/data/local/`, con `PedidoDao_Impl.kt` y `CafeteriaDatabase_Impl.kt`. Merece la pena abrir `PedidoDao_Impl.kt` una vez: es el código SQLite que no hemos tenido que escribir.

### Ejercicio 8 — `CafeteriaApp.kt`

```kotlin
package com.iesejemplo.cafeteria

import android.app.Application
import androidx.room.Room
import com.iesejemplo.cafeteria.data.local.CafeteriaDatabase

class CafeteriaApp : Application() {

    val database: CafeteriaDatabase by lazy {
        Room.databaseBuilder(
            this,
            CafeteriaDatabase::class.java,
            "cafeteria.db"
        ).build()
    }
}
```

`AndroidManifest.xml` (solo cambia la etiqueta `<application>`):

```xml
<application
    android:name=".CafeteriaApp"
    android:allowBackup="true"
    ... >
```

El punto inicial de `.CafeteriaApp` significa "dentro del paquete de la app", igual que en `android:name=".MainActivity"`.

### Ejercicio 9 — `data/HistorialRepository.kt`

```kotlin
package com.iesejemplo.cafeteria.data

import com.iesejemplo.cafeteria.data.local.PedidoDao
import com.iesejemplo.cafeteria.data.local.PedidoEntity

class HistorialRepository(private val pedidoDao: PedidoDao) {

    suspend fun registrar(ticket: Ticket): Long =
        pedidoDao.insertar(
            PedidoEntity(
                nombre = ticket.nombre,
                paraLlevar = ticket.paraLlevar,
                fechaHora = ticket.fechaHora,
                unidades = ticket.lineas.sumOf { it.cantidad },
                total = ticket.total
            )
        )

    suspend fun contarPedidos(nombre: String): Int =
        pedidoDao.contarPorNombre(nombre)

    suspend fun contarTodos(): Int = pedidoDao.contarTodos()

    suspend fun borrarTodo() = pedidoDao.borrarTodos()
}
```

No se indica el `id` al crear `PedidoEntity`: vale `0` por defecto y la base de datos le asigna el siguiente. No hay `withContext`: las funciones `suspend` del DAO ya se ejecutan en los hilos de Room.

### Ejercicio 4 y 10 — `ui/carta/PedidoViewModel.kt` (completo)

```kotlin
package com.iesejemplo.cafeteria.ui.carta

import android.app.Application
import android.util.Log
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import com.iesejemplo.cafeteria.CafeteriaApp
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.HistorialRepository
import com.iesejemplo.cafeteria.data.LineaTicket
import com.iesejemplo.cafeteria.data.PreferenciasRepository
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.data.Ticket
import com.iesejemplo.cafeteria.data.TicketRepository
import com.iesejemplo.cafeteria.data.dataStore
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.launch

private const val TAG = "PedidoVM"

class PedidoViewModel(application: Application) :
    AndroidViewModel(application) {

    // 1. Primero, todo lo que usa el init
    private val preferenciasRepository =
        PreferenciasRepository(application.dataStore)
    private val ticketRepository = TicketRepository(application.filesDir)
    private val historialRepository = HistorialRepository(
        (application as CafeteriaApp).database.pedidoDao()
    )

    val productos: List<Producto> = DatosCarta.productos

    var uiState by mutableStateOf(PedidoUiState())
        private set

    // 2. Después, el init
    init {
        Log.d(TAG, "PedidoViewModel creado")
        viewModelScope.launch {
            val guardadas = preferenciasRepository.preferencias.first()
            val ticket = ticketRepository.cargar()
            uiState = uiState.copy(
                nombre = guardadas.nombre,
                paraLlevar = guardadas.paraLlevar,
                ultimoTicket = ticket
            )
            Log.d(TAG, "Preferencias cargadas: $guardadas")
            Log.d(TAG, "Ticket cargado: $ticket")
        }
    }

    // --- Valores calculados: sin cambios (productosVisibles, unidades,
    //     total, errorNombre, lineasPedido) y buscarProducto ---

    // --- Eventos sin cambios: cambiarCantidad, cambiarNombre,
    //     cambiarParaLlevar, cambiarAvisar, seleccionarCategoria,
    //     vaciarPedido, confirmar y cancelarConfirmacion ---

    fun enviarPedido() {
        val nombre = uiState.nombre
        val paraLlevar = uiState.paraLlevar
        // El ticket se construye ANTES de vaciar el pedido
        val ticket = Ticket(
            nombre = nombre,
            paraLlevar = paraLlevar,
            fechaHora = System.currentTimeMillis(),
            lineas = lineasPedido.map { linea ->
                LineaTicket(
                    idProducto = linea.producto.id,
                    cantidad = linea.cantidad,
                    precioUnitario = linea.producto.precio
                )
            },
            total = total
        )
        uiState = uiState.copy(
            cantidades = emptyMap(),
            mostrarConfirmacion = false,
            intentoConfirmar = false,
            ultimoTicket = ticket
        )
        viewModelScope.launch {
            preferenciasRepository.guardar(nombre, paraLlevar)
            ticketRepository.guardar(ticket)
            val id = historialRepository.registrar(ticket)
            val enHistorial = historialRepository.contarTodos()
            Log.d(TAG, "Pedido registrado con id $id " +
                "($enHistorial en el historial)")
        }
    }

    fun verTicket() {
        val ticket = uiState.ultimoTicket ?: return
        uiState = uiState.copy(mostrarTicket = true, pedidosCliente = null)
        viewModelScope.launch {
            val pedidos = historialRepository.contarPedidos(ticket.nombre)
            uiState = uiState.copy(pedidosCliente = pedidos)
        }
    }

    fun cerrarTicket() {
        uiState = uiState.copy(mostrarTicket = false)
    }

    fun olvidarDatos() {
        uiState = uiState.copy(
            nombre = "",
            paraLlevar = false,
            ultimoTicket = null,
            mostrarTicket = false,
            pedidosCliente = null
        )
        viewModelScope.launch {
            preferenciasRepository.borrar()
            ticketRepository.borrar()
            historialRepository.borrarTodo()
            Log.d(TAG, "Preferencias, ticket e historial borrados")
        }
    }

    override fun onCleared() {
        super.onCleared()
        Log.d(TAG, "PedidoViewModel destruido (onCleared)")
    }
}
```

Puntos a revisar:

- **El orden de `enviarPedido()`**: `lineasPedido` y `total` son propiedades **calculadas** a partir de `uiState.cantidades`. Si se vaciaran las cantidades primero, el ticket saldría vacío y con total 0. Es el error más habitual de esta práctica.
- **Los tres repositorios se declaran antes del `init`**, que usa dos de ellos (UD3, 6.7).
- **`(application as CafeteriaApp)`**: si falta el `android:name` del manifiesto, esta línea lanza la `ClassCastException` de la prueba P1-11, al crearse el `ViewModel`.
- **Dentro del `launch` de `enviarPedido()`** las operaciones van una detrás de otra: primero se guarda el fichero, después se inserta en la base de datos y, por último, se cuenta. Como cada llamada `suspend` espera a que termine la anterior, `contarTodos()` ya incluye el pedido recién insertado.
- **`verTicket()`** abre el diálogo enseguida con `pedidosCliente = null` (sin la línea del recuento) y lo rellena cuando llega el resultado de la consulta, unos milisegundos después. Así el diálogo nunca espera a la base de datos.
- **El `ViewModel` no conoce `PedidoEntity`**: solo trabaja con `Ticket`. Si el día de mañana la tabla cambia, solo cambia el repositorio.

### Ejercicio 11 — `PRUEBAS.md`, sección "UD4 · Sesiones 1 y 2" (resultados esperables)

| # | Prueba | Resultado obtenido |
|---|---|---|
| P1-1 | Primera ejecución | OK: "Último pedido" desactivado. Logcat: `Ticket cargado: null` (el fichero no existe; no es un error y no hay aviso de `TicketRepo`) |
| P1-2 | Enviar 2 cafés + 1 croissant como "Ana", para llevar | OK: fecha y hora actuales, "Ana · Para llevar", la línea de los 2 cafés con 2,40 €, la del croissant con 1,50 € y total 3,90 € |
| P1-3 | Cerrar y abrir | OK: el mismo ticket. Logcat: `PedidoViewModel creado` y `Ticket cargado: Ticket(nombre=Ana, ...)` |
| P1-4 | *Device Explorer* | `/data/data/com.iesejemplo.cafeteria/files/ultimo_ticket.json`, junto a la carpeta `datastore/`. Se lee como el JSON del apartado 1.4. El de DataStore está en *Protocol Buffers*, un formato **binario** pensado para ocupar poco y leerse rápido, no para personas |
| P1-5 | Idioma inglés | OK: los productos aparecen con su nombre en inglés. El fichero guarda el id 1, no el texto; el nombre se busca con `stringResource`, que elige el recurso según el idioma activo |
| P1-6 | Fichero dañado (`{roto`) | OK: la app arranca y "Último pedido" está desactivado. Logcat: `W TicketRepo: El ticket guardado está dañado; se ignora` seguido de la traza de una `JsonDecodingException` (subclase de `SerializationException`) |
| P1-7 | Tres pedidos (Ana, Ana, Luis) | OK: el último ticket es el de Luis y muestra "Pedidos de Luis: 1". Logcat: `Pedido registrado con id 1 (1 en el historial)`, luego id 2 y id 3. Si antes se hicieron otras pruebas sin desinstalar, los ids siguen la numeración: SQLite no reutiliza los ids |
| P1-8 | *Database Inspector* | Tabla `pedidos` con tres filas. `para_llevar` aparece como `1` o `0`: SQLite no tiene tipo booleano y Room convierte `true`/`false` al guardar y al leer |
| P1-9 | Consulta propia | `SELECT * FROM pedidos WHERE nombre = 'Ana' ORDER BY fecha_hora DESC` → dos filas, la más reciente primero. En el inspector se usa el nombre de la **columna** (`fecha_hora`), no el de la propiedad de Kotlin |
| P1-10 | Carpeta `databases/` | `cafeteria.db`, `cafeteria.db-wal` y `cafeteria.db-shm`. El `.db` es la base de datos; el `-wal` guarda los últimos cambios antes de pasarlos al `.db`; el `-shm` coordina el acceso. Los tres juntos forman la base de datos |
| P1-11 | Sin `android:name` | La app se cierra al arrancar: `ClassCastException: android.app.Application cannot be cast to com.iesejemplo.cafeteria.CafeteriaApp`. Android ha creado su `Application` por defecto, no la nuestra, y la conversión `as CafeteriaApp` del `ViewModel` falla |
| P1-12 | Cambiar la entidad sin desinstalar | `IllegalStateException: Room cannot verify the data integrity. Looks like you've changed schema but forgot to update the version number`. La base de datos del emulador tiene la estructura de la versión 1 y la clase ya no coincide. Al desinstalar se borra `cafeteria.db` y Room crea una nueva con la estructura actual |
| P1-13 | "Olvidar mis datos" | `ultimo_ticket.json` **desaparece** (`delete()`); el fichero de DataStore sigue existiendo pero vacío (`clear()`); la tabla `pedidos` sigue existiendo pero sin filas (`DELETE FROM pedidos`): tres formas distintas de "borrar" |

P1-6, P1-11 y P1-12 son las pruebas clave: muestran que la app sobrevive a un fichero dañado y enseñan a reconocer los dos errores más frecuentes al empezar con Room.

### Opcional — Total gastado por cliente

En `PedidoDao`:

```kotlin
    @Query("SELECT SUM(total) FROM pedidos WHERE nombre = :nombre")
    suspend fun totalGastado(nombre: String): Double?
```

`SUM` sobre cero filas no devuelve 0, sino `NULL`. Por eso el tipo de retorno es `Double?`: si fuera `Double`, Room no tendría con qué valor devolver ese `NULL` y la consulta fallaría para un cliente sin pedidos. En el repositorio se puede convertir con `?: 0.0`, y en el `ViewModel` se consulta junto al recuento, en el mismo `launch` de `verTicket()`.

## Qué comprobar en cada elemento obligatorio

- **Modelo del ticket** → `@Serializable` en `Ticket` y `LineaTicket`; en `LineaTicket` no aparece ningún `R.` ni ningún `Producto`.
- **`TicketRepository`** → constructor con `File`; `withContext(Dispatchers.IO)` en las tres funciones; dos `catch` en `cargar()`.
- **Gradle** → `room-compiler` con `ksp(...)`, no con `implementation(...)`; el plugin KSP en los dos `build.gradle.kts`.
- **Room** → `PedidoDao` es una `interface`; `CafeteriaDatabase` es `abstract`; todas las funciones del DAO son `suspend`; las consultas usan `:nombre`.
- **Una sola base de datos** → `Room.databaseBuilder` aparece **una vez** en todo el proyecto, dentro del `by lazy` de `CafeteriaApp`; `android:name=".CafeteriaApp"` en el manifiesto.
- **Capas** → `PedidoDao` solo aparece en `HistorialRepository`, `CafeteriaDatabase` y `PedidoViewModel` (al crear el repositorio); `PedidoEntity` no aparece en el `ViewModel` ni en la interfaz.
- **`PRUEBAS.md`** → P1-6, P1-11 y P1-12 con el error copiado **y** su explicación; P1-13 con las tres formas de borrar.
