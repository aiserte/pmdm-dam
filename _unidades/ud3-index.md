---
layout: unidad
title: "UD3 · Interfaz de usuario con Jetpack Compose"
unit: UD3
permalink: /unidades/ud3/
duration: "11h"
---

En esta unidad pasamos de escribir Kotlin "de consola" a construir **aplicaciones Android de verdad**. Aprenderás cómo se organiza una app hecha con Jetpack Compose, cómo se usan los recursos del proyecto (textos, idiomas, imágenes), cómo gestionar el estado para que la interfaz reaccione a lo que hace el usuario, cómo mostrar listas, cómo moverse entre pantallas y cómo guardar las preferencias del usuario.

Si ya diseñas interfaces con Compose en otros módulos, aquí el foco no está en el diseño visual sino en lo que convierte una pantalla en una **app**: la estructura de clases, el ciclo de vida, el estado que sobrevive a un giro de pantalla, la navegación y la persistencia de preferencias.

A lo largo de la unidad construiremos, sesión a sesión, una misma aplicación: **Cafetería del instituto**, una carta digital con productos, pedidos y ajustes. La solución de cada práctica es el punto de partida de la siguiente.

## RA2 y criterios de evaluación

**RA2.** Desarrolla aplicaciones para dispositivos móviles analizando y empleando las tecnologías y librerías específicas.

El RA2 se reparte entre tres unidades (UD3, UD4 y UD5). En esta unidad se trabajan los criterios **a)**, **b)** y **g)**; el resto se trabajan en las unidades siguientes.

<div class="ra-grid">
  <div class="ra-card">
    <span class="ra-card__id">a) · UD3</span>
    <p class="ra-card__desc">Se ha generado la estructura de clases necesaria para la aplicación.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">b) · UD3</span>
    <p class="ra-card__desc">Se han analizado y utilizado las clases que modelan ventanas, menús, alertas y controles para el desarrollo de aplicaciones gráficas sencillas.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">c) · UD5</span>
    <p class="ra-card__desc">Se han utilizado las clases necesarias para la conexión y comunicación con dispositivos inalámbricos.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">d) · UD5</span>
    <p class="ra-card__desc">Se han desarrollado aplicaciones que hacen uso de las funcionalidades proporcionadas por los sensores.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">e) · UD5</span>
    <p class="ra-card__desc">Se han utilizado las clases necesarias para establecer conexiones y comunicaciones HTTP y HTTPS.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">f) · UD4</span>
    <p class="ra-card__desc">Se han utilizado las clases necesarias para establecer conexiones con almacenes de datos garantizando la persistencia.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">g) · UD3</span>
    <p class="ra-card__desc">Se han realizado pruebas de interacción usuario-aplicación para optimizar las aplicaciones desarrolladas a partir de emuladores.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">h) · UD5</span>
    <p class="ra-card__desc">Se han empaquetado y desplegado las aplicaciones desarrolladas en dispositivos móviles reales.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">i) · UD4-UD5</span>
    <p class="ra-card__desc">Se han documentado los procesos necesarios para el desarrollo de las aplicaciones.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">j) · UD5</span>
    <p class="ra-card__desc">Se han establecido los permisos requeridos para el funcionamiento de las aplicaciones.</p>
  </div>
</div>

<div class="callout-info">
Sobre el criterio b): Jetpack Compose no usa las clases clásicas de Android basadas en XML (<code>AlertDialog.Builder</code>, <code>Menu</code>, <code>View</code>…), sino sus equivalentes declarativos (<code>AlertDialog</code>, <code>DropdownMenu</code>, <code>Scaffold</code>, <code>TopAppBar</code>…). Cumplen la misma función —modelar ventanas, menús, alertas y controles— con la forma actual recomendada por Google para crear interfaces en Android.
</div>

## Sesiones de esta unidad

1. [Anatomía de una app Compose y recursos]({{ '/unidades/ud3-bloque1-anatomia-compose/' | relative_url }}) — 2h
2. [Ciclo de vida de la Activity, Logcat y estado]({{ '/unidades/ud3-bloque2-ciclo-vida-estado/' | relative_url }}) — 2h
3. [State hoisting, controles de entrada, diálogos y menús]({{ '/unidades/ud3-bloque3-hoisting-controles-dialogos/' | relative_url }}) — 2h
4. [Listas eficientes y ViewModel]({{ '/unidades/ud3-bloque4-listas-viewmodel/' | relative_url }}) — 2h
5. Navegación entre pantallas — 2h *(próximamente)*
6. Preferencias con DataStore — 1h *(próximamente)*

## Prácticas por sesión

- Sesión 1 (Anatomía y recursos): [Cafetería del instituto: la carta — enunciado]({{ '/practicas/ud3-sesion1-practica-enunciado/' | relative_url }}) · [solución]({{ '/practicas/ud3-sesion1-practica-solucion/' | relative_url }})
- Sesión 2 (Ciclo de vida y estado): [Cafetería del instituto: el pedido — enunciado]({{ '/practicas/ud3-sesion2-practica-enunciado/' | relative_url }}) · [solución]({{ '/practicas/ud3-sesion2-practica-solucion/' | relative_url }})
- Sesión 3 (State hoisting, controles, diálogos y menús): [Cafetería del instituto: confirmar el pedido — enunciado]({{ '/practicas/ud3-sesion3-practica-enunciado/' | relative_url }}) · [solución]({{ '/practicas/ud3-sesion3-practica-solucion/' | relative_url }})
- Sesión 4 (Listas y ViewModel): [Cafetería del instituto: carta completa y ViewModel — enunciado]({{ '/practicas/ud3-sesion4-practica-enunciado/' | relative_url }}) · [solución]({{ '/practicas/ud3-sesion4-practica-solucion/' | relative_url }})

<div class="callout-info">
Al terminar esta unidad serás capaz de construir una app Android de varias pantallas con Jetpack Compose, que reacciona a lo que hace el usuario, conserva su estado, muestra listas de datos y recuerda las preferencias del usuario entre ejecuciones.
</div>
