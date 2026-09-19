---
title: "Soporte técnico — Solución (Sesión 2)"
unit: UD2
sesion: 2
tipo: solucion
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Dejad que el alumnado llegue a su propia versión; usadla solo si hace falta destrabar a algún grupo.
</div>

```kotlin
// Ejercicio 0 (traducción) + Ejercicio 1 — data class Ticket
// tecnicoAsignado es nullable porque una incidencia puede crearse sin
// asignar todavía; el resto de propiedades son val porque no cambian
// tras crear el ticket (para "cambiarlas" se usará copy()).
data class Ticket(
    val id: Int,
    val descripcion: String,
    val tecnicoAsignado: String? = null,
    val resuelta: Boolean = false
)

var siguienteId = 1

// Ejercicio 2 — Elvis con salida temprana
fun crearTicket(descripcion: String?): Ticket {
    val desc = descripcion ?: throw IllegalArgumentException("La descripción no puede estar vacía")
    if (desc.isBlank()) {
        throw IllegalArgumentException("La descripción no puede estar vacía")
    }
    val ticket = Ticket(id = siguienteId, descripcion = desc)
    siguienteId++
    return ticket
}

fun descripcionTecnico(ticket: Ticket): String = ticket.tecnicoAsignado ?: "Sin asignar"

// Ejercicio 3 — ?.let { }
fun notificar(ticket: Ticket) {
    ticket.tecnicoAsignado?.let {
        println("Notificando a $it")
    }
}

fun main() {
    // Comprobación ejercicios 2 y 3
    val t1 = crearTicket("No enciende el proyector del aula 3")
    println(descripcionTecnico(t1))   // "Sin asignar"

    val t2 = t1.copy(tecnicoAsignado = "Marta")
    notificar(t1)   // no imprime nada: sin técnico
    notificar(t2)   // "Notificando a Marta"

    // Ejercicio 4 — Conversión segura con as?
    val elementos: List<Any> = listOf(
        Ticket(10, "Ordenador bloqueado", "Marta"),
        "Aviso: mantenimiento programado esta noche",
        Ticket(11, "No hay conexión wifi"),
        42
    )
    for (elemento in elementos) {
        val ticket = elemento as? Ticket
        if (ticket != null) {
            println(ticket.descripcion)
        } else {
            println("Elemento ignorado (no es un ticket)")
        }
    }

    // Ejercicio 5 — Igualdad estructural
    val a = Ticket(20, "Impresora sin tóner")
    val b = Ticket(20, "Impresora sin tóner")
    println(a == b)     // true: == llama a equals(), compara contenido
    println(a === b)    // false: son dos objetos distintos en memoria

    val c = a.copy(resuelta = true)
    println(a == c)     // false: resuelta es distinta

    // Ejercicio 6 — Panel de tickets
    val panel: MutableList<Ticket> = mutableListOf(
        Ticket(1, "No enciende el proyector del aula 3"),
        Ticket(2, "Ordenador bloqueado", "Marta"),
        Ticket(3, "No hay conexión wifi"),
        Ticket(4, "Impresora sin tóner", "Marta", resuelta = true),
        Ticket(5, "El altavoz no suena")
    )

    println("\n--- Panel de tickets ---")
    for (ticket in panel) {
        println("${ticket.descripcion} -> ${descripcionTecnico(ticket)}")
    }

    var sinAsignar = 0
    for (ticket in panel) {
        if (ticket.tecnicoAsignado == null) {
            sinAsignar++
        }
    }
    println("Tickets sin asignar: $sinAsignar")
}
```

## Qué comprobar en cada elemento obligatorio

- **Traducción Java → Kotlin y `data class`** → `Ticket` con constructor primario; `tecnicoAsignado: String?` es la única propiedad nullable.
- **Elvis como valor por defecto y como salida temprana** → `descripcionTecnico` usa `?:` con un valor por defecto; `crearTicket` usa `?: throw` para validar antes de continuar.
- **`?.let { }`** → `notificar` solo imprime cuando `tecnicoAsignado` no es `null`, sin necesidad de un `if`.
- **`as?`** → al recorrer `elementos` (`List<Any>`), el cast fallido en `"Aviso..."` o `42` da `null` en vez de lanzar una excepción.
- **`==` / `===` y `copy()`** → `a == b` compara contenido (ambos `Ticket` iguales aunque sean objetos distintos); `a === b` es `false`; `a.copy(resuelta = true)` crea un tercer ticket que ya no es `==` al original.
- **Colección recorrida con `for`** → `panel` es un `MutableList<Ticket>`; el conteo de tickets sin asignar usa un contador y un `for`, sin `filter`.
- **Sin adelantar contenido** → no aparecen funciones de extensión ni `filter`/`map` con `it`: todo se resuelve con lo visto hasta la sesión 2.
