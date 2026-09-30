---
title: "Biblioteca de barrio — Enunciado (Sesión 5)"
unidad: UD2
sesion: 5
orden: 5
tipo: enunciado
---

## Contexto

Práctica de **consolidación** de la Sesión 5: no introduce contenido nuevo. Repasa, sobre un dominio nuevo (una pequeña biblioteca de barrio), todo lo visto en las sesiones 1 a 4 —herencia e interfaces, null-safety y data classes, funciones de extensión y lambdas, y las funciones de colecciones y *scope functions* de la sesión 4—. Puedes resolverla en el [Kotlin Playground](https://play.kotlinlang.org/) o en cualquier proyecto Kotlin con una función `main()`.

<div class="callout-warning">
Esta práctica es la última antes de entrar en la UD3 (Jetpack Compose): no debería quedar ningún bloque de las sesiones 1-4 sin usar aquí. Si al terminar hay algún ejercicio que no sabéis resolver con soltura, es el momento de repasarlo, no de arrastrarlo a la UD3.
</div>

## Enunciado

Vas a modelar, en Kotlin puro, el catálogo y los préstamos de una pequeña biblioteca de barrio.

### Ejercicio 0 — El modelo (herencia e interfaces, sesión 1)

Crea una interfaz `Prestable` con un único método `descripcionCorta(): String`.

Crea una clase abierta `Publicacion(val titulo: String, val autor: String)` que implemente `Prestable`, con `descripcionCorta()` devolviendo `"<titulo>, de <autor>"`.

Crea dos subclases de `Publicacion`:
- `Libro(titulo: String, autor: String, val paginas: Int)`, que no sobrescribe `descripcionCorta()` (hereda la de `Publicacion`).
- `Revista(titulo: String, autor: String, val numero: Int)`, que **sí** sobrescribe `descripcionCorta()` para devolver `"<descripción heredada> (nº <numero>)"` reutilizando la implementación del padre con `super`.

### Ejercicio 1 — El préstamo (data class y null-safety, sesión 2)

Crea `data class Prestamo(val publicacion: Publicacion, val socio: String, val diasPrestado: Int, val devuelto: Boolean = false, val observaciones: String? = null)`.

Crea una lista `prestamos: List<Prestamo>` con al menos 6 préstamos, mezclando `Libro` y `Revista`, con al menos dos `devuelto = true`, al menos uno con más de 30 días de préstamo, y al menos uno con `observaciones` distinto de `null`.

### Ejercicio 2 — Funciones de extensión y Elvis (sesión 3)

Escribe una función de extensión `Prestamo.resumen(): String`, de expresión única, que devuelva un texto tipo `"Drácula (Bram Stoker) — Ana (12 días)"`, usando `descripcionCorta()` de la publicación. Si `observaciones` no es `null`, debe añadirse entre paréntesis al final (usa el operador Elvis para la parte por defecto, es decir, no añadir nada si es `null`).

### Ejercicio 3 — filter, map y forEach encadenados (sesión 3)

Escribe una función de extensión `List<Prestamo>.sinDevolver(): List<Prestamo>` con `filter`. Encadénala con `map { it.resumen() }` y `forEach` para imprimir, en una sola expresión, el resumen de todos los préstamos pendientes de devolver.

### Ejercicio 4 — Ordenar, contar y buscar (sesión 4)

- Usa `sortedByDescending` para obtener la lista de préstamos ordenada por `diasPrestado`, de más atrasado a menos.
- Usa `count { }` para saber cuántos préstamos están devueltos.
- Usa `any { }` para comprobar si hay algún préstamo de más de 30 días sin devolver (combina `diasPrestado > 30` y `!devuelto`).
- Usa `find` (o `firstOrNull`) para localizar el primer préstamo de una `Revista`. Guarda el resultado en una variable de tipo `Prestamo?` e imprime su resumen, o `"No hay préstamos de revistas"` si no hay ninguno.

### Ejercicio 5 — apply y also (sesión 4)

Crea un nuevo `Prestamo` y, con `apply { }`, imprime dentro del propio bloque un mensaje `"Nuevo préstamo registrado: <resumen>"` antes de guardarlo en una variable.

Encadena una expresión que (a) filtre solo los préstamos devueltos, (b) los registre con `also { }` imprimiendo cuántos son, y (c) los ordene alfabéticamente por el nombre del socio.

### Ejercicio 6 (integrador) — Tu propia función con dos lambdas

Escribe una función de extensión con **dos** parámetros lambda:

```kotlin
fun List<Prestamo>.procesarSegun(
    condicion: (Prestamo) -> Boolean,
    accion: (Prestamo) -> Unit
) {
    // TODO
}
```

Llámala para imprimir el resumen de todos los préstamos de un socio concreto, pasando la condición con nombre (`condicion = { ... }`) y la acción como *trailing lambda*.

## Requisitos técnicos obligatorios

- Jerarquía `Prestable` / `Publicacion` / `Libro` / `Revista`, con `override` y `super` usados correctamente en `Revista`.
- `data class Prestamo` con al menos una propiedad nullable, resuelta con el operador Elvis en algún punto.
- Al menos dos funciones de extensión (una de expresión única) y una función propia con dos parámetros lambda, llamada con el último en sintaxis de *trailing lambda*.
- Al menos un uso de `filter`/`map`/`forEach` encadenados, uno de `sortedByDescending`, uno de `count`/`any`, uno de `find`/`firstOrNull`, uno de `apply` y uno de `also`.
- Cero bucles `for`.

## Criterios de evaluación

- El código compila y se ejecuta sin errores.
- Aparecen y se usan correctamente todos los elementos obligatorios de las sesiones 1 a 4.
- La herencia y el polimorfismo de `Revista`/`Libro` están bien resueltos (uso correcto de `override`/`super`).
- El código sigue las convenciones de la unidad (`val` por defecto, sin `!!` innecesarios, nombres en camelCase).

<div class="callout-info">
Con esto se cierra la parte de Kotlin "puro" de la unidad. A partir de la próxima sesión entramos en la UD3: Jetpack Compose. Si no habéis terminado todavía la práctica <em>Catálogo de dispositivos</em> (propuesta hoy como trabajo autónomo), es un buen momento para dejarla resuelta antes de esa primera sesión.
</div>
