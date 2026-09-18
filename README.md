# PMDM - Materiales del módulo

Sitio Jekyll (tema `minima`) con los materiales del módulo de Programación
Multimedia y Dispositivos Móviles (DAM), currículum Comunitat Valenciana.

## Estructura

- `_config.yml` — configuración del sitio y declaración de colecciones.
- `assets/css/style.scss` — paleta de colores propia, sobrescribe `minima`.
- `_layouts/unidad.html` — plantilla para las páginas de unidad didáctica.
- `_layouts/practica.html` — plantilla para enunciados y soluciones de prácticas.
- `_unidades/` — una página por unidad didáctica (UD0...UD7).
- `_practicas/` — enunciado y solución en ficheros separados por práctica.

## Convención de nombres

Unidades:
```
_unidades/ud00-nivelacion.md
_unidades/ud01-tecnologias.md
_unidades/ud02-interfaz-compose.md
...
```

Prácticas (enunciado y solución SIEMPRE en ficheros distintos):
```
_practicas/ud02-p01-enunciado.md
_practicas/ud02-p01-solucion.md
```

## Front matter esperado

Unidad:
```yaml
---
title: "UD1 · Tecnologías para dispositivos móviles"
ra: "RA1"
horas: 14
---
```

Práctica:
```yaml
---
title: "Práctica 1 · Mi primera pantalla con Compose"
unidad: "UD2"
tipo: "enunciado"   # o "solucion"
---
```
