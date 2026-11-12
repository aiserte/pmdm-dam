---
title: "Almacenamiento local: ficheros JSON y bases de datos con Room"
unit: UD4
order: 1
duration: "3h"
---

## Objetivos

- Conocer los lugares donde una app Android puede guardar datos y elegir el mecanismo adecuado para cada tipo de dato: preferencias, fichero o base de datos.
- Leer y escribir ficheros en la carpeta privada de la app sin bloquear la interfaz, y guardar objetos completos en formato JSON.
- Entender qué es SQLite y escribir las consultas SQL básicas que necesita una app.
- Crear una base de datos local con **Room**: entidad, DAO y base de datos.
- Saber **dónde** debe crearse la base de datos para que exista una sola en toda la app, y por qué.
- Comprobar en el emulador, con *Device Explorer* y *Database Inspector*, qué se guarda y dónde.

<div class="callout-info">
Este bloque ocupa las dos primeras sesiones de la unidad (3h). Trabaja sobre todo el criterio <strong>f)</strong> del RA2: la app se conecta con dos almacenes de datos (un fichero JSON y una base de datos SQLite) y garantiza que la información persiste. También el <strong>g)</strong>, porque comprobaremos en el emulador qué se guarda, cuándo se pierde y qué hace la app cuando algo falla.
</div>

## Antes de empezar: qué necesitamos saber

Este bloque se apoya en piezas de unidades anteriores y añade otras nuevas. Repasad las de la primera tabla si alguna no os suena: el bloque **no** las vuelve a explicar.

| Ya visto | Dónde | Para qué lo usamos hoy |
|---|---|---|
| Hilo principal y ANR | UD2, sesión 5 (5.1); UD3, sesión 6 (6.2) | Por qué el disco no se toca desde el hilo que dibuja la interfaz |
| `suspend`, `launch` | UD2, sesión 5 (5.3 y 5.4); UD3, sesión 3 | Las operaciones de disco y de base de datos son funciones `suspend` |
| `viewModelScope` | UD3, sesión 6 (6.7) | Lanzar el guardado y la carga desde el `ViewModel` |
| Qué es un `Context` (`Activity` y `Application`) | UD3, sesión 6 (6.4) | Cada almacén se abre a partir de un `Context` |
| `Application` y `AndroidViewModel` | UD3, sesión 6 (6.7) | El `ViewModel` necesita la `Application` para llegar a los almacenes |
| Repositorio | UD3, sesión 6 (6.6) | Cada almacén tendrá su repositorio |
| Almacén de `ViewModel` de la `Activity` y de cada pantalla de la pila | UD3, sesiones 4 (4.5) y 5 (5.8) | Decidir dónde vive la base de datos (apartado 1.12) |
| `Bundle` | UD3, sesión 2 (2.9) | Comparar qué sobrevive en cada sitio |
| Plugin `@Serializable` | UD3, sesión 5 (5.3) | Convertir objetos a JSON sin tocar Gradle |
| Interfaces y clases `abstract` | UD2, sesión 1 | Room nos pide una interfaz y una clase abstracta |
| `object` y `as?` | UD3, práctica de la sesión 1; UD2, sesión 2 (2.5) | Objetos únicos y conversiones de tipo |
| `init` y orden de las propiedades | UD3, sesiones 4 (4.7) y 6 (6.7) | Crear los repositorios antes de usarlos |

| Nuevo hoy | Apartado |
|---|---|
| Cuánto vive cada `Context` y por qué cada app tiene su carpeta privada | 1.1 |
| *Dispatchers* y `withContext` | 1.3 |
| JSON y serialización de objetos | 1.4 |
| `try` como expresión y `return@etiqueta` | 1.5 |
| SQLite y el SQL mínimo | 1.7 |
| Qué es una anotación y quién la procesa (KSP, generación de código) | 1.9 |
| Referencias a clases (`::class.java`) | 1.11 |
| Una `Application` propia, `by lazy` y cuánto vive cada pieza de la app | 1.12 |

## Parte A · Almacenamiento y ficheros

## 1.1 Dónde puede guardar datos una app

### Repaso: el `Context` y cuánto vive cada uno

En la UD3 (sesión 6, apartado 6.4) vimos que el **`Context`** es la **puerta de entrada de la app al sistema**: a través de él se piden las carpetas, los recursos de `strings.xml` y los servicios del móvil. Es una clase abstracta de Android, y las dos clases que heredan de ella y usamos son la `Activity` y la `Application`. Hoy vamos a abrir ficheros y bases de datos a partir de un `Context`, así que conviene tener claras sus diferencias:

| Clase | Cuántas hay | Cuánto vive | Para qué usarla |
|---|---|---|---|
| `Activity` | Una por pantalla abierta (en nuestra app, una) | Se destruye y se recrea, por ejemplo, al girar | Lo que tiene que ver con la ventana: mostrar diálogos del sistema, pedir permisos (UD5) |
| `Application` | **Una por proceso** | Desde que arranca la app hasta que el proceso termina | Lo que debe existir mientras exista la app: ficheros, base de datos, DataStore |

Por eso un `ViewModel` o una base de datos no deben guardar nunca una `Activity` (vive menos que ellos: quedaría una referencia a una pantalla destruida, una **fuga de memoria**), pero sí pueden guardar la `Application`.

### Concepto previo: cada app tiene su carpeta privada

Android está construido sobre Linux y, al instalar una app, le crea un **usuario de Linux propio**. Cada app tiene su carpeta, `/data/data/<paquete>/`, de la que es la única propietaria: ninguna otra app puede leerla. A este aislamiento se le llama *sandbox* (caja de arena), y es la razón por la que en la UD3 ya encontramos el fichero de DataStore en esa carpeta y por la que no hace falta ningún permiso para escribir en ella.

### Las tres zonas de almacenamiento

| Zona | Cómo se obtiene | ¿Permisos? | ¿Visible para otras apps o el usuario? | ¿Se borra al desinstalar la app? |
|---|---|---|---|---|
| **Interna privada** | `context.filesDir` (datos) y `context.cacheDir` (caché) | No | No | Sí |
| **Externa propia de la app** | `context.getExternalFilesDir(null)` | No | Sí (explorador de archivos, PC) | Sí |
| **Compartida** (fotos, música, descargas...) | `MediaStore` o el selector de documentos del sistema | Depende del tipo de fichero | Sí | No |

- **`filesDir`** es `/data/data/<paquete>/files/`, la carpeta que vimos con *Device Explorer*. Es la opción por defecto para casi todo.
- **`cacheDir`** es para datos que se pueden volver a generar o descargar. Android puede **vaciarla por su cuenta** si el móvil se queda sin espacio, así que nunca debe guardarse ahí nada importante.
- La **zona compartida** se usa cuando el fichero pertenece al usuario y no a la app: una foto que ha hecho, un PDF que quiere guardar en *Descargas*. La trabajaremos en la UD6 (contenidos multimedia).

<div class="callout-info">
<strong>Almacenamiento con ámbito</strong> (<em>scoped storage</em>). Desde Android 10, una app ya no puede leer y escribir libremente en todo el almacenamiento del móvil, como ocurría antes. Para acceder a ficheros de otras apps o del usuario debe pasar por <code>MediaStore</code> o por el selector del sistema. Por eso en tutoriales antiguos veréis el permiso <code>WRITE_EXTERNAL_STORAGE</code>, que hoy ya no sirve para eso. Para las carpetas propias de la app (<code>filesDir</code>, <code>cacheDir</code> y <code>getExternalFilesDir</code>) no se necesita ningún permiso.
</div>

**Copia de seguridad automática.** Por defecto, Android hace una copia de seguridad en la cuenta de Google del usuario de los datos privados de la app (excepto la caché) y la restaura si reinstala la app en otro móvil. Se controla con el atributo `android:allowBackup` del manifiesto, que la plantilla de Android Studio deja en `true`. Si la app guarda algo que no debería salir del móvil, hay que excluirlo de la copia.

## 1.2 Qué mecanismo usar para cada dato

Con DataStore ya sabemos guardar **preferencias**. En este bloque añadimos los **ficheros** y la **base de datos**. Las tres opciones guardan en la zona interna privada; lo que cambia es la forma de los datos y cómo se van a leer:

| Necesito guardar... | Ejemplo en la cafetería | Mecanismo |
|---|---|---|
| Pocos valores sueltos, cada uno con un nombre | Nombre del cliente, "para llevar" | **DataStore** (UD3) |
| **Un** objeto o documento completo, que se lee y se escribe entero | El ticket del último pedido | **Fichero** JSON (Parte A) |
| **Muchos** registros con la misma forma, que se consultan, filtran y relacionan | El historial de todos los pedidos | **Base de datos** con Room (Parte B) |

Un fichero se lee entero: si solo hay un ticket, perfecto. Si hubiera mil tickets y quisiéramos "cuántos pedidos ha hecho Ana", habría que leer el fichero completo y contar a mano cada vez. Para eso existen las bases de datos.

## 1.3 Leer y escribir un fichero de texto

Kotlin añade funciones muy cómodas a la clase `File` de Java (la misma que usasteis en Programación de primero):

```kotlin
val fichero = File(context.filesDir, "notas.txt")   // carpeta + nombre

fichero.writeText("Hola, cafetería")   // crea o SOBRESCRIBE el fichero
val texto = fichero.readText()           // lee todo su contenido como String
fichero.exists()                         // ¿existe?
fichero.delete()                         // lo borra (true si lo ha borrado)
```

`File(carpeta, nombre)` no crea nada en el disco: solo representa una ruta. El fichero se crea al escribir en él. Si se intenta **leer** un fichero que no existe, `readText()` lanza una `FileNotFoundException`, así que antes hay que comprobar `exists()`.

### Fuera del hilo principal: `Dispatchers.IO`

Leer o escribir en el disco **tarda**, y no debe hacerse en el hilo principal. DataStore ya lo resolvía por dentro; con `File` nos toca a nosotros.

Las coroutines se ejecutan en un **dispatcher** (*repartidor*), que decide en qué hilos corren:

| Dispatcher | Para qué |
|---|---|
| `Dispatchers.Main` | La interfaz. Es donde corren por defecto las coroutines de `viewModelScope` y de los composables |
| `Dispatchers.IO` | Operaciones que **esperan** a algo externo: disco, red, base de datos |
| `Dispatchers.Default` | Cálculos pesados de CPU (ordenar una lista enorme, procesar una imagen) |

Para ejecutar un trozo de código en otro dispatcher y volver con el resultado se usa `withContext`:

```kotlin
suspend fun leerNotas(fichero: File): String = withContext(Dispatchers.IO) {
    fichero.readText()       // esto se ejecuta en un hilo de IO
}                            // al terminar, el resultado vuelve a quien llamó
```

`withContext` es `suspend`: quien llama espera el resultado **sin bloquear** su hilo. La regla práctica es que una función `suspend` que toca el disco debe ser **segura para el hilo principal** (*main-safe*): que se pueda llamar desde `viewModelScope.launch` sin preocuparse de nada más. Por eso el `withContext` va dentro de la función que accede al fichero, no en quien la llama.

## 1.4 Guardar objetos: JSON con `kotlinx.serialization`

Un fichero de texto guarda texto, pero queremos guardar un **objeto**: un ticket con su fecha, su total y una lista de líneas. Hay que convertirlo a texto al guardarlo (**serializar**) y reconstruirlo al leerlo (**deserializar**). El formato más usado para eso es **JSON**:

```json
{
    "nombre": "Ana",
    "paraLlevar": true,
    "fechaHora": 1761742800000,
    "lineas": [
        { "idProducto": 1, "cantidad": 2, "precioUnitario": 1.2 },
        { "idProducto": 3, "cantidad": 1, "precioUnitario": 1.5 }
    ],
    "total": 3.9
}
```

Un objeto JSON va entre llaves, con pares `"nombre": valor`; una lista va entre corchetes. Es el mismo formato en que nos llegarán los datos de los servicios web en la UD5.

### Ya tenéis todo lo necesario

En la UD3, para las rutas de Navigation, añadimos el **plugin** de serialización y la librería `kotlinx-serialization-json`. Entonces `@Serializable` servía para que Navigation guardara la ruta en el `Bundle`; ahora usamos la misma herramienta para convertir nuestros propios objetos a JSON. No hay que tocar Gradle:

```kotlin
@Serializable
data class LineaTicket(
    val idProducto: Int,
    val cantidad: Int,
    val precioUnitario: Double
)

@Serializable
data class Ticket(
    val nombre: String,
    val paraLlevar: Boolean,
    val fechaHora: Long,              // milisegundos desde el 1/1/1970
    val lineas: List<LineaTicket>,
    val total: Double
)
```

Todas las propiedades de una clase `@Serializable` deben ser de tipos que la librería sepa convertir: tipos básicos, `String`, listas, mapas y otras clases `@Serializable`. Si una propiedad es de otro tipo (por ejemplo, `Producto`, que no es `@Serializable`), el proyecto no compila.

La fecha se guarda como un `Long`: los **milisegundos transcurridos desde el 1 de enero de 1970**, que es lo que devuelve `System.currentTimeMillis()`. Es la forma estándar de guardar un instante: un número, fácil de comparar y ordenar, que se convierte en fecha legible solo al mostrarlo.

### De objeto a JSON y de vuelta

La conversión la hace un objeto `Json`, que se configura una vez:

```kotlin
private val json = Json {
    prettyPrint = true          // con saltos de línea: legible al abrirlo
    ignoreUnknownKeys = true    // ignora campos que la clase no conoce
}

val texto: String = json.encodeToString(ticket)          // Ticket -> JSON
val ticket: Ticket = json.decodeFromString<Ticket>(texto) // JSON -> Ticket
```

`ignoreUnknownKeys = true` es importante para una app que se actualiza. Si en la versión 2 quitamos un campo de `Ticket`, los ficheros que guardó la versión 1 todavía lo tienen; sin esta opción, leerlos daría error. En el sentido contrario, si en la versión 2 **añadimos** un campo, hay que darle un **valor por defecto** (`val avisar: Boolean = true`) para poder leer los ficheros antiguos, que no lo traen.

<div class="callout-warning">
<strong>Nunca guardéis en disco un identificador de recurso</strong> (<code>R.string.producto_cafe</code>, <code>R.drawable.…</code>). Son números que genera Android Studio al compilar y pueden cambiar en la siguiente versión de la app: un ticket guardado hoy podría mostrar mañana otro texto, o hacer que la app falle. Por eso <code>LineaTicket</code> guarda el <strong>id del producto</strong> (que lo decidimos nosotros y no cambia). Al mostrar el ticket, el nombre se busca a partir del id.
</div>

Fijaos también en que la línea guarda el **precio unitario** en el momento de la compra. Si mañana el café sube a 1,30 €, el ticket de hoy debe seguir diciendo 1,20 €. Un ticket es un registro de lo que pasó, no una vista de la carta actual.

## 1.5 Cuando el fichero no es lo que esperábamos

Al leer un fichero hay dos situaciones normales que no deben cerrar la app:

1. **El fichero no existe**: es la primera vez que se usa la app, o el usuario ha borrado sus datos. No es un error, simplemente no hay nada guardado.
2. **El contenido está dañado**: la escritura se cortó porque el móvil se apagó a mitad, o lo guardó una versión de la app con otro formato. `decodeFromString` lanza una `SerializationException`.

Además, cualquier operación de disco puede lanzar una `IOException` (disco lleno, fichero bloqueado...). Lo razonable en los tres casos es lo mismo: registrar el problema en Logcat y continuar como si no hubiera nada guardado.

```kotlin
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
```

- `return@withContext null` termina **el bloque entre llaves** de `withContext`, el trabajo que se hace en el hilo de IO, y hace que ese bloque devuelva `null`. Después, la ejecución vuelve al hilo desde el que se llamó a `cargar()` (en la cafetería, el hilo principal, porque se llama desde `viewModelScope.launch`) y sigue con lo que haya detrás del `withContext`. Aquí detrás no hay nada: todo el cuerpo de `cargar()` es ese `withContext` (fíjate en el `=` después de `Ticket?`), así que la función termina y devuelve ese mismo `null`. La etiqueta `@withContext` es obligatoria: un `return` sin etiqueta dentro de esta lambda no compila.
- En Kotlin, `try` es una **expresión**: su valor es el de la última línea del bloque que se ejecute. Por eso la función devuelve el `Ticket` leído o el `null` de un `catch`.
- El tipo de retorno es `Ticket?`: quien llama está **obligado** por el compilador a tratar el caso "no hay ticket".

<div class="callout-info">
<strong>En una app profesional</strong> no se sobrescribe un fichero importante directamente: se escribe en un fichero temporal y, cuando se ha escrito entero, se renombra encima del original. Así, si el móvil se apaga a mitad, el fichero bueno sigue intacto. Android ofrece la clase <code>AtomicFile</code> para hacerlo. DataStore y Room ya hacen algo equivalente por dentro; con ficheros propios es responsabilidad nuestra.
</div>

## 1.6 Un repositorio para el ticket

Igual que con las preferencias, todo lo relativo al fichero se esconde en un **repositorio**. Fuera de él nadie sabe cómo se llama el fichero ni que se guarda en JSON:

```kotlin
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
        // ... como en el apartado 1.5
    }

    suspend fun borrar() {
        withContext(Dispatchers.IO) {
            fichero.delete()
        }
    }
}
```

El repositorio recibe la **carpeta** (`File`), no un `Context`: no necesita nada más de Android. Desde el `ViewModel` se crea con `TicketRepository(application.filesDir)`.

Para comprobarlo, con *View › Tool Windows › Device Explorer* se navega a `/data/data/<paquete>/files/`. Allí aparecen la carpeta `datastore/` de la UD3 y el nuevo `ultimo_ticket.json`, que se puede **abrir y leer** directamente porque es texto. Para probar qué hace la app con un fichero dañado, desde el *Terminal* de Android Studio se puede escribir en él con `adb`:

```
adb shell run-as com.iesejemplo.cafeteria \
    sh -c "echo '{roto' > files/ultimo_ticket.json"
```

`run-as` ejecuta el comando como si fuera la propia app, la única que puede escribir en su carpeta privada. Solo funciona con apps en modo *debug*, como las que se lanzan desde Android Studio.

## Parte B · Bases de datos con Room

## 1.7 SQLite y el SQL mínimo

Cada móvil Android lleva incorporado **SQLite**, un motor de base de datos relacional muy ligero. No es un servidor al que la app se conecta por red, como MySQL u Oracle: es una librería que trabaja directamente sobre **un fichero** guardado en la carpeta privada de la app. Cada app tiene sus propias bases de datos, que nadie más puede abrir.

Una base de datos relacional guarda los datos en **tablas**. Cada fila es un registro y cada columna, un dato de ese registro. El historial de pedidos de la cafetería sería así:

| id | nombre | para_llevar | fecha_hora | unidades | total |
|---|---|---|---|---|---|
| 1 | Ana | 1 | 1761742800000 | 3 | 3.9 |
| 2 | Luis | 0 | 1761746400000 | 1 | 2.8 |
| 3 | Ana | 0 | 1761829200000 | 2 | 2.3 |

- La columna `id` es la **clave primaria**: identifica cada fila sin ambigüedad. Dejamos que la base de datos la numere sola (*autoincremento*).
- SQLite no tiene tipo booleano: guarda `true` como `1` y `false` como `0`.

Para trabajar con las tablas se usa el lenguaje **SQL**. Room escribe por nosotros la creación de las tablas, pero las **consultas** las escribiremos a mano, así que necesitamos estas cinco instrucciones:

```sql
-- Insertar una fila (el id lo pone la base de datos)
INSERT INTO pedidos (nombre, para_llevar, fecha_hora, unidades, total)
VALUES ('Ana', 1, 1761742800000, 3, 3.9);

-- Consultar: los pedidos de Ana, del más reciente al más antiguo
SELECT * FROM pedidos WHERE nombre = 'Ana' ORDER BY fecha_hora DESC;

-- Contar filas que cumplen una condición
SELECT COUNT(*) FROM pedidos WHERE nombre = 'Ana';

-- Modificar filas
UPDATE pedidos SET para_llevar = 0 WHERE id = 1;

-- Borrar filas (¡sin WHERE se borran TODAS!)
DELETE FROM pedidos WHERE id = 1;
```

| Parte | Qué hace |
|---|---|
| `SELECT columnas FROM tabla` | Elige qué columnas leer (`*` = todas) y de qué tabla |
| `WHERE condición` | Filtra las filas (`=`, `<>`, `<`, `>`, `AND`, `OR`) |
| `ORDER BY columna ASC/DESC` | Ordena el resultado de menor a mayor (`ASC`) o al revés (`DESC`) |
| `COUNT(*)` | En lugar de las filas, devuelve cuántas hay |
| `LIMIT n` | Devuelve como máximo `n` filas |

## 1.8 Qué aporta Room

Android trae desde siempre clases para usar SQLite directamente, pero obligan a escribir mucho código repetitivo: abrir la base de datos, preparar cada consulta, recorrer los resultados columna a columna y convertir cada fila en un objeto a mano. Un error en el nombre de una columna solo se descubre al ejecutar.

**Room** es la librería oficial de Jetpack que hace ese trabajo por nosotros. Es lo que se llama un **ORM** (*Object-Relational Mapping*, mapeo objeto-relacional): relaciona clases de Kotlin con tablas, y objetos con filas. Con Room se escriben tres piezas:

```
┌───────────────────┐      ┌─────────────────┐      ┌─────────────────────────┐
│ @Entity           │      │ @Dao            │      │ @Database               │
│ PedidoEntity      │      │ PedidoDao       │      │ CafeteriaDatabase       │
│                   │      │                 │      │                         │
│ Una clase =       │      │ Las operaciones │      │ Reúne entidades y DAO   │
│ una tabla         │      │ sobre la tabla  │      │ y abre el fichero .db   │
└───────────────────┘      └─────────────────┘      └─────────────────────────┘
```

- **Entidad** (`@Entity`): una `data class` que describe una tabla. Cada propiedad es una columna y cada objeto, una fila.
- **DAO** (`@Dao`, *Data Access Object*): una **interfaz** con las operaciones sobre la tabla: insertar, consultar, borrar.
- **Base de datos** (`@Database`): una **clase abstracta** que declara qué entidades hay y da acceso a los DAO.

Y una ventaja que se nota enseguida: Room **comprueba las consultas SQL al compilar**. Si escribís `SELECT * FROM pedido` (sin la *s*) o una columna que no existe, el proyecto no compila y el error dice exactamente qué falla.

## 1.9 Concepto previo: anotaciones y generación de código

Room se usa casi solo con anotaciones (`@Entity`, `@Dao`, `@Query`…). Llevamos desde la UD3 escribiendo anotaciones (`@Composable`, `@Preview`, `@StringRes`, `@Serializable`), pero hasta ahora no ha hecho falta saber qué son. Con Room sí.

Una **anotación** es una **etiqueta** que se pega a una clase, función o propiedad. Por sí sola no hace nada: no cambia el código ni se ejecuta. Sirve para que **otra herramienta** la lea y actúe. Según quién la lea, el efecto es muy distinto:

| Anotación | Quién la lee | Qué hace esa herramienta |
|---|---|---|
| `@StringRes`, `@DrawableRes` | Android Studio (el analizador de código) | Avisa si se pasa un id de imagen donde se espera un texto |
| `@Preview` | Android Studio | Dibuja la función en el panel de diseño |
| `@Composable` | Un **plugin del compilador** de Kotlin | Transforma la función al compilar para que Compose pueda recomponerla |
| `@Serializable` | Un **plugin del compilador** (el que añadimos en la UD3) | Añade a la clase el código que la convierte a JSON y desde JSON |
| `@Entity`, `@Dao`, `@Database` | El **procesador de Room**, a través de **KSP** | **Genera ficheros de código nuevos** que implementan la base de datos |

Fijaos en la última fila. El DAO es una **interfaz**: declara funciones pero no las implementa. Y la base de datos es una clase **abstracta**: no se puede crear con su constructor. ¿Quién escribe el código que falta? Lo escribe Room **al compilar**:

1. Al compilar, **KSP** (*Kotlin Symbol Processing*, una herramienta de Google) lee vuestro código y busca las anotaciones de Room.
2. Por cada una, el procesador de Room **genera** clases de Kotlin: `PedidoDao_Impl` (que implementa la interfaz con todo el código SQLite) y `CafeteriaDatabase_Impl` (que hereda de la clase abstracta).
3. Esas clases se compilan junto con las vuestras. Nunca las escribís ni las tocáis, pero podéis verlas en `app/build/generated/ksp/debug/kotlin/`.

```
Lo que escribís                  Lo que genera Room (KSP) al compilar
───────────────────────────────  ─────────────────────────────────────
interface PedidoDao          ──► class PedidoDao_Impl : PedidoDao
abstract class               ──► class CafeteriaDatabase_Impl
    CafeteriaDatabase                : CafeteriaDatabase()

La app usa las clases _Impl sin que las veáis.
```

<div class="callout-warning">
Por eso Room necesita <strong>dos</strong> dependencias: la librería que usa la app al ejecutarse (<code>room-runtime</code>) y el procesador que genera el código al compilar (<code>room-compiler</code>, que se añade con <code>ksp(...)</code> en lugar de <code>implementation(...)</code>). Si falta el procesador, el proyecto compila pero la app se cierra al abrir la base de datos con un error como <em>Cannot find implementation for CafeteriaDatabase. CafeteriaDatabase_Impl does not exist</em>: nadie ha generado la clase.
</div>

## 1.10 Añadir Room al proyecto

Es el mismo proceso que con la serialización en la UD3: una librería, un procesador y un plugin (KSP).

**Paso 1.** En `gradle/libs.versions.toml`:

```
[versions]
# ... las que ya hay
room = "2.8.5"
ksp = "2.3.12"

[libraries]
# ... las que ya hay
androidx-room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
androidx-room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }

[plugins]
# ... los que ya hay
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

**Paso 2.** En el `build.gradle.kts` **del proyecto** (el de la raíz), dentro de `plugins { }`:

```kotlin
alias(libs.plugins.ksp) apply false
```

**Paso 3.** En `app/build.gradle.kts`, dentro de `plugins { }`:

```kotlin
alias(libs.plugins.ksp)
```

y dentro de `dependencies { }`:

```kotlin
implementation(libs.androidx.room.runtime)
ksp(libs.androidx.room.compiler)
```

**Paso 4.** *Sync Now*.

<div class="callout-info">
Desde la versión 2.3.0, la versión de KSP ya no tiene que coincidir con la de Kotlin: funciona con Kotlin 2.2 o superior. Si Android Studio sugiere versiones más recientes de <code>room</code> o <code>ksp</code>, podéis aceptarlas. Room 2.8 necesita un <em>Minimum SDK</em> de 23 o más; el proyecto de la cafetería usa 24. En tutoriales antiguos veréis también <code>room-ktx</code>: desde Room 2.7 ya no hace falta, porque su contenido está dentro de <code>room-runtime</code>.
</div>

## 1.11 Las tres piezas en código

### La entidad

```kotlin
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

- `tableName` da nombre a la tabla; sin él se llamaría como la clase.
- `@PrimaryKey(autoGenerate = true)` marca la clave primaria y pide a SQLite que la numere. El valor por defecto `0` significa "todavía no tiene id": al insertar, Room deja que la base de datos le asigne el siguiente número.
- `@ColumnInfo(name = ...)` cambia el nombre de la columna. Lo usamos para seguir la costumbre de SQL de escribir los nombres en minúsculas con guiones bajos (`fecha_hora`), mientras que en Kotlin seguimos con `fechaHora`. En las consultas SQL se usa el nombre de la **columna**.
- Las columnas pueden ser de tipos básicos (`Int`, `Long`, `Double`, `Boolean`, `String`). Si la propiedad es nullable (`String?`), la columna admite `NULL`. Una propiedad `List<...>` **no** puede ser una columna: una lista de líneas es otra tabla relacionada, y lo veremos en la próxima sesión.

### El DAO

```kotlin
@Dao
interface PedidoDao {

    @Insert
    suspend fun insertar(pedido: PedidoEntity): Long

    @Query("SELECT COUNT(*) FROM pedidos WHERE nombre = :nombre")
    suspend fun contarPorNombre(nombre: String): Int

    @Query("SELECT * FROM pedidos ORDER BY fecha_hora DESC")
    suspend fun obtenerTodos(): List<PedidoEntity>

    @Query("DELETE FROM pedidos")
    suspend fun borrarTodos()
}
```

- `@Insert` no necesita SQL: Room lo deduce de la entidad. Devuelve el **id** asignado a la fila nueva.
- `@Query` lleva la consulta SQL completa. `:nombre` es un **parámetro**: Room pone ahí el valor del argumento de la función que se llama igual. Nunca se construye la consulta concatenando textos (`"... WHERE nombre = '" + nombre + "'"`): sería vulnerable a **inyección SQL**, el mismo problema que en cualquier aplicación con base de datos.
- Room convierte el resultado al tipo de retorno: una lista de entidades, un `Int` para `COUNT(*)`, una sola entidad (o `null`) si la función devuelve `PedidoEntity?`.
- Todas las funciones son `suspend`, igual que en DataStore. Veremos por qué en el apartado 1.13.

### La base de datos

```kotlin
@Database(entities = [PedidoEntity::class], version = 1, exportSchema = false)
abstract class CafeteriaDatabase : RoomDatabase() {
    abstract fun pedidoDao(): PedidoDao
}
```

- `entities` es la lista de tablas. `PedidoEntity::class` es una **referencia a la clase**: no crea ningún objeto, sino que le dice a Room "esta clase". Es la misma idea que las referencias a funciones de la UD3 (`viewModel::confirmar`), aplicada a una clase. Cuando una librería de Android espera la clase en su formato de Java, se escribe `::class.java`.
- `version` es la versión del **esquema** (la estructura de tablas y columnas). Cuando cambie, habrá que subirla: lo trabajaremos en la sesión 4 con las *migraciones*.
- `exportSchema = false` evita un aviso al compilar. Con `true`, Room guarda en un fichero JSON la estructura de cada versión, algo necesario para las migraciones; lo activaremos entonces.
- Por cada DAO hay una función abstracta que lo devuelve. La implementa `CafeteriaDatabase_Impl`.

## 1.12 Una sola base de datos para toda la app: ¿dónde se crea?

La base de datos se crea con `Room.databaseBuilder`:

```kotlin
val db = Room.databaseBuilder(
    context,                          // un Context: ¿cuál?
    CafeteriaDatabase::class.java,    // la clase de la base de datos
    "cafeteria.db"                    // el nombre del fichero
).build()
```

Abrir una base de datos es **caro** (lee el fichero, prepara conexiones, comprueba el esquema) y debe existir **una sola** para todo el proceso, igual que el DataStore de la UD3. Si hubiera dos objetos sobre el mismo fichero, funcionarían, pero gastarían el doble de memoria y, como veremos en la próxima sesión, uno no se enteraría de los cambios que hace el otro: una pantalla podría mostrar datos antiguos.

La pregunta es dónde ponerla. Para responder hay que tener claro **cuánto vive cada pieza** de la app:

```
Proceso de la app        ← hasta que se cierra la app o Android lo mata
└── Application          ← UNA por proceso, antes que ninguna pantalla
    └── MainActivity     ← se destruye y se recrea al girar
        ├── Almacén de ViewModel de la Activity   ← se conserva al girar
        │   └── PedidoViewModel
        └── NavHost: una entrada por pantalla de la pila
            └── Almacén de ViewModel de cada entrada  ← se destruye al salir
                                                         de esa pantalla
```

Repasemos las opciones con lo que ya sabemos de la UD3:

| Si la base de datos se crea en... | Qué pasaría |
|---|---|
| Un composable | Se crearía otra en cada recomposición o, con `remember`, otra en cada giro |
| La `Activity` | Otra en cada giro de pantalla |
| Un `ViewModel` | Una por cada `ViewModel`. Hoy solo tenemos uno, pero en cuanto una pantalla tenga el suyo (y vive en el almacén de **su entrada** de la pila, sesión 5 de la UD3), habría una base de datos por pantalla, que se abriría y cerraría al entrar y salir |
| La **`Application`** | **Una por proceso**, que existe antes que cualquier pantalla o `ViewModel` y vive tanto como ellos o más. Esta es la buena |

### Una `Application` propia

Hasta ahora la app usaba la `Application` que crea Android por defecto. Podemos crear una subclase propia y decirle a Android que use esa:

```kotlin
class CafeteriaApp : Application() {

    val database: CafeteriaDatabase by lazy {
        Room.databaseBuilder(
            this,                     // la propia Application es el Context
            CafeteriaDatabase::class.java,
            "cafeteria.db"
        ).build()
    }
}
```

Y en `AndroidManifest.xml`, en la etiqueta `<application>`, se añade el atributo `android:name`:

```xml
<application
    android:name=".CafeteriaApp"
    android:allowBackup="true"
    ... >
```

Sin ese atributo, Android sigue creando la `Application` por defecto y vuestra clase nunca se usa.

**`by lazy`** es otra **delegación** de propiedades, como el `by mutableStateOf` de la UD3, y significa "**no la crees ahora: créala la primera vez que alguien la use, y después devuelve siempre la misma**". Así la app no paga el coste de abrir la base de datos al arrancar si el usuario no llega a usarla, y nunca se crean dos. Además, `by lazy` es **seguro entre hilos**: si dos hilos la piden a la vez, solo uno la crea.

### Llegar a la base de datos desde el `ViewModel`

`PedidoViewModel` ya recibe la `Application` (es un `AndroidViewModel`). Pero la recibe con el tipo genérico `Application`, que no tiene ninguna propiedad `database`. Hay que decirle al compilador que, en realidad, es nuestra `CafeteriaApp`, con una **conversión de tipo** (`as`, la versión sin interrogación del `as?` de la UD2):

```kotlin
class PedidoViewModel(application: Application) :
    AndroidViewModel(application) {

    private val database = (application as CafeteriaApp).database
    private val pedidoDao = database.pedidoDao()
    // ...
}
```

Si se os olvida el `android:name` del manifiesto, la `Application` que llega es la genérica de Android, la conversión falla y la app se cierra al arrancar con una `ClassCastException` (*android.app.Application cannot be cast to …CafeteriaApp*). Es uno de los errores más típicos de esta sesión.

## 1.13 Room y los hilos

Igual que el disco, una consulta a la base de datos **tarda**, y Room lo controla de forma estricta: si una función del DAO **no** es `suspend` y se llama desde el hilo principal, la app se cierra con *Cannot access database on the main thread since it may potentially lock the UI for a long period of time*.

Con funciones `suspend`, Room ejecuta cada operación en **sus propios hilos** de base de datos y devuelve el resultado a la coroutine que la llamó. Es decir, las funciones `suspend` de un DAO ya son *main-safe*: se pueden llamar directamente desde `viewModelScope.launch`, **sin** `withContext(Dispatchers.IO)`. Es la diferencia con los ficheros del apartado 1.3, donde el cambio de hilo lo hacíamos nosotros.

| Almacén | ¿Quién saca el trabajo del hilo principal? |
|---|---|
| `File` | Nosotros, con `withContext(Dispatchers.IO)` en el repositorio |
| DataStore | La propia librería |
| Room (funciones `suspend` y `Flow`) | La propia librería |

## 1.14 Un repositorio para el historial

Como siempre, la capa de datos se presenta al `ViewModel` a través de un repositorio. Este, además, **traduce** entre el modelo de la app (`Ticket`) y la fila de la tabla (`PedidoEntity`): el `ViewModel` no necesita saber cómo es la tabla.

```kotlin
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

    suspend fun borrarTodo() = pedidoDao.borrarTodos()
}
```

La app queda con tres repositorios en su capa de datos, cada uno con su almacén:

```
PedidoViewModel
   ├──► PreferenciasRepository ──► DataStore (preferencias)
   ├──► TicketRepository       ──► ultimo_ticket.json
   └──► HistorialRepository    ──► PedidoDao ──► cafeteria.db (SQLite)
```

## 1.15 Comprobar la base de datos en el emulador

**Los ficheros.** En *Device Explorer*, en `/data/data/<paquete>/databases/`, aparecen **tres** ficheros, no uno: `cafeteria.db`, `cafeteria.db-wal` y `cafeteria.db-shm`. SQLite trabaja en modo *WAL* (*Write-Ahead Log*): los cambios se escriben primero en el fichero `-wal` y se pasan al `.db` de vez en cuando; el `-shm` coordina el acceso. Por eso, si se copia solo el `.db` al ordenador, puede parecer que faltan los últimos pedidos. Los tres forman la base de datos.

**El contenido.** Para ver las tablas no hace falta descargar nada: con la app en marcha, *View › Tool Windows › App Inspection*, pestaña **Database Inspector**. Muestra las bases de datos de la app, sus tablas y sus filas, y permite:

- ver los datos **en vivo** mientras se usa la app (casilla *Live updates*);
- escribir y ejecutar **consultas SQL** propias (*New Query*), por ejemplo `SELECT nombre, total FROM pedidos WHERE total > 3`;
- probar ahí una consulta antes de copiarla a un `@Query`.

### Si cambiáis una entidad durante el desarrollo

Imaginad que, con la app ya instalada y algún pedido guardado, añadís una columna a `PedidoEntity` y volvéis a ejecutar. La app se cierra con:

```
java.lang.IllegalStateException: Room cannot verify the data integrity.
Looks like you've changed schema but forgot to update the version number.
```

La base de datos del emulador se creó con la estructura antigua, y Room se niega a usarla con una clase que ya no coincide: podría perder o corromper datos. Mientras desarrollamos, la solución rápida es **desinstalar la app** del emulador (o *Ajustes › Aplicaciones › Cafetería › Almacenamiento › Borrar datos*) y volver a ejecutar: se crea una base de datos nueva con la estructura actual. Con una app ya publicada eso no es aceptable, porque el usuario perdería sus datos: en la sesión 4 veremos cómo se hace bien, con una **migración**.

<div class="callout-info">
<strong>¿Y Room 3?</strong> En 2026 Google ha publicado Room 3.0, una versión nueva pensada también para apps multiplataforma (Android, iOS, escritorio y web), con otro nombre de paquete (<code>androidx.room3</code>). Room 2.x sigue recibiendo correcciones y es la que usan casi toda la documentación y los ejemplos actuales, por eso la usamos en el módulo. Las ideas de esta unidad (entidades, DAO, base de datos, KSP, funciones <code>suspend</code> y <code>Flow</code>) son las mismas en las dos.
</div>

## Tabla resumen del bloque

| Necesito... | Uso |
|---|---|
| Carpeta privada de datos | `context.filesDir` (en el `ViewModel`: `application.filesDir`) |
| Representar un fichero | `File(carpeta, "nombre.json")` |
| Escribir / leer / comprobar / borrar | `writeText(texto)`, `readText()`, `exists()`, `delete()` |
| Código de disco fuera del hilo principal | `withContext(Dispatchers.IO) { ... }` dentro de una función `suspend` |
| Objeto ↔ JSON | `@Serializable`; `json.encodeToString(obj)` / `json.decodeFromString<Clase>(texto)` |
| JSON dañado / error de disco | capturar `SerializationException` / `IOException` |
| Añadir Room | `room-runtime` con `implementation`, `room-compiler` con `ksp`, plugin `com.google.devtools.ksp` |
| Una tabla | `@Entity(tableName = "...") data class` con `@PrimaryKey(autoGenerate = true) val id: Long = 0` |
| Nombre de columna distinto | `@ColumnInfo(name = "...")` |
| Operaciones sobre la tabla | `@Dao interface` con `@Insert`, `@Query("...")`, funciones `suspend` |
| Parámetro en una consulta | `:nombre` en el SQL, igual que el parámetro de la función |
| La base de datos | `@Database(entities = [...], version = 1) abstract class ... : RoomDatabase()` |
| Una única instancia | `val database by lazy { Room.databaseBuilder(...).build() }` en una `Application` propia + `android:name` en el manifiesto |
| Llegar a ella desde el `ViewModel` | `(application as CafeteriaApp).database` |
| Ver las tablas | *App Inspection › Database Inspector* |
| Cambié una entidad y la app se cierra | Desinstalar la app (en desarrollo); migración (sesión 4) |

## Para la próxima sesión

La cafetería guarda ahora el último ticket en un fichero y cada pedido en la base de datos, pero las dos cosas se solapan: el último ticket no deja de ser el pedido más reciente del historial. Y el historial solo guarda la cabecera de cada pedido, sin sus líneas, porque una lista no cabe en una columna. En la próxima sesión relacionaremos dos tablas (pedidos y líneas), haremos que la interfaz **se actualice sola** cuando cambie la base de datos, y todo ello sin tocar `PedidoViewModel`: un buen ejemplo de por qué conviene tener los datos detrás de un repositorio.
