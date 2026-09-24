---
title: "Funciones de extensión y lambdas (consolidación)"
unit: UD2
order: 4
duration: "2h"
---

## Objetivos

- Usar más funciones de la librería estándar sobre colecciones (`sortedBy`, `count`, `any`/`all`/`none`, `find`/`firstOrNull`, `sumOf`) para resolver tareas habituales sin bucles.
- Conocer la familia de *scope functions* (`let`, `apply`, `also`, `run`, `with`) y saber elegir la adecuada según el caso.
- Escribir funciones propias de orden superior con más de un parámetro lambda.

<div class="callout-info">
Esta sesión no introduce ningún concepto radicalmente nuevo: amplía el vocabulario de funciones de colección que ya conocéis (<code>filter</code>, <code>map</code>, <code>forEach</code>) y formaliza algo que ya usasteis sin nombre en la sesión 2 —<code>?.let { }</code>— como parte de una familia más amplia. El objetivo es llegar a la UD3 (Jetpack Compose) reconociendo este vocabulario de forma automática.
</div>

## 4.1 Repaso rápido

En la sesión 3 vimos tres funciones de orden superior sobre colecciones —`filter`, `map`, `forEach`— y cómo encadenarlas. Hoy añadimos seis funciones más, todas construidas sobre la misma idea: **una función de la librería estándar que recibe una lambda para decidir qué hacer con cada elemento**.

## 4.2 Ordenar: `sortedBy` y `sortedByDescending`

```kotlin
data class Cancion(val titulo: String, val artista: String, val duracionSegundos: Int)

val lista = listOf(
    Cancion("Bohemian Rhapsody", "Queen", 355),
    Cancion("Imagine", "John Lennon", 183),
    Cancion("Stairway to Heaven", "Led Zeppelin", 482)
)

val porDuracion = lista.sortedBy { it.duracionSegundos }
val masLargasPrimero = lista.sortedByDescending { it.duracionSegundos }
```

`sortedBy` devuelve una lista nueva ordenada de menor a mayor según el valor que devuelva la lambda; `sortedByDescending` la misma idea, pero de mayor a menor. La lista original no se modifica, igual que con `filter`/`map`.

## 4.3 Comprobar condiciones: `count`, `any`, `all`, `none`

```kotlin
val cortas = lista.count { it.duracionSegundos < 200 }        // cuántas cumplen la condición (Int)
val hayAlgunaCorta = lista.any { it.duracionSegundos < 200 }   // ¿al menos una? (Boolean)
val todasSonLargas = lista.all { it.duracionSegundos > 300 }   // ¿todas? (Boolean)
val ningunaEsDeQueen = lista.none { it.artista == "Queen" }    // ¿ninguna? (Boolean)
```

Estas cuatro funciones sustituyen patrones que en Java se resolvían con un bucle y una variable acumuladora (un contador, o un `boolean encontrado = false` que se pone a `true` dentro del bucle). En Kotlin no hace falta esa variable intermedia: la función ya devuelve directamente el resultado que buscáis.

<div class="callout-warning">
<code>count { }</code> (con lambda) es distinto de <code>.size</code>: <code>count { condición }</code> cuenta cuántos elementos cumplen la condición; <code>.size</code> cuenta todos los elementos de la lista. <code>lista.count()</code> sin lambda (los paréntesis vacíos) es equivalente a <code>.size</code>, pero es más habitual usar directamente <code>.size</code> cuando no hay condición.
</div>

## 4.4 Buscar un elemento: `find` y `firstOrNull`

```kotlin
val primeraCorta: Cancion? = lista.find { it.duracionSegundos < 200 }
val primeraDeQueen: Cancion? = lista.firstOrNull { it.artista == "Queen" }
```

`find` y `firstOrNull` hacen exactamente lo mismo (son sinónimos en la librería estándar de Kotlin: `find` está definido internamente como una llamada a `firstOrNull`) y devuelven el **primer** elemento que cumple la condición, o `null` si ninguno la cumple. Por eso el tipo de retorno es siempre nullable (`Cancion?`, no `Cancion`).

<div class="callout-warning">
Existe también <code>.first { condición }</code>, sin el <code>OrNull</code>: hace lo mismo pero <strong>lanza una excepción</strong> (<code>NoSuchElementException</code>) si no encuentra ningún elemento, en vez de devolver <code>null</code>. Usad <code>find</code>/<code>firstOrNull</code> por defecto —combina de forma natural con lo que ya sabéis de <code>null</code>-safety (<code>?.</code>, <code>?:</code>)— y reservad <code>first { }</code> para cuando estéis seguros de que el elemento tiene que existir y preferís que falle ruidosamente si no es así.
</div>

## 4.5 Sumar un valor calculado: `sumOf`

```kotlin
val duracionTotal: Int = lista.sumOf { it.duracionSegundos }
```

`sumOf` recorre la lista, aplica la lambda a cada elemento (que debe devolver un número) y suma todos los resultados. Sustituye el patrón `var total = 0; for (x in lista) { total += ... }`.

## 4.6 Tabla comparativa con Java

| Tarea | Java (bucle o Stream) | Kotlin |
|---|---|---|
| Ordenar | `list.sort(Comparator.comparingInt(...))` | `.sortedBy { ... }` |
| Contar los que cumplen | `long n = list.stream().filter(...).count();` | `.count { ... }` |
| ¿Alguno cumple? | `list.stream().anyMatch(...)` | `.any { ... }` |
| ¿Todos cumplen? | `list.stream().allMatch(...)` | `.all { ... }` |
| Buscar el primero (o null) | `list.stream().filter(...).findFirst().orElse(null)` | `.find { ... }` / `.firstOrNull { ... }` |
| Sumar un valor calculado | `list.stream().mapToInt(...).sum()` | `.sumOf { ... }` |

## 4.7 Las *scope functions*: una lambda, un contexto

Ya usasteis `?.let { }` en la sesión 2 para ejecutar código solo cuando un valor no era `null`. `let` no es un caso especial: pertenece a una familia de cinco funciones —`let`, `run`, `with`, `apply`, `also`— que la librería estándar de Kotlin ofrece para un mismo propósito general: **ejecutar un bloque de código en el "contexto" de un objeto**, sin tener que repetir su nombre en cada línea.

Las cinco funcionan sobre el mismo mecanismo (reciben una lambda), y se diferencian en dos preguntas:

1. Dentro de la lambda, ¿el objeto se llama `it` (parámetro con nombre) o `this` (receptor implícito, como en una función de extensión)?
2. ¿Qué devuelve la función: el resultado de la lambda, o el propio objeto original?

| Función | Dentro de la lambda | Devuelve | Uso típico |
|---|---|---|---|
| `let` | `it` | resultado de la lambda | transformar un valor, o ejecutar código solo si no es `null` (`?.let { }`) |
| `run` | `this` | resultado de la lambda | agrupar varias operaciones sobre un objeto y quedarte con un resultado final |
| `with` | `this` | resultado de la lambda | igual que `run`, pero **no** es una función de extensión: se llama `with(objeto) { ... }` |
| `apply` | `this` | el propio objeto | configurar un objeto recién creado (estilo *builder*) |
| `also` | `it` | el propio objeto | efecto secundario (log, comprobación) sin romper una cadena de llamadas |

Veámoslas con el mismo objeto, `Cancion`:

```kotlin
// let: transformar el resultado. Devuelve lo que devuelva la lambda.
val duracionEnMinutos = lista.first().let { it.duracionSegundos / 60.0 }

// run: agrupar operaciones sobre "this" y quedarte con un resultado.
val descripcion = lista.first().run {
    "$titulo dura ${duracionSegundos / 60} min"   // this. es opcional dentro de run
}

// with: como run, pero no es una función de extensión (se pasa el objeto como argumento)
val descripcion2 = with(lista.first()) {
    "$titulo — $artista"
}

// apply: configurar un objeto y quedarte con el objeto ya configurado
val cancionNueva = Cancion("Hey Jude", "The Beatles", 431).apply {
    println("Canción creada: $titulo")   // this. es opcional
}
// cancionNueva es la propia Cancion, no lo que devuelva la lambda

// also: efecto secundario (por ejemplo, un log) sin romper la cadena
val cancionRegistrada = cancionNueva.also {
    println("Registrando en el historial: ${it.titulo}")
}
// cancionRegistrada es también la propia Cancion; "it" porque also usa it, no this
```

<div class="callout-warning">
No hace falta memorizar la tabla de golpe: en la práctica, casi todo se resuelve con dos preguntas. <strong>¿Quiero quedarme con el objeto original o con un resultado nuevo?</strong> Si es el objeto original → <code>apply</code> (para configurarlo) o <code>also</code> (para un efecto secundario). Si es un resultado nuevo → <code>let</code> o <code>run</code>/<code>with</code>. <strong>¿Necesito nombrar el receptor (<code>it</code>) o me basta con <code>this</code> implícito?</strong> Si vais a llamar a un método propio del objeto con su nombre corto, usad las de <code>this</code> (<code>run</code>, <code>with</code>, <code>apply</code>); si preferís dejar explícito sobre qué operáis (por ejemplo, con un nombre más claro que <code>it</code>), usad <code>let</code>/<code>also</code>.
</div>

## 4.8 Dónde vais a ver esto en Android

En Jetpack Compose y en el resto del SDK de Android, `apply` es extremadamente habitual para configurar objetos justo después de crearlos (por ejemplo, un `Intent` con varias propiedades), y `let` es la forma estándar de trabajar con un valor nullable sin necesidad de un `if`. No hace falta ver ejemplos de Android todavía —eso empieza en la UD3—; basta con reconocer que, cuando aparezcan, no serán sintaxis nueva.

## 4.9 Vuestras propias funciones de orden superior con varios parámetros

En la sesión 3 escribisteis una función de orden superior con **una** lambda como parámetro (`aplicarATodos(accion: (Ticket) -> Unit)`). Una función puede recibir más de una lambda, cada una con su propio propósito:

```kotlin
fun List<Cancion>.procesarSegun(
    condicion: (Cancion) -> Boolean,
    accion: (Cancion) -> Unit
) {
    for (cancion in this) {
        if (condicion(cancion)) {
            accion(cancion)
        }
    }
}

// Solo el ÚLTIMO parámetro lambda puede salir con sintaxis de trailing lambda;
// el resto se pasan con nombre, dentro de los paréntesis.
lista.procesarSegun(
    condicion = { it.duracionSegundos > 300 }
) { cancion ->
    println("${cancion.titulo} es larga")
}
```

<div class="callout-info">
Fijaos en el patrón: cuando una función tiene varios parámetros lambda, se nombran los primeros (<code>condicion = { ... }</code>) y solo el último puede escribirse como <em>trailing lambda</em> fuera de los paréntesis. Es exactamente lo que hace <code>Button(onClick = { ... }) { ... }</code> en Compose: <code>onClick</code> con nombre, el contenido como <em>trailing lambda</em>.
</div>

## Tabla resumen de la sesión

| Necesito... | Función |
|---|---|
| Ordenar una lista según un criterio | `sortedBy` / `sortedByDescending` |
| Contar cuántos cumplen una condición | `count { }` |
| Saber si al menos uno/todos/ninguno cumple | `any` / `all` / `none` |
| Buscar el primero que cumple (o `null`) | `find` / `firstOrNull` |
| Sumar un valor calculado por elemento | `sumOf { }` |
| Ejecutar código con un valor y quedarme con un resultado | `let` (`it`) / `run` o `with` (`this`) |
| Configurar un objeto y quedarme con él | `apply` (`this`) |
| Efecto secundario sin romper la cadena | `also` (`it`) |

<div class="callout-practica">
<strong>Ejercicio de sesión</strong>: sobre el panel de tickets (sesiones 2 y 3), sin mirar la solución.

1. Usa <code>count { }</code> para saber cuántos tickets están resueltos.
2. Usa <code>any { }</code> para comprobar si hay algún ticket sin asignar.
3. Usa <code>sortedBy { }</code> para ordenar el panel por <code>id</code> descendente (pista: <code>sortedByDescending</code>).
4. Crea un <code>Ticket</code> nuevo con <code>apply { }</code>, imprimiendo dentro del bloque un mensaje de confirmación.

15 minutos de trabajo autónomo; se corrige en los últimos 10 minutos de la sesión.
</div>

## Para la próxima sesión

Sesión 5: una introducción ligera a las *coroutines*, lo justo para entender por qué existen antes de retomarlas en profundidad más adelante en el módulo (conectividad). Después, la sesión 6 cierra la unidad con la práctica integradora de Compose (*Catálogo de dispositivos*), antes de entrar de lleno en la UD3.