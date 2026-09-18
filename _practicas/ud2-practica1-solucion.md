---
title: "Catálogo de dispositivos — Solución"
unit: UD2
practica: 1
tipo: solucion
---

## Solución de referencia

<div class="callout-warning">
Esta es una posible solución, no el único enunciado válido. Dejad que el alumnado llegue a su propia versión; usadla solo si hace falta destrabar a algún grupo.
</div>

```kotlin
data class Dispositivo(
    val modelo: String,
    val fabricante: String,
    val telefono: String? = null
)

fun List<Dispositivo>.deFabricante(nombre: String) =
    this.filter { it.fabricante == nombre }

fun formatear(d: Dispositivo, extra: (Dispositivo) -> String): String =
    "${d.modelo} — ${extra(d)}"

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val dispositivos = listOf(
            Dispositivo("Pixel 8", "Google", "600111222"),
            Dispositivo("Galaxy S24", "Samsung", null)
        )
        setContent {
            Column {
                dispositivos.forEach { d ->
                    Text(formatear(d) { it.telefono ?: "Sin teléfono" })
                }
            }
        }
    }
}
```

## Qué comprobar en cada elemento obligatorio

- **data class** → `Dispositivo` genera `equals`, `toString` y `copy` automáticamente.
- **Función de extensión** → `deFabricante` se usaría como `dispositivos.deFabricante("Google")`.
- **Operador Elvis** → dentro de la lambda `{ it.telefono ?: "Sin teléfono" }`.
- **Lambda como parámetro** → `extra: (Dispositivo) -> String` en `formatear`.
- **setContent** → toda la interfaz se declara dentro del bloque de `MainActivity`.
