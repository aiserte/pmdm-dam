---
title: "Soporte técnico: extensión y lambdas — Solución (Sesión 3)"
unidad: UD2
sesion: 3
orden: 3
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Dejad que el alumnado llegue a su propia versión; usadla solo si hace falta destrabar a algún grupo.
</div>

```kotlin
// data class y panel heredados de la práctica de la sesión 2
data class Ticket(
    val id: Int,
    val descripcion: String,
    val tecnicoAsignado: String? = null,
    val resuelta: Boolean = false
)

val panel: MutableList<Ticket> = mutableListOf(
    Ticket(1, "No enciende el proyector del aula 3"),
    Ticket(2, "Ordenador bloqueado", "Marta"),
    Ticket(3, "No hay conexión wifi"),
    Ticket(4, "Impresora sin tóner", "Marta", resuelta = true),
    Ticket(5, "El altavoz no suena")
)

// Ejercicio 1
fun List<Ticket>.sinAsignar(): List<Ticket> = this.filter { it.tecnicoAsignado == null }

// Ejercicio 2
fun Ticket.resumen(): String =
    "#$id — $descripcion (${tecnicoAsignado?.let { "asignado a $it" } ?: "sin asignar"})"

// Ejercicio 5
fun List<Ticket>.resueltos(): List<Ticket> = this.filter { it.resuelta }

// Ejercicio 6 — extensión sobre receptor nullable, se llama SIN ?.
fun Ticket?.descripcionSegura(): String =
    if (this == null) "(sin ticket)" else this.resumen()

// Ejercicio 7 — función propia de orden superior, también extensión
fun List<Ticket>.aplicarATodos(accion: (Ticket) -> Unit) {
    this.forEach(accion)
}

fun main() {
    // Ejercicio 3 — map + forEach
    val resumenes = panel.map { it.resumen() }
    resumenes.forEach { println(it) }

    println("\n--- Solo pendientes de asignar ---")
    // Ejercicio 4 — encadenado
    panel.sinAsignar().map { it.resumen() }.forEach { println(it) }

    println("\n--- Tickets resueltos ---")
    println("Resueltos: ${panel.resueltos().size}")

    println("\n--- Receptor nullable ---")
    // Sin adelantar find/firstOrNull (eso es sesión 4): basta con guardar un
    // Ticket real, o null, en una variable declarada como Ticket?.
    val conTicket: Ticket? = panel[2]
    val sinTicket: Ticket? = null
    println(conTicket.descripcionSegura())   // resumen del ticket #3
    println(sinTicket.descripcionSegura())   // "(sin ticket)"

    println("\n--- Informe con trailing lambda ---")
    panel.aplicarATodos { ticket ->
        println(ticket.resumen())
        if (ticket.tecnicoAsignado == null) {
            println("  -> URGENTE")
        }
    }
}
```

## Qué comprobar en cada elemento obligatorio

- **Funciones de extensión con `filter`** → `sinAsignar()` y `resueltos()` no modifican `panel`; devuelven listas nuevas.
- **Expresión única + Elvis** → `resumen()` combina `?.let { }` (visto en sesión 2, para construir el texto solo si hay técnico) con `?:` para el valor por defecto, todo en una sola expresión.
- **Extensión sobre receptor nullable** → `descripcionSegura()` se llama como `conTicket.descripcionSegura()` y `sinTicket.descripcionSegura()`, **sin** `?.`, exactamente igual que `String?.estaVacioOEsNulo()` de la sesión.
- **`map`, `forEach` y encadenado** → aparecen los tres usos: por separado (ejercicio 3) y encadenados (`sinAsignar().map { }.forEach { }`, ejercicio 4).
- **Función propia de orden superior + trailing lambda** → `aplicarATodos` recibe `accion: (Ticket) -> Unit` y se llama como `panel.aplicarATodos { ticket -> ... }`, sin paréntesis por ser su único parámetro.
- **Cero bucles `for` y sin adelantar contenido** → toda la solución usa `filter`/`map`/`forEach`; no aparece `find`, `sortedBy`, `count` ni ninguna *scope function* nueva (eso es la sesión 4).