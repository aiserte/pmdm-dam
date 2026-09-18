---
title: "Funciones de extensión y lambdas"
unit: UD2
order: 3
duration: "2h"
---

## Objetivos

- Crear y usar funciones de extensión sobre tipos existentes.
- Escribir y leer lambdas en Kotlin, incluido el parámetro implícito `it`.
- Aplicar funciones de orden superior (`map`, `filter`, `forEach`) sobre colecciones.

## 1. Funciones de extensión

Una función de extensión permite añadir un nuevo método a una clase que ya existe, sin heredar de ella ni modificar su código fuente.

```kotlin
fun String.esEmailValido(): Boolean {
    return this.contains("@") && this.contains(".")
}

val correo = "alumno@instituto.es"
println(correo.esEmailValido())   // true
```

Dentro de la función, `this` hace referencia al objeto sobre el que se llama. Por debajo, el compilador genera una función estática normal; no modifica la clase `String` real.

<div class="callout-info">
En Android vais a ver funciones de extensión constantemente: <code>Modifier.padding()</code>, etc. Entender cómo se definen ayuda a entender por qué se encadenan con el punto.
</div>

## 2. Lambdas: funciones como valores

Una lambda es una función sin nombre que se puede guardar en una variable o pasar como argumento: `{ parámetros -> cuerpo }`

```kotlin
val cuadrado: (Int) -> Int = { numero -> numero * numero }
println(cuadrado(4))   // 16
```

Cuando la lambda recibe un único parámetro, se puede omitir su nombre y referirse a él como `it`:

```kotlin
val cuadrado: (Int) -> Int = { it * it }
```

## 3. Funciones de orden superior sobre colecciones

```kotlin
val modelos = listOf("Pixel 8", "Galaxy S24", "iPhone 15")

val enMayusculas = modelos.map { it.uppercase() }
val conP = modelos.filter { it.startsWith("P") }
modelos.forEach { println(it) }
```

- `map`: transforma cada elemento y devuelve una lista nueva del mismo tamaño.
- `filter`: se queda solo con los elementos que cumplen la condición.
- `forEach`: ejecuta una acción por cada elemento, sin devolver nada.

## 4. Por qué esto importa para Compose

```kotlin
Button(onClick = { println("Pulsado") }) {
    Text("Aceptar")
}

LazyColumn {
    items(modelos) { modelo ->
        Text(modelo)
    }
}
```

<div class="callout-info">
Si hoy entendéis bien qué es una lambda y qué es <code>it</code>, la sintaxis de Compose de la UD3 os va a resultar mucho más natural.
</div>

<div class="callout-practica">
Ejercicio de sesión: escribe una función de extensión <code>Int.esPar(): Boolean</code> y, con una lista de dispositivos, usa <code>filter</code> y <code>map</code> para obtener los nombres en mayúsculas de los que tengan teléfono registrado.
</div>

## Para la próxima sesión

Bloque 4: introducción ligera a las coroutines, lo justo para entender por qué existen antes de retomarlas en profundidad en la UD5 (conectividad).
