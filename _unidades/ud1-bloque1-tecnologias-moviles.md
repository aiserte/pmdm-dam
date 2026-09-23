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

<figure class="diagrama-ciclo-vida">
<svg viewBox="0 0 900 520" role="img" aria-labelledby="ciclovida-titulo ciclovida-desc" style="width:100%;height:auto;font-family:Helvetica,Arial,sans-serif;">
  <title id="ciclovida-titulo">Ciclo de vida de una aplicación móvil</title>
  <desc id="ciclovida-desc">Cinco fases en secuencia -Descubrimiento, Instalación, Ejecución, Actualización y Borrado-, con la actualización volviendo a la ejecución; durante la ejecución, la app alterna entre los estados Activa y En pausa hasta que finalmente pasa a Destruida.</desc>

  <defs>
    <marker id="fl-acento" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#534AB7"></path>
    </marker>
    <marker id="fl-aviso" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#BA7517"></path>
    </marker>
    <marker id="fl-primario" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#0F6E56"></path>
    </marker>
  </defs>

  <!-- Actualización vuelve a Ejecución (detrás de las cajas, para que no tape el texto) -->
  <path d="M 645 70 C 645 20, 460 20, 460 66" fill="none" stroke="#534AB7" stroke-width="2" stroke-dasharray="5 4" marker-end="url(#fl-acento)"></path>
  <text x="552" y="16" text-anchor="middle" font-size="12" fill="#5F5E5A">vuelve a ejecutarse</text>

  <!-- ===== Fila superior: fases del ciclo de vida ===== -->
  <!-- Descubrimiento -->
  <rect x="15" y="70" width="150" height="64" rx="10" fill="#E1F5EE" stroke="#0F6E56" stroke-width="2"></rect>
  <text x="90" y="107" text-anchor="middle" font-size="15" font-weight="700" fill="#0F6E56">Descubrimiento</text>

  <!-- Instalación -->
  <rect x="200" y="70" width="150" height="64" rx="10" fill="#E1F5EE" stroke="#0F6E56" stroke-width="2"></rect>
  <text x="275" y="107" text-anchor="middle" font-size="15" font-weight="700" fill="#0F6E56">Instalación</text>

  <!-- Ejecución (destacada) -->
  <rect x="385" y="70" width="150" height="64" rx="10" fill="#0F6E56" stroke="#0F6E56" stroke-width="2"></rect>
  <text x="460" y="107" text-anchor="middle" font-size="15" font-weight="700" fill="#E1F5EE">Ejecución</text>

  <!-- Actualización -->
  <rect x="570" y="70" width="150" height="64" rx="10" fill="#E1F5EE" stroke="#0F6E56" stroke-width="2"></rect>
  <text x="645" y="107" text-anchor="middle" font-size="15" font-weight="700" fill="#0F6E56">Actualización</text>

  <!-- Borrado -->
  <rect x="755" y="70" width="130" height="64" rx="10" fill="#E1F5EE" stroke="#0F6E56" stroke-width="2"></rect>
  <text x="820" y="107" text-anchor="middle" font-size="15" font-weight="700" fill="#0F6E56">Borrado</text>

  <!-- Flechas entre fases -->
  <line x1="165" y1="102" x2="196" y2="102" stroke="#534AB7" stroke-width="2.5" marker-end="url(#fl-acento)"></line>
  <line x1="350" y1="102" x2="381" y2="102" stroke="#534AB7" stroke-width="2.5" marker-end="url(#fl-acento)"></line>
  <line x1="535" y1="102" x2="566" y2="102" stroke="#534AB7" stroke-width="2.5" marker-end="url(#fl-acento)"></line>
  <line x1="720" y1="102" x2="751" y2="102" stroke="#534AB7" stroke-width="2.5" marker-end="url(#fl-acento)"></line>

  <!-- Conector de Ejecución al detalle de estados -->
  <line x1="460" y1="134" x2="460" y2="170" stroke="#0F6E56" stroke-width="2.5" marker-end="url(#fl-primario)"></line>

  <!-- ===== Panel inferior: estados durante la ejecución ===== -->
  <rect x="60" y="180" width="780" height="320" rx="14" fill="none" stroke="#5F5E5A" stroke-width="1.5" stroke-dasharray="6 5"></rect>
  <text x="90" y="208" font-size="13" font-weight="700" fill="#5F5E5A">MIENTRAS LA APP ESTÁ EN EJECUCIÓN: TRES ESTADOS POSIBLES</text>

  <!-- Activa -->
  <circle cx="300" cy="340" r="80" fill="#E1F5EE" stroke="#0F6E56" stroke-width="2.5"></circle>
  <text x="300" y="334" text-anchor="middle" font-size="17" font-weight="700" fill="#0F6E56">Activa</text>
  <text x="300" y="356" text-anchor="middle" font-size="12" fill="#2C2C2A">primer plano,</text>
  <text x="300" y="372" text-anchor="middle" font-size="12" fill="#2C2C2A">interactuando</text>

  <!-- En pausa -->
  <circle cx="600" cy="340" r="80" fill="#EEEDFE" stroke="#534AB7" stroke-width="2.5"></circle>
  <text x="600" y="334" text-anchor="middle" font-size="17" font-weight="700" fill="#534AB7">En pausa</text>
  <text x="600" y="356" text-anchor="middle" font-size="12" fill="#2C2C2A">visible, sin</text>
  <text x="600" y="372" text-anchor="middle" font-size="12" fill="#2C2C2A">el foco</text>

  <!-- Activa <-> En pausa -->
  <line x1="382" y1="326" x2="518" y2="326" stroke="#534AB7" stroke-width="2.5" marker-end="url(#fl-acento)"></line>
  <line x1="518" y1="354" x2="382" y2="354" stroke="#534AB7" stroke-width="2.5" marker-end="url(#fl-acento)"></line>

  <!-- Destruida -->
  <ellipse cx="450" cy="472" rx="95" ry="20" fill="#FAEEDA" stroke="#BA7517" stroke-width="2.5"></ellipse>
  <text x="450" y="478" text-anchor="middle" font-size="15" font-weight="700" fill="#8F5B12">Destruida</text>

  <!-- Activa -> Destruida / En pausa -> Destruida -->
  <line x1="330" y1="412" x2="410" y2="458" stroke="#BA7517" stroke-width="2.5" marker-end="url(#fl-aviso)"></line>
  <line x1="570" y1="412" x2="490" y2="458" stroke="#BA7517" stroke-width="2.5" marker-end="url(#fl-aviso)"></line>
</svg>
<figcaption>Ciclo de vida de una app: cinco fases (arriba) y, dentro de la fase de ejecución, los tres estados posibles (abajo).</figcaption>
</figure>

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
