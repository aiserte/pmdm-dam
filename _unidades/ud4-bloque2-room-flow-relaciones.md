---
title: "Room II: datos que se actualizan solos y relaciones entre tablas"
unit: UD4
order: 2
duration: "2h"
---

## Objetivos

- Hacer que una consulta de Room devuelva un `Flow` y entender por qué, así, la interfaz se actualiza sola cuando cambian los datos.
- Convertir ese `Flow` en un `StateFlow` en el `ViewModel` con `stateIn`, y mostrarlo en Compose con `collectAsStateWithLifecycle`, que deja de escuchar cuando la app no se ve.
- Relacionar dos tablas (los pedidos y sus líneas) con una **clave ajena**, y leer un pedido con todas sus líneas en una sola llamada.
- Usar una **transacción** para que guardar un pedido con sus líneas sea "todo o nada".
- Dar a cada pantalla nueva su propio `ViewModel`, alimentado por la misma base de datos.

<div class="callout-info">
Este bloque ocupa la tercera sesión de la unidad (2h). Sigue trabajando el criterio <strong>f)</strong> del RA2: la app guarda datos relacionados en dos tablas y los muestra siempre al día. Y el <strong>g)</strong>, porque comprobaremos en el emulador, con Logcat y el <em>Database Inspector</em>, cuándo la app consulta la base de datos y cuándo deja de hacerlo.
</div>

## Antes de empezar: qué necesitamos saber

Repasad las piezas de la primera tabla si alguna no os suena: el bloque **no** las vuelve a explicar.

| Ya visto | Dónde | Para qué lo usamos hoy |
|---|---|---|
| `Flow`, `map` y `first()`; declarar un `Flow` no lee nada (es una "receta") | UD3, sesión 6 (6.2 y 6.6) | Room también devuelve `Flow` |
| `viewModelScope` y `launch` | UD3, sesión 6 (6.7) | Dónde se ejecuta la escucha de la base de datos |
| Métodos del ciclo de vida: cuándo la `Activity` es visible (`onStart` / `onStop`) | UD3, sesión 2 (2.2) | Saber cuándo una pantalla escucha a la base de datos |
| Añadir una librería de *lifecycle* con el catálogo de versiones | UD3, sesión 4 (4.6) | La dependencia nueva de hoy usa la misma versión |
| `copy` en las `data class` | UD3, sesión 4 (4.7) | Completar las líneas con el id de su pedido |
| `items` con `key` | UD3, sesión 4 (4.2) | La lista del historial |
| `SavedStateHandle` | UD3, sesión 4 (4.9) | Leer el id del pedido desde el `ViewModel` del detalle |
| Un `ViewModel` por pantalla: `viewModel()` dentro de `composable<...>` usa el almacén de esa entrada de la pila | UD3, sesión 5 (5.8); UD4, bloque 1 (1.12) | Las dos pantallas nuevas tienen su propio `ViewModel` |
| Argumentos de una ruta y `toRoute()`; por la ruta solo viaja el id | UD3, sesión 5 (5.6 y 5.9) | Abrir el detalle de un pedido |
| Interfaces con métodos con cuerpo | UD2, sesión 1 (1.5) | Una función del DAO con varias operaciones |
| Funciones de extensión y `map` sobre listas | UD2, sesión 3 (3.1, 3.2 y 3.7) | Convertir filas de la tabla en objetos de la app |
| Guardar "todo junto" con `edit` | UD3, sesión 6 (6.5) | La idea de transacción |
| SQL mínimo, entidad, DAO, base de datos y parámetros `:nombre` | UD4, bloque 1 (1.7 y 1.11) | La tabla nueva y sus consultas |
| Una sola base de datos, creada en `CafeteriaApp` | UD4, bloque 1 (1.12) | Por qué una pantalla se entera de lo que cambia otra |
| Si cambia una entidad durante el desarrollo, desinstalar la app | UD4, bloque 1 (1.15) | Al añadir la tabla de líneas |
| *Database Inspector* | UD4, bloque 1 (1.15) | Ver las dos tablas y modificar datos en vivo |

| Nuevo hoy | Apartado |
|---|---|
| Consultas de Room que devuelven `Flow` | 2.2 |
| Escuchar un `Flow` sin fin: `collect` | 2.3 |
| `StateFlow`, `stateIn` y `SharingStarted.WhileSubscribed` | 2.4 |
| `collectAsStateWithLifecycle` y por qué "con ciclo de vida" | 2.5 |
| Relación 1:N, clave ajena (`@ForeignKey`), borrado en cascada e índices | 2.6 |
| Transacciones con `@Transaction` | 2.7 |
| Leer un pedido con sus líneas: `@Embedded` y `@Relation` | 2.8 |
| Leer los argumentos de la ruta desde el `ViewModel` | 2.10 |

## Parte A · Datos que se actualizan solos

## 2.1 El problema: preguntar una vez no basta

En el bloque 1, todas las funciones del DAO eran `suspend`: se llaman, devuelven **un** resultado y terminan. Es lo mismo que hace `first()` con DataStore. Para contar los pedidos al abrir el diálogo del ticket es suficiente, porque el diálogo pregunta una vez y muestra la respuesta.

Hoy la cafetería tendrá una pantalla de **historial** con todos los pedidos. Si la lista se cargara una sola vez al abrir la pantalla, habría que acordarse de volver a cargarla cada vez que algo cambie: al borrar un pedido desde la pantalla de detalle, al pulsar "Olvidar mis datos", al enviar un pedido nuevo... Con cada pantalla nueva que modifique la base de datos, habría un sitio más donde olvidarse.

La solución es darle la vuelta: en lugar de que la pantalla **pregunte**, que la base de datos **avise** cuando algo cambie. Es exactamente lo que es un `Flow` (UD3, 6.2): una suscripción que entrega un valor y, después, uno nuevo cada vez que los datos cambian.

## 2.2 Consultas que devuelven `Flow`

Para que una consulta de Room avise de los cambios, basta con cambiar su tipo de retorno:

```kotlin
@Query("SELECT * FROM pedidos ORDER BY fecha_hora DESC")
fun observarTodos(): Flow<List<PedidoEntity>>
```

Fijaos en tres detalles:

- **No es `suspend`.** La función no consulta nada: devuelve al momento un `Flow`, que es una *receta*, igual que la propiedad `preferencias` del repositorio de la UD3 (6.6). La consulta se ejecuta cuando alguien empieza a escuchar ese `Flow`.
- **Room vuelve a ejecutar la consulta cada vez que cambia la tabla `pedidos`**, haga quien haga el cambio dentro de la app, y entrega la lista nueva. Room no comprueba si el cambio afecta a vuestra consulta concreta: si se modifica la tabla, la vuelve a ejecutar.
- Por convención, las funciones que devuelven `Flow` se nombran con *observar* (`observarTodos`) y las `suspend` de una sola lectura con *obtener* o *contar*.

| | Función suspend | Función que devuelve un Flow |
|---|---|---|
| Ejemplo | `suspend fun contarTodos(): Int` | `fun observarTodos(): Flow<List<PedidoEntity>>` |
| Cuándo se ejecuta la consulta | Al llamarla | Cuando alguien empieza a escuchar el `Flow` |
| Cuántos resultados entrega | Uno | El primero y uno más cada vez que cambia la tabla |
| Uso típico | Insertar, borrar, una lectura puntual | Lo que muestra una pantalla |

<div class="callout-info">
<strong>Por esto había que tener una sola base de datos</strong> (lo anunciamos en el apartado 1.12). Room sabe que una tabla ha cambiado porque el cambio pasa por <strong>su</strong> objeto <code>CafeteriaDatabase</code>, que lleva la cuenta de qué tablas se han modificado y avisa a los <code>Flow</code> que dependen de ellas. Si hubiera dos objetos sobre el mismo fichero, lo que se insertara por uno no lo sabría el otro, y una pantalla podría quedarse con datos antiguos.
</div>

## 2.3 Escuchar un `Flow`: `first()` y `collect`

Para leer un `Flow` ya conocéis `first()`: espera al primer valor y deja de escuchar. Para recibir **todos** los valores se usa `collect`:

```kotlin
viewModelScope.launch {
    pedidoDao.observarTodos().collect { pedidos ->
        Log.d(TAG, "Hay ${pedidos.size} pedidos")
    }
    Log.d(TAG, "Esta línea no se ejecuta nunca")
}
```

- `collect` es `suspend` y ejecuta la lambda **una vez por cada valor** que llega.
- Un `Flow` de Room no termina nunca: mientras haya alguien escuchando, puede llegar otro cambio. Por eso `collect` **no termina** y la coroutine se queda suspendida en esa línea. Lo que se escriba detrás, dentro del mismo `launch`, no se ejecuta.
- La escucha acaba cuando se cancela la coroutine. Con `viewModelScope`, eso pasa cuando se destruye el `ViewModel` (`onCleared`).

| Operación | Qué hace | Cuándo termina |
|---|---|---|
| `flow.first()` | Devuelve el primer valor | En cuanto llega ese valor |
| `flow.collect { }` | Ejecuta la lambda con cada valor | Cuando se cancela la coroutine |

Con `collect` ya podríamos copiar cada lista nueva al estado de la pantalla. Funcionaría, pero tiene un problema: el `ViewModel` sigue vivo mientras la app está en segundo plano (el usuario ha pulsado Inicio), así que la app seguiría consultando la base de datos y actualizando una pantalla que **nadie ve**, gastando batería. Lo resolvemos en los dos apartados siguientes.

## 2.4 `StateFlow` y `stateIn`: el `Flow` dentro del `ViewModel`

### Qué es un `StateFlow`

Un `Flow` normal es como la radio: si no estás escuchando cuando dan la noticia, te la pierdes, y cada oyente que enciende la radio empieza desde cero. Un **`StateFlow`** es como el marcador de un partido: **siempre tiene un valor actual** (`.value`), y quien empieza a mirarlo recibe ese valor al momento, y después cada cambio.

Es la forma habitual en que un `ViewModel` ofrece datos a la interfaz: siempre hay algo que dibujar, aunque la base de datos todavía no haya respondido.

### Convertir un `Flow` en `StateFlow`: `stateIn`

```kotlin
class HistorialViewModel(application: Application) :
    AndroidViewModel(application) {

    private val historialRepository = HistorialRepository(
        (application as CafeteriaApp).database.pedidoDao()
    )

    val pedidos: StateFlow<List<ResumenPedido>?> =
        historialRepository.historial.stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = null
        )
}
```

`stateIn` recibe tres cosas:

1. **`scope`**: dónde se ejecuta la escucha del `Flow` de Room. Con `viewModelScope`, se cancela sola cuando se destruye el `ViewModel`.
2. **`initialValue`**: el valor que tiene el `StateFlow` antes de que Room entregue el primer resultado. Usamos `null` para distinguir dos situaciones que la pantalla debe tratar distinto: `null` es "todavía no sé nada" y una lista vacía es "no hay ningún pedido". Si el valor inicial fuera `emptyList()`, al abrir el historial aparecería un instante el mensaje "no hay pedidos" antes de la lista de verdad.
3. **`started`**: **cuándo** se escucha a Room.

| Valor de started | Escucha a Room... |
|---|---|
| `SharingStarted.Eagerly` | Desde que se crea el `ViewModel`, aunque nadie mire |
| `SharingStarted.Lazily` | Desde que alguien mira por primera vez, y ya no para |
| `SharingStarted.WhileSubscribed(5_000)` | **Mientras alguien mira**, y 5 segundos más después de que deje de mirar |

La opción recomendada para una pantalla es `WhileSubscribed`. Los 5 segundos de margen tienen una razón muy concreta: al **girar** el móvil, la `Activity` se destruye y se vuelve a crear (UD3, sesión 2), y durante ese momento la pantalla deja de mirar. Sin margen, Room dejaría de escuchar y volvería a lanzar la consulta tras cada giro. Con 5 segundos, un giro (que dura mucho menos) no corta la escucha; pero si el usuario sale de la app, a los 5 segundos se deja de consultar.

<div class="callout-info">
<strong>Concepto previo: <code>5_000</code>.</strong> En Kotlin se pueden escribir guiones bajos dentro de un número para leerlo mejor; el compilador los ignora. <code>5_000</code> es exactamente <code>5000</code>, y aquí son milisegundos: 5 segundos.
</div>

Además, `stateIn` **comparte** una única escucha de Room entre todos los que miran el `StateFlow`: aunque tres composables lean `pedidos`, la consulta se ejecuta una vez por cambio.

### ¿Y por qué no `mutableStateOf`, como en `PedidoViewModel`?

Las dos formas conviven sin problema en una app. `mutableStateOf` va bien para el estado que **cambia el propio `ViewModel`** en respuesta al usuario (el nombre que escribe, las cantidades). `StateFlow` va bien para datos que **cambian por su cuenta**, como los de una base de datos, porque se encadena directamente con el `Flow` de Room. En la cafetería, `PedidoViewModel` se queda como está.

## 2.5 Mostrarlo en Compose: `collectAsStateWithLifecycle`

Falta la última pieza: que la pantalla "mire" el `StateFlow` y se recomponga con cada valor nuevo.

```kotlin
val pedidos by historialViewModel.pedidos.collectAsStateWithLifecycle()
```

- Convierte el `StateFlow` en un **estado de Compose**, como el de `mutableStateOf` (UD3, sesión 2): cada valor nuevo provoca una recomposición. Por eso se usa con `by`, igual que `mutableStateOf`. Si Android Studio marca el `by` en rojo, falta el mismo `import` que con `mutableStateOf`: el de `getValue`.
- **"With lifecycle"** (*con ciclo de vida*): solo escucha mientras la `Activity` está **visible**, es decir, entre `onStart` y `onStop` (UD3, 2.2). Cuando el usuario pulsa Inicio, la `Activity` pasa por `onStop` y la pantalla deja de mirar; al volver, `onStart`, vuelve a mirar.

Juntando las dos piezas, esto es lo que ocurre cuando el usuario sale de la app con el historial abierto y vuelve más tarde:

```
Pulsa Inicio ── onStop ── la pantalla deja de mirar el StateFlow
                            │
                            └─ 5 s después: stateIn deja de escuchar a Room
                                            (no se consulta nada en segundo plano)

Vuelve a la app ── onStart ── la pantalla vuelve a mirar
                                │
                                ├─ recibe al momento el último valor guardado
                                └─ stateIn vuelve a escuchar a Room:
                                   nueva consulta y, si hay cambios, lista nueva
```

Y esta es la cadena completa, de la base de datos a la pantalla:

```
cafeteria.db  ── cambia la tabla pedidos ──► Room vuelve a consultar
   │
PedidoDao.observarTodos()          Flow<List<PedidoEntity>>
   │  .map { ... }                 (el repositorio traduce, apartado 2.9)
HistorialRepository.historial      Flow<List<ResumenPedido>>
   │  .stateIn(...)
HistorialViewModel.pedidos         StateFlow<List<ResumenPedido>?>
   │  .collectAsStateWithLifecycle()
PantallaHistorial                  estado de Compose ──► recomposición
```

### La dependencia

`collectAsStateWithLifecycle` está en la librería `lifecycle-runtime-compose`, de la misma familia que la de los `ViewModel` (UD3, 4.6), y usa su misma versión.

En `gradle/libs.versions.toml`, dentro de `[libraries]`:

```
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycleRuntimeKtx" }
```

En `app/build.gradle.kts`, dentro de `dependencies { }`:

```kotlin
implementation(libs.androidx.lifecycle.runtime.compose)
```

Y *Sync Now*.

<div class="callout-info">
En tutoriales encontraréis también <code>collectAsState()</code>, sin <em>WithLifecycle</em>. Viene con Compose y no necesita dependencia, pero sigue escuchando aunque la app esté en segundo plano. En apps Android, la recomendación oficial es la versión con ciclo de vida.
</div>

## Parte B · Relaciones entre tablas

## 2.6 Un pedido tiene varias líneas: la clave ajena

Hasta ahora, la tabla `pedidos` guardaba solo la cabecera de cada pedido (quién, cuándo, cuánto), porque una lista no cabe en una columna (apartado 1.11). Las líneas van en **otra tabla**, y cada línea apunta al pedido al que pertenece:

```
pedidos                      lineas_pedido
id  nombre  total            id  pedido_id  id_producto  cantidad
 1  Ana      3,90  ◄──┬─────  1      1           1           2
                      └─────  2      1           3           1
 2  Luis     2,80  ◄────────  3      2           2           1
```

Un pedido tiene **varias** líneas y cada línea es de **un** pedido: es una relación **uno a muchos** (1:N). La columna `pedido_id` guarda la clave primaria de otra tabla, y por eso se llama **clave ajena** (*foreign key*). Para ver las líneas del pedido 1 bastaría con:

```sql
SELECT * FROM lineas_pedido WHERE pedido_id = 1
```

Declarar la clave ajena en la base de datos, y no solo "usarla", tiene una ventaja: SQLite **vigila** que los datos sean coherentes. No deja guardar una línea de un pedido que no existe, y obliga a decidir qué pasa con las líneas cuando se borra su pedido. Hay dos opciones principales:

- **`ForeignKey.NO_ACTION`** (la opción por defecto): SQLite rechaza el borrado si el pedido tiene líneas.
- **`ForeignKey.CASCADE`** (en cascada): al borrar el pedido se borran también todas sus líneas.

Para la cafetería, una línea sin su pedido no tiene sentido: borrado **en cascada**.

### La entidad de las líneas

```kotlin
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

- **`foreignKeys`** es una lista (entre corchetes, como `entities` en `@Database`) de claves ajenas. Cada una es un `ForeignKey(...)`, que también es una anotación: cuando una anotación va **dentro** de otra, se escribe sin `@`.
- **`entity`** es la tabla a la que se apunta (la tabla "padre"). **`parentColumns`** y **`childColumns`** son las columnas que se relacionan, con su nombre de **columna**: `pedido_id`, no `pedidoId`.
- **`onDelete = ForeignKey.CASCADE`**: al borrar un pedido, SQLite borra sus líneas.
- **`indices = [Index("pedido_id")]`** crea un **índice** sobre la columna. Un índice es como el índice de un libro: sin él, para encontrar las líneas del pedido 7, SQLite tiene que leer la tabla entera, fila a fila. Como esa búsqueda se hace cada vez que se borra o se consulta un pedido, Room **avisa al compilar** si una clave ajena no tiene índice.
- La línea guarda el id del producto y el precio del momento, igual que `LineaTicket` (apartado 1.4).

SQLite trae la comprobación de claves ajenas desactivada por defecto. Room la activa al abrir la base de datos cuando alguna entidad tiene `foreignKeys`, así que no hay que hacer nada más.

### La base de datos con dos tablas

```kotlin
@Database(
    entities = [PedidoEntity::class, LineaPedidoEntity::class],
    version = 1,
    exportSchema = false
)
abstract class CafeteriaDatabase : RoomDatabase() {
    abstract fun pedidoDao(): PedidoDao
}
```

Hemos cambiado el esquema, así que la base de datos del emulador ya no coincide: mientras desarrollamos, **desinstalamos la app** y se crea una nueva (apartado 1.15). Seguimos en la versión 1; en la sesión 4 aprenderemos a cambiar de versión sin perder datos.

## 2.7 Guardar un pedido con sus líneas: transacciones

Para guardar un pedido completo hay dos pasos, y el orden importa: primero el pedido, porque hasta que no se inserta no tiene `id`, y después sus líneas con ese `id` en `pedido_id`.

¿Y si la app se cierra justo entre los dos pasos? Quedaría un pedido **sin líneas** en la base de datos. Para evitarlo existen las **transacciones**: un grupo de operaciones que se hace **entero o no se hace**. Si algo falla a mitad, la base de datos deshace lo que se hubiera hecho del grupo. Es la misma idea que el `edit` de DataStore (UD3, 6.5), donde todo lo que se cambiaba dentro de una llamada se guardaba junto.

En Room, una función del DAO marcada con **`@Transaction`** se ejecuta entera dentro de una transacción:

```kotlin
@Dao
interface PedidoDao {

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

    // ... el resto de funciones
}
```

- `insertarConLineas` es un método **con cuerpo** dentro de una interfaz (UD2, 1.5). Room no genera su código, como con los demás: la clase `PedidoDao_Impl` la "envuelve" en una transacción y llama a vuestro código dentro.
- `@Insert` acepta también una **lista**: inserta una fila por elemento.
- Las líneas llegan con un `pedidoId` provisional (`0`), porque quien las crea todavía no conoce el `id`. Con `copy` (UD3, 4.7) se crea cada línea de nuevo con el `id` de verdad.
- Los `Flow` que dependen de estas tablas reciben **un solo** aviso, al terminar la transacción, no uno por cada inserción.

## 2.8 Leer un pedido con sus líneas: `@Embedded` y `@Relation`

Para mostrar el detalle de un pedido necesitamos su cabecera **y** sus líneas. Room lo resuelve con una clase de resultado, que **no** es una entidad (no tiene `@Entity` ni crea ninguna tabla):

```kotlin
data class PedidoConLineas(
    @Embedded
    val pedido: PedidoEntity,
    @Relation(parentColumn = "id", entityColumn = "pedido_id")
    val lineas: List<LineaPedidoEntity>
)
```

- **`@Embedded`** (*incrustado*): las columnas de la consulta (`id`, `nombre`, `para_llevar`...) se usan para construir un `PedidoEntity`.
- **`@Relation`**: para cada pedido, Room busca en la tabla de `LineaPedidoEntity` las filas cuya columna `pedido_id` (**`entityColumn`**) sea igual a la columna `id` del pedido (**`parentColumn`**), y llena la lista.

Y en el DAO:

```kotlin
@Transaction
@Query("SELECT * FROM pedidos WHERE id = :id")
fun observarConLineas(id: Long): Flow<PedidoConLineas?>
```

Por dentro, Room hace **dos** consultas: la vuestra, sobre `pedidos`, y otra sobre `lineas_pedido` con los ids que haya devuelto la primera (en SQL, `pedido_id IN (1, 2, 5)` significa "cuyo `pedido_id` sea alguno de esta lista"). Después junta los resultados:

```
1. SELECT * FROM pedidos WHERE id = 1
       → PedidoEntity(id = 1, nombre = "Ana", ...)
2. SELECT * FROM lineas_pedido WHERE pedido_id IN (1)
       → [línea 1, línea 2]
3. PedidoConLineas(pedido = ..., lineas = [línea 1, línea 2])
```

- **`@Transaction`** hace que las dos consultas vean los mismos datos: si alguien borrara las líneas justo entre una y otra, podríamos recibir un pedido sin líneas. Si se olvida, Room avisa al compilar.
- El `Flow` se vuelve a emitir cuando cambia **cualquiera** de las dos tablas.
- El tipo es `PedidoConLineas?` porque puede que no exista ningún pedido con ese id (por ejemplo, si se acaba de borrar).

<div class="callout-info">
Si conocéis SQL, la otra forma de juntar dos tablas es un <code>JOIN</code>, y Room también lo admite. <code>@Relation</code> es la forma más habitual en Room cuando el resultado es "un objeto con su lista de hijos", porque devuelve directamente esa estructura.
</div>

### Borrar un pedido

```kotlin
@Query("DELETE FROM pedidos WHERE id = :id")
suspend fun borrarPorId(id: Long)
```

Gracias al `CASCADE`, al borrar el pedido SQLite borra también sus líneas. Lo mismo ocurre con el `DELETE FROM pedidos` de "Olvidar mis datos": vacía las dos tablas.

Existe también la anotación `@Delete`, que recibe una entidad completa y la borra por su clave primaria. Cuando solo se tiene el id, como aquí, es más cómodo un `@Query`.

## Parte C · Las pantallas del historial

## 2.9 El repositorio traduce

Como en el bloque 1, el resto de la app no ve las entidades: el repositorio las traduce a clases del modelo de la app. Para el historial basta con un resumen de cada pedido:

```kotlin
data class ResumenPedido(
    val id: Long,
    val nombre: String,
    val fechaHora: Long,
    val unidades: Int,
    val total: Double
)
```

Y para el detalle, reutilizamos `Ticket`, que ya tiene exactamente la forma de un pedido con sus líneas.

```kotlin
class HistorialRepository(private val pedidoDao: PedidoDao) {

    val historial: Flow<List<ResumenPedido>> =
        pedidoDao.observarTodos().map { pedidos ->
            Log.d(TAG, "Consulta del historial: ${pedidos.size} pedidos")
            pedidos.map { it.aResumen() }
        }

    fun observarPedido(id: Long): Flow<Ticket?> =
        pedidoDao.observarConLineas(id).map { it?.aTicket() }

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

    // contarPedidos, contarTodos y borrarTodo: sin cambios
}

private fun PedidoEntity.aResumen() = ResumenPedido(
    id = id, nombre = nombre, fechaHora = fechaHora,
    unidades = unidades, total = total
)

private fun PedidoConLineas.aTicket() = Ticket(
    nombre = pedido.nombre,
    paraLlevar = pedido.paraLlevar,
    fechaHora = pedido.fechaHora,
    lineas = lineas.map { linea ->
        LineaTicket(linea.idProducto, linea.cantidad, linea.precioUnitario)
    },
    total = pedido.total
)
```

<div class="callout-warning">
<strong>Dos <code>map</code> distintos en la misma línea.</strong> En <code>historial</code>, el primer <code>map</code> es el del <code>Flow</code> (UD3, 6.2): se ejecuta <strong>cada vez que Room entrega una lista nueva</strong> y transforma esa lista entera. El segundo, dentro, es el <code>map</code> de las listas (UD2, 3.7): transforma <strong>cada pedido</strong> de esa lista en un <code>ResumenPedido</code>. Leedlo así: "cada vez que llegue una lista de pedidos, conviértela en una lista de resúmenes".
</div>

- `aResumen` y `aTicket` son **funciones de extensión** (UD2, 3.1) privadas: solo existen dentro de este fichero, que es el único que conoce las entidades.
- En `observarPedido`, `it?.aTicket()` deja pasar el `null` si el pedido no existe.
- El `Log.d` dentro del `map` del `Flow` sirve para **ver en Logcat cada vez que Room ejecuta la consulta**. Lo usaremos para comprobar que `WhileSubscribed` deja de consultar (apartado 2.12).
- `registrar` sigue recibiendo un `Ticket` y devolviendo el id. **`PedidoViewModel` no cambia**: ni se entera de que ahora el pedido se guarda en dos tablas. Es la ventaja de tener los datos detrás de un repositorio.

<div class="callout-info">
<strong>¿Y el fichero del último ticket?</strong> El ticket más reciente también se podría obtener con una consulta a la base de datos, pero lo mantenemos en su fichero JSON: la cafetería sigue trabajando con dos almacenes distintos, cada uno con su repositorio, que es justo lo que pide el criterio f).
</div>

## 2.10 Un `ViewModel` por pantalla

En la UD3 (5.8) la cafetería usaba un único `PedidoViewModel` creado fuera del `NavHost`, porque todas las pantallas compartían **el mismo pedido en memoria**. Ya avisamos entonces de que, en apps más grandes, lo normal es un `ViewModel` por pantalla, con los datos compartidos en una capa común. Las dos pantallas nuevas son ese caso: sus datos no están en memoria, están en la base de datos, que ya es común a toda la app.

Por eso cada una tiene su propio `ViewModel`, y se pide con `viewModel()` **dentro** de su `composable<...>`, de modo que vive en el almacén de su entrada de la pila (apartado 1.12) y se destruye al salir de esa pantalla:

- **`HistorialViewModel`**: el del apartado 2.4, con la lista de pedidos.
- **`DetallePedidoViewModel`**: el pedido que se está viendo y la acción de borrarlo.

### Leer el id de la ruta desde el `ViewModel`

El detalle necesita saber **qué** pedido mostrar, y ese dato llega en la ruta, `DetallePedido(idPedido = 3)`. En la UD3 leíamos los argumentos en el `NavHost`, con `entrada.toRoute()`. También se pueden leer desde el propio `ViewModel`:

```kotlin
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

    fun borrar(alTerminar: () -> Unit) {
        viewModelScope.launch {
            historialRepository.borrar(idPedido)
            alTerminar()
        }
    }
}
```

- Cada entrada de la pila guarda los argumentos de su ruta en el **`SavedStateHandle`** (el pequeño almacén de la UD3, 4.9) de los `ViewModel` que se crean en ella. `toRoute<DetallePedido>()` los convierte de nuevo en el objeto de la ruta, igual que `entrada.toRoute()`.
- `viewModel()` ve que el constructor pide una `Application` y un `SavedStateHandle` y le pasa los dos, sin que tengamos que hacer nada.
- Como el `SavedStateHandle` se guarda en el `Bundle`, el id **sobrevive** a que Android elimine la app en segundo plano (UD3, 5.9): al volver, el `ViewModel` nuevo vuelve a leer el id y Room vuelve a cargar el pedido.
- **`borrar` recibe una lambda** con lo que hay que hacer al terminar (volver atrás). Así la pantalla no se cierra hasta que el pedido se ha borrado de verdad. Si se volviera atrás primero, la entrada de la pila se destruiría y, con ella, su `ViewModel` y su `viewModelScope`, que podría cancelarse antes de borrar.

### Varios repositorios, una sola base de datos

```
PedidoViewModel ──────── HistorialRepository ─┐
HistorialViewModel ───── HistorialRepository ─┼─► PedidoDao ─► cafeteria.db
DetallePedidoViewModel ─ HistorialRepository ─┘   (la única database,
                                                   la de CafeteriaApp)
```

Cada `ViewModel` crea su propio `HistorialRepository`, pero todos usan la **misma** base de datos de `CafeteriaApp`. Por eso, cuando el detalle borra un pedido, el `Flow` del historial se entera y la lista se actualiza sola. Crear un repositorio en cada `ViewModel` funciona, pero repite código; en la sesión 4 lo ordenaremos para que haya uno solo.

## 2.11 Las pantallas y la navegación

### Las rutas nuevas

```kotlin
@Serializable
data object Historial

@Serializable
data class DetallePedido(val idPedido: Long)
```

### La pantalla del historial

```kotlin
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
```

- `historialViewModel: HistorialViewModel = viewModel()`: aquí **sí** se pide con valor por defecto, porque queremos uno por pantalla (UD3, 5.8).
- `pedidos` se copia a una variable local: con una propiedad declarada con `by`, el compilador no puede asegurar que no cambie entre la comprobación y el uso, y no deja tratarla como no nula después del `if`.
- `ContenidoHistorial` es stateless (recibe la lista y una lambda) y es la que lleva la preview. Si la lista está vacía muestra el texto "Todavía no hay pedidos guardados"; si no, una `LazyColumn` con `items(pedidos, key = { it.id })` y una tarjeta por pedido.

### En el `NavHost`

```kotlin
composable<Historial> {
    PantallaHistorial(
        onVolver = { navController.navigateUp() },
        onPedidoClick = { id -> navController.navigate(DetallePedido(id)) }
    )
}

composable<DetallePedido> {
    PantallaDetallePedido(
        onVolver = { navController.navigateUp() }
    )
}
```

Fijaos en que `composable<DetallePedido>` ya **no** lee la ruta: lo hace su `ViewModel`. Y en que estas dos pantallas no reciben `pedidoViewModel`: no lo necesitan.

En la carta, una opción nueva del menú, "Historial de pedidos", navega a `Historial`.

## 2.12 Comprobar en el emulador

**Las dos tablas.** En el *Database Inspector* aparecen `pedidos` y `lineas_pedido`. Una consulta propia como `SELECT * FROM lineas_pedido WHERE pedido_id = 1` muestra las líneas de un pedido.

**La interfaz que se actualiza sola.** Con la app en el historial, en el *Database Inspector*, desmarcad *Live updates* (con esa casilla marcada, la tabla es de solo lectura), haced doble clic en el nombre de un pedido, escribid otro y pulsad Intro. La documentación oficial lo explica así: *"Si tu app usa Room y la IU observa la base de datos, como con `LiveData` o `Flow`, cualquier cambio que realices en los datos se verá de inmediato en la app en ejecución"*. El historial cambia sin tocar la app.

**Cuándo se consulta y cuándo no.** Filtrad Logcat por la etiqueta del repositorio y fijaos en el mensaje *"Consulta del historial"*:

| Con el historial abierto... | ¿Nueva consulta en Logcat? | Por qué |
|---|---|---|
| Se abre la pantalla | Sí | Alguien empieza a mirar: `stateIn` escucha a Room |
| Se gira el móvil | No | El giro dura menos de 5 s: `WhileSubscribed` no corta |
| Inicio y se vuelve antes de 5 s | No | Igual que el giro |
| Inicio, se esperan más de 5 s y se vuelve | Sí | A los 5 s se dejó de escuchar; al volver, consulta nueva |
| Se borra un pedido desde el detalle | Sí | Ha cambiado la tabla `pedidos` |

## Tabla resumen del bloque

| Necesito... | Uso |
|---|---|
| Una consulta que avise de los cambios | `@Query(...) fun observarX(): Flow<...>` (sin `suspend`) |
| Recibir todos los valores de un `Flow` | `flow.collect { valor -> ... }` dentro de una coroutine (no termina) |
| Ofrecer datos de la base de datos desde el `ViewModel` | `flow.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), valorInicial)` |
| Mostrar un `StateFlow` en un composable | `val x by viewModel.x.collectAsStateWithLifecycle()` (dependencia `lifecycle-runtime-compose`) |
| Relacionar dos tablas | `@Entity(foreignKeys = [ForeignKey(entity, parentColumns, childColumns, onDelete)], indices = [Index("columna")])` |
| Borrar los hijos con el padre | `onDelete = ForeignKey.CASCADE` |
| Varias operaciones "todo o nada" | Función con cuerpo en el DAO marcada con `@Transaction` |
| Insertar varias filas | `@Insert suspend fun insertar(lista: List<Entidad>)` |
| Un objeto con su lista de hijos | `data class` con `@Embedded` y `@Relation(parentColumn, entityColumn)`, consulta con `@Transaction` |
| Borrar por id | `@Query("DELETE FROM tabla WHERE id = :id")` |
| Un `ViewModel` propio para una pantalla | `viewModel()` como valor por defecto, dentro de su `composable<...>` |
| Leer los argumentos de la ruta en el `ViewModel` | Constructor con `SavedStateHandle` y `savedStateHandle.toRoute<Ruta>()` |

## Para la próxima sesión

La cafetería ya guarda pedidos completos y los muestra siempre al día. Pero tiene tres cosas que mejorar antes de llegar a una app "de verdad": cada `ViewModel` construye su propio repositorio repitiendo código; si mañana añadimos una columna, los usuarios que ya tengan la app instalada perderían sus datos al desinstalar; y nadie ha documentado cómo está organizada la capa de datos. En la sesión 4 crearemos un **contenedor** con un único repositorio para toda la app, aprenderemos a hacer **migraciones** entre versiones de la base de datos y documentaremos todo lo que hemos construido.
