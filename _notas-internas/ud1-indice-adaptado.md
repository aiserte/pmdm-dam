---
title: UD1 · Tecnologías móviles — Índice adaptado y sesión introductoria
---

# UD1 · Tecnologías móviles (RA1) — Adaptación del índice y sesión introductoria

**Estado:** entregado (apuntes .docx + presentación .pptx de la sesión introductoria). Cubre solo la sesión introductoria (teoría); las actividades prácticas de esta misma UD1 (instalación de Android Studio, uso de emuladores, análisis y modificación de una app existente) se desarrollan en sesiones posteriores dentro de las 14h asignadas a UD1.

**Fuente normativa:** RD 405/2023, Anexo I, módulo 0489 (`BOEA202313221.pdf`). RA1 y sus 8 CE (a-h) tomados literalmente de ese documento.

## Punto de partida

El profesor aportó el índice de un libro de texto genérico de "Tecnologías móviles" (temario 11-15 de un manual no oficial) para adaptarlo al currículo actual y a esta programación.

## Cambios aplicados sobre el índice original

**Eliminado (obsoleto):** epígrafes propios para Windows Phone, BlackBerry OS, Symbian, Palm OS/WebOS, Firefox OS y Ubuntu Touch como sistemas operativos independientes, y el desglose de un IDE por cada uno de ellos (sección 15 original). Se sustituyen por una única sección "Otros sistemas operativos: panorama actual y sistemas descatalogados" que los cita solo como contexto histórico y añade HarmonyOS como alternativa activa (China).

**Actualizado:** sección de lenguajes/tecnologías de desarrollo reestructurada en desarrollo nativo, multiplataforma compilado a nativo (Kotlin Multiplatform, Flutter, React Native) y multiplataforma basado en web (PWA, Ionic/Capacitor) — sustituye la referencia a "HTML5" del índice original. Sección de IDEs reducida a Android Studio + Xcode + emuladores (sin IDEs de plataformas descatalogadas).

**Añadido (hueco frente al RD):** el índice original no cubría "Aplicaciones móviles: estructura, jerarquía de clases", el modelo de estados (activo/pausa/destruido), el ciclo de vida (descubrimiento/instalación/ejecución/actualización/borrado), la modificación de apps existentes ni el entorno de ejecución del administrador de aplicaciones — todo ello contenido básico explícito del bloque RA1 en el RD y ligado a los CE f) y g). Se añadió como epígrafe nuevo **1.6 Estructura y ciclo de vida de una aplicación móvil**.

## Índice adaptado (final)

1.1 Introducción
1.2 Dispositivos móviles: características y limitaciones
&nbsp;&nbsp;1.2.1 Características y hardware de los dispositivos móviles
&nbsp;&nbsp;1.2.2 Tipos de dispositivos móviles
&nbsp;&nbsp;1.2.3 Conectividad y tecnología de comunicación móvil
&nbsp;&nbsp;1.2.4 Limitaciones de los dispositivos móviles
1.3 Sistemas operativos móviles
&nbsp;&nbsp;1.3.1 Android
&nbsp;&nbsp;1.3.2 iOS
&nbsp;&nbsp;1.3.3 Otros sistemas operativos: panorama actual y sistemas descatalogados
1.4 Tecnologías y lenguajes de desarrollo
&nbsp;&nbsp;1.4.1 Desarrollo nativo
&nbsp;&nbsp;1.4.2 Desarrollo multiplataforma compilado a nativo
&nbsp;&nbsp;1.4.3 Desarrollo multiplataforma basado en tecnologías web
1.5 Entornos de desarrollo y emuladores
&nbsp;&nbsp;1.5.1 Android Studio
&nbsp;&nbsp;1.5.2 Xcode
&nbsp;&nbsp;1.5.3 Emuladores y dispositivos virtuales
1.6 Estructura y ciclo de vida de una aplicación móvil
&nbsp;&nbsp;1.6.1 Estructura de una app y jerarquía de clases
&nbsp;&nbsp;1.6.2 Modelo de estados de una app
&nbsp;&nbsp;1.6.3 Ciclo de vida de una aplicación
&nbsp;&nbsp;1.6.4 Modificación de apps existentes y administrador de aplicaciones

(Se mantienen Objetivos, Mapa conceptual, Glosario, Resumen y Actividades de autoevaluación como en el original.)

## Cobertura de CE de RA1 en este índice

a) limitaciones → 1.2.4 · b) tecnologías de desarrollo → 1.4 · c) entornos de trabajo → 1.5 (instalación real en sesión práctica posterior) · d) clasificación por características → 1.2.1/1.2.2 · e) perfiles dispositivo-app → 1.5.3 · f) estructura/clases de apps existentes → 1.6.1 (análisis en profundidad, práctica posterior) · g) modificación de apps existentes → 1.6.4 (práctica posterior) · h) emuladores → 1.5.3.

## Entregables generados

- `Tema1_Tecnologias_Moviles.docx`: apuntes de la sesión introductoria (contenido breve, nivel de entrada sin experiencia previa en el sector), portada con "Profesora: Alicia Iserte Tena", RA1 y tabla de CE, paleta de `style.scss.txt` aplicada (verde teal `#0F6E56` en títulos, violeta `#534AB7` en subtítulos, ámbar `#BA7517` para el cuadro de ejercicios). Encabezado en todas las páginas con "PMDM · Programación Multimedia y Dispositivos Móviles" / "UD1 · Tecnologías móviles" y pie de página "Página X de Y" (requisito de proyecto vigente desde esta unidad, válido para todos los .docx de contenidos que se generen en adelante).
- `Tema1_Tecnologias_Moviles.pptx`: presentación de apoyo a la misma sesión (18 diapositivas, formato 16:9), siguiendo el mismo índice y la paleta del proyecto (fondo blanco en contenido, portada y cierre en verde oscuro `#0C3D2E`, tarjetas en verde/violeta/ámbar suaves). Incluye portada con "Profesora: Alicia Iserte Tena", diapositiva de RA1+CE, mapa conceptual, glosario, una diapositiva por epígrafe (con iconos), resumen, autoevaluación y cierre con la próxima sesión. Sin encabezado/pie de página (ese requisito del proyecto aplica solo a los .docx).
