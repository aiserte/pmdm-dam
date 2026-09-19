---
title: "Soporte técnico — Enunciado (Sesión 2)"
unit: UD2
sesion: 2
tipo: enunciado
---

## Contexto

Práctica de consolidación de la **Sesión 2** (*Null-safety y data classes*). Igual que en la sesión 1, puedes resolverla en el [Kotlin Playground](https://play.kotlinlang.org/) o en cualquier proyecto Kotlin con una función `main()`. Es independiente de la práctica de la sesión 1 (*Flota de alquiler*): aquí el dominio es un sistema de tickets de soporte técnico de un centro educativo.

<div class="callout-warning">
Usa lo visto hasta ahora: tipos nullable (<code>?</code>, <code>?.</code>, <code>?:</code>, <code>!!</code>, <code>?: return</code>/<code>?: throw</code>), <code>?.let { }</code>, <code>as?</code>, <code>data class</code> e igualdad estructural (<code>==</code>/<code>===</code>). Todavía <strong>no</strong> hemos visto funciones de extensión ni lambdas con <code>filter</code>/<code>map</code> —eso es la sesión 3— ni coroutines —sesión 4—. Donde haga falta recorrer una lista, usa un bucle <code>for</code> normal.
</div>

## Enunciado

Vas a modelar, en Kotlin puro, un pequeño sistema de tickets de incidencias de soporte técnico.

### Ejercicio 0 — Traducción Java → Kotlin

Traduce esta clase Java a Kotlin. Decide qué propiedades deben ser `val` y cuáles `var`, y qué tipo debe ser nullable.

```java
public class Incidencia {
    private int id;
    private String descripcion;
    private String tecnicoAsignado; // puede ser null si aún no se ha asignado
    private boolean resuelta;

    public Incidencia(int id, String descripcion, String tecnicoAsignado, boolean resuelta) {
        this.id = id;
        this.descripcion = descripcion;
        this.tecnicoAsignado = tecnicoAsignado;
        this.resuelta = resuelta;
    }

    public int getId() { return id; }
    public String getDescripcion() { return descripcion; }
    public String getTecnicoAsignado() { return tecnicoAsignado; }
    public void setTecnicoAsignado(String tecnicoAsignado) { this.tecnicoAsignado = tecnicoAsignado; }
    public boolean isResuelta() { return resuelta; }
    public void setResuelta(boolean resuelta) { this.resuelta = resuelta; }
}
```

### Ejercicio 1 — `data class Ticket`

Convierte la traducción anterior en `data class Ticket`, con `id: Int`, `descripcion: String`, `tecnicoAsignado: String? = null` y `resuelta: Boolean = false`.

### Ejercicio 2 — Validación con Elvis y salida temprana

- Escribe `fun crearTicket(descripcion: String?): Ticket` que use `?:` con `throw` para lanzar un `IllegalArgumentException` si `descripcion` es `null` o está en blanco (`.isBlank()`), y que si es válida devuelva un `Ticket` nuevo (puedes usar un contador simple para el `id`).
- Escribe `fun descripcionTecnico(ticket: Ticket): String` que use `?:` para devolver `"Sin asignar"` cuando `tecnicoAsignado` sea `null`.

### Ejercicio 3 — `?.let { }`

Escribe `fun notificar(ticket: Ticket)` que, usando `ticket.tecnicoAsignado?.let { ... }`, imprima `"Notificando a <nombre>"` solo si hay un técnico asignado, sin hacer nada en caso contrario.

### Ejercicio 4 — Conversión segura con `as?`

Se te da una lista `List<Any>` que mezcla tickets y avisos de sistema (`String`, `Int`...). Recorre la lista con un `for` y, para cada elemento, usa `as? Ticket` para intentar convertirlo: si el resultado no es `null`, imprime su descripción; si lo es, imprime `"Elemento ignorado (no es un ticket)"`.

### Ejercicio 5 — Igualdad estructural

Crea dos `Ticket` con los mismos datos y comprueba con `==` y `===` que son estructuralmente iguales pero no el mismo objeto en memoria. Después usa `copy()` para crear un tercer ticket cambiando solo `resuelta = true`, y comprueba que ya no es `==` al primero.

### Ejercicio 6 (integrador) — Panel de tickets

Crea un `MutableList<Ticket>` con al menos 5 tickets, alguno sin técnico asignado. Recorre la lista con un `for` y muestra, para cada ticket, su descripción junto con el resultado de `descripcionTecnico()`. Al final, cuenta —con un contador y un `for`, sin `filter`— cuántos tickets están sin asignar y muéstralo.

## Requisitos técnicos obligatorios

- Traducción correcta de `Incidencia` a Kotlin (constructor primario, `val`/`var` y nulabilidad bien decididos).
- `data class Ticket` con una propiedad nullable (`tecnicoAsignado: String?`).
- Uso de `?:` en dos variantes: como valor por defecto y como salida temprana (`?: throw`).
- Uso de `?.let { }` para ejecutar código solo si un valor no es `null`.
- Uso de `as?` para una conversión de tipo segura, comprobando el resultado nullable.
- Comparación explícita con `==` y `===`, y uso de `copy()`.
- `MutableList<Ticket>` recorrida con `for` (sin `filter`/`map`/lambdas: todavía no se han visto).

## Criterios de evaluación

- El código compila y se ejecuta sin errores (en Kotlin Playground o Android Studio).
- Aparecen y se usan correctamente todos los elementos obligatorios listados arriba.
- El código sigue las convenciones vistas en la unidad (`val` por defecto salvo que se necesite `var`, nombres en camelCase, `!!` evitado).
- Las comprobaciones de `==`/`===` y el uso de `copy()` son correctos y están comentados.

<div class="callout-info">
Trabajo para casa: tráelo resuelto a la sesión 3, donde se corrige brevemente antes de empezar con funciones de extensión y lambdas.
</div>
