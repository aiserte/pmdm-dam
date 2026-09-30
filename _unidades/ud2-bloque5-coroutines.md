---
title: "Introducción ligera a las coroutines"
unit: UD2
order: 5
duration: "30-45 min"
---

## Objetivos

- Entender **qué problema** resuelven las coroutines: no bloquear el hilo principal (UI) con tareas que tardan.
- Reconocer la sintaxis básica: `suspend fun`, `launch`, `delay`.
- Saber que esto es solo una toma de contacto: se retoma en profundidad en la UD3 (operaciones con Room) y sobre todo en la UD4 (llamadas de red).

<div class="callout-info">
Esta sesión es deliberadamente breve (30-45 min). Todavía no tenéis ningún caso real que necesite esperar por algo (ni red, ni base de datos), así que no tiene sentido entrar en Dispatchers, excepciones en coroutines o Flow: eso llegará cuando haya una razón real para usarlo. Hoy el objetivo es solo que la palabra "coroutine" y la palabra clave <code>suspend</code> os suenen y sepáis qué problema atacan.
</div>

## 5.1 El problema: el hilo principal no puede esperar

Toda aplicación Android tiene un **hilo principal** (*main thread* o *UI thread*) encargado de dibujar la pantalla y responder a los toques. Si ese hilo se queda "ocupado" más de un instante —por ejemplo, esperando la respuesta de un servidor o leyendo un fichero grande—, la interfaz se congela. Si tarda demasiado, el sistema operativo puede llegar a matar la app con un error ANR (*Application Not Responding*).

En Java, esto se resolvía tradicionalmente moviendo ese trabajo a otro hilo (`Thread`, `AsyncTask` —ya en desuso—, `Executor`...), y había que coordinar manualmente la vuelta al hilo principal para actualizar la interfaz con el resultado. Funciona, pero es propenso a errores (callbacks anidados, fugas de memoria, código difícil de seguir).

<div class="callout-warning">
No hace falta que recordéis <code>AsyncTask</code> ni la gestión manual de hilos de Java/Android para seguir esta sesión: basta con quedarse con la idea de que "hacer esperar al hilo principal es un problema", que seguramente ya os suena de otros contextos (una interfaz de escritorio que se queda "pillada" mientras carga algo).
</div>

## 5.2 La idea de una coroutine

Una **coroutine** es, para lo que nos interesa hoy, una función que se puede **pausar sin bloquear el hilo** en el que se ejecuta, y **reanudarse** más tarde donde se quedó. Mientras está pausada, el hilo queda libre para hacer otras cosas (como seguir respondiendo a la interfaz).

La palabra clave que marca este comportamiento es `suspend`.

## 5.3 Funciones suspendidas: `suspend fun`

```kotlin
suspend fun cargarDatos(): String {
    delay(2000)   // simula una operación larga (red, disco...) SIN bloquear el hilo
    return "Datos cargados"
}
```

- `suspend` marca la función como "pausable": puede ceder el control mientras espera, en vez de bloquear el hilo.
- `delay()` es el equivalente, dentro de una coroutine, a "espera un rato" — pero, a diferencia de `Thread.sleep()`, no bloquea el hilo mientras espera.
- Una función `suspend` **solo se puede llamar** desde otra función `suspend` o desde dentro de una coroutine. El compilador os avisará si intentáis llamarla como una función normal.

## 5.4 Lanzar una coroutine: `launch`

Una función `suspend` no hace nada por sí sola: hace falta lanzarla dentro de una coroutine. `launch` es la forma más simple de hacerlo:

```kotlin
fun main() = runBlocking {
    println("Empieza")
    launch {
        val resultado = cargarDatos()
        println(resultado)
    }
    println("Esto se imprime antes de que termine cargarDatos()")
}
```

`runBlocking` es un "arrancador" de coroutines pensado para pruebas rápidas en `main()` (como en el Kotlin Playground); no se usa así dentro de una app Android real. Fijaos en el orden de salida: el segundo `println` se imprime **antes** de que termine `cargarDatos()`, porque `launch` no espera a que acabe para seguir con el resto del código.

<div class="callout-info">
En una app Android real no escribiréis <code>runBlocking</code> ni gestionaréis hilos a mano: usaréis un <em>scope</em> ya integrado en el ciclo de vida del componente (por ejemplo, algo llamado <code>viewModelScope</code> o <code>lifecycleScope</code>) que lanza y cancela las coroutines automáticamente. No hace falta que os suenen esos nombres todavía — los veréis con contexto real en la UD3 y la UD4.
</div>

## 5.5 Qué NO vamos a hacer todavía

A propósito, hoy no entramos en: `Dispatchers` (en qué hilo concreto se ejecuta cada cosa), cómo se gestionan las excepciones dentro de una coroutine, `Flow` (secuencias de valores asíncronas), ni la cancelación o la concurrencia estructurada en detalle. Todo eso tiene mucho más sentido cuando ya haya una tarea real esperando —una consulta a Room en la UD3, una llamada HTTP en la UD4— y lo retomaremos entonces con calma.

## Tabla resumen de la sesión

| Necesito... | Herramienta |
|---|---|
| Marcar una función como "pausable" | `suspend fun` |
| Esperar sin bloquear el hilo (simulando una tarea larga) | `delay(ms)` |
| Lanzar una coroutine | `launch { }` |
| Arrancar coroutines en una prueba rápida (`main()`, Playground) | `runBlocking { }` (solo para pruebas, no para Android real) |

## Para el resto de la sesión

El resto de la sesión se dedica a una **práctica de consolidación en Kotlin puro** que repasa todo lo visto en las sesiones 1 a 4 (POO, null-safety, funciones de extensión, lambdas, colecciones y *scope functions*) antes de entrar en la UD3.

Además, en esta sesión se os propone también, como **trabajo autónomo para terminar en casa**, la práctica *Catálogo de dispositivos*: vuestro primer proyecto Compose, que ya no tiene una sesión de aula dedicada pero que conviene dejar resuelto antes de empezar la UD3.
