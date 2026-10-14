# 02 — Tema y tokens SCSS

Fecha: 2026-10-13 · Fase 2 del plan

## Qué se hizo

Se definió el tema visual de la aplicación a partir de los valores del diseño en Adobe XD (sección 2 de `PLAN.md`).

| Archivo | Cambio |
|---|---|
| `src/theme/_tokens.scss` | Nuevo. Fuente única de valores de diseño: colores, tipografía, espaciados, radios, tamaños |
| `src/theme/_typography.scss` | Nuevo. Mixin `text()` que aplica un escalón de la escala tipográfica |
| `src/theme/variables.scss` | Reescrito. Traduce los tokens a variables CSS de Ionic (`--ion-*`) y de la app (`--app-color-*`) |
| `src/global.scss` | Reescrito. Importa el CSS base de Ionic y fija el texto por defecto en 14/16 |
| `angular.json` | Se agregó `stylePreprocessorOptions.includePaths: ["src/theme"]` |
| `src/main.ts` | `provideIonicAngular({ mode: 'md' })` |
| `src/index.html` | `lang="es"`, título "Demo BHD", `color-scheme` solo `light` |
| `src/app/home/` | La página de ejemplo ahora es una vista previa temporal del tema |

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | Los tokens viven en `_tokens.scss`, no en `_variables.scss` | El plan proponía `_variables.scss` junto a `variables.scss`. Sass no puede distinguir dos archivos con el mismo nombre en la misma carpeta (uno parcial y otro no) y falla al importarlos. Se conserva `variables.scss` porque es el nombre que usa Ionic |
| 2 | Tokens SCSS como fuente única; las variables CSS se generan a partir de ellos | Un color se cambia en un solo lugar. Si el diseño cambia el verde, se edita `$color-primary` y se actualizan los estilos propios y los componentes de Ionic |
| 3 | Unidades en **px**, no en rem | El criterio principal de la prueba es que la interfaz coincida con el diseño, que está medido en px. Con rem, el tamaño depende de la configuración de fuente del usuario. Contrapartida: la app no escala con el tamaño de texto del sistema |
| 4 | Modo `md` forzado en todas las plataformas | Ionic cambia tamaños, fuentes y animaciones según el sistema (`ios` o `md`). El diseño es uno solo, así que se fija un modo para que el resultado sea el mismo en iOS, Android y navegador |
| 5 | Se quitó el modo oscuro de la plantilla | La plantilla importaba `dark.system.css`, que oscurece la app si el sistema está en modo oscuro. El diseño solo define tema claro |
| 6 | Se quitaron las utilidades CSS opcionales de Ionic (`padding`, `float-elements`, `text-alignment`, `text-transformation`, `flex-utils`) | Aplican espaciados propios de Ionic (16px) que no corresponden a la grilla del diseño. Menos CSS global y ninguna medida que no venga de los tokens |
| 7 | `@use` en lugar de `@import` | `@import` está obsoleto en Sass y se eliminará. `@use` además da espacio de nombres a cada archivo |
| 8 | Sombras y tintes de los colores de Ionic calculados con Sass | Ionic necesita seis variables por color. Se generan con un mixin usando la misma fórmula de Ionic (12% de negro para `shade`, 10% de blanco para `tint`) en vez de escribir valores a mano |
| 9 | `includePaths: ["src/theme"]` | Cualquier componente importa los tokens con `@use 'tokens' as *;` sin rutas relativas |
| 10 | Aún no existe `_mixins.scss` | El plan lo listaba, pero todavía no hay ningún mixin que lo necesite. Se creará cuando haga falta |
| 11 | Pila de fuentes de respaldo: Trebuchet MS → Lucida Grande → Lucida Sans Unicode → Lucida Sans → Tahoma → sans-serif | Trebuchet MS no viene en Android. La decisión definitiva para Android sigue pendiente |

## Tokens

### Colores

| Token | Valor |
|---|---|
| `$color-primary` | `#49B742` |
| `$color-secondary` | `#00477A` |
| `$color-danger` | `#C92340` |
| `$color-text` | `#666666` |
| `$color-text-muted` | `#999999` |
| `$color-placeholder` | `#CCCCCC` |
| `$color-border` | `rgba(204, 204, 204, 0.6)` |
| `$color-background` | `#F5F5F5` |
| `$color-surface` | `#FFFFFF` |
| `$color-overlay` | `rgba(0, 0, 0, 0.16)` |
| `$color-backdrop` | `rgba(0, 0, 0, 0.4)` |
| `$color-toggle-off` | `rgba(255, 255, 255, 0.6)` |

### Escala tipográfica

| Escalón | Tamaño / interlineado |
|---|---|
| `xs` | 10 / 11 |
| `sm` | 12 / 14 |
| `md` | 14 / 16 |
| `lg` | 16 / 19 |
| `xl` | 24 / 28 |

Pesos: `$font-weight-regular` (400) y `$font-weight-bold` (700).

### Espaciado, radios y tamaños

| Token | Valor | Origen en el diseño |
|---|---|---|
| `$space-xxs` / `xs` / `sm` / `md` / `lg` | 4 / 8 / 16 / 24 / 32 | Grilla base de 8 |
| `$screen-margin` | 24 | Margen lateral del artboard |
| `$content-width` | 326 | Ancho de inputs, botones y cards |
| `$radius-sm` / `md` / `pill` | 4 / 8 / 12 | Cards y toast / inputs y botones / toggle |
| `$border-width` | 1 | Bordes |
| `$accent-bar-height` | 4 | Barra verde de header, tabs y menú |
| `$control-height` | 44 | Inputs y botones |
| `$header-height` | 66 | Header (62 + barra de 4) |
| `$tab-bar-height` | 80 | Barra de pestañas |
| `$side-menu-width` | 269 | Menú lateral |
| `$shadow-knob` | `1px 1px 1.5px rgba(0,0,0,0.16)` | Perilla del toggle |
| `$transition-duration` | 0.3s | Transiciones del prototipo |

## Cómo se usa

En el SCSS de un componente:

```scss
@use 'tokens' as *;
@use 'typography' as type;

.section-title {
  @include type.text('md', $font-weight-bold);
  color: $color-primary;
  margin-bottom: $space-sm;
}
```

Si se pide un escalón que no existe (`type.text('xxl')`), la compilación falla con un mensaje que lista los escalones válidos.

Para personalizar un componente de Ionic desde su hoja de estilos:

```scss
ion-button {
  --border-radius: #{$radius-md};
}
```

## Conceptos

### Tokens de diseño

Un token es un valor de diseño con nombre (`$color-primary` en lugar de `#49B742`). El código nunca escribe el valor directo. Así, un cambio de diseño se hace en un solo archivo y es fácil comprobar que no hay medidas inventadas.

### Variables SCSS frente a variables CSS

- Una **variable SCSS** (`$color-primary`) existe solo al compilar. En el CSS final queda el valor ya resuelto.
- Una **variable CSS** (`--ion-color-primary`) existe en el navegador y se puede leer o cambiar en tiempo de ejecución.

Los componentes de Ionic usan Shadow DOM: sus estilos internos están aislados y no se pueden alcanzar con selectores normales. La forma de cambiarlos es a través de las variables CSS que cada componente expone. Por eso los tokens se traducen a `--ion-*` en `variables.scss`.

### Las seis variables de un color de Ionic

Cada color (`primary`, `secondary`, `danger`…) necesita: el color base, su versión en canales RGB, el color de contraste para el texto, el contraste en RGB, una variante más oscura (`shade`, estado presionado) y una más clara (`tint`, estado hover). El mixin `ionic-color` las genera todas.

Los valores `-rgb` existen porque Ionic aplica transparencias con `rgba(var(--ion-color-primary-rgb), 0.5)`.

### Modo `ios` y modo `md`

Ionic incluye dos juegos de estilos. Por defecto elige `ios` en dispositivos Apple y `md` (Material Design) en el resto. El modo se ve en la clase del elemento `<html>`.

### Parciales de Sass

Un archivo que empieza con guion bajo (`_tokens.scss`) es un parcial: no genera CSS por sí mismo, solo se importa desde otros. Se importa sin el guion ni la extensión.

## Verificación

Se abrió la app a 375 × 812 y se leyeron los estilos calculados por el navegador:

| Comprobación | Resultado |
|---|---|
| Clase de `<html>` | `md` |
| Texto por defecto | Trebuchet MS, 14px / 16px, `rgb(102, 102, 102)` |
| Los 12 colores `--app-color-*` | Coinciden con el diseño |
| Los 5 escalones tipográficos | 10/11, 12/14, 14/16, 16/19, 24/28 |
| `--ion-color-primary` y derivados | `#49b742`, shade `rgb(64, 161, 58)`, tint `rgb(91, 190, 85)` |
| Consola del navegador | Sin errores |
| `ng build` | Sin advertencias |
| `ng test` | 2 pruebas en verde |
| `ng lint` | Sin errores |

## Pendientes

- La página `home` es una vista previa temporal; se reemplazará cuando exista la primera pantalla real.
- Definir la fuente para Android.
- Los assets de `design-assets/` aún no se han copiado a `src/assets`.
