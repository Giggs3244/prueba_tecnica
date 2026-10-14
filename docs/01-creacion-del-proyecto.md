# 01 — Creación del proyecto

Fecha: 2026-10-13

## Qué se hizo

Se creó el proyecto `demo-bhd-app` dentro de `prueba_tecnica/` con la CLI de Ionic, que además inicializó el repositorio git (rama `main`) con un commit inicial.

Comando ejecutado:

```bash
npx @ionic/cli@7.2.1 start demo-bhd-app blank --type=angular-standalone --capacitor --package-id=com.demo.bhdapp --no-interactive
```

## Versiones resultantes

| Paquete | Versión |
|---|---|
| Angular | 22.1.7 |
| Angular CLI / `@angular/build` | 22.1.8 |
| Ionic (`@ionic/angular`) | 9.0.7 |
| Capacitor (`@capacitor/core` y CLI) | 8.5.3 |
| Ionicons | 8.x |
| TypeScript | 6.0 |
| Vitest (pruebas unitarias) | 4.0 |
| ESLint + angular-eslint | 10 / 22.5 |
| Node.js usado | 24.21.0 (gestionado con fnm) |

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | **Ionic 9** en lugar de Ionic 8 | Es la versión estable publicada (`latest` = 9.0.7) y declara soporte para Angular 18 o superior, por lo que es compatible con Angular 22. La 8 ya no es la que instala la CLI |
| 2 | Tipo `angular-standalone` | Angular 22 trabaja con componentes standalone por defecto; evita NgModules y permite cargar por ruta solo lo que cada pantalla usa |
| 3 | Plantilla `blank` | El diseño combina tabs, menú lateral, login y onboarding. Ninguna plantilla (`tabs`, `sidemenu`) coincide con esa mezcla, así que partir de cero evita borrar código generado |
| 4 | Capacitor integrado desde el inicio (`--capacitor`) | La prueba lista Capacitor como tecnología a evaluar |
| 5 | Identificador `com.demo.bhdapp` | Valor provisional en notación DNS inversa. No se usó un dominio del banco porque no nos pertenece. Se cambia en `capacitor.config.ts` antes de agregar plataformas nativas |
| 6 | CLI de Ionic vía `npx`, sin instalación global | No modifica el entorno de la máquina y fija la versión usada |
| 7 | Repositorio git propio dentro de `demo-bhd-app/` | Solo el código de la aplicación queda versionado; el enunciado, el plan, los assets de diseño y esta documentación (`docs/`) quedan fuera, en `prueba_tecnica/` |

## Conceptos

### Ionic, Angular y Capacitor: quién hace qué

- **Angular** es el framework de la aplicación: componentes, rutas, formularios, inyección de dependencias, HTTP.
- **Ionic** es una librería de componentes de interfaz pensados para móvil (`ion-header`, `ion-content`, `ion-tabs`, `ion-menu`, `ion-toast`…) más un sistema de navegación con transiciones nativas. No reemplaza a Angular; se monta encima.
- **Capacitor** empaqueta la aplicación web dentro de un proyecto nativo de Android o iOS y expone APIs del dispositivo (almacenamiento, barra de estado, teclado) como funciones de JavaScript.

El resultado de `ng build` es una aplicación web normal en la carpeta `www/`. Capacitor copia esa carpeta dentro del proyecto nativo.

### Standalone frente a NgModules

Un componente standalone declara en su propio `imports` lo que necesita. No existe `AppModule`: la aplicación arranca en `src/main.ts` con `bootstrapApplication` y una lista de proveedores.

### Archivos principales generados

| Archivo | Para qué sirve |
|---|---|
| `src/main.ts` | Punto de arranque: registra Ionic, el router y la estrategia de reutilización de rutas de Ionic |
| `src/app/app.component.*` | Componente raíz con `ion-app` e `ion-router-outlet` |
| `src/app/app.routes.ts` | Tabla de rutas |
| `src/app/home/` | Página de ejemplo de la plantilla; se reemplazará |
| `src/theme/variables.scss` | Variables de color de Ionic |
| `src/global.scss` | Estilos globales e importación del CSS de Ionic |
| `capacitor.config.ts` | Identificador, nombre de la app y carpeta web (`www`) |
| `ionic.config.json` | Tipo de proyecto para la CLI de Ionic |
| `angular.json` | Configuración de build, servidor de desarrollo, pruebas y lint |

### Pruebas unitarias

El proyecto usa **Vitest** con jsdom a través del builder `@angular/build:unit-test`, que es lo que genera Angular 22. No usa Karma ni Jasmine.

## Verificación

| Comando | Resultado |
|---|---|
| `npx ng build` | Compila, salida en `www/` |
| `npx ng test --watch=false` | 2 archivos, 2 pruebas en verde |
| `npx ng lint` | Sin errores |

## Comandos de uso diario

Node se gestiona con fnm, así que en una terminal nueva puede hacer falta activarlo primero:

```powershell
fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression
```

```bash
npm start
```

```bash
npm test
```

```bash
npm run build
```

```bash
npm run lint
```

## Pendientes y riesgos

- La carpeta está dentro de **OneDrive**. `node_modules` tiene decenas de miles de archivos y la sincronización puede volver lentos la instalación y el build o bloquear archivos. Conviene excluir `node_modules` y `www` de la sincronización o mover el proyecto fuera de OneDrive.
- `appName` en `capacitor.config.ts` es `demo-bhd-app`; es el nombre visible bajo el icono en el dispositivo. Definir el nombre final antes de agregar Android.
- `package.json` trae `author`, `homepage` y `description` de la plantilla de Ionic; hay que ajustarlos.
- `src/test-setup.ts` trae comentarios generados por la plantilla.
