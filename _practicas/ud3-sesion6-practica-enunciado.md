---
title: "Cafetería del instituto: recordar al cliente — Enunciado (Sesión 6)"
unidad: UD3
sesion: 6
orden: 6
tipo: enunciado
---

## Contexto

Práctica de la **Sesión 6** (*Preferencias del usuario con DataStore*), la última de la unidad. Partimos de la solución de la sesión 5: la cafetería con cuatro pantallas (carta, detalle, resumen y pedido enviado) y un único `PedidoViewModel` compartido.

Hasta ahora, cada vez que se abre la app hay que volver a escribir el nombre y marcar si el pedido es para llevar. Hoy la app **recordará los datos del último pedido enviado**, aunque se cierre la app o se reinicie el móvil, y permitirá **olvidarlos** desde el menú de la carta.

<div class="callout-warning">
Usa lo visto hasta ahora: todo lo de las sesiones 1 a 5 más DataStore (<code>preferencesDataStore</code>, claves, <code>data.map</code>, <code>first</code>, <code>edit</code>, <code>clear</code>), <code>AndroidViewModel</code> y <code>viewModelScope</code>.
</div>

<div class="callout-info">
Si no terminaste la práctica de la sesión 5, parte de su solución publicada. Es una práctica corta (unos 40 minutos): solo se toca la capa de datos, el <code>ViewModel</code> y una opción del menú.
</div>

## Enunciado

### Ejercicio 1 — Dependencia y recursos

1. Añade la librería `datastore-preferences` al catálogo de versiones y a `app/build.gradle.kts`, como se explica en el apartado 6.3 de la teoría, y sincroniza Gradle.
2. Añade estos textos en español y en inglés: `olvidar_datos` → "Olvidar mis datos" y `datos_olvidados` → "Se han borrado tus datos".

### Ejercicio 2 — El repositorio de preferencias

Crea `data/PreferenciasRepository.kt` con:

1. La propiedad de extensión `Context.dataStore`, con el fichero `"preferencias"`, declarada en el nivel superior del fichero.
2. Una `data class PreferenciasCliente(nombre: String = "", paraLlevar: Boolean = false)`.
3. Una clase `PreferenciasRepository(private val dataStore: DataStore<Preferences>)` con:
   - dos claves privadas: `nombre_cliente` (`String`) y `para_llevar` (`Boolean`);
   - `val preferencias: Flow<PreferenciasCliente>`, con los valores por defecto si no hay nada guardado;
   - `suspend fun guardar(nombre: String, paraLlevar: Boolean)`;
   - `suspend fun borrar()`.

### Ejercicio 3 — El `ViewModel` recuerda al cliente

Modifica `PedidoViewModel`:

1. Conviértelo en `AndroidViewModel`, que recibe la `Application` en su constructor, y crea dentro el `PreferenciasRepository`.
2. En el bloque `init`, con `viewModelScope`, lee **una vez** las preferencias guardadas y cópialas al `nombre` y a `paraLlevar` del estado. Añade un `Log.d` con la etiqueta `PedidoVM` que muestre lo que se ha cargado.
3. En `enviarPedido()`, después de vaciar el pedido, guarda el nombre y la opción "para llevar" del pedido enviado. Añade otro `Log.d` cuando se guarden.
4. Crea `fun olvidarDatos()`: vacía el nombre, pone `paraLlevar` a `false` y borra las preferencias guardadas.

<div class="callout-warning">
Revisa el orden de las declaraciones dentro de la clase: el repositorio y <code>uiState</code> deben estar escritos <strong>antes</strong> que el bloque <code>init</code> que los usa (apartado 6.7).
</div>

### Ejercicio 4 — "Olvidar mis datos" en el menú

1. Añade a `BarraSuperiorCarta` un parámetro `onOlvidarDatos: () -> Unit` y una segunda opción en su `DropdownMenu`, "Olvidar mis datos", siempre habilitada.
2. En `PantallaCarta`, al elegirla, llama a `olvidarDatos()` del `ViewModel` y muestra un *Snackbar* con el texto `datos_olvidados`.

### Ejercicio 5 — Pruebas en el emulador

Añade a `PRUEBAS.md` una sección **"Sesión 6"**. Filtra Logcat con `tag:PedidoVM`.

| # | Prueba | Resultado esperado |
|---|---|---|
| P6-1 | Primera ejecución (o tras desinstalar): abrir el resumen | Campo de nombre vacío y "Para llevar" desactivado |
| P6-2 | Enviar un pedido a nombre de "Ana", para llevar. Cerrar la app deslizándola fuera de las apps recientes y volver a abrirla | En el resumen aparecen "Ana" y "Para llevar" activado. Copia los dos mensajes de Logcat (guardado y carga) |
| P6-3 | Escribir otro nombre en el resumen **sin enviar** el pedido, cerrar la app y volver a abrirla | Vuelve a aparecer "Ana": solo se recuerdan los datos de pedidos enviados |
| P6-4 | Con *Device Explorer*, localizar el fichero de preferencias | Anota la ruta completa y el tamaño del fichero |
| P6-5 | Elegir "Olvidar mis datos" en la carta, cerrar la app y volver a abrirla | Aparece el *Snackbar*; al volver, el nombre está vacío. Comprueba también en *Device Explorer* qué ha pasado con el fichero |
| P6-6 | Enviar un pedido y después borrar los datos de la app desde *Ajustes › Aplicaciones* | Documenta qué se pierde y por qué |

### Opcional — Recordar el filtro de la carta

Guarda también la última categoría seleccionada en el filtro y restáurala al abrir la app. Pista: no existe una clave para `enum`, pero puedes guardar el **nombre** del valor (`categoria.name`, un `String`) y recuperarlo con `Categoria.valueOf(...)`. Decide qué guardar cuando la selección es "Todo" (`null`).

<div class="callout-info">
Este ejercicio es opcional: no cuenta en los requisitos técnicos ni en los criterios de evaluación.
</div>

## Requisitos técnicos obligatorios

- Dependencia de DataStore añadida mediante el catálogo de versiones.
- Un único `Context.dataStore` declarado en el nivel superior de `PreferenciasRepository.kt`.
- `PreferenciasRepository` con claves privadas, un `Flow<PreferenciasCliente>` con valores por defecto, y las funciones `suspend` `guardar` y `borrar`.
- `PedidoViewModel` como `AndroidViewModel`, sin ninguna referencia a la `Activity`.
- Carga de preferencias en `init` y guardado en `enviarPedido()`, ambos con `viewModelScope`.
- Opción "Olvidar mis datos" en el menú, con *Snackbar*.
- Ningún composable usa DataStore directamente: solo el repositorio lo conoce.
- `PRUEBAS.md` con la sección "Sesión 6" completa.

## Criterios de evaluación

Esta práctica contribuye a los criterios **a)**, **b)** y **g)** del RA2:

- **a)** La app tiene una capa de datos separada (`PreferenciasRepository`) que usa el `ViewModel`, sin que la interfaz conozca cómo se guardan los datos.
- **b)** Se amplía el menú de la barra superior y se informa al usuario con un *Snackbar*.
- **g)** Se ha comprobado y documentado en el emulador que las preferencias sobreviven al cierre de la app y cuándo se pierden.
- El proyecto compila y se ejecuta sin errores.
