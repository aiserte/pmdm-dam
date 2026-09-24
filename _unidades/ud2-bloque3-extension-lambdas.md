---
title: "Funciones de extensión y lambdas"
unit: UD2
order: 3
duration: "2h"
---

## Objetivos

- Crear y usar funciones de extensión sobre tipos que ya existen, entendiendo qué son realmente "por debajo".
- Escribir y leer lambdas en Kotlin: sintaxis, el parámetro implícito `it` y la sintaxis de *trailing lambda*.
- Aplicar funciones de orden superior (`map`, `filter`, `forEach`) sobre colecciones, sustituyendo bucles `for` manuales por código declarativo.

<div class="callout-info">
Esta sesión es la más importante de la unidad de cara a lo que viene: en la UD3 (Jetpack Compose) prácticamente todo el código de interfaz se escribe pasando lambdas como argumentos. Si hoy quedan dudas sobre `it` o sobre por qué un `{ ... }` puede ir fuera de los paréntesis, merece la pena resolverlas antes de avanzar.
</div>

## 3.1 Por qué existen las funciones de extensión

Imaginad que queréis añadir un método útil a una clase que no controláis —por ejemplo, `String`, que además en Kotlin es `final` y no se puede heredar—. En Java, la solución habitual es una clase de utilidades con métodos estáticos:

```java
public class StringUtils {
    public static boolean esEmailValido(String s) {
        return s.contains("@") && s.contains(".");
    }
}

// Uso:
boolean valido = StringUtils.esEmailValido(correo);
```

Funciona, pero se lee "al revés": la operación no aparece junto al dato sobre el que actúa, sino en una clase aparte que hay que conocer y localizar. Una **función de extensión** en Kotlin permite escribir exactamente la misma idea, pero con la sintaxis de un método real:

```kotlin
fun String.esEmailValido(): Boolean {
    return this.contains("@") && this.contains(".")
}

val correo = "alumno@instituto.es"
println(correo.esEmailValido())   // true
```

`String` es el **tipo receptor** (el tipo que "recibe" la nueva función) y, dentro del cuerpo, `this` hace referencia al objeto sobre el que se ha llamado —aquí, el propio `correo`—.

## 3.2 Sintaxis general y un ejemplo con vuestras propias clases

La forma general es:

```kotlin
fun TipoReceptor.nombreFuncion(parametros): TipoRetorno {
    // this = el objeto sobre el que se ha llamado
}
```

No hace falta que el tipo receptor sea de la librería estándar: funciona igual con vuestras propias `data class`. Retomando `Ticket` de la práctica de la sesión anterior:

```kotlin
data class Ticket(
    val id: Int,
    val descripcion: String,
    val tecnicoAsignado: String? = null,
    val resuelta: Boolean = false
)

fun Ticket.estaAsignado(): Boolean = tecnicoAsignado != null

val t = Ticket(1, "Impresora atascada", tecnicoAsignado = "Marta")
println(t.estaAsignado())   // true
```

<div class="callout-info">
Fijaos en la sintaxis <code>fun Ticket.estaAsignado(): Boolean = tecnicoAsignado != null</code>: es una <strong>función de expresión única</strong> (el cuerpo es una sola expresión, sin llaves ni <code>return</code>). Es habitual en Kotlin para funciones cortas; equivale a escribir <code>{ return tecnicoAsignado != null }</code>, pero más compacto.
</div>

## 3.3 Qué ocurre "por debajo": no es herencia, es azúcar sintáctico

Esto es importante y suele generar confusión: una función de extensión **no modifica la clase real** ni añade nada a su jerarquía de herencia. El compilador la traduce, por debajo, en una función normal (estática) que recibe el receptor como primer argumento oculto. `correo.esEmailValido()` se compila, en esencia, como `esEmailValido(correo)`.

La consecuencia práctica es que **se resuelve en tiempo de compilación según el tipo declarado de la variable, no según el tipo real del objeto en ejecución** —al contrario que un método sobrescrito con `override`, que sí es polimórfico—:

```kotlin
open class Animal
class Perro : Animal()

fun Animal.sonido() = "..."
fun Perro.sonido() = "Guau"

fun main() {
    val mascota: Animal = Perro()   // tipo declarado: Animal; objeto real: Perro
    println(mascota.sonido())       // imprime "...", NO "Guau"
}
```

<div class="callout-warning">
Si necesitáis comportamiento distinto según la subclase real en tiempo de ejecución, no uséis una función de extensión: usad un método normal de la clase, con <code>open</code>/<code>override</code> (lo visteis en el bloque 1). Las funciones de extensión son para <strong>añadir</strong> utilidades, no para sustituir el polimorfismo.
</div>

Otra consecuencia del mismo motivo: si la clase ya tiene un miembro real con el mismo nombre y firma que vuestra función de extensión, **siempre gana el miembro real**. La extensión nunca puede "tapar" un método existente.

## 3.4 Extensión sobre tipos nullable

El tipo receptor puede ser nullable (`Tipo?`), lo que permite escribir la comprobación de `null` dentro de la propia función en vez de en cada punto donde se usa:

```kotlin
fun String?.estaVacioOEsNulo(): Boolean {
    return this == null || this.isEmpty()
}

val a: String? = null
val b: String? = "hola"

println(a.estaVacioOEsNulo())   // true — se puede llamar sin ?. aunque sea null
println(b.estaVacioOEsNulo())   // false
```

Fijaos en que `a.estaVacioOEsNulo()` se llama sin `?.`, aunque `a` sea `String?`: es la propia función la que se ocupa de comprobar `this == null` por dentro, así que no hace falta protegerla desde fuera.

<div class="callout-info">
En Android vais a encontrar funciones de extensión constantemente: <code>Modifier.padding(16.dp)</code>, <code>Context.showToast(...)</code>, etc. Ya sabéis exactamente qué son: no son "magia" del framework, son funciones normales definidas con <code>fun Receptor.nombre(...)</code>, como las de esta sesión.
</div>

## 3.5 Lambdas: funciones como valores

Hasta ahora, una función se definía con `fun` y se llamaba por su nombre. Una **lambda** es una función *sin nombre* que se puede guardar en una variable, devolver desde otra función o pasar como argumento, igual que un `Int` o un `String`.

En Java, esto se resolvía tradicionalmente con clases anónimas que implementaban una interfaz funcional, y desde Java 8 con lambdas ligadas siempre a una interfaz de un solo método abstracto (`Runnable`, `Comparator<T>`, o una interfaz `@FunctionalInterface` propia):

```java
// Java: interfaz funcional + lambda
Comparator<String> porLongitud = (a, b) -> a.length() - b.length();
```

En Kotlin, las funciones son un tipo de dato de primera clase; no hace falta ninguna interfaz intermedia. El **tipo función** se escribe `(TiposParametros) -> TipoRetorno`:

```kotlin
val cuadrado: (Int) -> Int = { numero -> numero * numero }
println(cuadrado(4))   // 16
```

Aquí `(Int) -> Int` es el tipo (una función que recibe un `Int` y devuelve un `Int`), y `{ numero -> numero * numero }` es la lambda en sí: entre llaves, los parámetros antes de la flecha `->` y el cuerpo después.

Cuando la lambda recibe **un único parámetro**, Kotlin permite omitir su nombre y referirse a él como `it`:

```kotlin
val cuadrado: (Int) -> Int = { it * it }
```

<div class="callout-warning">
<code>it</code> solo está disponible cuando la lambda tiene exactamente un parámetro y no le habéis dado nombre explícito. Si anidáis una lambda dentro de otra (por ejemplo, un <code>filter</code> dentro de un <code>map</code>), <strong>no uséis <code>it</code> en las dos</strong>: la interna "tapa" a la externa y el código se vuelve imposible de leer. En ese caso, nombrad los parámetros explícitamente: <code>{ ticket -> ... { tecnico -> ... } }</code>.
</div>

## 3.6 Sintaxis de *trailing lambda*: la clave para entender Compose

Cuando el **último parámetro** de una función es una lambda, Kotlin permite sacarla fuera de los paréntesis. Si además es el **único** parámetro, los paréntesis se pueden omitir del todo. Veámoslo progresivamente con una función propia:

```kotlin
fun repetir(veces: Int, accion: () -> Unit) {
    for (i in 1..veces) accion()
}

// Sintaxis "normal": la lambda es un argumento más, dentro de los paréntesis
repetir(3, { println("Hola") })

// Trailing lambda: al ser el último parámetro, se saca fuera de los paréntesis
repetir(3) { println("Hola") }
```

Y cuando la lambda es el único parámetro:

```kotlin
// Sintaxis "normal"
modelos.forEach({ println(it) })

// Sin paréntesis: al ser el único parámetro, no hace falta ni escribirlos
modelos.forEach { println(it) }
```

Esta es la razón por la que el código de Jetpack Compose se ve como se ve. En la UD3 escribiréis cosas como:

```kotlin
Button(onClick = { println("Pulsado") }) {
    Text("Aceptar")
}
```

`Button` recibe **dos** parámetros que son lambdas: `onClick` (con nombre, dentro de los paréntesis, porque no es el último) y el contenido del botón, que al ser el último parámetro sale como *trailing lambda* fuera de los paréntesis. No hay ninguna sintaxis nueva que aprender: es exactamente el mismo mecanismo de `repetir(3) { ... }`, aplicado dos veces en la misma llamada.

## 3.7 Funciones de orden superior sobre colecciones

Una **función de orden superior** es aquella que recibe otra función como parámetro (como `repetir` en el ejemplo anterior) o que devuelve una función. La librería estándar de Kotlin las usa masivamente para trabajar con colecciones sin necesidad de bucles `for` explícitos.

Recuperad el "Ejercicio 6" de la práctica de la sesión anterior, donde recorríais una `MutableList<Ticket>` con un `for` normal porque todavía no habíamos visto `filter`:

```kotlin
// Cómo lo hicisteis en la sesión 2 (sin filter)
var sinAsignarCount = 0
for (ticket in tickets) {
    if (ticket.tecnicoAsignado == null) {
        sinAsignarCount++
    }
}
println(sinAsignarCount)
```

Con `filter`, la misma idea se escribe así:

```kotlin
val sinAsignar = tickets.filter { it.tecnicoAsignado == null }
println(sinAsignar.size)
```

`filter` recorre la lista y devuelve **una lista nueva** solo con los elementos que cumplen la condición. La lista original (`tickets`) no se modifica.

`map` también devuelve una lista nueva, pero en vez de filtrar, **transforma** cada elemento uno a uno; el resultado tiene siempre el mismo número de elementos que la lista original:

```kotlin
val descripciones = tickets.map { it.descripcion.uppercase() }
// descripciones tiene el mismo tamaño que tickets, con cada texto en mayúsculas
```

`forEach` no transforma ni filtra nada: simplemente ejecuta una acción por cada elemento y no devuelve ningún resultado útil (su tipo de retorno es `Unit`, el equivalente Kotlin de `void`):

```kotlin
tickets.forEach { println(it.descripcion) }
```

<div class="callout-warning">
Error habitual al empezar: usar <code>map</code> cuando en realidad solo hace falta <code>forEach</code> (por ejemplo, <code>tickets.map { println(it) }</code> "funciona", pero crea y descarta una lista de <code>Unit</code> que no sirve para nada). Regla práctica: si el resultado de la lambda os interesa como lista nueva, es <code>map</code>; si solo os interesa el efecto secundario (imprimir, guardar, notificar), es <code>forEach</code>.
</div>

## 3.8 Encadenar operaciones

`filter`, `map` y el resto de funciones de colecciones se pueden encadenar, porque cada una devuelve una lista sobre la que se puede volver a llamar otra función:

```kotlin
val resumen = tickets
    .filter { it.tecnicoAsignado == null }
    .map { it.descripcion.uppercase() }

resumen.forEach { println(it) }
```

Quien venga de Java reconocerá el patrón: es la misma idea que la Stream API (`tickets.stream().filter(...).map(...).collect(Collectors.toList())`), pero sin la ceremonia de abrir el *stream* y "recolectar" el resultado al final.

| Operación | Java (Stream API) | Kotlin |
|---|---|---|
| Filtrar | `.stream().filter(t -> ...).collect(Collectors.toList())` | `.filter { ... }` |
| Transformar | `.stream().map(t -> ...).collect(Collectors.toList())` | `.map { ... }` |
| Recorrer con efecto secundario | `.forEach(t -> ...)` | `.forEach { ... }` |
| Encadenar filtro + transformación | `.stream().filter(...).map(...).collect(...)` | `.filter { ... }.map { ... }` (directo, sin *stream*/*collect*) |

<div class="callout-warning">
Encadenar más de dos o tres operaciones seguidas puede volverse difícil de leer, igual que en Java. Si una cadena empieza a ocupar muchas líneas o mezcla condiciones complejas, valorad partirla en pasos intermedios con nombre (<code>val sinAsignar = tickets.filter { ... }</code> y luego operar sobre <code>sinAsignar</code>), aunque sea "menos compacto".
</div>

## 3.9 Por qué esto importa para Compose

Volviendo al ejemplo del apartado 3.6: en la UD3 vais a escribir interfaces con funciones como `Button` o `LazyColumn`, que reciben lambdas como argumento para definir qué ocurre al pulsar o cómo se dibuja cada elemento de una lista:

```kotlin
LazyColumn {
    items(modelos) { modelo ->
        Text(modelo)
    }
}
```

Aquí `LazyColumn { ... }` es una *trailing lambda* (su único parámetro relevante es una lambda de configuración), y dentro, `items(modelos) { modelo -> ... }` vuelve a ser el mismo patrón: `modelos` como argumento normal y una lambda final que se ejecuta una vez por cada elemento, con `modelo` como su parámetro (el equivalente con nombre explícito de un `it`).

<div class="callout-info">
Conexión con lo que viene: si hoy entendéis bien qué es una lambda, qué es <code>it</code> y por qué un bloque <code>{ ... }</code> puede ir fuera de los paréntesis, la sintaxis de Compose de la próxima unidad os va a resultar mucho más natural. No es sintaxis nueva de Android: es exactamente lo que hemos visto hoy con <code>Kotlin</code> puro.
</div>

## Tabla resumen de equivalencias

| Concepto | Java | Kotlin |
|---|---|---|
| Añadir un método a un tipo que no controláis | Clase de utilidades con métodos `static` (`StringUtils.foo(s)`) | Función de extensión (`s.foo()`) |
| Tipo "función" como valor | Interfaz funcional (`Runnable`, `Comparator<T>`, `@FunctionalInterface`) | `(TiposParametros) -> TipoRetorno` |
| Crear una función sin nombre | Lambda ligada a una interfaz funcional (`(a, b) -> a.compareTo(b)`) | `{ parametros -> cuerpo }` |
| Parámetro único de una lambda | Hay que nombrarlo siempre | Se puede usar `it` |
| Lambda como último argumento | Siempre dentro de los paréntesis | Se puede sacar fuera (*trailing lambda*) |
| Filtrar una colección | `.stream().filter(...).collect(Collectors.toList())` | `.filter { ... }` |
| Transformar una colección | `.stream().map(...).collect(Collectors.toList())` | `.map { ... }` |
| Recorrer con efecto secundario | `for` clásico o `.forEach(...)` | `.forEach { ... }` |

<div class="callout-practica">
<strong>Ejercicio de sesión</strong>: retoma la <code>data class Ticket</code> y la lista de tickets de la práctica de la Sesión 2 (<em>Soporte técnico</em>).

1. Escribe una función de extensión <code>List&lt;Ticket&gt;.sinAsignar(): List&lt;Ticket&gt;</code> que use <code>filter</code> para devolver solo los tickets sin técnico asignado —sustituye así el contador manual con <code>for</code> del Ejercicio 6 de la sesión anterior—.
2. Escribe una función de extensión <code>Ticket.resumen(): String</code> (expresión única) que devuelva un texto tipo <code>"#3 — Impresora atascada (sin asignar)"</code>, usando el operador Elvis visto en la sesión 2 para la parte "(sin asignar)"/"(asignado a X)".
3. Usa <code>map</code> sobre la lista de tickets para obtener la lista de todos los <code>resumen()</code>, y <code>forEach</code> para imprimirlos, uno por línea.
4. Encadena <code>sinAsignar()</code> y <code>map { it.resumen() }</code> para imprimir solo el resumen de los tickets pendientes de asignar.

20 minutos de trabajo autónomo; se corrige en los últimos 10 minutos de la sesión.
</div>

## Para la próxima sesión

Sesión 4: seguimos consolidando funciones de extensión y lambdas —más funciones de colecciones de uso muy habitual (`sortedBy`, `count`, `any`/`all`, `find`...), las *scope functions* (`let`, `apply`, `also`, `run`, `with`) y cómo escribir vuestras propias funciones de orden superior— antes de pasar a Jetpack Compose en la UD3.