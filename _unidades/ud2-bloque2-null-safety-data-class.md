---
title: "Null-safety y data classes"
unit: UD2
order: 2
duration: "2h"
---

## Objetivos

- Entender por qué existe el `NullPointerException` y cómo lo evita Kotlin desde el propio sistema de tipos.
- Manejar tipos nullable con `?.`, `?:`, `!!` y el idioma `?: return` / `?: throw`.
- Ejecutar código solo cuando un valor no es `null` con `?.let { }`, y convertir tipos de forma segura con `as?`.
- Sustituir POJOs de Java por `data class` y entender qué genera automáticamente, incluida la igualdad estructural.

## 2.1 El problema del NullPointerException

En Java, cualquier variable de tipo referencia puede valer `null` en cualquier momento, y el compilador no avisa. El típico `NullPointerException` —la "excepción del billón de dólares", como la llamó su propio inventor, Tony Hoare— aparece en tiempo de ejecución, normalmente en el peor momento.

Java 8 introdujo `Optional<T>` para mitigarlo, pero es un mecanismo opcional: nada obliga a usarlo, y una API antigua puede seguir devolviendo `null` sin avisar. Kotlin ataca el problema en el propio sistema de tipos: **por defecto, ningún tipo admite `null`**. Si queréis permitirlo, tenéis que decirlo explícitamente en la declaración del tipo.

```kotlin
var nombre: String = "Ana"
nombre = null   // ERROR de compilación, no en ejecución

var apodo: String? = null   // OK: el ? permite null
```

<div class="callout-info">
La diferencia clave con <code>Optional&lt;T&gt;</code>: en Kotlin la nulabilidad forma parte del tipo (<code>String</code> frente a <code>String?</code>), así que el compilador os avisa en el momento de escribir el código, no en tiempo de ejecución.
</div>

## 2.2 Trabajar con tipos nullable

Una vez que una variable es nullable (`String?`), el compilador os obliga a comprobarlo antes de usarla. Para eso existen varios operadores:

- `?.` (*safe call*): accede a la propiedad solo si el objeto no es `null`; si lo es, devuelve `null` sin lanzar excepción.
- `?:` (*elvis operator*): da un valor por defecto cuando la expresión de la izquierda es `null`.
- `!!` (*not-null assertion*): afirma "estoy seguro de que no es null"; si os equivocáis, lanza `NullPointerException` igualmente. Usar con mucha cautela.

```kotlin
val apodo: String? = null

val longitud = apodo?.length              // null, sin crashear
val longitudSegura = apodo?.length ?: 0   // 0, valor por defecto
val forzado = apodo!!.length              // lanza NPE si apodo es null
```

<div class="callout-warning">
Regla de oro en este módulo: evitad <code>!!</code> salvo que estéis absolutamente seguros. Preferid siempre <code>?:</code> con un valor por defecto sensato.
</div>

### Cómo se resolvía esto en Java

| Necesidad | Java | Kotlin |
|---|---|---|
| Declarar que un valor puede faltar | Cualquier referencia (no se distingue en el tipo) | `String?` (explícito en el tipo) |
| Evitar el NPE al encadenar accesos | `if (x != null) { x.foo(); }` o `Optional.ofNullable(x).map(...)` | `x?.foo()` |
| Valor por defecto si es `null` | `x != null ? x : def` (ternario) | `x ?: def` |
| Afirmar "sé que no es null" | Cast implícito, sin aviso del compilador | `x!!` (explícito, lanza NPE si os equivocáis) |

## 2.3 Elvis con salida temprana: `?: return` / `?: throw`

El operador `?:` no solo sirve para dar un valor por defecto: a su derecha puede ir cualquier expresión, incluida una que corte el flujo de la función. Es un idioma muy habitual para validar parámetros al principio de una función:

```kotlin
fun procesarTelefono(telefono: String?): String {
    val valor = telefono ?: return "Sin teléfono registrado"
    return "Teléfono: $valor"
}
```

Si `telefono` es `null`, la función termina inmediatamente devolviendo el texto por defecto; si no lo es, `valor` queda con el tipo `String` (ya no nullable) para el resto de la función. También se usa con `throw` cuando un valor nulo debería considerarse un error de programación:

```kotlin
fun cargarUsuario(id: String?): String {
    val idValido = id ?: throw IllegalArgumentException("id no puede ser null")
    return "Cargando usuario $idValido"
}
```

## 2.4 Ejecutar código solo si no es null: `?.let { }`

`let` es una *función de ámbito* (scope function) de Kotlin: recibe el objeto sobre el que se llama como `it` y ejecuta el bloque que le paséis. Combinada con `?.`, es la forma idiomática de "haz esto solo si el valor no es null":

```kotlin
val apodo: String? = "Ana"

apodo?.let {
    println("El apodo tiene ${it.length} caracteres")
}
```

Si `apodo` fuese `null`, el bloque `{ ... }` simplemente no se ejecuta: no hace falta envolverlo en un `if`.

<div class="callout-info">
En Android vais a ver <code>?.let { }</code> constantemente para ejecutar código de interfaz solo cuando un dato ha llegado —por ejemplo, la respuesta de una petición de red—. Kotlin tiene más funciones de ámbito (<code>also</code>, <code>apply</code>, <code>run</code>, <code>with</code>); de momento solo necesitáis <code>let</code>.
</div>

## 2.5 Conversión segura de tipos: `as?`

Igual que `?.` es la versión seguriza de acceder a una propiedad, `as?` es la versión segura del cast `as`: si la conversión no es posible, devuelve `null` en lugar de lanzar una excepción.

```kotlin
val obj: Any = "Hola"

val texto: String? = obj as? String     // "Hola": el cast sí es válido
val numero: Int? = obj as? Int          // null: el cast falla, sin excepción
```

## 2.6 data class: adiós a los POJOs manuales

En Java, una clase de datos sencilla (un POJO) obliga a escribir a mano `equals()`, `hashCode()`, `toString()` y, a veces, un método para copiar. En Kotlin, con la palabra clave `data`, todo eso se genera automáticamente:

```kotlin
data class Usuario(
    val nombre: String,
    val telefono: String?
)
```

Con esa única declaración, Kotlin genera para vosotros:

- `equals()`/`hashCode()` — comparan por contenido, no por referencia.
- `toString()` — imprime `Usuario(nombre=Ana, telefono=null)` en lugar de una dirección de memoria.
- `copy()` — crea una copia del objeto cambiando solo algunos campos: `usuario.copy(telefono = "600123456")`.
- `componentN()` — permite la desestructuración: `val (nombre, tel) = usuario`.

## 2.7 Igualdad estructural: `==` frente a `===`

En Java, `==` sobre objetos compara referencias (si son el mismo objeto en memoria), y hay que llamar a `.equals()` explícitamente para comparar contenido. En Kotlin es al revés por defecto:

```kotlin
data class Usuario(val nombre: String, val telefono: String?)

fun main() {
    val u1 = Usuario("Ana", "600111222")
    val u2 = Usuario("Ana", "600111222")

    println(u1 == u2)    // true: == llama a equals(), compara contenido
    println(u1 === u2)   // false: no es el mismo objeto en memoria
}
```

<div class="callout-warning">
Cuidado con la confusión si venís de Java: en Kotlin, <code>==</code> compara <strong>contenido</strong> (llama a <code>equals()</code>) y <code>===</code> compara <strong>referencia</strong> (es el equivalente al <code>==</code> de Java sobre objetos). Con una <code>data class</code>, <code>equals()</code> ya viene generado automáticamente comparando todas sus propiedades.
</div>

## 2.8 Ejemplo práctico: de Java a Kotlin

Partiendo de esta clase Java:

```java
public class Usuario {
    private String nombre;
    private String telefono; // puede ser null
    // constructor, getters, setters, equals, hashCode, toString...
}
```

La traducción idiomática a Kotlin, combinando `data class` y tipo nullable, queda así:

```kotlin
data class Usuario(
    val nombre: String,
    val telefono: String? = null
)

fun mostrarTelefono(u: Usuario) {
    val texto = u.telefono ?: "No disponible"
    println(texto)
}
```

## 2.9 Tabla resumen de equivalencias

| Concepto | Java | Kotlin |
|---|---|---|
| Tipo que puede ser `null` | Cualquier referencia + `@Nullable` / `Optional<T>` | `Tipo?` |
| Acceso seguro encadenado | `if (x != null) { ... }` / `Optional.map(...)` | `x?.foo()` |
| Valor por defecto si `null` | Ternario `x != null ? x : def` | `x ?: def` |
| Salir de la función si `null` | `if (x == null) return ...` | `x ?: return ...` / `x ?: throw ...` |
| Ejecutar código solo si no es `null` (función de ámbito `let`) | `if (x != null) { ... }` | `x?.let { ... }` |
| Conversión segura de tipo (cast seguro `as?`) | `instanceof` + cast | `x as? Tipo` |
| Comparación por contenido | `.equals()` explícito | `==` (llama a `equals()` automáticamente) |
| POJO con `equals`/`hashCode`/`toString`/`copy` | Escritos a mano o generados por el IDE | `data class` |

<div class="callout-practica">
Ejercicio de sesión: retoma la clase <code>Conductor</code> que tradujiste en la práctica de la Sesión 1 (<em>Flota de alquiler</em>).

1. Conviértela en <code>data class</code> y añade un nuevo campo <code>telefono: String?</code> (nullable).
2. Escribe una función <code>mostrarContacto(c: Conductor): String</code> que use el operador Elvis para devolver <code>"Sin teléfono registrado"</code> cuando <code>telefono</code> sea <code>null</code>.
3. Crea dos <code>Conductor</code> con los mismos datos y comprueba con <code>==</code> y <code>===</code> la diferencia entre igualdad estructural y de referencia.
4. Usa <code>?.let { }</code> para imprimir un mensaje solo cuando el conductor sí tenga teléfono registrado.

20 minutos de trabajo autónomo; se corrige en los últimos 10 minutos de la sesión.
</div>

## Para la próxima sesión

Sesión 3: funciones de extensión y lambdas — la base de cómo se escribirá todo el código de interfaz en Jetpack Compose.
