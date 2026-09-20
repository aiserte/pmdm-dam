---
title: "Flota de alquiler — Enunciado (Sesión 1)"
unidad: UD2
sesion: 1
orden: 1
tipo: enunciado
---

## Contexto

Práctica de consolidación de la **Sesión 1** (*Repaso de POO: de Java a Kotlin*). No hace falta Android Studio: puedes resolverla en el [Kotlin Playground](https://play.kotlinlang.org/) o en cualquier proyecto Kotlin con una función `main()`.

<div class="callout-warning">
Usa solo lo visto hasta ahora: clases, constructores, herencia (<code>open</code>/<code>override</code>/<code>super</code>), <code>Any</code>, interfaces y colecciones/arrays. Todavía <strong>no</strong> hemos visto <code>data class</code> ni los operadores de seguridad ante nulos (<code>?.</code>, <code>?:</code>) — eso es la sesión 2 — ni lambdas con <code>filter</code>/<code>map</code> — eso es la sesión 3. Resuélvelo con bucles <code>for</code> normales.
</div>

## Enunciado

Vas a modelar, en Kotlin puro (sin interfaz gráfica), la flota de vehículos de una empresa de alquiler.

### Ejercicio 0 — Traducción Java → Kotlin

Traduce esta clase Java a Kotlin usando constructor primario. Decide razonadamente qué propiedades deben ser `val` y cuáles `var`.

```java
public class Conductor {
    private String nombre;
    private String numeroLicencia;
    private int aniosExperiencia;

    public Conductor(String nombre, String numeroLicencia, int aniosExperiencia) {
        this.nombre = nombre;
        this.numeroLicencia = numeroLicencia;
        this.aniosExperiencia = aniosExperiencia;
    }

    public String getNombre() { return nombre; }
    public String getNumeroLicencia() { return numeroLicencia; }
    public int getAniosExperiencia() { return aniosExperiencia; }
    public void setAniosExperiencia(int aniosExperiencia) {
        this.aniosExperiencia = aniosExperiencia;
    }
}
```

### Ejercicio 1 — Jerarquía de vehículos (herencia)

- Crea `open class Vehiculo` con las propiedades `matricula: String`, `marca: String` y `precioPorDia: Double`, y un método `open fun describir(): String` que devuelva algo como `"Toyota (1234ABC) — 35.0€/día"`.
- Crea `class Turismo` que herede de `Vehiculo` y añada una propiedad propia `plazas: Int`. Sobrescribe `describir()` reutilizando la del padre con `super.describir()` y añadiendo el número de plazas.
- Crea `class Furgoneta` que herede de `Vehiculo` y añada `capacidadCargaKg: Int`. Sobrescribe `describir()` igual que en `Turismo`, añadiendo la capacidad de carga.

### Ejercicio 2 — Interfaz `Alquilable`

Crea una interfaz `Alquilable` con:

- Una propiedad abstracta `val precioPorDia: Double`.
- Un método `calcularPrecioTotal(dias: Int): Double` **con implementación por defecto**, que devuelva `precioPorDia * dias`.

Haz que `Vehiculo` implemente `Alquilable`. En un comentario, explica por qué la propiedad `precioPorDia` de la interfaz no necesita (ni puede tener) un *backing field* propio.

### Ejercicio 3 — La clase `Any`

- Crea un `Vehiculo` (o subtipo) y haz `println(vehiculo)` **sin** sobrescribir `toString()`. Observa qué se imprime y explica de dónde sale ese comportamiento.
- Sobrescribe `toString()` en `Vehiculo` para que devuelva lo mismo que `describir()`, y comprueba cómo cambia el resultado de `println(vehiculo)`.

### Ejercicio 4 — Colecciones

- Crea un `MutableList<Vehiculo>` llamado `flota` con al menos 2 `Turismo` y 2 `Furgoneta`.
- Sin usar `filter` ni `map` (todavía no los hemos visto), recorre la flota con un `for` y construye un `Set<String>` con las marcas distintas que aparecen.
- Crea un `Map<String, Double>` llamado `kilometraje` que asocie la matrícula de cada vehículo con sus kilómetros recorridos (mínimo 4 entradas). Muestra por pantalla el kilometraje de una matrícula concreta accediendo al mapa por su clave.

### Ejercicio 5 — Arrays

- Crea un `IntArray` llamado `reservasSemana` con el número de reservas de cada día de la semana (7 valores).
- Calcula y muestra el total de reservas de la semana con `.sum()`.
- Sin usar funciones de orden superior, recorre el array con un `for` y averigua el índice del día con más reservas.

### Ejercicio 6 (integrador) — Informe de la flota

Recorre `flota` con un `for` e imprime, para cada vehículo, el resultado de `describir()` junto con el precio total de alquilarlo 3 días (usando `calcularPrecioTotal`, heredado de la interfaz).

## Requisitos técnicos obligatorios

- Traducción correcta de `Conductor` a Kotlin (constructor primario, `val`/`var` bien decididos).
- Jerarquía `Vehiculo` → `Turismo` / `Furgoneta` con `open`, `override` y `super` usados correctamente.
- Interfaz `Alquilable` con propiedad abstracta y método con implementación por defecto, implementada por `Vehiculo`.
- Uso explícito y comentado del comportamiento de `Any` (con y sin `toString()` sobrescrito).
- `MutableList<Vehiculo>`, un `Set<String>` construido con bucle `for` y un `Map<String, Double>`.
- Un `IntArray` recorrido con `for`, con `.sum()` y búsqueda manual del máximo.
- Ni `data class`, ni operadores `?.` / `?:` / `!!`, ni `filter`/`map`/lambdas: todavía no se han visto.

## Criterios de evaluación

- El código compila y se ejecuta sin errores (en Kotlin Playground o Android Studio).
- Aparecen y se usan correctamente todos los elementos obligatorios listados arriba.
- El código sigue las convenciones vistas en la sesión (`val` por defecto salvo que se necesite `var`, nombres en camelCase, `open`/`override` explícitos).
- Las explicaciones pedidas en comentarios (backing field, comportamiento de `Any`) son correctas.

<div class="callout-info">
Trabajo para casa: tráelo resuelto a la sesión 2, donde se corrige brevemente antes de empezar con <em>null-safety</em> y <code>data class</code>.
</div>