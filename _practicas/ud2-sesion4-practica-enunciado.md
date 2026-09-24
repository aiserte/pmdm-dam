---
title: "Playlist — Enunciado (Sesión 4)"
unidad: UD2
sesion: 4
orden: 4
tipo: enunciado
---

## Contexto

Práctica de consolidación de la **Sesión 4** (*Funciones de extensión y lambdas: consolidación*). Cambiamos de dominio respecto a las sesiones 2 y 3: aquí trabajarás con una playlist de canciones. Puedes resolverla en el [Kotlin Playground](https://play.kotlinlang.org/) o en cualquier proyecto Kotlin con una función `main()`.

<div class="callout-warning">
Usa lo visto hasta ahora: funciones de extensión, lambdas y <em>trailing lambda</em>, <code>filter</code>/<code>map</code>/<code>forEach</code> (sesión 3), y las funciones de esta sesión —<code>sortedBy</code>/<code>sortedByDescending</code>, <code>count</code>/<code>any</code>/<code>all</code>/<code>none</code>, <code>find</code>/<code>firstOrNull</code>, <code>sumOf</code> y las <em>scope functions</em> (<code>let</code>, <code>run</code>, <code>with</code>, <code>apply</code>, <code>also</code>)—. Todavía <strong>no</strong> hemos visto coroutines ni nada de Jetpack Compose.
</div>

## Enunciado

Vas a modelar, en Kotlin puro, la playlist de una app de música.

### Ejercicio 0 — El modelo

Crea `data class Cancion(val titulo: String, val artista: String, val duracionSegundos: Int, val genero: String, val favorita: Boolean = false)`.

Crea una lista `playlist: List<Cancion>` con al menos 6 canciones, variando artista, género y duración, con al menos 2 marcadas como `favorita = true`.

### Ejercicio 1 — Ordenar

Usa `sortedBy`/`sortedByDescending` para obtener: (a) la playlist ordenada por duración de menor a mayor, y (b) la playlist ordenada alfabéticamente por artista.

### Ejercicio 2 — Contar y comprobar condiciones

- Cuenta cuántas canciones hay de un género concreto, con `count { }`.
- Comprueba con `any { }` si hay alguna canción de más de 5 minutos (300 segundos).
- Comprueba con `all { }` si todas las canciones duran más de 1 minuto.

### Ejercicio 3 — Buscar

Usa `find` (o `firstOrNull`) para localizar la primera canción favorita. Guarda el resultado en una variable de tipo `Cancion?` e imprime su título, o `"Sin favoritas"` si no hay ninguna (puedes reutilizar el patrón de extensión sobre receptor nullable de la sesión 3, o resolverlo con `?.` / `?:`).

### Ejercicio 4 — Sumar

Usa `sumOf { }` para calcular la duración total de la playlist en segundos, y muéstrala también convertida a minutos (puedes usar `let` para hacer la conversión sin crear una variable intermedia).

### Ejercicio 5 — `apply` para construir

Crea una nueva `Cancion` y, con `apply { }`, imprime dentro del propio bloque un mensaje `"Añadida: <titulo>"` antes de guardarla en una variable.

### Ejercicio 6 — `also` en una cadena

Escribe una expresión que, encadenada, (a) filtre solo las canciones favoritas, (b) las registre con `also { }` imprimiendo cuántas son, y (c) las ordene por título. Ejemplo de forma (complétalo): `playlist.filter { it.favorita }.also { ... }.sortedBy { it.titulo }`.

### Ejercicio 7 (integrador) — Tu propia función con dos lambdas

Escribe una función de extensión con **dos** parámetros lambda:

```kotlin
fun List<Cancion>.procesarSegun(
    condicion: (Cancion) -> Boolean,
    accion: (Cancion) -> Unit
) {
    // TODO
}
```

Llámala para imprimir el título de todas las canciones de un género concreto, pasando la condición con nombre (`condicion = { ... }`) y la acción como *trailing lambda*.

## Requisitos técnicos obligatorios

- `data class Cancion` con al menos 5 propiedades, una de ellas `Boolean`.
- Al menos un uso de `sortedBy`/`sortedByDescending`, uno de `count`/`any`/`all`, uno de `find`/`firstOrNull` y uno de `sumOf`.
- Al menos un uso de `apply` (configurar) y uno de `also` (efecto secundario en una cadena).
- Una función propia con dos parámetros lambda, llamada con el último en sintaxis de *trailing lambda*.
- Cero bucles `for`.

## Criterios de evaluación

- El código compila y se ejecuta sin errores.
- Aparecen y se usan correctamente todos los elementos obligatorios.
- Se elige la *scope function* adecuada en cada caso (no se usa `apply` donde tocaría `also`, ni al revés).
- El código sigue las convenciones de la unidad (`val` por defecto, sin `!!` innecesarios).

<div class="callout-info">
Con esto se cierra el bloque de Kotlin "moderno" de la unidad (extensión, lambdas, colecciones, <em>scope functions</em>). La sesión 5 cambia de tema: una introducción ligera a las <em>coroutines</em>.
</div>