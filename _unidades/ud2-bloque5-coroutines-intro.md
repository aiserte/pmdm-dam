---
title: "Introducción ligera a coroutines"
unit: UD2
order: 5
duration: "2h"
---

## Objetivos

- Entender por qué el hilo principal de Android no puede bloquearse.
- Saber qué es una coroutine a nivel conceptual y qué significa `suspend`.
- Lanzar una coroutine sencilla con `launch` dentro de un `CoroutineScope`.

## 1. El problema: el hilo principal

Toda app Android tiene un hilo principal (*main thread*) encargado de dibujar la interfaz y responder a los toques. Si se bloquea más de unos segundos, el sistema muestra el diálogo "La aplicación no responde" (ANR).

<div class="callout-warning">
Todo lo que tarde —red, ficheros grandes, base de datos— debe hacerse fuera del hilo principal. Las coroutines son la herramienta que usaremos en Kotlin para eso.
</div>

## 2. ¿Qué es una coroutine?

Una coroutine es una tarea que se puede pausar y reanudar sin bloquear el hilo que la ejecuta. A diferencia de un `Thread` del sistema operativo, es muy ligera: se pueden lanzar miles sin apenas coste de memoria.

No profundizamos hoy en cómo funciona por dentro — eso se retoma con calma en la UD5, al conectar con servicios web.

## 3. La palabra clave suspend

```kotlin
suspend fun cargarDatos(): String {
    delay(1000)   // simula una espera, sin bloquear el hilo
    return "Datos cargados"
}
```

`delay()` es el equivalente "amigo de las coroutines" de `Thread.sleep()`: pausa la coroutine, pero deja libre el hilo para que siga atendiendo la interfaz.

## 4. Lanzar una coroutine: launch

```kotlin
CoroutineScope(Dispatchers.Main).launch {
    val resultado = cargarDatos()
    println(resultado)
}
```

- `launch` abre una nueva coroutine y la lanza sin bloquear quien la llama.
- `Dispatchers.Main` indica que sigue en el hilo principal, ideal para actualizar la interfaz al terminar.

## 5. Qué no vemos todavía

Se deja para la UD5: `Dispatchers.IO`, manejo de excepciones en coroutines, `withContext` y *flows*. Hoy solo hace falta reconocer `suspend`, `launch` y `delay`.

<div class="callout-practica">
Ejercicio de consolidación: sobre la <code>data class Usuario</code>, añade una función de extensión <code>List&lt;Usuario&gt;.conTelefono(): List&lt;Usuario&gt;</code> usando <code>filter</code>, y una función <code>suspend fun simularCarga()</code> que use <code>delay(500)</code> e imprima un mensaje. Lánzala con <code>launch</code>.
</div>

## Para la próxima sesión

Sesión 6: práctica de cierre de la unidad (*Catálogo de dispositivos*), antes de entrar de lleno en la UD3.
