---
title: "Cafetería del instituto: historial y detalle de pedidos — Solución (Sesión 3)"
unidad: UD4
sesion: 3
orden: 2
tipo: solucion
disponible: false
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Solo se muestran los ficheros nuevos o que cambian respecto a la solución de la práctica 1. No cambian: <code>PedidoViewModel</code>, <code>TicketRepository</code>, <code>PreferenciasRepository</code>, <code>CafeteriaApp</code>, <code>PedidoEntity</code> y las pantallas de detalle de producto, resumen y pedido enviado.
</div>

### Estructura final

```
com.iesejemplo.cafeteria/
├── CafeteriaApp.kt
├── MainActivity.kt
├── data/
│   ├── ...
│   ├── ResumenPedido.kt            ← nuevo
│   ├── HistorialRepository.kt      ← Flow, líneas, borrar
│   └── local/
│       ├── PedidoEntity.kt
│       ├── LineaPedidoEntity.kt    ← nuevo
│       ├── PedidoConLineas.kt      ← nuevo
│       ├── PedidoDao.kt            ← transacción, Flow, borrar
│       └── CafeteriaDatabase.kt    ← dos entidades
└── ui/
    ├── navegacion/
    │   ├── Rutas.kt                ← + Historial, DetallePedido
    │   └── AppCafeteria.kt         ← + dos destinos
    ├── carta/
    │   ├── DialogoTicket.kt        ← ContenidoTicket público
    │   ├── BarraSuperiorCarta.kt   ← + "Historial de pedidos"
    │   ├── PantallaCarta.kt        ← + onVerHistorial
    │   └── (resto sin cambios)
    └── historial/                  ← nuevo paquete
        ├── HistorialViewModel.kt
        ├── PantallaHistorial.kt
        ├── DetallePedidoViewModel.kt
        └── PantallaDetallePedido.kt
```

### Ejercicio 1 — Dependencia y recursos

`gradle/libs.versions.toml`, dentro de `[libraries]`:

```
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycleRuntimeKtx" }
```

`app/build.gradle.kts`, dentro de `dependencies { }`:

```kotlin
implementation(libs.androidx.lifecycle.runtime.compose)
```

`res/values/strings.xml`:

```xml
    <string name="historial_pedidos">Historial de pedidos</string>
    <string name="historial_vacio">Todavía no hay pedidos guardados.</string>
    <string name="titulo_detalle_pedido">Pedido n.º %1$d</string>
    <string name="borrar_pedido">Borrar pedido</string>
```

`res/values-en/strings.xml`:

```xml
    <string name="historial_pedidos">Order history</string>
    <string name="historial_vacio">No orders saved yet.</string>
    <string name="titulo_detalle_pedido">Order #%1$d</string>
    <string name="borrar_pedido">Delete order</string>
```

`%1$d` admite tanto `Int` como `Long`, así que sirve para el id del pedido.

### Ejercicio 2 — `data/local/LineaPedidoEntity.kt`

```kotlin
package com.iesejemplo.cafeteria.data.local

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.ForeignKey
import androidx.room.Index
import androidx.room.PrimaryKey

@Entity(
    tableName = "lineas_pedido",
    foreignKeys = [
        ForeignKey(
            entity = PedidoEntity::class,
            parentColumns = ["id"],
            childColumns = ["pedido_id"],
            onDelete = ForeignKey.CASCADE
        )
    ],
    indices = [Index("pedido_id")]
)
data class LineaPedidoEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    @ColumnInfo(name = "pedido_id")
    val pedidoId: Long,
    @ColumnInfo(name = "id_producto")
    val idProducto: Int,
    val cantidad: Int,
    @ColumnInfo(name = "precio_unitario")
    val precioUnitario: Double
)
```

`pedidoId` es `Long` porque guarda el `id` de `PedidoEntity`, que es `Long`. En `parentColumns` y `childColumns` van nombres de **columna**.

### Ejercicio 2 — `data/local/PedidoConLineas.kt`

```kotlin
package com.iesejemplo.cafeteria.data.local

import androidx.room.Embedded
import androidx.room.Relation

data class PedidoConLineas(
    @Embedded
    val pedido: PedidoEntity,
    @Relation(parentColumn = "id", entityColumn = "pedido_id")
    val lineas: List<LineaPedidoEntity>
)
```

No lleva `@Entity` ni se añade a `@Database`: no es una tabla, es la forma del resultado de una consulta.

### Ejercicio 2 — `data/local/CafeteriaDatabase.kt`

```kotlin
package com.iesejemplo.cafeteria.data.local

import androidx.room.Database
import androidx.room.RoomDatabase

@Database(
    entities = [PedidoEntity::class, LineaPedidoEntity::class],
    version = 1,
    exportSchema = false
)
abstract class CafeteriaDatabase : RoomDatabase() {
    abstract fun pedidoDao(): PedidoDao
}
```

Si se ejecuta sin desinstalar, la app se cierra con el error de esquema del apartado 1.15 del bloque 1.

### Ejercicio 3 — `data/local/PedidoDao.kt`

```kotlin
package com.iesejemplo.cafeteria.data.local

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.Query
import androidx.room.Transaction
import kotlinx.coroutines.flow.Flow

@Dao
interface PedidoDao {

    // --- Escribir ---

    @Insert
    suspend fun insertarPedido(pedido: PedidoEntity): Long

    @Insert
    suspend fun insertarLineas(lineas: List<LineaPedidoEntity>)

    @Transaction
    suspend fun insertarConLineas(
        pedido: PedidoEntity,
        lineas: List<LineaPedidoEntity>
    ): Long {
        val idPedido = insertarPedido(pedido)
        insertarLineas(lineas.map { it.copy(pedidoId = idPedido) })
        return idPedido
    }

    @Query("DELETE FROM pedidos WHERE id = :id")
    suspend fun borrarPorId(id: Long)

    @Query("DELETE FROM pedidos")
    suspend fun borrarTodos()

    // --- Observar (Flow, sin suspend) ---

    @Query("SELECT * FROM pedidos ORDER BY fecha_hora DESC")
    fun observarTodos(): Flow<List<PedidoEntity>>

    @Transaction
    @Query("SELECT * FROM pedidos WHERE id = :id")
    fun observarConLineas(id: Long): Flow<PedidoConLineas?>

    // --- Lecturas puntuales (sin cambios) ---

    @Query("SELECT COUNT(*) FROM pedidos WHERE nombre = :nombre")
    suspend fun contarPorNombre(nombre: String): Int

    @Query("SELECT COUNT(*) FROM pedidos")
    suspend fun contarTodos(): Int
}
```

- El `import` de `Flow` es el de `kotlinx.coroutines.flow`. Android Studio a veces propone otros `Flow` de otras librerías: hay que elegir ese.
- `borrarPorId` y `borrarTodos` no necesitan tocar `lineas_pedido`: el `CASCADE` borra las líneas.
- Tras compilar, merece la pena abrir `PedidoDao_Impl.kt` (apartado 1.9 del bloque 1) y buscar `insertarConLineas`: se ve cómo Room envuelve vuestra función en una transacción y llama a vuestro código dentro.

### Ejercicio 4 — `data/ResumenPedido.kt`

```kotlin
package com.iesejemplo.cafeteria.data

data class ResumenPedido(
    val id: Long,
    val nombre: String,
    val fechaHora: Long,
    val unidades: Int,
    val total: Double
)
```

### Ejercicio 4 — `data/HistorialRepository.kt` (completo)

```kotlin
package com.iesejemplo.cafeteria.data

import android.util.Log
import com.iesejemplo.cafeteria.data.local.LineaPedidoEntity
import com.iesejemplo.cafeteria.data.local.PedidoConLineas
import com.iesejemplo.cafeteria.data.local.PedidoDao
import com.iesejemplo.cafeteria.data.local.PedidoEntity
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

private const val TAG = "HistorialRepo"

class HistorialRepository(private val pedidoDao: PedidoDao) {

    // --- Observar ---

    val historial: Flow<List<ResumenPedido>> =
        pedidoDao.observarTodos().map { pedidos ->
            Log.d(TAG, "Consulta del historial: ${pedidos.size} pedidos")
            pedidos.map { it.aResumen() }
        }

    fun observarPedido(id: Long): Flow<Ticket?> =
        pedidoDao.observarConLineas(id).map { it?.aTicket() }

    // --- Escribir ---

    suspend fun registrar(ticket: Ticket): Long =
        pedidoDao.insertarConLineas(
            pedido = PedidoEntity(
                nombre = ticket.nombre,
                paraLlevar = ticket.paraLlevar,
                fechaHora = ticket.fechaHora,
                unidades = ticket.lineas.sumOf { it.cantidad },
                total = ticket.total
            ),
            lineas = ticket.lineas.map { linea ->
                LineaPedidoEntity(
                    pedidoId = 0,           // lo pone insertarConLineas
                    idProducto = linea.idProducto,
                    cantidad = linea.cantidad,
                    precioUnitario = linea.precioUnitario
                )
            }
        )

    suspend fun borrar(id: Long) = pedidoDao.borrarPorId(id)

    suspend fun borrarTodo() = pedidoDao.borrarTodos()

    // --- Lecturas puntuales (sin cambios) ---

    suspend fun contarPedidos(nombre: String): Int =
        pedidoDao.contarPorNombre(nombre)

    suspend fun contarTodos(): Int = pedidoDao.contarTodos()
}

// --- Conversiones: solo este fichero conoce las entidades ---

private fun PedidoEntity.aResumen() = ResumenPedido(
    id = id,
    nombre = nombre,
    fechaHora = fechaHora,
    unidades = unidades,
    total = total
)

private fun PedidoConLineas.aTicket() = Ticket(
    nombre = pedido.nombre,
    paraLlevar = pedido.paraLlevar,
    fechaHora = pedido.fechaHora,
    lineas = lineas.map { linea ->
        LineaTicket(
            idProducto = linea.idProducto,
            cantidad = linea.cantidad,
            precioUnitario = linea.precioUnitario
        )
    },
    total = pedido.total
)
```

- `historial` es una propiedad (`val`) y `observarPedido` una función: el historial es siempre el mismo `Flow`; el de un pedido depende del id que se pida.
- El `map` de fuera es el del `Flow` (necesita `import kotlinx.coroutines.flow.map`); el de dentro, el de las listas. Si falta ese `import`, Android Studio marca en rojo el primer `map`.
- `PedidoViewModel` sigue llamando a `registrar(ticket)` y recibe el id igual que antes: no cambia ni una línea.

### Ejercicio 5 — `ui/carta/DialogoTicket.kt`

Solo cambia la parte de arriba; `FilaTicket` y la preview siguen igual.

```kotlin
@Composable
fun DialogoTicket(
    ticket: Ticket,
    pedidosCliente: Int?,
    buscarProducto: (Int) -> Producto?,
    onCerrar: () -> Unit,
    modifier: Modifier = Modifier
) {
    AlertDialog(
        onDismissRequest = onCerrar,
        modifier = modifier,
        title = { Text(stringResource(R.string.titulo_ticket)) },
        text = {
            Column(verticalArrangement = Arrangement.spacedBy(8.dp)) {
                ContenidoTicket(ticket = ticket, buscarProducto = buscarProducto)
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
fun ContenidoTicket(
    ticket: Ticket,
    buscarProducto: (Int) -> Producto?,
    modifier: Modifier = Modifier
) {
    val fecha = remember(ticket.fechaHora) {
        DateFormat.getDateTimeInstance(DateFormat.SHORT, DateFormat.SHORT)
            .format(Date(ticket.fechaHora))
    }

    Column(
        modifier = modifier,
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
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
    }
}
```

`ContenidoTicket` sigue el patrón de siempre: recibe los datos y un `modifier` opcional, sin saber dónde se va a mostrar.

### Ejercicio 6 — `ui/historial/HistorialViewModel.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.historial

import android.app.Application
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import com.iesejemplo.cafeteria.CafeteriaApp
import com.iesejemplo.cafeteria.data.HistorialRepository
import com.iesejemplo.cafeteria.data.ResumenPedido
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn

class HistorialViewModel(application: Application) :
    AndroidViewModel(application) {

    private val historialRepository = HistorialRepository(
        (application as CafeteriaApp).database.pedidoDao()
    )

    // null = Room todavía no ha respondido; lista vacía = no hay pedidos
    val pedidos: StateFlow<List<ResumenPedido>?> =
        historialRepository.historial.stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = null
        )
}
```

`historialRepository` va **antes** que `pedidos`, porque `pedidos` lo usa al crearse (mismo motivo que el orden de las propiedades y el `init` en la UD3, 6.7).

### Ejercicio 6 — `ui/historial/PantallaHistorial.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.historial

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.PaddingValues
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Card
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import androidx.lifecycle.viewmodel.compose.viewModel
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.data.ResumenPedido
import com.iesejemplo.cafeteria.ui.carta.BarraConVolver
import com.iesejemplo.cafeteria.ui.theme.CafeteriaTheme
import java.text.DateFormat
import java.util.Date

@Composable
fun PantallaHistorial(
    onVolver: () -> Unit,
    onPedidoClick: (Long) -> Unit,
    modifier: Modifier = Modifier,
    historialViewModel: HistorialViewModel = viewModel()
) {
    val pedidos by historialViewModel.pedidos.collectAsStateWithLifecycle()

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraConVolver(
                titulo = stringResource(R.string.historial_pedidos),
                onVolver = onVolver
            )
        }
    ) { innerPadding ->
        val lista = pedidos              // copia local para comprobar null
        if (lista != null) {             // null: Room todavía no ha respondido
            ContenidoHistorial(
                pedidos = lista,
                onPedidoClick = onPedidoClick,
                modifier = Modifier.padding(innerPadding)
            )
        }
    }
}

@Composable
fun ContenidoHistorial(
    pedidos: List<ResumenPedido>,
    onPedidoClick: (Long) -> Unit,
    modifier: Modifier = Modifier
) {
    if (pedidos.isEmpty()) {
        Text(
            text = stringResource(R.string.historial_vacio),
            style = MaterialTheme.typography.bodyLarge,
            modifier = modifier.padding(16.dp)
        )
        return
    }
    LazyColumn(
        modifier = modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        items(items = pedidos, key = { it.id }) { pedido ->
            TarjetaPedido(pedido = pedido, onClick = { onPedidoClick(pedido.id) })
        }
    }
}

@Composable
private fun TarjetaPedido(
    pedido: ResumenPedido,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    val fecha = remember(pedido.fechaHora) {
        DateFormat.getDateTimeInstance(DateFormat.SHORT, DateFormat.SHORT)
            .format(Date(pedido.fechaHora))
    }
    Card(onClick = onClick, modifier = modifier.fillMaxWidth()) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(text = pedido.nombre, style = MaterialTheme.typography.titleMedium)
            Text(text = fecha, style = MaterialTheme.typography.bodySmall)
            Row(modifier = Modifier.fillMaxWidth()) {
                Text(
                    text = stringResource(R.string.unidades_pedido,
                        pedido.unidades),
                    modifier = Modifier.weight(1f)
                )
                Text(text = stringResource(R.string.precio, pedido.total))
            }
        }
    }
}

@Preview(name = "Con pedidos", showBackground = true)
@Composable
fun ContenidoHistorialPreview() {
    val ahora = System.currentTimeMillis()
    CafeteriaTheme {
        Surface {
            ContenidoHistorial(
                pedidos = listOf(
                    ResumenPedido(3, "Ana", ahora, 1, 2.00),
                    ResumenPedido(2, "Luis", ahora - 3_600_000, 1, 2.80),
                    ResumenPedido(1, "Ana", ahora - 7_200_000, 3, 3.90)
                ),
                onPedidoClick = {}
            )
        }
    }
}

@Preview(name = "Vacío", showBackground = true)
@Composable
fun ContenidoHistorialVacioPreview() {
    CafeteriaTheme {
        Surface {
            ContenidoHistorial(pedidos = emptyList(), onPedidoClick = {})
        }
    }
}
```

- `BarraConVolver` está en el paquete `ui.carta` y es pública: se importa y se reutiliza.
- En `ContenidoHistorial`, el `return` sale del composable después de dibujar el texto de lista vacía (igual que en `PantallaDetalle` de la UD3, 5.6).
- En la preview, `3_600_000` milisegundos es una hora: los pedidos de ejemplo tienen fechas distintas.
- `TarjetaPedido` es `private`: solo la usa este fichero.

### Ejercicio 7 — `ui/historial/DetallePedidoViewModel.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.historial

import android.app.Application
import android.util.Log
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.SavedStateHandle
import androidx.lifecycle.viewModelScope
import androidx.navigation.toRoute
import com.iesejemplo.cafeteria.CafeteriaApp
import com.iesejemplo.cafeteria.data.DatosCarta
import com.iesejemplo.cafeteria.data.HistorialRepository
import com.iesejemplo.cafeteria.data.Producto
import com.iesejemplo.cafeteria.data.Ticket
import com.iesejemplo.cafeteria.ui.navegacion.DetallePedido
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

private const val TAG = "DetalleVM"

class DetallePedidoViewModel(
    application: Application,
    savedStateHandle: SavedStateHandle
) : AndroidViewModel(application) {

    val idPedido: Long = savedStateHandle.toRoute<DetallePedido>().idPedido

    private val historialRepository = HistorialRepository(
        (application as CafeteriaApp).database.pedidoDao()
    )

    val pedido: StateFlow<Ticket?> =
        historialRepository.observarPedido(idPedido).stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = null
        )

    init {
        Log.d(TAG, "DetallePedidoViewModel creado para el pedido $idPedido")
    }

    fun buscarProducto(id: Int): Producto? =
        DatosCarta.productos.find { it.id == id }

    fun borrar(alTerminar: () -> Unit) {
        viewModelScope.launch {
            historialRepository.borrar(idPedido)
            Log.d(TAG, "Pedido $idPedido borrado")
            alTerminar()
        }
    }

    override fun onCleared() {
        super.onCleared()
        Log.d(TAG, "DetallePedidoViewModel destruido (onCleared)")
    }
}
```

- El `import` de `toRoute` es `androidx.navigation.toRoute`, el mismo que en `AppCafeteria`; aquí se usa sobre el `SavedStateHandle` en lugar de sobre la entrada.
- `idPedido` es pública porque la pantalla la usa en el título.
- `alTerminar()` se llama **después** de `borrar`, dentro de la misma coroutine: la pantalla no se cierra hasta que el pedido ya no está en la base de datos.

### Ejercicio 7 — `ui/historial/PantallaDetallePedido.kt`

```kotlin
package com.iesejemplo.cafeteria.ui.historial

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.OutlinedButton
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import androidx.lifecycle.viewmodel.compose.viewModel
import com.iesejemplo.cafeteria.R
import com.iesejemplo.cafeteria.ui.carta.BarraConVolver
import com.iesejemplo.cafeteria.ui.carta.ContenidoTicket

@Composable
fun PantallaDetallePedido(
    onVolver: () -> Unit,
    modifier: Modifier = Modifier,
    detalleViewModel: DetallePedidoViewModel = viewModel()
) {
    val pedido by detalleViewModel.pedido.collectAsStateWithLifecycle()

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraConVolver(
                titulo = stringResource(R.string.titulo_detalle_pedido,
                    detalleViewModel.idPedido),
                onVolver = onVolver
            )
        }
    ) { innerPadding ->
        val ticket = pedido              // copia local para comprobar null
        if (ticket != null) {
            Column(
                modifier = Modifier
                    .padding(innerPadding)
                    .padding(16.dp)
                    .fillMaxSize(),
                verticalArrangement = Arrangement.spacedBy(24.dp)
            ) {
                ContenidoTicket(
                    ticket = ticket,
                    buscarProducto = detalleViewModel::buscarProducto
                )
                OutlinedButton(
                    onClick = { detalleViewModel.borrar(alTerminar = onVolver) },
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Text(stringResource(R.string.borrar_pedido))
                }
            }
        }
    }
}
```

`OutlinedButton` es la variante de `Button` de Material con borde y sin relleno, y se usa exactamente igual. Para una acción secundaria y destructiva, como borrar, llama menos la atención que un botón relleno.

La pantalla no tiene preview propia: depende del `ViewModel` y de la base de datos. Lo que se dibuja ya tiene preview en el diálogo del ticket, que usa el mismo `ContenidoTicket`.

### Ejercicio 8 — `ui/navegacion/Rutas.kt` (se añaden al final)

```kotlin
@Serializable
data object Historial

@Serializable
data class DetallePedido(val idPedido: Long)
```

### Ejercicio 8 — `ui/navegacion/AppCafeteria.kt`

```kotlin
@Composable
fun AppCafeteria() {
    val navController = rememberNavController()
    // Un único PedidoViewModel para las pantallas del pedido: FUERA del NavHost
    val pedidoViewModel: PedidoViewModel = viewModel()

    NavHost(navController = navController, startDestination = Carta) {

        composable<Carta> {
            PantallaCarta(
                pedidoViewModel = pedidoViewModel,
                onProductoClick = { id -> navController.navigate(Detalle(id)) },
                onVerPedido = {
                    navController.navigate(Resumen) { launchSingleTop = true }
                },
                onVerHistorial = {
                    navController.navigate(Historial) { launchSingleTop = true }
                }
            )
        }

        // ... Detalle, Resumen y PedidoEnviado sin cambios

        // UD4 · Sesión 3: cada una con su propio ViewModel (dentro del destino)
        composable<Historial> {
            PantallaHistorial(
                onVolver = { navController.navigateUp() },
                onPedidoClick = { id ->
                    navController.navigate(DetallePedido(id))
                }
            )
        }

        composable<DetallePedido> {
            PantallaDetallePedido(
                onVolver = { navController.navigateUp() }
            )
        }
    }
}
```

(con los `import` de `Historial`, `DetallePedido`, `PantallaHistorial` y `PantallaDetallePedido`). `launchSingleTop` evita abrir dos historiales si se pulsa la opción dos veces seguidas (UD3, 5.7).

### Ejercicio 8 — `ui/carta/BarraSuperiorCarta.kt`

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BarraSuperiorCarta(
    hayPedido: Boolean,
    hayTicket: Boolean,
    onVaciarPedido: () -> Unit,
    onVerTicket: () -> Unit,
    onVerHistorial: () -> Unit,
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
                        text = {
                            Text(stringResource(R.string.historial_pedidos))
                        },
                        onClick = {
                            menuAbierto = false
                            onVerHistorial()
                        }
                    )
                    // ... "Vaciar pedido" y "Olvidar mis datos" sin cambios
                }
            }
        }
    )
}
```

### Ejercicio 8 — `ui/carta/PantallaCarta.kt` (solo cambia la cabecera y la barra)

```kotlin
@Composable
fun PantallaCarta(
    pedidoViewModel: PedidoViewModel,
    onProductoClick: (Int) -> Unit,
    onVerPedido: () -> Unit,
    onVerHistorial: () -> Unit,
    modifier: Modifier = Modifier
) {
    // ... estado, snackbarHostState, scope y mensajes sin cambios

    Scaffold(
        modifier = modifier,
        topBar = {
            BarraSuperiorCarta(
                hayPedido = pedidoViewModel.unidades > 0,
                hayTicket = estado.ultimoTicket != null,
                onVaciarPedido = { /* sin cambios */ },
                onVerTicket = pedidoViewModel::verTicket,
                onVerHistorial = onVerHistorial,
                onOlvidarDatos = { /* sin cambios */ }
            )
        },
        // ... bottomBar, snackbarHost, contenido y diálogo sin cambios
    )
}
```

La carta solo avisa de que el usuario quiere ver el historial; qué pantalla se abre lo decide `AppCafeteria` (UD3, 5.5).

### Ejercicio 9 — `PRUEBAS.md`, sección "UD4 · Sesión 3" (resultados esperables)

| # | Prueba | Resultado obtenido |
|---|---|---|
| P2-1 | Historial recién instalado | OK: "Todavía no hay pedidos guardados.". Logcat: `Consulta del historial: 0 pedidos`. Es una lista vacía, no `null`: Room ya ha respondido |
| P2-2 | Tres pedidos y abrir el historial | OK: Ana (zumo, 2,00 €), Luis (bocadillo, 2,80 €) y Ana (3 unidades, 3,90 €), en ese orden. **Una** consulta al abrir la pantalla. Al enviar los pedidos no apareció ninguna, porque nadie miraba el historial y `WhileSubscribed` no escuchaba a Room |
| P2-3 | *Database Inspector* | `pedidos` con 3 filas y `lineas_pedido` con 4: dos con `pedido_id = 1` (café y croissant), una con `2` y una con `3`. Consulta: `SELECT * FROM lineas_pedido WHERE pedido_id = 1`. `pedido_id` es una clave ajena: guarda el `id` de una fila de `pedidos` |
| P2-4 | Cambiar un nombre desde el inspector | OK: la tarjeta cambia sola. Logcat: una nueva `Consulta del historial: 3 pedidos`. El inspector avisa a Room de que la tabla `pedidos` ha cambiado; Room vuelve a ejecutar la consulta del `Flow`, el `StateFlow` recibe la lista nueva y Compose recompone |
| P2-5 | Girar dos veces | **No** hay consultas nuevas. Al girar, la `Activity` se destruye y se recrea, pero el `HistorialViewModel` sobrevive (está en el almacén de su entrada) y la pantalla vuelve a mirar el `StateFlow` en mucho menos de 5 s: `WhileSubscribed` no llega a dejar de escuchar y la pantalla recibe el último valor guardado |
| P2-6 | Inicio y volver (antes y después de 5 s) | Volviendo enseguida: `onPause` → `onStop` → `onRestart`… `onResume` (`CicloVida`), sin consulta nueva. Esperando 10 s: los mismos mensajes y, tras `onStart`, una nueva `Consulta del historial`. Con `onStop` la pantalla dejó de mirar; a los 5 s, `stateIn` dejó de escuchar a Room; con `onStart` volvió a mirar y se lanzó la consulta otra vez |
| P2-7 | Detalle e idioma | OK: fecha, "Ana · Para llevar" (o "Para tomar aquí"), las dos líneas con su importe y 3,90 €. En inglés, *Coffee with milk* y *Croissant*: igual que en el diálogo del ticket, el nombre se busca por id al dibujar |
| P2-8 | Borrar el pedido | OK: se vuelve al historial con dos pedidos, sin recargar nada a mano. Logcat: `Pedido 1 borrado`, una `Consulta del historial: 2 pedidos` y `DetallePedidoViewModel destruido`. En `lineas_pedido` ya no hay filas con `pedido_id = 1`: el `CASCADE` de la clave ajena las ha borrado con su pedido |
| P2-9 | *No conservar actividades* en el detalle | OK: la app vuelve al mismo detalle con sus datos. Logcat: al salir, `onStop`, `onDestroy` y `DetallePedidoViewModel destruido`; al volver, `onCreate` y `DetallePedidoViewModel creado para el pedido 2`. La pila de pantallas, con la ruta `DetallePedido(idPedido = 2)`, se guardó en el `Bundle`; el `ViewModel` nuevo lee ese id de su `SavedStateHandle` y Room vuelve a cargar el pedido |
| P2-10 | Sin índice | Aviso de Room en la ventana *Build*, del estilo de *pedido_id column references a foreign key but it is not part of an index. This may trigger full table scans whenever parent table is modified…*: sin índice, cada vez que se borra o modifica un pedido, SQLite tiene que recorrer toda la tabla de líneas para encontrar las suyas. Es un aviso, no un error: la app compila, pero sería más lenta con muchos datos |
| P2-11 | "Olvidar mis datos" | Historial vacío. Las dos tablas existen pero sin filas: `DELETE FROM pedidos` borra los pedidos y el `CASCADE`, sus líneas |

P2-4, P2-6 y P2-8 son las pruebas clave: demuestran que la interfaz refleja siempre la base de datos sin código de recarga, que la app deja de consultarla cuando no se ve y que la clave ajena mantiene los datos coherentes.

### Opcional — Confirmar antes de borrar

En `PantallaDetallePedido`, el diálogo se guarda en la propia pantalla con `rememberSaveable`, porque es estado visual de esa pantalla (UD3, 4.9):

```kotlin
    var confirmarBorrado by rememberSaveable { mutableStateOf(false) }

    // ... el botón ahora solo abre el diálogo:
    OutlinedButton(onClick = { confirmarBorrado = true }, ...) { ... }

    if (confirmarBorrado) {
        AlertDialog(
            onDismissRequest = { confirmarBorrado = false },
            title = { Text(stringResource(R.string.confirmar_borrado)) },
            confirmButton = {
                TextButton(onClick = {
                    confirmarBorrado = false
                    detalleViewModel.borrar(alTerminar = onVolver)
                }) { Text(stringResource(R.string.borrar)) }
            },
            dismissButton = {
                TextButton(onClick = { confirmarBorrado = false }) {
                    Text(stringResource(R.string.cancelar))
                }
            }
        )
    }
```

(con dos recursos nuevos: `confirmar_borrado` y `borrar`; `cancelar` ya existía). Con `rememberSaveable`, el diálogo sigue abierto tras girar el móvil; con `remember`, se cerraría. Guardarlo en el `ViewModel` también funcionaría (sobrevive al giro), pero es un detalle de cómo se muestra esta pantalla, no un dato de la app.

## Qué comprobar en cada elemento obligatorio

- **Entidad de líneas** → `foreignKeys` con `entity = PedidoEntity::class`, `parentColumns = ["id"]`, `childColumns = ["pedido_id"]` y `onDelete = ForeignKey.CASCADE`; `indices = [Index("pedido_id")]`.
- **DAO** → `insertarConLineas` con `@Transaction` y cuerpo; `observarTodos` y `observarConLineas` **sin** `suspend` y devolviendo `Flow`; `observarConLineas` con `@Transaction`.
- **Repositorio** → buscad `Entity` en el proyecto: solo debe aparecer en `data/local` y en `HistorialRepository.kt`. `aResumen` y `aTicket` son `private`.
- **`PedidoViewModel`** → sin cambios respecto a la práctica 1.
- **`ViewModel` nuevos** → `stateIn` con `WhileSubscribed(5_000)` e `initialValue = null`; `viewModel()` como valor por defecto en las pantallas; ninguna de las dos pantallas recibe `pedidoViewModel`. El detalle usa `savedStateHandle.toRoute<DetallePedido>()` y `composable<DetallePedido>` no lee la ruta.
- **Borrado** → `alTerminar()` se llama dentro del `launch`, después de `borrar`.
- **Pantallas** → `collectAsStateWithLifecycle` (no `collectAsState`); `ContenidoTicket` usado en el diálogo y en el detalle; previews de `ContenidoHistorial` con lista y vacía.
- **`PRUEBAS.md`** → P2-4, P2-6 y P2-8 con los mensajes de Logcat **y** su explicación.
