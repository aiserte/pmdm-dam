---
title: "Playlist — Solución (Sesión 4)"
unidad: UD2
sesion: 4
orden: 4
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Dejad que el alumnado llegue a su propia versión; usadla solo si hace falta destrabar a algún grupo.
</div>

```kotlin
// Ejercicio 0
data class Cancion(
    val titulo: String,
    val artista: String,
    val duracionSegundos: Int,
    val genero: String,
    val favorita: Boolean = false
)

val playlist: List<Cancion> = listOf(
    Cancion("Bohemian Rhapsody", "Queen", 355, "Rock", favorita = true),
    Cancion("Imagine", "John Lennon", 183, "Pop"),
    Cancion("Stairway to Heaven", "Led Zeppelin", 482, "Rock"),
    Cancion("Blinding Lights", "The Weeknd", 200, "Synthpop", favorita = true),
    Cancion("Clair de Lune", "Debussy", 300, "Clásica"),
    Cancion("Billie Jean", "Michael Jackson", 294, "Pop")
)

// Ejercicio 7 — función propia con dos lambdas
fun List<Cancion>.procesarSegun(
    condicion: (Cancion) -> Boolean,
    accion: (Cancion) -> Unit
) {
    this.filter(condicion).forEach(accion)
}

fun main() {
    // Ejercicio 1 — ordenar
    val porDuracion = playlist.sortedBy { it.duracionSegundos }
    val porArtista = playlist.sortedBy { it.artista }
    println("--- Por duración ---")
    porDuracion.forEach { println("${it.titulo} (${it.duracionSegundos}s)") }
    println("\n--- Por artista ---")
    porArtista.forEach { println("${it.artista} — ${it.titulo}") }

    // Ejercicio 2 — contar y comprobar
    val numRock = playlist.count { it.genero == "Rock" }
    val hayLarga = playlist.any { it.duracionSegundos > 300 }
    val todasMasDeUnMinuto = playlist.all { it.duracionSegundos > 60 }
    println("\nCanciones de Rock: $numRock")
    println("¿Alguna de más de 5 min?: $hayLarga")
    println("¿Todas duran más de 1 min?: $todasMasDeUnMinuto")

    // Ejercicio 3 — buscar
    val primeraFavorita: Cancion? = playlist.find { it.favorita }
    println("\nPrimera favorita: ${primeraFavorita?.titulo ?: "Sin favoritas"}")

    // Ejercicio 4 — sumar
    val duracionTotal = playlist.sumOf { it.duracionSegundos }
    val duracionEnMinutos = duracionTotal.let { it / 60.0 }
    println("\nDuración total: $duracionTotal s (${"%.1f".format(duracionEnMinutos)} min)")

    // Ejercicio 5 — apply
    val nueva = Cancion("Hey Jude", "The Beatles", 431, "Rock").apply {
        println("\nAñadida: $titulo")
    }

    // Ejercicio 6 — also en una cadena
    val favoritasOrdenadas = playlist
        .filter { it.favorita }
        .also { println("Favoritas encontradas: ${it.size}") }
        .sortedBy { it.titulo }
    println("Favoritas (orden alfabético): ${favoritasOrdenadas.map { it.titulo }}")

    // Ejercicio 7 — llamada con condición nombrada + trailing lambda
    println("\n--- Canciones de Pop ---")
    playlist.procesarSegun(condicion = { it.genero == "Pop" }) { cancion ->
        println(cancion.titulo)
    }
}
```

## Qué comprobar en cada elemento obligatorio

- **`data class Cancion`** → 5 propiedades, `favorita: Boolean` con valor por defecto `false`.
- **`sortedBy`/`sortedByDescending`** → dos listas nuevas (`porDuracion`, `porArtista`); `playlist` no se modifica.
- **`count`/`any`/`all`** → cada una devuelve directamente el tipo esperado (`Int`, `Boolean`, `Boolean`) sin variables acumuladoras.
- **`find`/`firstOrNull`** → `primeraFavorita` es `Cancion?`; se resuelve con `?:` al imprimir, sin `if`.
- **`sumOf`** → `duracionTotal` sustituye a un `var total = 0; for (...) { total += ... }`.
- **`apply`** → `nueva` es la propia `Cancion` construida (no lo que devuelve el `println` de dentro); dentro del bloque, `titulo` se usa como `this.titulo` implícito.
- **`also` en una cadena** → `favoritasOrdenadas` sigue siendo el resultado de `filter().sortedBy()`; el `also { }` del medio solo imprime, no altera el valor que fluye por la cadena (usa `it`, no `this`).
- **Función propia con dos lambdas** → `procesarSegun` recibe `condicion` y `accion`; se llama con `condicion` nombrada dentro de los paréntesis y `accion` como *trailing lambda*, el único orden válido.
- **Cero bucles `for`** → toda la solución usa las funciones de colección de las sesiones 3 y 4.