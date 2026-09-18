---
layout: default
title: Inicio
---

# PMDM · Programación Multimedia y Dispositivos Móviles

Materiales del módulo, organizados por unidad didáctica.

## Unidades

<ul>
{% assign unidades_ordenadas = site.unidades | sort: "unit" %}
{% for unidad in unidades_ordenadas %}
  <li><a href="{{ unidad.url }}">{{ unidad.title }}</a></li>
{% endfor %}
</ul>
