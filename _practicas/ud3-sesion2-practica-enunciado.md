---
title: "Cafetería del instituto: el pedido — Enunciado (Sesión 2)"
unidad: UD3
sesion: 2
orden: 2
tipo: enunciado
---

## Contexto

Práctica de la **Sesión 2** (*Ciclo de vida de la Activity, Logcat y estado*). Continuamos la app **Cafetería del instituto**: partimos de la solución de la sesión 1 (la carta en dos idiomas) y hoy la carta deja de ser estática. Cada producto tendrá un selector de cantidad para ir montando el pedido.

Antes de añadir interacción, instrumentaremos la app con Logcat para ver su ciclo de vida. Así, cuando algo falle al girar la pantalla, podremos explicar **por qué**.

<div class="callout-warning">
Usa lo visto hasta ahora: todo lo de la sesión 1 más los métodos del ciclo de vida, <code>Log</code>, <code>remember</code>, <code>mutableStateOf</code>, <code>rememberSaveable</code> y eventos <code>onClick</code>. Todavía <strong>no</strong> hemos visto <em>state hoisting</em>, campos de texto, <code>LazyColumn</code> ni <code>ViewModel</code>: no los necesitas. En particular, <strong>no</strong> se pide mostrar el total del pedido (eso es la sesión 3).
</div>

<div class="callout-info">
Si no terminaste la práctica de la sesión 1, parte de su solución publicada. Guarda bien el resultado de hoy: es el punto de partida de la sesión 3.
</div>

## Enunciado

### Ejercicio 1 — Instrumentar el ciclo de vida

En `MainActivity.kt`:

1. Declara al principio del fichero, fuera de la clase, `private const val TAG = "CicloVida"`.
2. Sobrescribe `onStart`, `onResume`, `onPause`, `onStop` y `onDestroy`, y añade a `onCreate` un mensaje con `Log.d`. Cada método debe llamar a su `super` y registrar su nombre.
3. En `onCreate`, el mensaje debe incluir además si `savedInstanceState` es `null` o no.

### Ejercicio 2 — Experimentos en Logcat

Ejecuta la app, filtra Logcat con `tag:CicloVida` y realiza estas acciones **en orden**. Anota en tu `PRUEBAS.md`, en una nueva sección **"Ciclo de vida"**, la secuencia exacta de métodos que aparece tras cada acción:

| # | Acción | Secuencia observada en Logcat |
|---|---|---|
| CV1 | Abrir la app | |
| CV2 | Pulsar Inicio | |
| CV3 | Volver a la app desde la pantalla de apps recientes | |
| CV4 | Girar el emulador a horizontal | |
| CV5 | Pulsar Atrás en la carta | |
| CV6 | Abrir de nuevo la app desde su icono | |

Indica también, al principio de la sección, la **versión de Android** de tu emulador (*Ajustes → Acerca del teléfono*): el resultado de CV5 depende de ella.

### Ejercicio 3 — Recursos nuevos

1. Añade a `res/drawable` dos iconos con *Vector Asset → Clip art*: `add` y `remove`.
2. Añade a `strings.xml` (español e inglés) los textos `anadir` ("Añadir una unidad" / "Add one") y `quitar` ("Quitar una unidad" / "Remove one"), que servirán de `contentDescription` de los botones.

### Ejercicio 4 — Selector de cantidad con `remember`

Modifica `ProductoCard` para que, **debajo** de la fila que ya tenía (icono, nombre y descripción, precio), muestre una fila alineada a la derecha con:

- un `IconButton` con el icono `remove`, que resta una unidad y **solo está habilitado si la cantidad es mayor que 0**;
- un `Text` con la cantidad actual;
- un `IconButton` con el icono `add`, que suma una unidad.

La cantidad se guarda en un estado local de la tarjeta con `remember { mutableStateOf(0) }` (o `mutableIntStateOf(0)`), usando `by`.

### Ejercicio 5 — Observar la recomposición

Añade temporalmente dentro de `ProductoCard` un `Log.d("Recomposicion", ...)` que muestre el **nombre del recurso** del producto (`producto.nombre`) y su cantidad. Con el filtro `tag:Recomposicion`, pulsa "+" en una sola tarjeta y responde en `PRUEBAS.md`: **¿cuántas tarjetas se recomponen y por qué?** Después elimina ese `Log.d`.

### Ejercicio 6 — El problema del giro

Pon cantidades distintas de 0 en dos productos y gira el emulador. Documenta en `PRUEBAS.md` qué ocurre con las cantidades y **explícalo** usando la secuencia del ciclo de vida que ves en Logcat (2-3 frases).

### Ejercicio 7 — Solución con `rememberSaveable`

Cambia `remember` por `rememberSaveable` y repite la prueba del ejercicio 6. Anota el resultado.

### Ejercicio 8 — Desplazamiento en horizontal

En la sesión 1, la prueba 5 (orientación horizontal) fallaba porque la carta no cabía y no se podía desplazar. Añade `.verticalScroll(rememberScrollState())` al `Modifier` de la `Column` de `PantallaCarta` y repite esa prueba. Comprueba además si, al desplazar la carta hasta abajo y volver a girar, **la posición del desplazamiento se conserva**, y explica por qué con lo visto hoy (pista: fíjate en el nombre de la función).

### Ejercicio 9 (integrador) — "No conservar actividades"

1. Activa en el emulador *Opciones de desarrollador → No conservar actividades*.
2. Pon cantidades en varios productos, pulsa Inicio y vuelve a la app.
3. Anota en `PRUEBAS.md` la secuencia de Logcat (¿aparece `onDestroy` al pulsar Inicio?, ¿qué vale `savedInstanceState` al volver?) y si las cantidades se han conservado.
4. Desactiva la opción al terminar.

### Opcional — Provocar y leer un error

Elimina la llamada `super.onStop()` de tu `onStop`, ejecuta y pulsa Inicio. Busca el error en Logcat con `level:error` y copia en `PRUEBAS.md` la línea de la excepción y su mensaje. Restaura la línea.

<div class="callout-info">
Este ejercicio es opcional: no cuenta en los requisitos técnicos ni en los criterios de evaluación.
</div>

## Requisitos técnicos obligatorios

- Los seis métodos del ciclo de vida sobrescritos en `MainActivity`, todos con su `super` y su `Log.d` con la etiqueta `TAG`.
- `PRUEBAS.md` con la sección "Ciclo de vida" (versión de Android y las seis secuencias CV1-CV6) y los resultados de los ejercicios 5 a 9.
- Selector de cantidad dentro de `ProductoCard`, con el botón de quitar deshabilitado en 0.
- Estado de la cantidad declarado con `rememberSaveable` y `by` en la versión final.
- Iconos y `contentDescription` de los botones tomados de recursos, en español e inglés.
- `PantallaCarta` desplazable verticalmente.
- Sin `Log.d` de recomposición en la versión final.

## Criterios de evaluación

Esta práctica contribuye a los criterios **a)**, **b)** y **g)** del RA2:

- **a)** Se distingue qué código va en la `Activity` (ciclo de vida) y qué va en los composables (interfaz y su estado).
- **b)** Se usan correctamente controles interactivos (`IconButton`), su estado habilitado/deshabilitado y los eventos `onClick`.
- **g)** Se han realizado y documentado pruebas de interacción en el emulador: secuencias del ciclo de vida, giro de pantalla y destrucción de actividades, con explicación del comportamiento observado.
- El proyecto compila y se ejecuta sin errores.
