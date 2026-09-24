---
title: "Soporte técnico: extensión y lambdas — Enunciado (Sesión 3)"
unidad: UD2
sesion: 3
orden: 3
tipo: enunciado
---

## Contexto

Práctica de consolidación de la **Sesión 3** (*Funciones de extensión y lambdas*). Continúa directamente sobre el `Ticket` y el panel de tickets de la práctica de la **Sesión 2** (*Soporte técnico*): si no la tienes a mano, parte del `data class Ticket` y del `panel: MutableList<Ticket>` del "Ejercicio 6" de aquella práctica.

<div class="callout-warning">
Usa lo visto hasta ahora: funciones de extensión (sobre tipo no nulo y sobre tipo nullable), lambdas y <code>it</code>, sintaxis de <em>trailing lambda</em>, y <code>filter</code>/<code>map</code>/<code>forEach</code> sobre listas. Todavía <strong>no</strong> hemos visto otras funciones de colecciones como <code>sortedBy</code> o <code>count</code>, ni las <em>scope functions</em> (<code>let</code>, <code>apply</code>...) más allá de <code>?.let { }</code> —eso es la sesión 4—.
</div>

## Enunciado

Vas a ampliar el panel de tickets de la sesión anterior con funciones de extensión y lambdas.

### Ejercicio 1 — `sinAsignar()`

Escribe una función de extensión `List<Ticket>.sinAsignar(): List<Ticket>` que use `filter` para devolver solo los tickets con `tecnicoAsignado == null`.

### Ejercicio 2 — `resumen()`

Escribe una función de extensión `Ticket.resumen(): String`, de expresión única, que devuelva un texto con el formato `"#3 — Impresora atascada (sin asignar)"` o `"#2 — Ordenador bloqueado (asignado a Marta)"`. Usa el operador Elvis (visto en la sesión 2) para la parte final.

### Ejercicio 3 — `map` + `forEach`

Usa `map` sobre el panel de tickets para obtener la lista de todos los `resumen()`, y `forEach` para imprimirlos por pantalla, uno por línea.

### Ejercicio 4 — Encadenado

Encadena `sinAsignar()` y `map { it.resumen() }` para imprimir solo el resumen de los tickets pendientes de asignar, en una sola expresión.

### Ejercicio 5 — `resueltos()`

Escribe una segunda función de extensión, `List<Ticket>.resueltos(): List<Ticket>`, que devuelva solo los tickets con `resuelta == true`. Imprime cuántos tickets resueltos hay usando `.size`.

### Ejercicio 6 — Extensión sobre receptor nullable

En algún punto del programa manejaréis un `Ticket?` (por ejemplo, el resultado de buscar un ticket por id que podría no existir). Escribe una función de extensión `Ticket?.descripcionSegura(): String` que devuelva `"(sin ticket)"` si el receptor es `null`, o `ticket.resumen()` si no lo es —sin usar `?.` al llamarla, igual que en el ejemplo de `String?.estaVacioOEsNulo()` de la sesión—.

### Ejercicio 7 (integrador) — Tu propia función de orden superior

Escribe una función de extensión que sea **a la vez** función de orden superior:

```kotlin
fun List<Ticket>.aplicarATodos(accion: (Ticket) -> Unit) {
    // TODO: usa forEach por dentro
}
```

Llámala con sintaxis de *trailing lambda* sobre el panel completo para imprimir, para cada ticket, una línea con su `resumen()` y, debajo, `"  -> URGENTE"` si lleva más de 2 días sin asignar (puedes simular esto con un `Boolean` extra en el `Ticket` o con un valor fijo para el ejercicio).

## Requisitos técnicos obligatorios

- Dos funciones de extensión sobre `List<Ticket>` que usan `filter` (`sinAsignar`, `resueltos`).
- Una función de extensión de expresión única sobre `Ticket` que usa el operador Elvis (`resumen`).
- Una función de extensión sobre receptor **nullable** (`Ticket?`), llamada sin `?.`.
- Uso de `map` y de `forEach` por separado, y al menos una vez encadenados.
- Una función propia que reciba una lambda como parámetro (`(Ticket) -> Unit` o similar), llamada con sintaxis de *trailing lambda*.
- Cero bucles `for` en todo el ejercicio: todo se resuelve con funciones de orden superior.

## Criterios de evaluación

- El código compila y se ejecuta sin errores (en Kotlin Playground o Android Studio).
- Aparecen y se usan correctamente todos los elementos obligatorios listados arriba.
- Las funciones de extensión están bien definidas: expresión única cuando el cuerpo es una sola expresión, `this` usado correctamente.
- No aparece ningún bucle `for` ni ninguna función no vista todavía (`sortedBy`, `let`, `apply`...).

<div class="callout-info">
Trabajo para casa: tráelo resuelto a la sesión 4, donde seguiremos ampliando este mismo panel de tickets con más funciones de colecciones y las <em>scope functions</em>.
</div>