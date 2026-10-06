---
title: "Cafetería del instituto: la carta — Enunciado (Sesión 1)"
unidad: UD3
sesion: 1
orden: 1
tipo: enunciado
---

## Contexto

Práctica de la **Sesión 1** (*Anatomía de una app Compose y recursos*). Es la primera entrega de la app que construiremos durante toda la UD3: **Cafetería del instituto**, una carta digital en la que, sesión a sesión, iremos añadiendo pedidos, diálogos, listas, varias pantallas y ajustes.

Hoy la app solo **muestra** la carta: no hay interacción todavía. El objetivo es montar bien la base: estructura de paquetes, modelo de datos, componentes reutilizables y todos los textos e imágenes como recursos, en dos idiomas.

<div class="callout-warning">
Usa solo lo visto hasta ahora: funciones <code>@Composable</code>, <code>@Preview</code>, layouts básicos (<code>Column</code>, <code>Row</code>, <code>Box</code>, <code>Spacer</code>), <code>Modifier</code>, <code>stringResource</code> y <code>painterResource</code>. Todavía <strong>no</strong> hemos visto estado (<code>remember</code>), ni listas <code>Lazy</code>, ni navegación: no los necesitas.
</div>

<div class="callout-info">
Guarda bien este proyecto: la solución de esta práctica es el punto de partida de la práctica de la sesión 2.
</div>

## Enunciado

### Ejercicio 0 — Proyecto y tema

1. Crea un proyecto nuevo con la plantilla **Empty Activity**, llamado `Cafeteria`, en Kotlin, con *Minimum SDK* API 24.
2. En `ui/theme/Theme.kt`, cambia el valor por defecto de `dynamicColor` a `false`, para que la app use siempre los colores de su tema (y no los del fondo de pantalla del usuario).

### Ejercicio 1 — Estructura de paquetes

Crea, dentro del paquete principal, esta estructura (vacía de momento):

```
data/        ← modelo de datos
ui/carta/    ← composables de la pantalla de la carta
```

### Ejercicio 2 — Textos como recursos

En `res/values/strings.xml` (español, idioma por defecto) define como mínimo:

- `app_name` (no traducible).
- `titulo_carta` → "Carta de hoy".
- `subtitulo_carta` → "Cafetería del instituto · Abierto de %1$s a %2$s" (dos argumentos: hora de apertura y de cierre).
- `precio` → "%1$.2f €".
- `logo_descripcion` → descripción del logotipo, para el lector de pantalla.
- Para **cuatro productos** (por ejemplo: café con leche, bocadillo de tortilla, croissant y zumo de naranja): un recurso para el nombre (`producto_xxx`) y otro para una descripción breve (`desc_xxx`).

Después, con el **Translations Editor**, añade el idioma **inglés** (`values-en`) y traduce todos los textos (salvo `app_name`). En inglés, el precio va con el símbolo delante: "€%1$.2f".

### Ejercicio 3 — Iconos de los productos

Con *New → Vector Asset → Clip art*, añade a `res/drawable` un icono para cada producto. Sugerencias: `local_cafe`, `lunch_dining`, `bakery_dining`, `local_drink`.

### Ejercicio 4 — El modelo

En el paquete `data`:

- `Producto.kt`: una `data class Producto` con el nombre y la descripción como **identificadores de recurso** (`@StringRes`), el icono como `@DrawableRes` y el precio como `Double`.
- `DatosCarta.kt`: un `object DatosCarta` con una propiedad `productos: List<Producto>` que contenga los cuatro productos.

### Ejercicio 5 — Componente `ProductoCard`

En `ui/carta/ProductoCard.kt`, crea una función `@Composable fun ProductoCard(producto: Producto, modifier: Modifier = Modifier)` que muestre, dentro de una `Card` que ocupe todo el ancho:

- A la izquierda, el icono del producto (usa `Icon` con `tint` del color primario del tema, 40 dp).
- En el centro, el nombre (estilo `titleMedium`) y debajo la descripción (estilo `bodySmall`), ocupando todo el espacio sobrante.
- A la derecha, el precio formateado con el recurso `precio`.

Añade una `@Preview` de la tarjeta con uno de los productos de `DatosCarta`.

### Ejercicio 6 — Componente `CabeceraCarta`

En `ui/carta/CabeceraCarta.kt`, crea `@Composable fun CabeceraCarta(modifier: Modifier = Modifier)` con, centrados:

- Un **logotipo circular de 96 dp** formado por dos imágenes superpuestas en un `Box`: `R.drawable.ic_launcher_background` (fondo, decorativa) y `R.drawable.ic_launcher_foreground` (con `logo_descripcion`). Ambas ya existen en el proyecto de la plantilla.
- El título de la carta (estilo `headlineMedium`).
- El subtítulo con el horario de apertura, pasando las horas como argumentos del recurso.

### Ejercicio 7 — Pantalla y `MainActivity`

1. En `ui/carta/PantallaCarta.kt`, crea `@Composable fun PantallaCarta(productos: List<Producto>, modifier: Modifier = Modifier)` que muestre la cabecera y, debajo, una `ProductoCard` por cada producto, con 12 dp de separación entre elementos y 16 dp de margen.
2. Deja `MainActivity` reducida a: tema → `Scaffold` → `PantallaCarta(productos = DatosCarta.productos, ...)`, aplicando correctamente el `innerPadding`.
3. Borra la función `Greeting` de la plantilla y su preview.

### Ejercicio 8 — Previews

Sobre `PantallaCarta`, crea una función de preview con **tres** `@Preview`: español, inglés y modo oscuro. Envuelve el contenido en el tema de la app y en un `Surface`, para que el fondo sea correcto también en modo oscuro.

### Ejercicio 9 — Pruebas en el emulador

Ejecuta la app en el emulador y realiza las pruebas de la tabla. Crea un fichero `PRUEBAS.md` en la raíz del proyecto con la tabla completa: el resultado obtenido en cada prueba y, si algo falla, qué ocurre.

| # | Prueba | Resultado esperado | Resultado obtenido |
|---|---|---|---|
| 1 | Idioma del sistema en inglés | Todos los textos en inglés, precio con "€" delante | |
| 2 | Volver a español | Todos los textos en español, precio con coma decimal | |
| 3 | Modo oscuro | Todo legible | |
| 4 | Tamaño de fuente máximo | Ningún texto se sale de su tarjeta | |
| 5 | Orientación horizontal | Se ve toda la carta | |
| 6 | TalkBack sobre el logotipo | Lee la descripción del logotipo | |

### Ampliación (opcional)

- Añade un tercer idioma: **valenciano** (`values-ca`) y pruébalo en el emulador.
- Si en la prueba 5 has detectado algún problema, intenta resolverlo buscando `Modifier.verticalScroll` en la documentación de Compose. (En la sesión 4 veremos la solución adecuada para listas: `LazyColumn`).

## Requisitos técnicos obligatorios

- Estructura de paquetes `data` y `ui/carta`, con un fichero por componente.
- **Ningún texto visible escrito directamente en el código**: todos desde `strings.xml` (las horas del horario, como argumentos, sí pueden ir en el código).
- `strings.xml` en español y en inglés, con los mismos recursos en los dos.
- Al menos un recurso con dos argumentos numerados (`%1$s`, `%2$s`) y uno con formato decimal (`%1$.2f`).
- `data class Producto` con `@StringRes` y `@DrawableRes`.
- Todos los composables propios con el parámetro `modifier: Modifier = Modifier`, aplicado a su elemento raíz.
- `contentDescription` rellenado en las imágenes con significado y `null` en las decorativas.
- Previews: al menos una de `ProductoCard` y tres de `PantallaCarta`.
- `PRUEBAS.md` con las seis pruebas documentadas.

## Criterios de evaluación

Esta práctica contribuye a los criterios **a)**, **b)** y **g)** del RA2:

- **a)** La estructura de clases y paquetes es la pedida y cada pieza está en su sitio (modelo en `data`, interfaz en `ui/carta`, `MainActivity` mínima).
- **b)** Se usan correctamente los componentes de interfaz (`Card`, `Icon`, `Image`, `Text`, layouts) y los recursos del proyecto.
- **g)** Se han realizado y documentado las pruebas en el emulador, incluyendo cambio de idioma, orientación y accesibilidad.
- El proyecto compila y se ejecuta sin errores ni avisos de *hardcoded string*.
