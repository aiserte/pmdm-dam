---
layout: default
title: "PMDM · Programación Multimedia y Dispositivos Móviles"
permalink: /
---

<div class="modulo-hero">
  <p class="modulo-hero__kicker">Ciclo Formativo de Grado Superior · Desarrollo de Aplicaciones Multiplataforma (DAM) · 2º curso</p>
  <h1 class="modulo-hero__titulo">PMDM</h1>
  <p class="modulo-hero__subtitulo">Programación Multimedia y Dispositivos Móviles</p>
  <p class="modulo-hero__profesora">Profesora: Alicia Iserte Tena</p>

  <div class="modulo-hero__stats">
    <div class="stat">
      <span class="stat__numero">87 h</span>
      <span class="stat__etiqueta">de formación en total</span>
    </div>
    <div class="stat">
      <span class="stat__numero">8</span>
      <span class="stat__etiqueta">unidades didácticas</span>
    </div>
    <div class="stat">
      <span class="stat__numero">2</span>
      <span class="stat__etiqueta">exámenes de evaluación</span>
    </div>
    <div class="stat">
      <span class="stat__numero">4 h</span>
      <span class="stat__etiqueta">por semana, en 2 sesiones de 2h</span>
    </div>
  </div>

  <p class="modulo-hero__intro">
    PMDM te lleva a construir aplicaciones Android reales: desde la interfaz que ve el usuario hasta el
    almacenamiento de datos, la conexión a internet, el uso de sensores, el contenido multimedia y, para
    cerrar, tus propios videojuegos 2D/3D. Iremos construyendo apps paso a paso, con explicaciones y
    práctica guiada en cada unidad: no hace falta experiencia previa en desarrollo móvil.
  </p>

  <p class="modulo-hero__herramientas">
    <strong>Herramientas y lenguaje de trabajo:</strong> Kotlin · Android Studio · Jetpack Compose · dispositivo real
  </p>
</div>

## Resultados de aprendizaje (RA)

Al terminar el módulo serás capaz de…

<div class="ra-grid">
  <div class="ra-card">
    <span class="ra-card__id">RA1</span>
    <h3 class="ra-card__titulo">Aplicar tecnologías para dispositivos móviles</h3>
    <p class="ra-card__desc">Analizar hardware, entornos de desarrollo, emuladores y el ciclo de vida de una app.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">RA2</span>
    <h3 class="ra-card__titulo">Desarrollar aplicaciones para dispositivos móviles</h3>
    <p class="ra-card__desc">Interfaz de usuario, persistencia de datos, conectividad, sensores y permisos.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">RA3</span>
    <h3 class="ra-card__titulo">Integrar contenidos multimedia</h3>
    <p class="ra-card__desc">Reproducción, procesamiento y animación de imágenes, audio y vídeo.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">RA4</span>
    <h3 class="ra-card__titulo">Seleccionar y probar motores de juegos</h3>
    <p class="ra-card__desc">Comparar motores, sus componentes y librerías 2D/3D.</p>
  </div>
  <div class="ra-card">
    <span class="ra-card__id">RA5</span>
    <h3 class="ra-card__titulo">Desarrollar juegos 2D y 3D sencillos</h3>
    <p class="ra-card__desc">Escenas, físicas, cámaras, audio y optimización de un videojuego propio.</p>
  </div>
</div>

## Así se organiza el módulo

8 unidades didácticas + 2 exámenes de evaluación · 87 horas en total.

{% assign bloques = site.data.unidades | group_by: "bloque" %}
{% for bloque in bloques %}
<div class="secuencia-bloque">
  <h3 class="secuencia-bloque__titulo">{{ bloque.name }}</h3>
  <div class="secuencia-lista">
    {% for u in bloque.items %}
    <div class="unidad-card{% if u.tipo == 'examen' %} unidad-card--examen{% endif %}{% if u.disponible %} unidad-card--disponible{% endif %}">
      <div class="unidad-card__cabecera">
        <span class="unidad-card__id">{{ u.id }}</span>
        <span class="unidad-card__horas">{{ u.horas }}h</span>
      </div>
      <h4 class="unidad-card__titulo">
        {% if u.disponible %}
        <a href="{{ '/unidades/' | append: u.slug | append: '/' | relative_url }}">{{ u.titulo }}</a>
        {% else %}
        {{ u.titulo }}
        {% endif %}
      </h4>
      <p class="unidad-card__ra">{{ u.ra }}</p>
      <p class="unidad-card__resumen">{{ u.resumen }}</p>
      {% if u.disponible %}
      <a class="unidad-card__enlace" href="{{ '/unidades/' | append: u.slug | append: '/' | relative_url }}">Ver contenidos →</a>
      {% else %}
      <span class="unidad-card__proximamente">Próximamente</span>
      {% endif %}
    </div>
    {% endfor %}
  </div>
</div>
{% endfor %}

<div class="callout-info">
La 1ª evaluación cierra tras UD5 (RA1 + RA2 completos); la 2ª evaluación cierra el módulo tras UD8 (RA3 + RA4 + RA5).
</div>