---
title: "Catálogo de dispositivos — Enunciado"
unit: UD2
practica: 1
tipo: enunciado
---

## Contexto

Práctica guiada de cierre de la UD2. No introduce contenido nuevo: es una consolidación de los cinco bloques de la unidad. El objetivo es salir con un pequeño proyecto Compose funcionando y confianza en el flujo básico de Android Studio, antes de entrar en la UD3.

## Enunciado

Crea un proyecto Compose ("Empty Activity") que muestre en pantalla, con `Text()` dentro de una `Column`, una lista de dispositivos a partir de una lista de objetos en Kotlin.

## Requisitos técnicos obligatorios

- Una `data class Dispositivo(val modelo: String, val fabricante: String, val telefono: String?)`.
- Una función de extensión `List<Dispositivo>.deFabricante(nombre: String): List<Dispositivo>` usando `filter`.
- Uso del operador Elvis (`?:`) al mostrar el teléfono cuando sea `null`.
- Una lambda pasada como parámetro a alguna función auxiliar propia (por ejemplo, para formatear el texto de cada dispositivo).
- El listado de dispositivos se muestra dentro de `setContent`, en `MainActivity.kt`.

## Criterios de evaluación

- El proyecto compila y se ejecuta sin errores.
- Aparecen y se usan correctamente los 5 elementos obligatorios listados arriba.
- El código sigue las convenciones vistas en la unidad (`val` por defecto, nombres en camelCase, sin `!!` innecesarios).

<div class="callout-info">
Tienes 1h25 de trabajo autónomo o en parejas. Los últimos 20 minutos de la sesión se dedican a que 2-3 compañeros muestren su pantalla y se comenten alternativas.
</div>
