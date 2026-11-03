---
layout: unidad
title: "UD4 · Persistencia y almacenamiento"
unit: UD4
permalink: /unidades/ud4/
duration: "7h"
---

Hasta ahora, casi todo lo que hace la **Cafetería del instituto** vive en la memoria del móvil: en cuanto el usuario cierra la app, el pedido desaparece. En la UD3 dimos el primer paso con DataStore para recordar unas pocas preferencias. En esta unidad aprenderás a **guardar datos de verdad en el dispositivo**: dónde puede escribir una app Android, cómo guardar objetos completos en ficheros y, sobre todo, cómo usar una **base de datos local** con SQLite y la librería **Room** para almacenar, consultar y relacionar datos estructurados.

Seguimos trabajando sobre la misma app que en la UD3. Cada práctica parte de la solución de la anterior: al terminar la unidad, la cafetería guardará el último ticket, el historial de pedidos con sus líneas y los productos favoritos del usuario, y tendrás documentada su capa de datos.

## RA2 y criterios de evaluación

**RA2.** Desarrolla aplicaciones para dispositivos móviles analizando y empleando las tecnologías y librerías específicas.

El RA2 se reparte entre tres unidades (UD3, UD4 y UD5). En esta unidad se trabaja a fondo el criterio **f)**, y de forma parcial los criterios **g)** (pruebas en el emulador) e **i)** (documentación), que se completan en la UD5.

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
    <span class="ra-card__id">g) · UD3-UD4</span>
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

## Bloques de esta unidad

1. [Almacenamiento local: ficheros JSON y bases de datos con Room]({{ '/unidades/ud4-bloque1-ficheros-room/' | relative_url }}) — 3h (sesiones 1 y 2)

<!-- Próximos bloques (se enlazarán al publicarlos):
2. Room II: datos reactivos y relaciones — 2h (sesión 3)
3. Repositorio, migraciones y documentación — 2h (sesión 4)
-->

## Prácticas

- Bloque 1 (Ficheros JSON y Room): [Cafetería del instituto: el último ticket y el historial — enunciado]({{ '/practicas/ud4-practica1-enunciado/' | relative_url }}) · [solución]({{ '/practicas/ud4-practica1-solucion/' | relative_url }})

<div class="callout-info">
Al terminar esta unidad serás capaz de elegir dónde y cómo guardar cada tipo de dato de una app Android, de guardar objetos en ficheros JSON y de construir una base de datos local con Room que la interfaz muestra y mantiene actualizada por sí sola.
</div>
