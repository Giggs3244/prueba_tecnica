# 09 — Detalle de producto y paso de datos entre pantallas

Fecha: 2026-10-13 · Fase 8 del plan

## Qué se hizo

Al tocar una card en "Mis productos" se abre el detalle de ese producto. La pantalla del diseño (`product_detail`) existe para validar dos cosas: la navegación y el paso de datos entre pantallas.

| Archivo | Cambio |
|---|---|
| `src/app/features/product-detail/product-detail.page.{ts,html,scss}` | Nueva página de detalle |
| `src/app/core/services/products.store.ts` | Nuevo. Guarda en memoria los productos cargados |
| `src/app/core/guards/product-exists.guard.ts` | Nuevo. Impide abrir el detalle de un producto que no está cargado |
| `src/app/core/config/app-routes.ts` | Nuevos `productDetail` y `productDetailLink()` |
| `src/app/features/products/products.page.{ts,html,scss}` | Lee los productos del store; cada card es un enlace al detalle |
| `src/app/app.routes.ts` | Nueva ruta `product/:type/:digits` |

Contacto y Sucursales ya se habían creado en la fase 7 (el menú las necesitaba); aquí solo se compararon contra el diseño.

## Cómo viajan los datos

```
ProductsPage ──(enlace /product/TC/0001)──▶ router ──(type, digits)──▶ ProductDetailPage
      │                                                                      │
      └── store.load() ──▶ ProductsStore ◀── store.find(type, digits) ───────┘
```

1. `ProductsPage` pide al store que cargue los productos y los pinta según llegan.
2. Cada card es un enlace: `/product/{tipo}/{últimos 4 dígitos}`.
3. El router entrega `type` y `digits` a `ProductDetailPage` como entradas del componente.
4. El detalle busca el producto en el store y lo pinta con la misma `app-product-card`.

La URL lleva solo el **identificador**; el **objeto** se obtiene del store. El detalle recibe la misma instancia de `Account` o `CreditCard` que se mostró en la lista, con sus métodos.

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | Identificador en la URL y datos en un store | Es la forma estándar en Angular: la URL describe qué se está viendo y sirve para el botón atrás y el historial. Alternativas descartadas en la sección "Otras formas de pasar datos" |
| 2 | La URL usa los **últimos 4 dígitos**, no el número completo | Un número de cuenta o de tarjeta no debe quedar en una URL (historial, registros, capturas). Los últimos dígitos ya son lo único que se muestra en pantalla |
| 3 | `ProductsStore` separado de `ProductsService` | El servicio habla con la API y no guarda nada. El store guarda el estado que comparten la lista y el detalle. Cada uno tiene una sola responsabilidad |
| 4 | El store se vacía al cerrar la sesión | Un `effect` observa `isAuthenticated`. Así los productos de un usuario nunca quedan en memoria para el siguiente |
| 5 | `productExistsGuard` | Si se intenta abrir un detalle que no está en memoria (enlace directo, producto inexistente), redirige a "Mis productos" en lugar de mostrar una pantalla vacía |
| 6 | El detalle está **fuera** de las pestañas | En el artboard `product_detail` no aparece la barra de pestañas. La ruta es de primer nivel, con `authGuard` |
| 7 | El detalle no vuelve a llamar a la API | El producto ya está cargado; no hay endpoint de detalle en el enunciado |
| 8 | Las cuatro cards navegan al detalle | El prototipo solo enlaza la primera cuenta, pero la pantalla de detalle es genérica |
| 9 | La card se envuelve en un enlace desde la página | `app-product-card` sigue siendo solo de presentación y se reutiliza tal cual en el detalle, donde no es pulsable |
| 10 | `productDetailLink()` junto a las rutas | La forma de la URL se define en un solo lugar |

### Limitación conocida

Dos productos del mismo tipo con los mismos últimos cuatro dígitos serían indistinguibles. La API del enunciado no entrega un identificador propio; con una API real el enlace usaría un id opaco (por ejemplo un UUID) en lugar de derivarlo del número.

## Otras formas de pasar datos (para la entrevista)

| Forma | Cómo | Cuándo sirve | Por qué no se usó |
|---|---|---|---|
| Parámetro de ruta + store o servicio | `/product/TC/0001` y búsqueda en memoria | Caso general | **Es la usada** |
| Parámetro de ruta + nueva petición | El detalle pide `GET /products/:id` | Cuando el detalle tiene más datos que la lista | No hay endpoint de detalle |
| Resolver | `resolve: { product: productResolver }` carga el dato antes de activar la ruta | Cuando la pantalla no debe pintarse sin el dato | El guard ya cubre el caso de "no existe" y el dato está en memoria |
| Estado de navegación | `router.navigate([...], { state: { product } })` | Datos efímeros que no deben ir en la URL | El navegador serializa el estado: se pierde la clase (`Account` pasa a ser un objeto plano sin `displayName` ni `balances`) y el dato desaparece al recargar |
| Parámetros de consulta | `?type=TC&digits=0001` | Filtros y opciones | No identifican un recurso; un segmento de ruta es más claro |
| Servicio con "producto seleccionado" | `selection.set(product)` antes de navegar | Asistentes de varios pasos | La URL no dice qué se está viendo y el botón atrás o un enlace directo dejan el estado incoherente |

## Verificación contra el diseño

Medido en el navegador a 375 × 812.

| Elemento | Diseño (XD) | Medido |
|---|---|---|
| Título del header | "Detalle de producto" | "Detalle de producto" |
| Flecha atrás | (26.4, 23.4) 19.2 × 19.2 | (26.42, 22.92) 19.16 × 19.16 |
| Card | (24, 90) 326 × 128 (cuenta) | (24, 90) 326 de ancho; alto según el producto |
| Título de relleno | 100 px bajo la card (y = 318 con una cuenta) | 100 px bajo la card |
| Párrafo | 30 px bajo el título, ancho 294 | 30 px, ancho 294 |
| Barra de pestañas | No aparece | No aparece |

Contacto y Sucursales (creadas en la fase 7):

| Elemento | Diseño (XD) | Medido |
|---|---|---|
| Título de relleno | y = 261 | y = 261 |
| Párrafo | y = 347, ancho 294 | y = 347, ancho 294 |
| Texto de Sucursales | "Pantalla de transacciones" | "Pantalla de sucursales" (inconsistencia 5 del plan) |

Comprobado en el navegador:

- Las cuatro cards enlazan a `/product/AC/0001`, `/product/AC/0002`, `/product/TC/0001` y `/product/TC/0002`.
- Tocar la primera tarjeta de crédito abre el detalle con su alias y sus dos balances.
- Volver atrás regresa a `/tabs/products`.
- Abrir `/product/AC/0001` sin sesión redirige al login.
- Envolver las cards en enlaces no movió ninguna de su posición.

**No verificado en el navegador:** las animaciones de entrada (deslizar a la izquierda) y de salida (deslizar a la derecha). El panel de vista previa seguía oculto y sin dibujar fotogramas.

## Conceptos

### Parámetros de ruta como entradas

En `product/:type/:digits`, los segmentos con dos puntos son parámetros. Con `withComponentInputBinding()` el router los asigna a las entradas `type` y `digits` del componente. Si la URL cambia sin destruir la página, las entradas se actualizan y el `computed` recalcula el producto.

### Store

Servicio que guarda estado compartido y lo expone de solo lectura. Aquí son dos: `SessionStore` (token y usuario) y `ProductsStore` (productos). Los componentes leen signals del store; solo el store modifica su estado.

### `effect`

Función que se vuelve a ejecutar cuando cambia algún signal que lee. Se usa para efectos secundarios, no para calcular valores (para eso está `computed`). En `ProductsStore` vacía los productos cuando la sesión deja de estar activa.

### Guards encadenados

`canActivate: [authGuard, productExistsGuard]`. Si alguno no devuelve `true`, se aplica el resultado del primero que falla según el orden del arreglo: sin sesión manda al login; con sesión pero sin producto, a "Mis productos".

### `ion-back-button` y `defaultHref`

El botón retrocede en la pila de navegación de Ionic. Si no hay página anterior (por ejemplo, tras recargar), usa `defaultHref`.

## Pruebas

148 pruebas en 24 archivos, todas en verde (14 nuevas).

| Archivo | Qué comprueba |
|---|---|
| `products.store.spec.ts` | Empieza vacío; publica cada actualización; descarta una carga anterior al iniciar otra; busca por tipo y dígitos sin mezclar tipos; no encuentra antes de cargar ni con dígitos desconocidos; se vacía y deja de escuchar al terminar la sesión |
| `product-detail.spec.ts` | Sin sesión va al login; producto no cargado o desconocido vuelve a productos; muestra la cuenta elegida; muestra la tarjeta con sus dos balances; botón atrás hacia productos y textos explicativos |
| `products.page.spec.ts` | Cada card enlaza a su detalle usando solo los últimos dígitos, con dirección de avance |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 148 / 148 |
| `ng lint` | Sin errores |
