---
title: "Instalación y primer proyecto en Android Studio"
unit: UD2
order: 5
duration: "2h"
---

## Objetivos

- Instalar y configurar Android Studio correctamente.
- Entender la estructura básica de un proyecto Android.
- Crear, ejecutar y depurar un primer proyecto con Jetpack Compose.

## 1. Instalación de Android Studio

- Descargar desde developer.android.com/studio (versión estable, no Canary/Beta).
- Requisitos mínimos: 8 GB de RAM recomendados, 8 GB de espacio libre en disco.
- Durante la instalación, aceptar el Android SDK, el Android Virtual Device (AVD) y las herramientas de línea de comandos.

<div class="callout-info">
Si vais a usar un dispositivo físico en lugar de emulador, activad las "Opciones de desarrollador" (Ajustes → Acerca del teléfono → pulsar 7 veces sobre "Número de compilación") y, dentro de ellas, la "Depuración USB".
</div>

## 2. Estructura de un proyecto Android

- `manifests/AndroidManifest.xml` — declara la app: permisos, actividades, nombre, icono.
- `java/` (o kotlin+java) — vuestro código Kotlin, organizado por paquetes.
- `res/` — recursos: `res/drawable` (imágenes), `res/values` (textos, colores, estilos), `res/mipmap` (iconos).
- `build.gradle.kts (Module :app)` — dependencias del proyecto.

## 3. Crear el primer proyecto

- File → New → New Project → plantilla "Empty Activity" (ya trae Jetpack Compose configurado).
- Nombre del proyecto, package name (p. ej. `com.iesvuestro.pmdm.ud2`), lenguaje: Kotlin, Minimum SDK: API 24 o superior.
- Esperar a que Gradle sincronice antes de tocar nada.

## 4. Anatomía de MainActivity.kt

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Text(text = "Hola, PMDM")
        }
    }
}
```

- `: ComponentActivity()` — herencia, igual que vimos en el bloque 1.
- `onCreate` — método del ciclo de vida (se retomará con detalle más adelante).
- `setContent { ... }` — recibe una lambda (bloque 3) donde se declara la interfaz con funciones `@Composable`.

## 5. Ejecutar la app

- Seleccionar el dispositivo/emulador en la barra superior.
- Pulsar ▶ Run (o Shift+F10).
- La primera compilación puede tardar varios minutos; las siguientes son mucho más rápidas.

## 6. Errores comunes al empezar

- **"No target device found"**: no hay ningún AVD creado ni dispositivo conectado — crear un AVD desde Device Manager.
- **Símbolos `remember`, `getValue`, `setValue` no resueltos**: falta el import — activar auto-import (Settings → Editor → General → Auto Import) o `Alt+Enter` sobre el error.
- **Gradle no sincroniza**: comprobar conexión a internet y, si persiste, File → Invalidate Caches / Restart.

## Para la próxima sesión

Bloque 6 (práctica de cierre): un mini-proyecto guiado que combina todo lo visto en esta unidad antes de entrar de lleno en la UD3.
