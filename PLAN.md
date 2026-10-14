# Plan de trabajo — Prueba técnica frontend (Ionic 9 + Angular 22 + Capacitor)

> Fuente del diseño: Adobe XD `BHDLeon_Frontend_Test` (modo Specs, 19 artboards, link actualizado el 15-ene-2020).
> Todos los valores de la sección 2 salen de los datos del propio archivo XD, no de estimaciones sobre capturas.

## 1. Alcance

### 1.1 Pantallas (19 artboards → 9 pantallas reales + estados)

| Artboard XD | Qué es | Ruta propuesta |
|---|---|---|
| `demo_1`, `demo_2`, `demo_3` | Onboarding de 3 slides | `/onboarding` |
| `login` | Login vacío (pantalla inicial del prototipo) | `/login` |
| `login – 2` | Login con datos y toggle activo | estado |
| `login – 3` | Login con indicador de carga | estado |
| `login – 4` | Login con toast de error | estado |
| `login – 1` | Login con menú lateral público (Contacto, Sucursales) | estado |
| `dashboard` | "Mis productos": cuentas + tarjetas (alto 893, hace scroll) | `/tabs/products` |
| `transactions`, `offers`, `configuration` | Pantallas placeholder con tabs | `/tabs/transactions`, `/tabs/offers`, `/tabs/configuration` |
| `dashboard – 1`, `transactions – 1`, `offers – 1`, `configuration – 1` | Las mismas con menú lateral privado abierto | estado |
| `product_detail` | Detalle de producto (valida navegación + paso de datos) | `/product/:type/:digits` (últimos 4 dígitos, no el número completo) |
| `contact`, `branches` | Placeholders con botón atrás | `/contact`, `/branches` |

### 1.2 Flujo de navegación (interacciones del prototipo)

- `demo_1 → demo_2 → demo_3 → login` con "CONTINUAR" o tap, transición slide-left 0.3 s.
- Login → "INICIAR SESIÓN" → carga → dashboard (slide-left). Con error → toast.
- Tabs: cambio entre las 4 pestañas con dissolve 0.3 s.
- Tap en una cuenta → `product_detail` (slide-left); flecha atrás → slide-right.
- Icono hamburguesa → menú lateral; tap en el overlay lo cierra.
- Menú privado: Mis productos, Transacciones, Ofertas, Configuración, Contacto, Sucursales, "CERRAR SESIÓN" → login.
- Menú público (desde login): solo Contacto y Sucursales.
- Contacto y Sucursales: slide-left al entrar, atrás vuelve a la pantalla anterior.

### 1.3 Servicios (mock)

`POST /sign_in`, `GET /user`, `GET /accounts`, `GET /credit_cards`. Los tres GET llevan `Authorization: Bearer <access_token>`.

## 2. Especificación de diseño

### 2.1 Lienzo y grilla

- Artboard: **375 × 812** (iPhone X). Grilla de 12 columnas, gutter 10, **margen lateral 24**, espaciado base 8.
- Ancho útil de contenido: **326** (375 − 24 − 24, el XD usa 326 y deja 25 a la derecha).

### 2.2 Colores

| Token | Valor | Uso |
|---|---|---|
| `$color-primary` | `#49B742` | Botones, barra de header/tabs, iconos de menú, título "CUENTAS", tab activo |
| `$color-secondary` | `#00477A` | Título "TARJETAS" y franja lateral de tarjetas de crédito |
| `$color-text` | `#666666` | Texto principal, iconos inactivos |
| `$color-text-muted` | `#999999` | "Balance disponible" |
| `$color-placeholder` | `#CCCCCC` | Placeholders, iconos de input, textos de pantallas placeholder |
| `$color-border` | `rgba(204,204,204,0.6)` | Borde 1px (interior) de inputs, cards, dots |
| `$color-background` | `#F5F5F5` | Fondo de todas las pantallas internas |
| `$color-surface` | `#FFFFFF` | Header, tabs, cards, inputs, menú |
| `$color-danger` | `#C92340` | Toast de error |
| `$color-overlay` | `rgba(0,0,0,0.16)` | Overlay del menú lateral |
| `$color-backdrop` | `rgba(0,0,0,0.4)` | Fondo del indicador de carga |
| `$color-toggle-off` | `rgba(255,255,255,0.6)` | Pista del toggle apagado |
| `$shadow-knob` | `1px 1px 1.5px rgba(0,0,0,0.16)` | Perilla del toggle |

### 2.3 Tipografía

Familia: **Trebuchet MS** (Regular y Bold).

| Token | Tamaño / line-height | Uso |
|---|---|---|
| `xs` | 10 / 11 | Labels de tabs, "Balance disponible" |
| `sm` | 12 / 14 | "Recuerda mi usuario", ítems del menú |
| `md` | 14 / 16 | Inputs, botones (Bold), títulos de sección (Bold), alias de producto, prefijo de moneda |
| `lg` | 16 / 19 | Título del header |
| `xl` | 24 / 28 | Montos, títulos de onboarding y placeholders |

Los montos son un solo texto con dos tamaños: el símbolo de moneda (`RD$ `) y los decimales (`.00`) a 14, y la parte entera (`1,000,000`) a 24.

### 2.4 Componentes

**Header** (66 de alto): fondo blanco 62 + barra verde 4. Título centrado 16/19 `#666`. Hamburguesa en (24, 23): 25 × 19, tres barras de 3px separadas 5px, `#49B742`. Flecha atrás (`arrow-back` de Ionicons md) 19.2 × 19.2 en (26.4, 23.4), verde.

**Input**: 326 × 44, radio 8, fondo blanco, borde 1px. Icono a 16 del borde izquierdo (≈15 × 17, trazo 1px `#CCCCCC`), texto a 47 del borde izquierdo, 14/16. Placeholder `#CCCCCC`, valor `#666666`.

**Botón principal**: 326 × 44, radio 8, `#49B742`, texto Bold 14/16 blanco centrado, mayúsculas escritas.
**Botón outline** ("CERRAR SESIÓN"): 237 × 44, radio 8, borde 1px verde, texto Bold 14 verde.

**Toggle**: pista 40 × 24 radio 12; perilla 22 con sombra, a 1px del borde. Apagado: pista blanca 60% + borde. Encendido: pista `#49B742`, perilla desplazada 16px. Label a 8px, 12/14 `#666`.

**Card de producto**: 326 de ancho, radio 4, fondo blanco, borde 1px, franja izquierda de 6px (verde en cuentas, azul en tarjetas). Padding 16.
- Icono 40 × 40 en (16, 16); alias a 72 del borde izquierdo de la card (x=96), centrado verticalmente con el icono, 14/16.
- Monto a 72 del borde superior, alto 28; "Balance disponible" 1px debajo, 10/11 `#999`; 16 de padding inferior.
- Cuenta: alto **128**. Tarjeta con RD$ y US$: alto **156** (segunda línea de monto de 28). Tarjeta solo RD$: **128** (el XD la dibuja de 122 con el icono desplazado; al unificar la posición del icono, inconsistencia 4, queda igual que la cuenta).
- Alias mostrado: `{alias} - {últimos 4 dígitos del número}`.

**Sección de productos**: título Bold 14/16 (`CUENTAS (n)` verde, `TARJETAS (n)` azul) a 25 del header; 16 entre título y primera card; 16 entre cards; 33 entre secciones.

**Tab bar** (80 de alto, fija abajo): fondo blanco, barra verde de 4px arriba. 4 pestañas de ancho igual. Icono 30 × 30 a 15 del borde superior; label 10/11 a 6 del icono. Activo `#49B742`, inactivo `#666666`.

**Menú lateral**: ancho **269**, fondo blanco; overlay sobre los 106 restantes. Barra verde de 4px en y=62 (alineada con el header).
- Privado: avatar circular 42 en (24, 12); nombre en x=82, 14/16. Ítems desde y=106 con paso de 48: icono 24 verde en x=24, texto en x=64, 12/14. Separador 1px en y=314. Contacto y Sucursales desde y=354. Botón outline en (16, 744).
- Público: logo 149 × 46 en (60, 12); Contacto y Sucursales desde y=106.

**Indicador de carga**: backdrop a pantalla completa; caja blanca 103 × 109 radio 4 centrada; spinner 28 × 28 negro.

**Toast de error**: 356 × 61 en (10, 12), radio 4, `#C92340`, texto 14 blanco centrado: "Usuario y/o contraseña incorrectos".

### 2.5 Pantallas

**Login**: imagen de fondo 768 × 1024 escalada a alto completo (609 × 812, centrada con recorte lateral). Hamburguesa en (24, 23). Logo 215 × 66 en (80, 120). Inputs en y=263 y y=331 (24 de separación). Toggle en (107, 399). Botón en y=447.

**Onboarding**: fondo `#F5F5F5`. Título 24/28 `#666` centrado en y=169. Ilustración ≈150 × 150 centrada en y≈364 (gris `#666` + acento verde al 50%). Dots de 16px en y=706, separación 8, activo verde, inactivos blancos con borde. Footer blanco de 60 en y=752 con "CONTINUAR" Bold 14 verde, a 32 del borde derecho.

Textos: "Puedes ver tus productos" · "Ahorra tiempo realizando transacciones de forma rápida y segura" · "Aprovecha las ofertas que tenemos para ti".

**Placeholders** (transacciones, ofertas, configuración, contacto, sucursales): título 24/28 `#CCC` centrado en y=261 ("Pantalla / de …"); párrafo 14/16 `#CCC` centrado, ancho 294, en y=347.

**Detalle de producto**: header con flecha atrás; card del producto seleccionado en y=90 (24 bajo el header); título placeholder en y=318 y párrafo en y=404.

### 2.6 Inconsistencias del diseño (decidir y comentarlas en la entrevista)

| # | Lo que dice el XD | Propuesta |
|---|---|---|
| 1 | Algunos textos están en **Helvetica Neue Medium** (título del header en las pantallas con tabs, toast, un "Balance disponible"); el resto en Trebuchet MS | Usar Trebuchet MS en todo; parece un reemplazo de fuente accidental |
| 2 | Separación entre tarjetas de crédito 22, entre cuentas 16 | Usar 16 |
| 3 | Alias de la primera cuenta en x=98, el resto en x=96 | Usar 96 |
| 4 | Icono de tarjeta a 19 del borde superior en la primera y a 13 en la segunda | Centrar el icono en la misma caja de 40 × 40 |
| 5 | `branches` dice "Pantalla de transacciones" | Escribir "Pantalla de sucursales" |
| 6 | Menú muestra "Juan Pérez"; la API devuelve "José Pérez" | Usar el dato de la API |
| 7 | Montos del diseño no coinciden con los de la API | Usar los de la API, con el formato del diseño |
| 8 | El onboarding solo se alcanza tocando el logo del login | Mostrarlo en el primer arranque y guardar la marca con Preferences |
| 9 | El dashboard con menú abierto no muestra logo ni foto real | Usar `photo` de `/user` en el avatar de 42px |
| 10 | El diseño no define cómo se ve "Mis productos" mientras carga ni cuando falla | Decidido: **un único skeleton** con la forma de una card de producto (326 × 128, radio 4, franja izquierda de 6). Las secciones no muestran título mientras cargan. Una sección sin productos no se pinta. No se diseña estado de error: una sección que falla no se pinta. El skeleton sigue visible mientras quede algún servicio sin responder, en la posición de la primera sección pendiente (ver sección 3.4) |

Trebuchet MS viene instalada en Windows, macOS e iOS, pero **no en Android**. Si se va a mostrar en un dispositivo Android hay que definir una fuente de respaldo y decirlo abiertamente.

## 3. Arquitectura

### 3.1 Stack

- Angular 22 con componentes standalone, signals, control flow (`@if`, `@for`) y detección de cambios OnPush.
- Ionic 9 (`@ionic/angular/standalone`), modo `md` forzado para que iOS y Android se vean igual que el diseño.
- Capacitor: `@capacitor/preferences` (recordar usuario, onboarding visto, token), `status-bar`, `splash-screen`, `keyboard`.
- SCSS con tokens (`_variables.scss`) expuestos también como CSS custom properties y mapeados a las variables de Ionic.
- Pruebas unitarias con el runner que genere el CLI.

Verificar al crear el proyecto que la versión de `@ionic/angular` instalada declara soporte para Angular 22.

### 3.2 Estructura de carpetas

```
src/app/
  core/
    models/         product.model.ts, account.model.ts, credit-card.model.ts, user.model.ts, auth-tokens.model.ts
    mappers/        product.factory.ts (DTO → modelo)
    services/       auth.service.ts, user.service.ts, products.service.ts, products.store.ts, storage.service.ts, user-preferences.service.ts, session.store.ts
    interceptors/   auth.interceptor.ts, mock-api.interceptor.ts, error.interceptor.ts
    guards/         auth.guard.ts, guest.guard.ts, onboarding.guard.ts, product-exists.guard.ts
    mocks/          datos de respuesta de los 4 servicios
  shared/
    components/     header, menu-button, icon, button, input, toggle, product-card, product-card-skeleton, placeholder-content, loading-overlay
    config/         product-presentation.ts (título, icono y color por tipo de producto), main-sections.ts (secciones y enlaces)
  layout/
    side-menu/      menú lateral público y privado, cierre de sesión
  features/
    onboarding/
    auth/login/
    tabs/           tabs.page (contenedor de pestañas)
    products/       "Mis productos"
    placeholder/    una página parametrizada por la ruta: transacciones, ofertas, configuración, contacto, sucursales
    product-detail/
  app.routes.ts, app.config.ts
src/theme/          _variables.scss, _typography.scss, _mixins.scss, variables.scss (Ionic)
```

### 3.3 Decisiones a defender

- **Mock con interceptor HTTP** (`mockApiInterceptor`, activado por `environment.useMocks`). Los servicios usan `HttpClient` real contra los paths del documento; cambiar a un backend real es apagar una bandera. Funciona igual dentro del build nativo de Capacitor, sin servidor aparte. Valida `userId`/`password` y devuelve 401 si no coinciden, y responde 401 a los GET sin token.
- **Interceptor de autenticación**: agrega `Authorization: Bearer` a todo menos `/sign_in`.
- **POO**: clase abstracta `Product` (alias, número, tipo, `displayName`, `balances`) con `Account` y `CreditCard` como subclases. `CreditCard` decide si muestra US$ según `isInternational`. La card de producto trabaja contra `Product` sin saber el tipo concreto.
- **Patrones**: Factory (crear el modelo según `productType` AC/TC), Adapter (DTO → modelo), Interceptor (cadena de responsabilidad), Facade/Store (estado de sesión con signals), Singleton por DI, Strategy para el formato de moneda, Guard para rutas.
- **Performance**: rutas lazy con `loadComponent`, OnPush + signals, `@for` con `track`, carga progresiva de productos (ver abajo), iconos SVG en lugar de fuentes, imagen de fondo optimizada, `@defer` donde aplique.
- **Carga progresiva de productos**: cada tipo de producto se pide en paralelo y su sección se pinta en cuanto llega su respuesta, sin esperar a las demás (`merge`, no `forkJoin`). El orden de las secciones es fijo y lo define el registro de tipos de `ProductsService`: primero Cuentas, después Tarjetas, aunque las tarjetas respondan antes. Cada sección tiene su propio estado (`loading`, `loaded`, `error`), de modo que el fallo de un servicio no impide mostrar los demás. Agregar un tipo de producto nuevo es añadir una entrada al registro, su modelo y su caso en la fábrica.
- **Retardo del mock**: cada petición simulada tarda 1, 2 o 3 segundos, elegidos al azar, para que la carga progresiva sea visible. Se configura en `environment.mockDelaysMs`.
- **Almacenamiento**: `StorageService` como abstracción sobre Capacitor Preferences, para poder simularlo en pruebas.

### 3.4 Estado de carga de "Mis productos" (fase 6)

Definido por el usuario el 2026-10-13.

- Al entrar a la pantalla se muestra **un solo skeleton** con la forma de una card de producto. Es uno solo porque no se sabe cuántos tipos de producto tiene el usuario: puede tener uno o dos.
- Cuando llega la respuesta de un tipo de producto, se pintan su título y sus cards.
- Una sección cuya respuesta llega vacía no se pinta (ni título ni cards).
- **Sin manejo de errores en pantalla** para la prueba técnica. El core conserva el estado `error` por sección para que una petición fallida cuente como terminada; en pantalla esa sección simplemente no aparece. Sin eso, un fallo dejaría el skeleton visible para siempre.
- Medidas del skeleton: las de la card de cuenta (326 × 128, radio 4, borde 1px, franja izquierda de 6px), con bloques grises en las posiciones del icono, el alias, el monto y la etiqueta.

Cuándo desaparece el skeleton. El usuario eligió la **opción A** el 2026-10-13:

| Opción | Comportamiento | Efecto |
|---|---|---|
| **A (elegida)** | El skeleton sigue visible mientras quede algún servicio sin responder, ubicado en la posición de la primera sección pendiente | Si las tarjetas llegan antes, el skeleton queda arriba ocupando el lugar de Cuentas y las tarjetas debajo. El usuario sabe que falta contenido y el salto al llegar las cuentas es menor |
| B (descartada) | El skeleton desaparece con la primera respuesta | Si las tarjetas llegan antes, la pantalla parece completa durante 1 o 2 segundos y luego las cuentas aparecen arriba empujando las tarjetas hacia abajo |

## 4. Fases

| # | Fase | Resultado verificable |
|---|---|---|
| 0 | Assets: ya descargados en `design-assets/` (logo, fondo, iconos SVG, ilustraciones); copiarlos a `src/assets` al crear el proyecto | Carpeta `src/assets` completa |
| 1 | Crear proyecto (`ionic start`, blank, standalone, Capacitor), git, ESLint/Prettier | App arranca en 375 × 812 |
| 2 | Tema: tokens SCSS, tipografía, variables Ionic, modo `md` | Página de prueba con colores y textos correctos |
| 3 | Core: modelos, factory, mocks, interceptores, servicios, storage, guards + sus pruebas | Pruebas en verde |
| 4 | Componentes compartidos: botón, input, toggle, header, card, loading, placeholder | Cada uno comparado con el XD |
| 5 | Login completo: formulario con Signal Forms (Reactive Forms queda solo como referencia en `docs/05`), recordar usuario, carga, toast de error | Los 4 estados coinciden con `login` … `login – 4` |
| 6 | Tabs + "Mis productos" con datos de la API, con carga progresiva por sección (Cuentas primero) y un único skeleton de card mientras carga (sección 3.4) | Coincide con `dashboard`, scroll correcto; cada sección aparece al llegar su respuesta |
| 7 | Menú lateral público y privado, cerrar sesión | Coincide con los artboards `– 1` |
| 8 | Detalle de producto (paso de datos), placeholders, contacto, sucursales | Navegación completa del prototipo |
| 9 | Onboarding con primer arranque | Coincide con `demo_1..3` |
| 10 | Pixel perfect: comparar cada pantalla contra el XD a 375 × 812 y corregir | Diferencias anotadas o resueltas |
| 11 | Capacitor: `cap add android`, status bar, splash, safe areas | Build nativo abre |
| 12 | Pruebas faltantes, README, repaso para la entrevista | `ng test` y `ng build` en verde |

Si el tiempo no alcanza, el orden de recorte es: 11, 9, y dejar contacto/sucursales como una sola página parametrizada. No recortar 10 ni las pruebas del core.

## 5. Verificación de pixel perfection

1. Navegador a 375 × 812, DPR 1, modo `md`.
2. Por cada pantalla, medir con `getBoundingClientRect` los elementos clave y compararlos con la sección 2.
3. Comparar visualmente contra el artboard del XD, lado a lado.
4. Registrar las diferencias aceptadas (las de la tabla 2.6).

## 6. Pruebas unitarias mínimas

- `ProductFactory`: AC → `Account`, TC → `CreditCard`, tipo desconocido.
- `CreditCard`: balances con y sin `isInternational`.
- `authInterceptor`: agrega el header, omite `/sign_in`.
- `mockApiInterceptor`: login correcto, login incorrecto (401), GET sin token (401), retardo aleatorio.
- `ProductsService.loadGroups`: estado inicial en carga, publicación por sección al llegar cada respuesta, orden fijo aunque las tarjetas respondan antes, error aislado por sección.
- `AuthService`: guarda token, cierra sesión, limpia storage.
- Guards: redirecciones con y sin sesión.
- Formato de monto y últimos dígitos (en `ProductCardComponent` y en el modelo `Product`).
- `LoginPage`: validación del formulario, estado de carga, toast en error.
- `ProductCardComponent`: render de cuenta, tarjeta internacional y tarjeta local.

## 7. Guion para la entrevista

- Cómo se obtuvieron las medidas (Specs del XD) y qué inconsistencias se encontraron y cómo se resolvieron.
- Por qué un interceptor para el mock y cómo se cambia a un backend real.
- Jerarquía `Product` y dónde se aplica cada patrón.
- Decisiones de performance y cómo se medirían.
- Qué quedó fuera y qué se haría con más tiempo (refresh token, i18n, pruebas e2e, accesibilidad).
