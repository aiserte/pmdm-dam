---
title: "Tecnologías móviles: panorama y elección de Kotlin + Compose"
unit: UD1
order: 1
duration: "2h"
---

## Objetivos

- Conocer las características, tipos y limitaciones de los dispositivos móviles actuales.
- Identificar los principales sistemas operativos móviles vigentes.
- Distinguir entre desarrollo nativo y multiplataforma, y los lenguajes y entornos de cada uno.
- Comprender para qué sirve un emulador y qué es un entorno integrado de desarrollo (IDE).
- Entender, a nivel de panorama, la estructura y el ciclo de vida de una aplicación móvil.
- Saber por qué este módulo trabaja con Kotlin y Jetpack Compose sobre Android.

## 1.1 Introducción

Los dispositivos móviles son hoy la plataforma de cómputo más usada del mundo, por delante del ordenador de escritorio. Esto ha creado un sector profesional propio —el desarrollo de aplicaciones móviles— con tecnologías, lenguajes y entornos de trabajo que no siempre coinciden con los que ya conocéis del desarrollo de escritorio o web.

En esta unidad no vamos a instalar nada todavía: vamos a construir el mapa. Dispositivo, sistema operativo, tecnología de desarrollo y entorno de trabajo son las cuatro piezas que hay que entender antes de tomar cualquier decisión técnica, y las cuatro confluyen en un mismo sitio: la aplicación móvil, con su estructura y su ciclo de vida propios. En UD2 empezaréis a trabajar ya con Android Studio, Kotlin y el emulador.

<div class="callout-info">
<strong>Glosario rápido:</strong> <em>app</em> es una aplicación de software pensada para ejecutarse en un dispositivo móvil; <em>SO móvil</em> es el software base que gestiona el hardware del dispositivo y ejecuta sus apps; <em>IDE</em> es el entorno integrado de desarrollo (editor, compilador, depurador...) en un solo programa; <em>emulador</em> es un programa que simula un dispositivo móvil en el ordenador para probar apps sin necesidad de un terminal físico.
</div>

## 1.2 Dispositivos móviles: características y hardware

Un dispositivo móvil se define por ser portátil, tener conectividad inalámbrica y funcionar con batería. Sus componentes hardware principales, que toda app debe tener en cuenta, son:

- **Pantalla**: tamaño, resolución y densidad de píxeles (afectan al diseño de la interfaz).
- **Procesador (SoC) y memoria RAM**: determinan el rendimiento disponible.
- **Almacenamiento**: espacio interno, a veces ampliable.
- **Batería**: recurso limitado que toda app debe gestionar con cuidado.
- **Cámara y sensores**: acelerómetro, GPS, giroscopio, huella digital, etc.
- **Conectividad**: red móvil, Wi-Fi, Bluetooth, NFC.

## 1.3 Tipos de dispositivos móviles

Smartphones, tablets, phablets, wearables (relojes inteligentes) y dispositivos embebidos/IoT con capacidades similares. Cada tipo condiciona el tamaño de pantalla, la interacción (táctil, voz, gestos) y el consumo energético que la app debe respetar.

## 1.4 Conectividad y comunicación móvil

La conexión a redes es una de las señas de identidad del dispositivo móvil:

- **Redes celulares**: evolución de 3G a 4G/LTE y 5G, con mejoras de velocidad y latencia.
- **Wi-Fi**: conexión de mayor ancho de banda dentro de un área limitada.
- **Bluetooth y NFC**: comunicación de corto alcance entre dispositivos (accesorios, pagos, IoT).

<div class="callout-warning">
Hay que programar asumiendo que la conectividad puede fallar o desaparecer en cualquier momento: ninguna app móvil debería dar por hecho que la red siempre está disponible.
</div>

## 1.5 Limitaciones de los dispositivos móviles

Frente a un ordenador de escritorio, un dispositivo móvil impone restricciones que condicionan el diseño de cualquier aplicación (CE a):

| Ordenador de escritorio | Dispositivo móvil |
|---|---|
| Alimentación continua | Autonomía de batería limitada |
| Alta capacidad de proceso y memoria | Menor proceso, memoria y almacenamiento |
| Pantalla y periféricos amplios | Pantalla reducida y entrada táctil |
| Conexión estable por cable | Conectividad intermitente e interrupciones frecuentes |

## 1.6 Sistemas operativos: Android e iOS

**Android (Google).** Basado en Linux, de código mayoritariamente abierto (AOSP). Es el sistema operativo móvil más usado del mundo, con gran fragmentación de fabricantes y versiones. Se programa oficialmente en Kotlin (antes Java) usando Android Studio. Es el sistema que usaremos en este módulo.

**iOS (Apple).** Sistema operativo exclusivo de los dispositivos de Apple (iPhone, iPad), muy homogéneo en hardware y versiones. Se programa en Swift usando Xcode, disponible solo en macOS. Lo mencionamos como referencia para entender el ecosistema, aunque en este módulo trabajaremos con Android.

## 1.7 Otros sistemas operativos: panorama actual y descatalogados

Android e iOS concentran hoy prácticamente todo el mercado. Como alternativa activa destaca **HarmonyOS** de Huawei, con presencia relevante en China. Otros sistemas que tuvieron peso en su momento —Windows Phone, BlackBerry OS, Symbian, Palm OS/webOS, Firefox OS o Ubuntu Touch— han sido descontinuados y ya no reciben desarrollo ni soporte comercial; se citan aquí solo como contexto histórico.

## 1.8 Tecnologías y lenguajes de desarrollo

**Desarrollo nativo.** Se programa con el lenguaje y el SDK oficial de cada sistema operativo (Kotlin/Java para Android, Swift/Objective-C para iOS). Ofrece el mejor rendimiento y acceso completo a las funciones del dispositivo, a cambio de mantener un proyecto distinto por cada plataforma.

**Desarrollo multiplataforma compilado a nativo.** Un único código fuente se compila a código nativo para varios sistemas operativos. Es la tendencia dominante actual: Kotlin Multiplatform (Google/JetBrains, comparte lógica de negocio entre Android e iOS), Flutter (Google, lenguaje Dart) y React Native (Meta, JavaScript/TypeScript).

**Desarrollo multiplataforma basado en tecnologías web.** Aplicaciones construidas con HTML, CSS y JavaScript que se ejecutan dentro de un contenedor (PWA, Ionic/Capacitor). Muy portables, pero con más limitaciones de rendimiento y acceso al hardware que las dos opciones anteriores.

## 1.9 Entornos de desarrollo y emuladores

**Android Studio** es el IDE oficial para el desarrollo Android, basado en IntelliJ IDEA. Incluye editor de código, diseñador visual, depurador, gestor de dependencias (Gradle) y el emulador de dispositivos. Será nuestra herramienta principal a partir de UD2.

**Xcode** es el IDE oficial de Apple para iOS/macOS, disponible únicamente sobre macOS. Se menciona como referencia del ecosistema iOS.

Un **emulador** permite ejecutar y probar una app en un dispositivo virtual sin necesidad de un terminal físico (CE h). Al crear uno se configuran: el perfil de dispositivo (modelo, tamaño de pantalla y densidad a simular), la versión del sistema operativo a emular y las características de hardware simuladas (cámara, sensores, tarjeta SD...). Estos perfiles son los que relacionan un dispositivo concreto con los requisitos de una aplicación (CE e).

## 1.10 Estructura y ciclo de vida de una aplicación móvil

Toda app móvil se organiza como un conjunto de componentes (pantallas, servicios, clases de datos) relacionados en una jerarquía. Analizar aplicaciones ya existentes para identificar estas clases (CE f) es una de las actividades prácticas de UD2.

Una app siempre se encuentra en uno de estos tres **estados**:

- **Activa**: en primer plano, visible e interactuando con el usuario.
- **En pausa**: visible pero sin el foco de interacción (por ejemplo, tapada parcialmente por otra ventana).
- **Destruida**: cerrada y eliminada de memoria por el sistema o por el usuario.

A lo largo de su existencia, una app pasa por las siguientes fases de su **ciclo de vida**: descubrimiento (tienda de apps), instalación, ejecución, actualización y borrado. El sistema operativo gestiona automáticamente estas transiciones mediante su entorno de ejecución del administrador de aplicaciones, responsable también de instalar, actualizar, desinstalar y gestionar permisos.

## 1.11 Por qué Kotlin + Jetpack Compose en este módulo

Con el panorama anterior ya podemos justificar la elección técnica del módulo:

- Trabajamos sobre **Android** porque es el sistema operativo con más cuota de mercado y el que mejor encaja con un entorno educativo sin dispositivos Apple garantizados.
- Programamos en **desarrollo nativo** (no multiplataforma) porque el objetivo es aprender en profundidad cómo funciona una app Android por dentro —estructura, ciclo de vida, permisos, acceso a hardware— y eso se entiende mejor con las herramientas oficiales que a través de una capa multiplataforma.
- Usamos **Kotlin** porque es el lenguaje oficial de Android desde 2019, plenamente soportado por Google, e interoperable con Java si en algún momento os encontráis código heredado.
- Usamos **Jetpack Compose** en lugar de las Views clásicas con XML porque es el sistema de interfaz de usuario moderno y recomendado por Google: declarativo, más conciso y con menos código repetitivo.

<div class="ejercicio" markdown="1">
1. Cita tres limitaciones de un dispositivo móvil frente a un ordenador de escritorio.
2. ¿Qué diferencia hay entre desarrollo nativo y desarrollo multiplataforma? Pon un ejemplo de tecnología de cada tipo.
3. ¿Para qué sirve un emulador y qué elementos se configuran en un perfil de dispositivo virtual?
4. Enumera los tres estados posibles de una aplicación móvil y pon un ejemplo de cuándo se produce cada uno.
5. Ordena las fases del ciclo de vida de una aplicación: instalación, borrado, actualización, descubrimiento, ejecución.
6. Cita un sistema operativo móvil descatalogado y el que lo ha sustituido en el mercado actual.
7. ¿Por qué este módulo usa Kotlin y Jetpack Compose en lugar de una tecnología multiplataforma?
</div>

## Resumen

Un dispositivo móvil es portátil, tiene batería y conectividad inalámbrica, lo que impone limitaciones (batería, proceso, pantalla) que condicionan cualquier desarrollo. Android e iOS dominan el mercado actual de sistemas operativos; otros sistemas históricos ya están descatalogados. Para programar apps se puede optar por desarrollo nativo (mejor rendimiento, un proyecto por SO) o multiplataforma (un solo código para varios SO); en este módulo elegimos nativo, en Android, con Kotlin y Jetpack Compose. El trabajo diario se apoyará en Android Studio y en emuladores que permiten probar la app sin dispositivo físico. Toda app tiene una estructura de clases propia y atraviesa un ciclo de vida con tres estados (activa, pausada, destruida) gestionado por el sistema operativo.

## Para la próxima sesión

UD2 · Introducción a Kotlin y Android Studio: repaso de programación orientada a objetos desde Java, sintaxis de Kotlin e instalación de Android Studio con vuestro primer proyecto.
