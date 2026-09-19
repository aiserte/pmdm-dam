---
title: "Repaso de POO: de Java a Kotlin"
unit: UD2
order: 1
duration: "2h"
---

## Objetivos

- Entender por qué usamos Kotlin en este módulo y su relación con Java.
- Traducir clases, constructores, herencia e interfaces de Java a Kotlin.
- Conocer las colecciones básicas de Kotlin (`List`, `MutableList`, `Map`).

## 1.1 ¿Por qué Kotlin?

Kotlin es el lenguaje oficial recomendado por Google para el desarrollo Android desde 2019. Funciona sobre la misma máquina virtual que Java (JVM) y es 100% interoperable con código Java existente.

<div class="callout-info">
Todo lo que sabéis de clases, objetos, herencia y polimorfismo en Java sigue siendo válido en Kotlin. Hoy solo aprendemos a escribirlo de otra forma.
</div>

## 1.2 Clases y constructores

En Java, una clase típica separa atributos, constructor y getters/setters:

```java
public class Dispositivo {
    private String modelo;
    private int anioLanzamiento;

    public Dispositivo(String modelo, int anioLanzamiento) {
        this.modelo = modelo;
        this.anioLanzamiento = anioLanzamiento;
    }

    public String getModelo() { return modelo; }
    public void setModelo(String modelo) { this.modelo = modelo; }
}
```

En Kotlin, el constructor primario y las propiedades se declaran en una sola línea:

```kotlin
class Dispositivo(var modelo: String, val anioLanzamiento: Int)
```

Con `var` la propiedad es mutable; con `val` es de solo lectura (equivalente a un atributo `final`). En Android, preferimos `val` siempre que sea posible.

## 1.3 Herencia

Diferencia clave: en Kotlin, las clases son `final` por defecto. Para permitir que se hereden, hay que marcarlas con `open`.

```kotlin
// Java
public class Vehiculo { }
public class Coche extends Vehiculo { }

// Kotlin
open class Vehiculo { }
class Coche : Vehiculo() { }
```

<div class="callout-warning">
Error típico al empezar: olvidar el <code>open</code> y que el compilador se queje al intentar heredar. Es una decisión de diseño de Kotlin para evitar herencias accidentales.
</div>

Además de `open`, hay otras palabras reservadas que aparecen siempre que trabajamos con herencia. Vale la pena verlas juntas antes de seguir:

```kotlin
open class Vehiculo(val marca: String) {
    open fun describir(): String = "Vehículo de la marca $marca"
}

class Coche(marca: String, val puertas: Int) : Vehiculo(marca) {
    override fun describir(): String {
        return super.describir() + " con $puertas puertas"
    }
}
```

La misma jerarquía en Java, para comparar:

```java
class Vehiculo {
    protected String marca;
    public Vehiculo(String marca) { this.marca = marca; }
    public String describir() { return "Vehículo de la marca " + marca; }
}

class Coche extends Vehiculo {
    private int puertas;
    public Coche(String marca, int puertas) {
        super(marca);
        this.puertas = puertas;
    }
    @Override
    public String describir() {
        return super.describir() + " con " + puertas + " puertas";
    }
}
```

| Palabra clave | Uso en Kotlin | Equivalente en Java |
|---|---|---|
| `open` | Permite que una clase o un miembro puedan heredarse/sobrescribirse | Todo es "open" por defecto |
| `override` | Obligatorio al sobrescribir un método o propiedad heredados | `@Override` (opcional, solo aviso) |
| `super` | Accede a la implementación de la superclase (`super.describir()`) | `super` (igual) |
| `final` | Evita que un miembro `open` siga sobrescribiéndose en subclases | `final` |
| `abstract` | Declara clase/miembro sin implementación; obliga a implementarlo en la subclase | `abstract` |

<div class="callout-info">
En Kotlin, la llamada al constructor de la superclase (equivalente al <code>super(marca)</code> de Java) se hace en la cabecera de la clase — <code>: Vehiculo(marca)</code> —, no como una instrucción dentro del cuerpo del constructor. <code>override</code>, en cambio, sí es obligatorio: si se os olvida, el compilador da error, a diferencia de la anotación <code>@Override</code> de Java, que es opcional.
</div>

## 1.4 La clase Any

Toda clase en Kotlin hereda implícitamente de `Any` cuando no se indica una superclase explícita, igual que en Java toda clase hereda de `Object`. Es la raíz de la jerarquía de tipos.

```kotlin
// Toda clase hereda implícitamente de Any, aunque no lo escribamos
class Dispositivo(var modelo: String)   // en realidad: class Dispositivo(...) : Any()

// Any define equals(), hashCode() y toString() para todas las clases
val d1 = Dispositivo("Pixel 8")
println(d1.toString())   // usa la implementación por defecto de Any
```

<div class="callout-info">
<code>Any</code> es la raíz de la jerarquía de clases en Kotlin, igual que <code>Object</code> en Java. En la sesión 2 veremos por qué <code>data class</code> os libera de reescribir <code>equals()</code>, <code>hashCode()</code> y <code>toString()</code>: son, precisamente, los métodos que define <code>Any</code>.
</div>

## 1.5 Interfaces

Las interfaces en Kotlin pueden tener métodos con implementación por defecto:

```kotlin
interface Sonable {
    fun emitirSonido()
    fun descripcion(): String = "Este objeto puede sonar"
}

class Altavoz : Sonable {
    override fun emitirSonido() = println("Bip")
}
```

## 1.6 Colecciones

Kotlin distingue entre colecciones de solo lectura y mutables:

- `listOf(...)` → `List`, de solo lectura. `mutableListOf(...)` → `MutableList`.
- `mapOf(...)` → `Map` de solo lectura. `mutableMapOf(...)` → `MutableMap`.
- No hace falta indicar el tipo casi nunca: Kotlin lo infiere.

```kotlin
val modelos = listOf("Pixel", "Galaxy", "iPhone")
val carrito = mutableListOf<String>()
carrito.add("Pixel")
```

## 1.7 Arrays

En Kotlin, a diferencia de Java, el array no es la estructura por defecto: se usa `List`/`MutableList` para casi todo, y `Array` queda reservado para interoperar con APIs Java que lo exigen, o para primitivos, donde `IntArray` evita el coste de autoboxing que sí tiene `List<Int>`.

```kotlin
// Array "normal" (genérico)
val modelos = arrayOf("Pixel", "Galaxy", "iPhone")
println(modelos.size)          // 3, no .length
println(modelos[0])            // acceso por índice, igual que Java

// Array de primitivos: evita autoboxing
val notas = intArrayOf(7, 8, 9)

// ¿Array o List? En Kotlin, por defecto: List
val modelosList = listOf("Pixel", "Galaxy", "iPhone")   // preferible salvo interop o rendimiento
```

<div class="callout-practica">
Cuidado con la confusión: el tamaño de un <code>Array</code>, igual que el de una <code>List</code> (no Mutable), es fijo. Lo que cambia entre <code>Array</code> y <code>List</code> no es la mutabilidad de tamaño, sino cuándo conviene usar cada uno: <code>Array</code> casi solo para interoperar con Java o para primitivos.
</div>

## 1.8 Tabla resumen de equivalencias

| Concepto | Java | Kotlin |
|---|---|---|
| Clase con atributos | Constructor + getters/setters | `class X(var a: T)` |
| Herencia | `extends` (permitida por defecto) | `open class` + `:` |
| Raíz de la jerarquía | `Object` | `Any` |
| Interfaz con lógica | método `default` | método con cuerpo `= ...` |
| Colección de solo lectura | `Collections.unmodifiableList` | `listOf(...)` |
| Colección mutable | `ArrayList<T>()` | `mutableListOf<T>()` |
| Array de tamaño fijo | `int[]` / `String[]` | `intArrayOf(...)` / `arrayOf(...)` |

## Para la próxima sesión

Bloque 2: el rasgo más característico de Kotlin, la seguridad ante nulos (*null-safety*), y las `data class`.
