# 12 — Capacitor y Android

Fecha: 2026-10-13 · Fase 11 del plan

## Qué se hizo

| Archivo | Cambio |
|---|---|
| `android/` | Nuevo. Proyecto nativo de Android generado por Capacitor |
| `capacitor.config.ts` | Nombre visible "Demo BHD"; configuración de barras del sistema y pantalla de arranque |
| `src/theme/_tokens.scss` | Pila de fuentes con la fuente del sistema como respaldo |
| `src/app/core/services/native-app.service.ts` | Nuevo. Envuelve la función nativa de cerrar la app |
| `src/app/app.component.ts` | El botón atrás del teléfono cierra la app cuando no hay página anterior |
| `package.json` | Nuevas dependencias `@capacitor/android` 8.5.3 y `@capacitor/splash-screen` 8.0.2 |

Comandos ejecutados:

```bash
npm install @capacitor/android@8.5.3 @capacitor/splash-screen@^8
npx ng build
npx cap add android
npx cap sync android
```

## Estado

| Paso | Resultado |
|---|---|
| Build web de producción (`www/`) | Correcto, sin advertencias |
| `npx cap add android` | Correcto; proyecto creado en `android/` |
| `npx cap sync android` | Correcto; web copiada y 6 plugins registrados |
| Compilar el APK (`gradlew assembleDebug`) | **Correcto**: `BUILD SUCCESSFUL` en 52 s |
| Abrir la app en un emulador o dispositivo | **No realizado** (ver "Qué falta para ejecutarla") |

El proyecto nativo quedó con `minSdk` 24 (Android 7), `compileSdk` y `targetSdk` 36, Gradle 8.14.3 y Android Gradle Plugin 8.13.

### Compilación

En el primer intento no se pudo compilar: el SDK de Android estaba incompleto y solo había JDK 25, que Gradle 8.14 no admite. Tras instalar el usuario el SDK desde Android Studio y un JDK 21 (Temurin), la compilación funcionó.

Comando usado, desde `android/`, en PowerShell:

```powershell
$env:JAVA_HOME = "$HOME\.jdks\temurin-21.0.12.1"
$env:ANDROID_HOME = "$HOME\AppData\Local\Android\Sdk"
.\gradlew.bat assembleDebug
```

Las dos variables solo se fijaron para esa terminal; no se cambió la configuración del sistema. El JDK por defecto de la máquina sigue siendo el 25, así que en una terminal nueva hay que volver a indicarlas, o elegir el JDK 21 como *Gradle JDK* en Android Studio.

APK generado: `android/app/build/outputs/apk/debug/app-debug.apk`

| Dato del APK | Valor |
|---|---|
| Tamaño | 4.42 MB |
| Paquete | `com.demo.bhdapp` |
| Nombre visible | Demo BHD |
| Versión | 1.0 (código 1) |
| `targetSdk` / `compileSdk` | 36 |
| Web empaquetada | 68 archivos en `assets/public/`, incluidos `index.html` y el logo |
| Configuración de Capacitor | Incluida (`assets/capacitor.config.json`) |

### Qué falta para ejecutarla

Se intentó arrancar el emulador creado en Android Studio (`Medium_Phone_API_37.0`) y no inició:

```
x86_64 emulation currently requires hardware acceleration!
Android Emulator hypervisor driver is not installed on this machine
```

El emulador necesita aceleración por hardware y Windows no tiene ningún hipervisor activo. Activarlo es un cambio de configuración del sistema que debe hacer el usuario. Dos caminos:

- **Emulador:** activar la característica de Windows "Plataforma del hipervisor de Windows" (*Activar o desactivar las características de Windows*) y reiniciar, o instalar el *Android Emulator hypervisor driver* desde el SDK Manager de Android Studio. Requiere que la virtualización esté habilitada en la BIOS.
- **Teléfono real:** activar la depuración por USB en el teléfono, conectarlo y ejecutar desde Android Studio, o instalar el APK con `adb install`.

Lo que sigue sin verificarse por no haber podido ejecutar la app: la pantalla de arranque, el estilo de las barras del sistema, las áreas seguras, el botón atrás físico y el aspecto con la fuente del sistema.

## Fuente en Android

**Decisión del usuario:** usar la fuente por defecto del sistema cuando Trebuchet MS no esté disponible.

```scss
$font-family-base: 'Trebuchet MS', Roboto, system-ui, sans-serif;
```

| Plataforma | Fuente que se usa |
|---|---|
| Navegador en Windows y macOS | Trebuchet MS |
| iOS | Trebuchet MS (viene con el sistema) |
| Android | Roboto, la fuente por defecto del sistema |

Antes la pila incluía varias fuentes de escritorio (Lucida Grande, Lucida Sans, Tahoma) que tampoco existen en Android; se quitaron porque no aportaban nada.

Consecuencias en Android:

- No se empaqueta ningún archivo de fuente: la app no crece ni hay que gestionar licencias.
- **El diseño no será pixel perfect en Android.** Roboto tiene otras proporciones que Trebuchet MS, así que los anchos de texto cambian (por ejemplo, el monto que mide 154 px en el diseño). Posiciones, tamaños de caja, colores e iconos no cambian.
- La alternativa para igualar el diseño en Android sería empaquetar una fuente web con licencia que lo permita. Trebuchet MS es propiedad de Microsoft y no se puede redistribuir dentro de una app.

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | Fuente del sistema como respaldo en Android | Decisión del usuario. Sin archivos de fuente ni licencias |
| 2 | Solo plataforma Android | iOS requiere macOS y Xcode; este equipo es Windows. El código es el mismo: bastaría `npx cap add ios` en un Mac |
| 3 | Barras del sistema con el plugin `SystemBars` del núcleo de Capacitor 8, por configuración | Capacitor 8 dibuja la app de borde a borde y expone las áreas seguras como variables CSS (`insetsHandling: 'css'`). Ionic ya usa esas variables en header, pestañas y contenido. No hace falta código |
| 4 | Estilo `LIGHT` de las barras | Iconos oscuros sobre fondo claro: el header es blanco y el fondo de la app gris muy claro |
| 5 | `initialViewportFitValueHint: 'cover'` | `index.html` declara `viewport-fit=cover`. Indicarlo de antemano evita un salto de diseño al arrancar |
| 6 | Pantalla de arranque blanca de 1 s, por configuración | Sin lógica propia: la app no tiene nada que cargar antes de mostrar el login |
| 7 | Botón atrás físico: cierra la app si no hay página anterior | Ionic ya retrocede de página, cierra el menú y los overlays con ese botón. Lo que no hace es salir de la app en una pantalla raíz (login o pestañas); sin esto el botón no haría nada |
| 8 | `NativeAppService` envuelve la llamada al plugin | Las pruebas lo sustituyen por un doble. Es la forma recomendada de usar plugins de Capacitor en Angular |
| 9 | Nombre visible "Demo BHD"; identificador `com.demo.bhdapp` | El identificador sigue siendo provisional; no se usó un dominio del banco |
| 10 | La carpeta `android/` se versiona | Es código fuente del proyecto nativo. Sus archivos generados (builds, copia de la web) ya están excluidos por su propio `.gitignore` |

## Áreas seguras

Lo que protege el contenido de la muesca, la barra de estado y la barra de gestos:

| Elemento | Cómo se resuelve |
|---|---|
| Header | `ion-toolbar` añade el margen superior por sí mismo |
| Barra de pestañas | `ion-tab-bar` añade el margen inferior por sí mismo |
| Login (hamburguesa y logo) | Posición sumada a `--ion-safe-area-top` |
| Onboarding (título, puntos, pie) | Posición sumada a `--ion-safe-area-top` y `--ion-safe-area-bottom` |
| Menú lateral | Relleno superior e inferior con las mismas variables |
| Toast | Ionic lo coloca bajo el área segura |

## Flujo de trabajo con Capacitor

```bash
npx ng build
```

```bash
npx cap sync android
```

```bash
npx cap open android
```

- `ng build` genera la web en `www/`.
- `cap sync` copia `www/` dentro del proyecto nativo y actualiza los plugins. Hay que repetirlo cada vez que cambia la web o se instala un plugin.
- `cap open` abre el proyecto en Android Studio.

Para desarrollo diario se sigue usando el navegador (`npm start`); el build nativo solo hace falta para probar funciones del dispositivo.

## Plugins instalados

| Plugin | Uso en la app |
|---|---|
| `@capacitor/preferences` | Usuario recordado y marca de onboarding visto |
| `@capacitor/app` | Botón atrás físico y cierre de la app |
| `@capacitor/splash-screen` | Pantalla de arranque |
| `@capacitor/keyboard` | Lo usa Ionic para ajustar la pantalla cuando aparece el teclado |
| `@capacitor/status-bar`, `@capacitor/haptics` | Vienen con la plantilla de Ionic; la app no los usa directamente |
| `SystemBars` (núcleo) | Estilo de las barras y áreas seguras |

## Conceptos

### Qué hace Capacitor

Crea un proyecto nativo real (Android Studio o Xcode) cuya única pantalla es un WebView que carga la web empaquetada. Los plugins son puentes: una función de JavaScript que llama a código nativo y devuelve una promesa.

### `webDir`

Carpeta con la web compilada que Capacitor copia al proyecto nativo. Debe coincidir con la salida de `ng build`, que en este proyecto es `www`.

### Detección de plataforma

`Capacitor.isNativePlatform()` indica si la app corre dentro del contenedor nativo. Los plugins usados aquí tienen implementación web (Preferences usa `localStorage`), así que el mismo código funciona en el navegador sin condicionales.

### Botón atrás en Android y prioridades

Ionic reparte el evento del botón atrás por prioridad: overlays (100), menú (99), navegación (0). Un manejador con prioridad −1 solo se ejecuta cuando ninguno de los anteriores ha consumido el evento, es decir, cuando no hay nada que cerrar ni página a la que volver.

```ts
platform.backButton.subscribeWithPriority(-1, () => {
  if (!this.outlet().canGoBack()) {
    void this.nativeApp.exit();
  }
});
```

### Borde a borde (edge-to-edge)

Desde Android 15 las apps se dibujan detrás de las barras del sistema. La app debe dejar libre ese espacio por su cuenta, usando los valores de área segura.

## Pruebas

165 pruebas en 26 archivos, todas en verde (3 nuevas).

| Archivo | Qué comprueba |
|---|---|
| `app.component.spec.ts` | El botón atrás cierra la app en una página raíz; no la cierra si hay página anterior; deja de escuchar al destruirse el componente |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 165 / 165 |
| `ng lint` | Sin errores |
| `npx cap sync android` | Correcto |
| `gradlew assembleDebug` | BUILD SUCCESSFUL |

## Pendientes

- Ejecutar la app en un emulador o en un teléfono (el APK ya compila; falta aceleración por hardware para el emulador).
- Icono y pantalla de arranque propios: hoy son los de Capacitor por defecto. Se generan a partir de una imagen con `@capacitor/assets`.
- Definir el identificador definitivo de la app.
