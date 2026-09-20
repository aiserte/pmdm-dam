---
title: "Flota de alquiler — Solución (Sesión 1)"
unidad: UD2
sesion: 1
orden: 1
tipo: solucion
disponible: true
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único código válido. Dejad que el alumnado llegue a su propia versión; usadla solo si hace falta destrabar a algún grupo.
</div>

```kotlin
// Ejercicio 0 — Traducción Java → Kotlin
// aniosExperiencia es var porque el Java original expone un setter;
// nombre y numeroLicencia son val porque el Java original no los modifica.
class Conductor(
    val nombre: String,
    val numeroLicencia: String,
    var aniosExperiencia: Int
)

// Ejercicio 2 — Interfaz Alquilable
// precioPorDia es abstracta: la interfaz no puede tener backing field propio
// (no se instancia), así que exige que quien la implemente aporte el valor.
// calcularPrecioTotal sí trae implementación por defecto porque solo depende
// de esa propiedad abstracta.
interface Alquilable {
    val precioPorDia: Double
    fun calcularPrecioTotal(dias: Int): Double = precioPorDia * dias
}

// Ejercicio 1 — Jerarquía de vehículos
open class Vehiculo(
    val matricula: String,
    val marca: String,
    override val precioPorDia: Double
) : Alquilable {

    open fun describir(): String = "$marca ($matricula) — ${precioPorDia}€/día"

    // Ejercicio 3 — Any: toString() es uno de los métodos que Any define
    // por defecto para todas las clases; aquí lo sobrescribimos para que
    // use la misma información que describir().
    override fun toString(): String = describir()
}

class Turismo(
    matricula: String,
    marca: String,
    precioPorDia: Double,
    val plazas: Int
) : Vehiculo(matricula, marca, precioPorDia) {
    override fun describir(): String = super.describir() + " · $plazas plazas"
}

class Furgoneta(
    matricula: String,
    marca: String,
    precioPorDia: Double,
    val capacidadCargaKg: Int
) : Vehiculo(matricula, marca, precioPorDia) {
    override fun describir(): String =
        super.describir() + " · carga máx. ${capacidadCargaKg}kg"
}

fun main() {
    // Ejercicio 3 — comprobación del comportamiento de Any
    val pruebaAny = Turismo("1234ABC", "Toyota", 35.0, 5)
    // Sin sobrescribir toString(), println habría mostrado algo como
    // "Turismo@1b6d3586" (paquete + hash), heredado de Any.
    // Con toString() sobrescrito, usa nuestra descripción:
    println(pruebaAny)

    // Ejercicio 4 — Colecciones
    val flota: MutableList<Vehiculo> = mutableListOf(
        Turismo("1234ABC", "Toyota", 35.0, 5),
        Turismo("5678BCD", "Seat", 30.0, 5),
        Furgoneta("9012CDE", "Renault", 55.0, 1200),
        Furgoneta("3456DEF", "Mercedes", 70.0, 1500)
    )

    val marcas = mutableSetOf<String>()
    for (vehiculo in flota) {
        marcas.add(vehiculo.marca)
    }
    println("Marcas en la flota: $marcas")

    val kilometraje = mapOf(
        "1234ABC" to 18500.0,
        "5678BCD" to 9200.0,
        "9012CDE" to 42300.0,
        "3456DEF" to 5100.0
    )
    println("Kilometraje de 9012CDE: ${kilometraje["9012CDE"]} km")

    // Ejercicio 5 — Arrays
    val reservasSemana = intArrayOf(3, 5, 2, 4, 6, 8, 7)
    val totalReservas = reservasSemana.sum()
    println("Reservas totales de la semana: $totalReservas")

    var diaConMasReservas = 0
    for (i in reservasSemana.indices) {
        if (reservasSemana[i] > reservasSemana[diaConMasReservas]) {
            diaConMasReservas = i
        }
    }
    println(
        "Día con más reservas: índice $diaConMasReservas " +
            "(${reservasSemana[diaConMasReservas]} reservas)"
    )

    // Ejercicio 6 — Informe de la flota
    println("\n--- Informe de la flota (alquiler de 3 días) ---")
    for (vehiculo in flota) {
        val total = vehiculo.calcularPrecioTotal(3)
        println("${vehiculo.describir()} → total 3 días: ${total}€")
    }
}
```

## Qué comprobar en cada elemento obligatorio

- **Traducción Java → Kotlin** → `Conductor` con constructor primario; `aniosExperiencia` como `var` porque el Java original tenía setter, el resto `val`.
- **Herencia** → `Vehiculo` es `open`, `Turismo` y `Furgoneta` heredan con `:`, y ambas sobrescriben `describir()` llamando a `super.describir()`.
- **Interfaz con propiedad abstracta** → `Alquilable` declara `precioPorDia` sin backing field propio; `Vehiculo` la implementa con `override val precioPorDia` en el propio constructor.
- **`Any`** → se evidencia comparando el `toString()` por defecto (heredado de `Any`) con el sobrescrito en `Vehiculo`.
- **Colecciones** → `flota` es `MutableList<Vehiculo>` (polimorfismo: contiene `Turismo` y `Furgoneta`); `marcas` es un `Set<String>` construido a mano con un `for`; `kilometraje` es un `Map<String, Double>` consultado por clave.
- **Arrays** → `reservasSemana` es un `IntArray` de 7 posiciones; el total usa `.sum()`; el día con más reservas se busca con un `for` clásico, sin `maxOrNull()` ni lambdas.
- **Sin adelantar contenido** → no aparece `data class`, ni `?.`/`?:`/`!!`, ni `filter`/`map` con `it`: todo se resuelve con lo visto hasta la sesión 1.