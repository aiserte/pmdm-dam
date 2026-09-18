---
title: "Null-safety y data classes"
unit: UD2
order: 2
duration: "2h"
---

## Objetivos

- Comprender el problema del `NullPointerException` y cómo lo resuelve Kotlin.
- Manejar tipos nullable con `?.`, `?:` y `!!`.
- Sustituir POJOs de Java por `data class` y aprovechar lo que generan automáticamente.

## 1. El problema del NullPointerException

En Java, cualquier variable de tipo referencia puede valer `null` en cualquier momento, y el compilador no avisa. Kotlin ataca este problema en el propio sistema de tipos: por defecto, ningún tipo admite `null`.

```kotlin
var nombre: String = "Ana"
nombre = null   // ERROR de compilación, no en ejecución

var apodo: String? = null   // OK: el ? permite null
```

## 2. Trabajar con tipos nullable

- `?.` (*safe call*): accede a la propiedad solo si el objeto no es `null`; si lo es, devuelve `null` sin lanzar excepción.
- `?:` (*elvis operator*): da un valor por defecto cuando la expresión de la izquierda es `null`.
- `!!` (*not-null assertion*): afirma "estoy seguro de que no es null"; si os equivocáis, lanza `NullPointerException` igualmente.

```kotlin
val apodo: String? = null

val longitud = apodo?.length              // null, sin crashear
val longitudSegura = apodo?.length ?: 0   // 0, valor por defecto
val forzado = apodo!!.length              // lanza NPE si apodo es null
```

<div class="callout-warning">
Regla de oro en este módulo: evitad <code>!!</code> salvo que estéis absolutamente seguros. Preferid siempre <code>?:</code> con un valor por defecto sensato.
</div>

## 3. data class: adiós a los POJOs manuales

Con la palabra clave `data`, Kotlin genera automáticamente `equals()`, `hashCode()`, `toString()`, `copy()` y `componentN()`:

```kotlin
data class Usuario(
    val nombre: String,
    val telefono: String?
)
```

- `equals()`/`hashCode()` — comparan por contenido, no por referencia.
- `toString()` — imprime `Usuario(nombre=Ana, telefono=null)`.
- `copy()` — crea una copia cambiando solo algunos campos: `usuario.copy(telefono = "600123456")`.
- `componentN()` — permite la desestructuración: `val (nombre, tel) = usuario`.

## 4. Ejemplo práctico: de Java a Kotlin

```java
public class Usuario {
    private String nombre;
    private String telefono; // puede ser null
    // constructor, getters, setters, equals, hashCode, toString...
}
```

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

<div class="callout-practica">
Ejercicio de sesión: convierte la clase <code>Usuario</code> en <code>data class</code> con el campo <code>telefono</code> nullable y usa el operador Elvis para mostrar un valor por defecto cuando no exista.
</div>

## Para la próxima sesión

Bloque 3: funciones de extensión y lambdas — la base de cómo se escribirá todo el código de interfaz en Jetpack Compose.
