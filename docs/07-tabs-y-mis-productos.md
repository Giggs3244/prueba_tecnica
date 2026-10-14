# 07 — Pestañas y "Mis productos"

Fecha: 2026-10-13 · Fase 6 del plan

## Qué se hizo

La zona privada de la app: barra de cuatro pestañas y la pantalla "Mis productos" (el home tras iniciar sesión), con los datos de la API y carga progresiva.

| Archivo | Cambio |
|---|---|
| `src/app/features/tabs/tabs.page.{ts,html,scss}` | Nueva. Contenedor con `ion-tabs` y la barra inferior |
| `src/app/features/products/products.page.{ts,html,scss}` | Nueva. "Mis productos" |
| `src/app/features/placeholder/placeholder.page.{ts,html,scss}` | Nueva. Una sola página para Transacciones, Ofertas y Configuración, parametrizada por la ruta |
| `src/app/shared/config/main-sections.ts` | Nuevo. Las cuatro secciones (ruta, etiqueta, icono) |
| `src/app/app.routes.ts` | Ruta `tabs` con cuatro rutas hijas |
| `src/app/home/` | Eliminada. Era la galería temporal de componentes |

## Rutas

| Ruta | Guard | Página |
|---|---|---|
| `/login` | `guestGuard` | `LoginPage` |
| `/tabs` | `authGuard` | `TabsPage`; redirige a `/tabs/products` |
| `/tabs/products` | (hereda) | `ProductsPage` |
| `/tabs/transactions` | (hereda) | `PlaceholderPage` con título "Transacciones" |
| `/tabs/offers` | (hereda) | `PlaceholderPage` con título "Ofertas" |
| `/tabs/configuration` | (hereda) | `PlaceholderPage` con título "Configuración" |

El guard está en la ruta padre, así que protege a todas las hijas.

## Carga progresiva en "Mis productos"

Comportamiento acordado con el usuario (opción A de la sección 3.4 del plan):

1. Al entrar se muestra **un solo skeleton** con la forma de una card.
2. Cada sección aparece, con su título y sus cards, cuando llega su respuesta.
3. Cuentas va siempre primero. Si las tarjetas llegan antes, el skeleton se queda arriba, en el lugar de Cuentas, y las tarjetas se pintan debajo.
4. El skeleton desaparece cuando ya no queda ningún servicio por responder.
5. Una sección sin productos, o cuyo servicio falló, no se pinta.

Secuencia observada en el navegador en una ejecución (los tiempos dependen del retardo aleatorio del mock):

| Momento | En pantalla |
|---|---|
| 0 ms | Skeleton en y = 123 |
| 900 ms | Skeleton en y = 123 · `TARJETAS (2)` en y = 284 |
| 1800 ms | `CUENTAS (2)` en y = 91 · `TARJETAS (2)` en y = 428 |

El skeleton está colocado donde irá la primera card (deja libre el espacio del título), así que al llegar las cuentas la primera card lo sustituye en la misma posición.

### Cómo está escrito

```ts
protected readonly groups = toSignal(inject(ProductsService).loadGroups(), { initialValue: [] });

protected readonly firstLoadingType = computed(
  () => this.groups().find((group) => group.status === 'loading')?.type ?? null,
);
```

```html
@for (group of groups(); track group.type) {
  @if (group.status === 'loaded' && group.products.length > 0) {
    <section>…título y cards…</section>
  } @else if (group.type === firstLoadingType()) {
    <app-product-card-skeleton />
  }
}
```

La página recorre los grupos en el orden que entrega el servicio. No conoce los tipos de producto ni los endpoints: con un tercer tipo de producto esta plantilla no cambia.

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | `ion-tabs` con rutas hijas | Es el patrón de Ionic: cada pestaña conserva su propia pila de navegación y su estado al cambiar de pestaña |
| 2 | Una sola `PlaceholderPage` para las tres pestañas de relleno | Las tres pantallas del diseño solo difieren en dos textos. Los textos llegan desde `data` de la ruta y se enlazan a entradas del componente (`withComponentInputBinding`). En la fase 8 servirá también para Contacto y Sucursales |
| 3 | Las secciones se definen una vez en `MAIN_SECTIONS` | La barra de pestañas y el menú lateral (fase 7) muestran las mismas cuatro opciones |
| 4 | Iconos de pestaña con máscara CSS y `currentColor` | Un solo SVG por icono; el color lo pone Ionic según la pestaña esté activa o no. Se dejan de necesitar las variantes verde y gris de cada archivo |
| 5 | Pestañas de ancho igual | Los iconos quedan a 1.1 px o menos de la posición del XD, donde no están repartidos de forma exacta. Se prefirió un reparto uniforme, que además se adapta a otros anchos de pantalla |
| 6 | Los productos se cargan una vez al crear la página | Ionic mantiene viva la página al cambiar de pestaña; no se repiten las peticiones. Al cerrar sesión la página se destruye y la siguiente sesión carga de nuevo |
| 7 | Guard solo en la ruta padre `tabs` | Evita repetirlo en cada hija |
| 8 | 53 px de espacio bajo la última card | En el XD el contenido mide 893 de alto; al hacer scroll hasta el final quedan 53 px entre la última card y la barra |
| 9 | Valor por defecto de `leading` con `transform` | Con `withComponentInputBinding`, el router asigna `undefined` a las entradas que no vienen en la ruta, y eso pisaba el valor por defecto `'menu'` |

## Verificación contra el diseño

Medido en el navegador a 375 × 812.

| Elemento | Diseño (XD) | Medido |
|---|---|---|
| Header | 375 × 66 | 375 × 66 |
| Título `CUENTAS (2)` | (24, 91), verde, Bold 14 | (24, 91), `rgb(73, 183, 66)` |
| Cards de cuenta | (24, 123) y (24, 267), 326 × 128 | (24, 123) y (24, 267), 326 × 128 |
| Título `TARJETAS (2)` | (24, 428), azul | (24, 428), `rgb(0, 71, 122)` |
| Card de tarjeta 1 | (24, 460) 326 × 156 | (24, 460) 326 × 156 |
| Card de tarjeta 2 | (24, 638) 326 × 122 | (24, 632) 326 × 128 |
| Barra de pestañas | (0, 732) 375 × 80, borde superior verde de 4 | (0, 732) 375 × 80, `4px solid rgb(73, 183, 66)` |
| Iconos de pestaña | y = 747, 30 × 30; x = 33, 126, 220, 314 | y = 747, 30 × 30; x = 31.9, 125.6, 219.4, 313.1 |
| Etiquetas de pestaña | y = 783, 10/11 | y = 783, 10/11 |
| Hamburguesa en pestañas de relleno | (24, 23) | (24, 23) |
| Título de relleno | y = 261, alto 56 | y = 261, alto 56 |
| Párrafo de relleno | y = 347, ancho 294 | y = 347, ancho 294 |

Diferencias aceptadas:

- **Card de tarjeta 2**: 6 px más arriba y 6 px más alta. Son las inconsistencias 2 y 4 del plan (separación de 16 en lugar de 22 y icono en la misma posición que las demás cards).
- **Iconos de pestaña**: hasta 1.1 px a la izquierda (decisión 5).
- **Montos**: los de la API, no los del dibujo (inconsistencia 7).

Comprobado en el navegador: inicio de sesión, aparición progresiva de las secciones, orden fijo con las tarjetas llegando primero, navegación entre las cuatro pestañas y textos de cada una.

**No verificado en el navegador:** el color verde de la pestaña activa y las animaciones. El panel de vista previa seguía oculto y no dibujaba fotogramas; Ionic marca la pestaña activa al terminar la transición, que ahí no llega a completarse. Conviene revisarlo a ojo.

## Pendiente respecto al diseño

- El prototipo cambia de pestaña con un fundido de 0.3 s. Ionic cambia de pestaña sin animación. Queda para la fase 10.
- Las cards todavía no navegan al detalle (fase 8).
- La hamburguesa todavía no abre el menú (fase 7).

## Conceptos

### `ion-tabs`

`ion-tabs` contiene un `ion-router-outlet` propio y un `ion-tab-bar`. Cada `ion-tab-button` lleva en `tab` el nombre de la ruta hija que abre. Ionic guarda una pila de navegación por pestaña: si dentro de una pestaña se entra a un detalle y luego se cambia de pestaña, al volver se sigue en el detalle.

### Rutas hijas

Una ruta con `children` pinta su componente (el contenedor de pestañas) y, dentro de su outlet, el componente de la hija activa. Los guards y resolvers del padre se aplican antes que los de las hijas.

### Datos de ruta como entradas

```ts
{ path: 'offers', loadComponent: …, data: { title: 'Ofertas', heading: 'Pantalla\nde ofertas' } }
```

Con `withComponentInputBinding()` en la configuración del router, cada clave de `data` (y cada parámetro de la URL) se asigna a la entrada del mismo nombre en el componente. El componente no necesita inyectar `ActivatedRoute`.

### `toSignal` con un observable que emite varias veces

`loadGroups()` emite la lista de grupos cada vez que responde un servicio. `toSignal` guarda siempre el último valor y la plantilla se repinta sola. La suscripción se cancela cuando la página se destruye.

### `computed`

`firstLoadingType` se recalcula solo cuando cambia `groups`. La plantilla lo lee como cualquier otro signal.

### Máscaras CSS

```scss
.tab-icon {
  background: currentColor;
  mask: var(--tab-icon) center / contain no-repeat;
}
```

El SVG actúa como plantilla recortable y el color viene del texto. Así un mismo archivo sirve para el estado activo, el inactivo y cualquier otro color.

## Pruebas

122 pruebas en 21 archivos, todas en verde (15 netas nuevas; se retiró la prueba de la galería).

| Archivo | Qué comprueba |
|---|---|
| `products.page.spec.ts` | Título; un solo skeleton al inicio; cuentas cargadas con tarjetas pendientes; tarjetas primero con el skeleton en el lugar de cuentas; todo cargado sin skeleton y en orden; color del título por tipo; grupo vacío no se pinta; grupo con error no se pinta ni deja skeleton; usuario sin productos |
| `tabs-navigation.spec.ts` | Sin sesión redirige al login; `/tabs` abre productos; cuatro botones en el orden del diseño; cada pestaña de relleno con su título, su texto y el botón de menú; valor por defecto del botón cuando la ruta no lo indica |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 122 / 122 |
| `ng lint` | Sin errores |
