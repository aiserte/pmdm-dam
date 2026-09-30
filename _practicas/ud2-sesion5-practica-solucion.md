---
title: "Biblioteca de barrio — Solución (Sesión 5)"
unidad: UD2
sesion: 5
orden: 5
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Dejad que el alumnado llegue a su propia versión; usadla solo si hace falta destrabar a algún grupo.
</div>

```kotlin
// Ejercicio 0
interface Prestable {
    fun descripcionCorta(): String
}

open class Publicacion(val titulo: String, val autor: String) : Prestable {
    override fun descripcionCorta(): String = "$titulo, de $autor"
}

class Libro(titulo: String, autor: String, val paginas: Int) : Publicacion(titulo, autor)

class Revista(titulo: String, autor: String, val numero: Int) : Publicacion(titulo, autor) {
    override fun descripcionCorta(): String = "${super.descripcionCorta()} (nº $numero)"
}

// Ejercicio 1
data class Prestamo(
    val publicacion: Publicacion,
    val socio: String,
    val diasPrestado: Int,
    val devuelto: Boolean = false,
    val observaciones: String? = null
)

val prestamos: List<Prestamo> = listOf(
    Prestamo(Libro("Drácula", "Bram Stoker", 418), "Ana", 12),
    Prestamo(Revista("National Geographic", "VVAA", 245), "Luis", 40),
    Prestamo(Libro("1984", "George Orwell", 328), "Marta", 5, devuelto = true),
    Prestamo(Libro("El Quijote", "Cervantes", 863), "Ana", 60, observaciones = "Portada dañada"),
    Prestamo(Revista("Muy Interesante", "VVAA", 512), "Pedro", 8, devuelto = true),
    Prestamo(Libro("Cien años de soledad", "García Márquez", 471), "Luis", 20)
)

// Ejercicio 2
fun Prestamo.resumen(): String =
    "${publicacion.descripcionCorta()} — $socio (${diasPrestado} días)" +
        (observaciones?.let { " ($it)" } ?: "")

// Ejercicio 3
fun List<Prestamo>.sinDevolver(): List<Prestamo> = this.filter { !it.devuelto }

// Ejercicio 6 — función propia con dos lambdas
fun List<Prestamo>.procesarSegun(
    condicion: (Prestamo) -> Boolean,
    accion: (Prestamo) -> Unit
) {
    this.filter(condicion).forEach(accion)
}

fun main() {
    // Ejercicio 3 — filter + map + forEach encadenados
    println("--- Pendientes de devolver ---")
    prestamos.sinDevolver().map { it.resumen() }.forEach { println(it) }

    // Ejercicio 4 — ordenar, contar, buscar
    val porAtraso = prestamos.sortedByDescending { it.diasPrestado }
    println("\n--- Por días de préstamo (desc.) ---")
    porAtraso.forEach { println(it.resumen()) }

    val numDevueltos = prestamos.count { it.devuelto }
    println("\nPréstamos devueltos: $numDevueltos")

    val hayAtrasoGrave = prestamos.any { it.diasPrestado > 30 && !it.devuelto }
    println("¿Algún préstamo de más de 30 días sin devolver?: $hayAtrasoGrave")

    val primeraRevista: Prestamo? = prestamos.find { it.publicacion is Revista }
    println("Primer préstamo de revista: ${primeraRevista?.resumen() ?: "No hay préstamos de revistas"}")

    // Ejercicio 5 — apply y also
    val nuevo = Prestamo(Libro("Fahrenheit 451", "Ray Bradbury", 256), "Elena", 0).apply {
        println("\nNuevo préstamo registrado: ${resumen()}")
    }

    val devueltosOrdenados = prestamos
        .filter { it.devuelto }
        .also { println("Préstamos devueltos encontrados: ${it.size}") }
        .sortedBy { it.socio }
    println("Devueltos (orden por socio): ${devueltosOrdenados.map { it.socio }}")

    // Ejercicio 6 — llamada con condición nombrada + trailing lambda
    println("\n--- Préstamos de Ana ---")
    prestamos.procesarSegun(condicion = { it.socio == "Ana" }) { prestamo ->
        println(prestamo.resumen())
    }
}
```

## Qué comprobar en cada elemento obligatorio

- **Jerarquía `Prestable`/`Publicacion`/`Libro`/`Revista`** → `Libro` no sobrescribe `descripcionCorta()` (usa la heredada de `Publicacion`, polimorfismo simple); `Revista` sí la sobrescribe y reutiliza la del padre con `super.descripcionCorta()`, sin duplicar el texto.
- **`data class Prestamo` con propiedad nullable** → `observaciones: String?`; se resuelve en `resumen()` con `?.let { }` combinado con `?:`, sin ningún `if` explícito.
- **Funciones de extensión** → `resumen()` es de expresión única; `sinDevolver()` también. Ambas se llaman como métodos normales (`prestamo.resumen()`, `prestamos.sinDevolver()`).
- **`filter`/`map`/`forEach` encadenados** → `sinDevolver().map { it.resumen() }.forEach { println(it) }`, en una sola cadena.
- **`sortedByDescending`, `count`, `any`, `find`** → cada uno se usa exactamente para lo que se pide; `primeraRevista` es `Prestamo?` y se resuelve con `?:` al imprimir.
- **`apply`** → `nuevo` es el propio `Prestamo` construido, no lo que devuelve el `println` de dentro. Dentro del bloque, `resumen()` se llama sin prefijo porque, igual que con una propiedad, `apply` deja `this` implícito: al ser `resumen()` una función de extensión sobre `Prestamo`, se resuelve exactamente igual que si fuera un miembro de la clase.
- **`also` en una cadena** → `devueltosOrdenados` sigue siendo el resultado de `filter().sortedBy()`; el `also { }` del medio solo imprime, usa `it` (no `this`) y no altera el valor que fluye por la cadena.
- **Función propia con dos lambdas** → `procesarSegun` recibe `condicion` y `accion`; se llama con `condicion` nombrada dentro de los paréntesis y `accion` como *trailing lambda*.
- **Cero bucles `for`** → toda la solución usa funciones de colección y de orden superior de las sesiones 3 y 4.
