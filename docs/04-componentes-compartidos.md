# 04 — Componentes compartidos

Fecha: 2026-10-13 · Fase 4 del plan

## Qué se hizo

Se creó `src/app/shared/` con los componentes de interfaz reutilizables. No tienen lógica de negocio: reciben datos por entradas y avisan por salidas.

```
src/app/shared/
  components/
    button/                  app-button
    input/                   app-input
    toggle/                  app-toggle
    header/                  app-header
    product-card/            app-product-card  (+ _product-card-layout.scss)
    product-card-skeleton/   app-product-card-skeleton
    loading-overlay/         app-loading-overlay
    placeholder-content/     app-placeholder-content
  config/
    product-presentation.ts  título, icono y color por tipo de producto
src/assets/
  icons/           20 SVG
  images/          logo (1x, 2x, 3x) y fondo del login
  illustrations/   3 SVG del onboarding
```

La página `home` es ahora una galería temporal con todos los componentes, usada para medirlos contra el diseño.

## Componentes

| Componente | Entradas | Salidas / enlace | Base de Ionic |
|---|---|---|---|
| `app-button` | `variant` (`solid`, `outline`), `type`, `disabled` | Clic nativo | `ion-button` |
| `app-input` | `placeholder`, `type`, `inputMode`, `autocomplete`, `icon`, `disabled` | `value` (dos vías), `touch` | `ion-input` |
| `app-toggle` | `disabled`; la etiqueta va como contenido | `checked` (dos vías), `touch` | `ion-toggle` |
| `app-header` | `title`, `leading` (`menu`, `back`, `none`), `backHref` | — | `ion-header`, `ion-toolbar`, `ion-title`, `ion-menu-button`, `ion-back-button` |
| `app-product-card` | `product` (cualquier `Product`) | — | — |
| `app-product-card-skeleton` | — | — | `ion-skeleton-text` |
| `app-loading-overlay` | — | — | — |
| `app-placeholder-content` | `title`, `description` | — | — |

Ejemplo de uso con formulario:

```html
<app-input placeholder="Contraseña" type="password" icon="assets/icons/key-input-gray.svg" [formField]="loginForm.password" />
<app-toggle [formField]="loginForm.rememberUser">Recuerda mi usuario</app-toggle>
<app-button type="submit">INICIAR SESIÓN</app-button>
```

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | Componentes propios que **envuelven** componentes de Ionic | Se aprovecha el comportamiento de Ionic (efecto de pulsación, accesibilidad, teclado, gestos del toggle, integración del botón de menú y de retroceso con la navegación) y el estilo exacto del diseño queda escrito una sola vez. Las pantallas no repiten variables CSS |
| 2 | Estilo de Ionic mediante variables CSS y `::part()` | Es la vía que Ionic ofrece para personalizar el interior de sus componentes, aislado por Shadow DOM |
| 3 | `app-input` y `app-toggle` cumplen los contratos de **Signal Forms** (`FormValueControl`, `FormCheckboxControl`) | Es el sistema de formularios recomendado desde Angular 22. Con `[formField]` el formulario enlaza valor, estado deshabilitado y "tocado" sin código adicional. También funcionan sueltos con `[(value)]` y `[(checked)]` |
| 4 | Card de producto y skeleton comparten medidas en `_product-card-layout.scss` | El skeleton debe ocupar exactamente lo mismo que la card para que el contenido no salte al llegar los datos |
| 5 | `app-product-card` recibe un `Product`, no una cuenta o una tarjeta | Pinta `displayName` y `balances` sin preguntar el tipo. El icono, el color y el título del grupo salen de `PRODUCT_PRESENTATION`, indexado por tipo. Un tipo nuevo no cambia la plantilla |
| 6 | Sin pipes propios (cambio respecto al plan) | Los últimos dígitos ya los calcula el modelo. El monto se formatea en la card porque hay que partirlo en dos tamaños de letra, algo que un pipe de texto no resuelve |
| 7 | Monto en dos tamaños: moneda y decimales a 14, parte entera a 24 | Corrección al leer los rangos de texto del XD. Se comprobó midiendo: `RD$ 1,000,000.00` ocupa 153.9 px (154 en el diseño) |
| 8 | Tarjeta de crédito con un solo balance: 128 de alto, no 122 | El XD la dibuja de 122 con el icono desplazado. Al unificar la posición del icono (inconsistencia 4 del plan) queda igual que la cuenta |
| 9 | Indicador de carga propio, sin `LoadingController` de Ionic | El diseño usa una caja de 103 × 109 con un spinner de 8 puntos que Ionic no trae. Un componente declarativo (`@if (loading())`) es más simple de controlar con signals y de probar |
| 10 | Iconos como `<img>` con SVG | Colores fijos por archivo, tamaño exacto del diseño y sin fuente de iconos. `ion-icon` solo se usa donde Ionic lo exige (flecha de retroceso) |
| 11 | Área táctil de 48 × 48 en menú y retroceso | El icono mide 25 × 19 en el diseño, demasiado pequeño para tocar. El botón es más grande que el dibujo y el dibujo queda en la posición exacta |
| 12 | Título del header centrado respecto a la pantalla | En modo `md` Ionic lo alinea a la izquierda. Se posiciona de forma absoluta para centrarlo con o sin botón |
| 13 | Skeleton con franja gris | Mientras carga no se sabe el tipo de producto, así que no se usa ni el verde ni el azul |
| 14 | Animación del spinner desactivada con `prefers-reduced-motion` | Respeta la preferencia de accesibilidad del sistema |

## Verificación contra el diseño

Medido en el navegador a 375 × 812 con `getBoundingClientRect`.

| Elemento | Diseño (XD) | Medido |
|---|---|---|
| Header | 375 × 66, barra verde de 4 | 375 × 66, borde inferior 4px `#49B742` |
| Hamburguesa | (24, 23) 25 × 19 | (24, 23) 25 × 19 |
| Título del header | y = 23, 16/19, centrado | y = 23, 16/19, centrado |
| Flecha atrás | (26.4, 23.4) 19.2 × 19.2 | (26.41, 22.91) 19.16 × 19.16 |
| Input | 326 × 44, radio 8 | 326 × 44, radio 8 |
| Icono del input | x = 16 desde el borde | x = 15.95 |
| Texto del input | x = 47 desde el borde, 14px | x = 47, 14px |
| Placeholder | `#CCCCCC` | `rgb(204, 204, 204)` |
| Toggle apagado | pista 40 × 24, perilla 22 a 1px | pista 40 × 24, perilla 22 a 1px |
| Toggle encendido | pista `#49B742`, perilla a 17 del borde | pista `rgb(73, 183, 66)`, perilla a 17 |
| Etiqueta del toggle | a 8 de la pista, 12/14 | a 8, 12/14 |
| Botón principal | 326 × 44, radio 8, Bold 14 | 326 × 44, radio 8, 700 14/16, sin sombra |
| Botón outline | borde 1px verde | `1px solid rgb(73, 183, 66)` |
| Card de cuenta | 326 × 128 | 326 × 128 |
| Card con dos balances | 326 × 156 | 326 × 156 |
| Icono de la card | (16, 16) 40 × 40 | (16, 16) 40 × 40 |
| Alias | x = 72 desde el borde | x = 72 |
| Monto | y = 72, alto 28 | y = 72, alto 28 |
| "Balance disponible" | y = 101, 10/11 | y = 101, alto 11 |
| Ancho de `RD$ 1,000,000.00` | 154 | 153.9 |
| Ancho de `US$ 1,000.00` | 106 | 106.4 |
| Skeleton | 326 × 128 | 326 × 128 |
| Caja de carga | (136, 352) 103 × 109, radio 4 | (136.1, 351.5) 103 × 109, radio 4 |
| Spinner | (174, 391) 28 × 28 | (173.6, 392) 28 × 28 |
| Fondo de carga | `rgba(0,0,0,0.4)` | `rgba(0, 0, 0, 0.4)` |
| Placeholder: título → párrafo | 30 de separación, párrafo de 294 | 30, 294 |

Diferencias aceptadas: el alias de la card queda 1 px más arriba que en el XD (se centra con el icono) y la flecha 0.5 px más arriba. Son efecto de cómo XD mide las cajas de texto.

También se probó la interacción: escribir en el input actualiza el formulario, pulsar el toggle cambia el valor y el indicador de carga aparece y desaparece.

## Conceptos

### Shadow DOM, variables CSS y `::part`

Los componentes de Ionic encapsulan su HTML interno. Un selector normal (`ion-button button { … }`) no lo alcanza. Hay dos puertas:

- **Variables CSS** que el componente declara (`--border-radius`, `--background`, `--handle-width`). Se asignan desde fuera sobre el elemento.
- **`::part(nombre)`**, para las piezas internas que el componente expone (`ion-toggle::part(track)`).

### Entradas, salidas y `model()`

- `input()` declara un dato que el padre entrega. `input.required()` obliga a entregarlo.
- `output()` declara un evento hacia el padre.
- `model()` es un dato de dos vías: el padre lo entrega y el componente puede cambiarlo. Permite `[(value)]="algo"`.

### Signal Forms y controles propios

`form(signal({...}))` crea un formulario a partir de un signal con los datos. La directiva `[formField]` conecta un campo con un control. Para que un componente propio sirva como control basta con que tenga un `model()` llamado `value` (o `checked` si es de tipo sí/no). Si además declara `disabled` y `touch`, el formulario los sincroniza.

### Proyección de contenido

`<ng-content />` coloca dentro del componente lo que el padre escribe entre sus etiquetas. Así el texto del botón y la etiqueta del toggle se escriben donde se usan.

### `ChangeDetectionStrategy.OnPush`

El componente solo se vuelve a pintar cuando cambia una entrada o un signal que lee su plantilla. Todos los componentes compartidos lo usan.

### Skeleton

Marcador con la forma del contenido final que se muestra mientras llegan los datos. Reduce la sensación de espera y evita saltos de la interfaz si ocupa el mismo espacio que el contenido real.

## Pruebas

95 pruebas en 19 archivos, todas en verde (36 nuevas en esta fase).

| Archivo | Qué comprueba |
|---|---|
| `button.component.spec.ts` | Texto proyectado, variante, tipo y estado deshabilitado |
| `input.component.spec.ts` | Placeholder y nombre accesible, icono opcional, tipo, valor en ambas direcciones, campo vaciado, evento de "tocado", deshabilitado |
| `toggle.component.spec.ts` | Etiqueta, valor en ambas direcciones, evento de "tocado" |
| `header.component.spec.ts` | Título; botón de menú, de retroceso (con ruta de respaldo) o ninguno |
| `product-card.component.spec.ts` | Cuenta, tarjeta internacional y local; formato del monto; color e icono por tipo |
| `static-components.spec.ts` | Atributos de accesibilidad del skeleton y del indicador de carga; textos del placeholder |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 95 / 95 |
| `ng lint` | Sin errores |

## Pendientes

- La galería de `home` es temporal; desaparece cuando existan las pantallas reales.
- `app-side-menu` y el toast de error se construyen con sus pantallas (fases 7 y 5).
- Los iconos de tabs y menú vienen en archivos separados por color; se evaluará unificarlos con `currentColor` en la fase 6.
- La card todavía no es pulsable; la navegación al detalle se agrega en la fase 8.
