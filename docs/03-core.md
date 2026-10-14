# 03 — Core: modelos, servicios, interceptores y guards

Fecha: 2026-10-13 · Fase 3 del plan · Actualizado el mismo día con la carga progresiva de productos y el retardo aleatorio del mock

## Qué se hizo

Se creó `src/app/core/`, la capa sin interfaz gráfica: datos, comunicación con la API, sesión y reglas de acceso. Ninguna pantalla la usa todavía; quedó probada con pruebas unitarias.

```
src/app/core/
  config/        api-endpoints.ts, app-routes.ts
  models/        api.dto.ts, auth-tokens.model.ts, user.model.ts,
                 product.model.ts, account.model.ts, credit-card.model.ts, product-group.model.ts
  mappers/       product.factory.ts
  mocks/         mock-data.ts
  services/      storage.service.ts, user-preferences.service.ts, session.store.ts,
                 auth.service.ts, user.service.ts, products.service.ts
  interceptors/  auth.interceptor.ts, error.interceptor.ts, mock-api.interceptor.ts
  guards/        auth.guard.ts, guest.guard.ts, onboarding.guard.ts
  testing/       in-memory-storage.service.ts
```

Otros cambios:

| Archivo | Cambio |
|---|---|
| `src/app/app.config.ts` | Nuevo. Lista de proveedores de la app, incluido `HttpClient` con los interceptores |
| `src/main.ts` | Ahora solo arranca la app con `appConfig` |
| `src/environments/*.ts` | Se agregaron `apiUrl`, `useMocks` y `mockDelaysMs` |
| `package.json` | Se instaló `@capacitor/preferences` 8.0.1 |

## Cómo fluye una petición

```
Pantalla → Servicio → HttpClient → authInterceptor → errorInterceptor → mockApiInterceptor
                                    (agrega token)   (vigila los 401)   (responde sin red)
```

1. El servicio pide datos con `HttpClient` contra la ruta real del enunciado (`/api/accounts`).
2. `authInterceptor` agrega `Authorization: Bearer <token>` si hay sesión y la petición no es `/sign_in`.
3. `mockApiInterceptor` reconoce la ruta y devuelve la respuesta simulada tras 1, 2 o 3 segundos (al azar). Si no la reconoce, la deja pasar a la red.
4. La respuesta vuelve en sentido inverso. Si es un 401 de una ruta protegida, `errorInterceptor` cierra la sesión y navega al login.
5. El servicio convierte el DTO en un modelo (`Account`, `CreditCard`, `User`).

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | **El token vive solo en memoria**, no en Preferences | Cambio respecto al plan. Capacitor Preferences guarda texto sin cifrar (SharedPreferences en Android, UserDefaults en iOS); no es un lugar adecuado para el token de un banco. En memoria, la sesión termina al cerrar la app, que es el comportamiento esperado en banca. Una app real usaría almacenamiento seguro (Keychain / Keystore) |
| 2 | Preferences se usa solo para datos no sensibles: usuario recordado y onboarding visto | Son preferencias, no credenciales. La contraseña nunca se guarda |
| 3 | Mock con interceptor HTTP, activado por `environment.useMocks` | Los servicios no saben que la API es simulada. Pasar a un backend real es poner `useMocks: false` y la URL en `apiUrl`. Funciona dentro del build nativo sin servidor aparte |
| 4 | `useMocks: true` también en `environment.prod.ts` | No existe backend real. Con `false`, el build de producción quedaría sin datos |
| 5 | El mock valida credenciales y token | Responde 401 si `userId`/`password` no coinciden o si falta el header. Así el flujo de error del login y el interceptor de autenticación se prueban de verdad |
| 6 | Retardo aleatorio de 1, 2 o 3 segundos por petición (`environment.mockDelaysMs`, token `MOCK_API_DELAY`) | Simula tiempos de red distintos por servicio, de modo que la carga progresiva y el indicador de carga del login sean visibles. El retardo es una estrategia inyectable (una función que devuelve milisegundos); en pruebas se inyecta `() => 0` |
| 7 | DTO y modelo separados | El DTO describe lo que envía la API (`access_token`, `productType`). El modelo es lo que usa la app. Si la API cambia un nombre de campo, solo cambia el mapeo |
| 8 | Jerarquía `Product` → `Account` / `CreditCard` | La card de producto trabajará contra `Product` (`displayName`, `balances`) sin preguntar el tipo. La regla "mostrar US$ solo si la tarjeta es internacional" vive en `CreditCard`, no en la plantilla |
| 9 | `StorageService` abstracto con implementación de Capacitor | Los servicios dependen de la abstracción. En pruebas se sustituye por `InMemoryStorageService` |
| 10 | Estado de sesión en `SessionStore` con signals | Único lugar que conoce el token y el usuario. Guards, interceptores y componentes leen de ahí |
| 11 | Interceptores y guards como funciones | Es la forma recomendada desde Angular 15; menos código que las clases |
| 12 | **Carga progresiva de productos** con `merge` (antes `forkJoin`) | Las peticiones salen en paralelo y cada sección se publica en cuanto llega su respuesta. Con `forkJoin` el usuario esperaba al servicio más lento para ver cualquier producto. Detalle en la sección "Carga progresiva de productos" |
| 17 | Orden de secciones fijo, definido por el registro `PRODUCT_SOURCES` | Cuentas siempre va primero aunque las tarjetas respondan antes. El orden de pantalla no depende de la velocidad de la red |
| 18 | Un método genérico `getProducts(type)` en lugar de `getAccounts()` y `getCreditCards()` | Hoy hay dos tipos de producto; una app real tendrá más (préstamos, certificados…). Agregar uno no obliga a escribir un método nuevo ni a tocar la pantalla |
| 19 | Estado por sección: `loading`, `loaded`, `error` | Si un servicio falla, las demás secciones se muestran igual. Para la prueba técnica no se muestra ningún mensaje de error: la sección que falla no se pinta. El estado `error` sirve para saber que esa petición terminó |
| 13 | Archivos con sufijo (`auth.service.ts`, `auth.guard.ts`) | Angular 20+ propone nombres sin sufijo, pero la plantilla de Ionic usa sufijos (`home.page.ts`, `app.component.ts`). Se mantiene la convención existente en el proyecto |
| 14 | `UserPreferencesService` (no estaba en el plan) | Evita que las pantallas conozcan las claves de almacenamiento. Expone métodos con nombre: `rememberUserId`, `hasSeenOnboarding`… |
| 15 | Carpeta `config/` con rutas de API y de navegación | Sin cadenas repetidas por el código |
| 16 | Los guards existen pero aún no están aplicados | Las rutas `/login`, `/onboarding` y `/tabs/products` se crean en fases posteriores |

## Patrones de diseño aplicados

| Patrón | Dónde | Para qué |
|---|---|---|
| Factory | `ProductFactory` | Crea `Account` o `CreditCard` según `productType`; rechaza tipos desconocidos |
| Adapter / Mapper | Servicios y `ProductFactory` | Convierte el formato de la API al modelo de la app |
| Herencia y polimorfismo | `Product`, `Account`, `CreditCard` | Un mismo contrato (`balances`, `displayName`) con comportamiento distinto por tipo |
| Chain of Responsibility | Interceptores HTTP | Cada interceptor hace una sola cosa y pasa la petición al siguiente |
| Inversión de dependencias | `StorageService` | El código depende de una abstracción, no de Capacitor |
| Store / Facade | `SessionStore` | Estado centralizado, de solo lectura hacia afuera |
| Singleton | `providedIn: 'root'` | Una sola instancia de cada servicio en toda la app |
| Guard | `authGuard`, `guestGuard`, `onboardingGuard` | Decide si una ruta se puede abrir |

## Carga progresiva de productos

Cambio pedido después de la primera versión del core. Objetivo: que el usuario vea sus productos a medida que responde cada servicio, sin esperar a que respondan todos.

### Cómo funciona

`ProductsService.loadGroups()` devuelve un `Observable` que emite la lista completa de secciones cada vez que cambia el estado de una:

| Momento | Emisión |
|---|---|
| Al suscribirse | `[AC: loading, TC: loading]` |
| Responde tarjetas (1 s) | `[AC: loading, TC: loaded]` |
| Responde cuentas (3 s) | `[AC: loaded, TC: loaded]` y el observable termina |

Cada elemento es un `ProductGroup`: `{ type, status, products }`. La pantalla solo recorre el arreglo y pinta cada sección según su `status`; no conoce los endpoints ni el orden.

### Registro de tipos

```ts
const PRODUCT_SOURCES = [
  { type: 'AC', endpoint: API_ENDPOINTS.accounts },
  { type: 'TC', endpoint: API_ENDPOINTS.creditCards },
];
```

Este arreglo define dos cosas: qué tipos de producto existen y en qué orden se muestran. Para agregar un tipo nuevo se necesita una entrada aquí, su clase modelo, su caso en `ProductFactory` y su endpoint. El servicio y la pantalla no cambian.

### `forkJoin` frente a `merge`

- `forkJoin` espera a que **todos** los observables terminen y emite una sola vez con todos los resultados. Si uno falla, falla todo.
- `merge` reenvía cada valor en cuanto llega, sin importar de qué observable viene.

Se combinó `merge` con `scan`, que acumula: parte del estado inicial (todo en carga) y, por cada respuesta, reemplaza solo la sección correspondiente. `startWith` emite el estado inicial de inmediato para que la pantalla pueda pintar las secciones en carga. `catchError` dentro de cada petición convierte un fallo en una sección con `status: 'error'`, así el flujo general no se rompe.

### Qué falta para verlo en pantalla

Esta fase deja lista la capa de datos. El render en "Mis productos" se hace en la fase 6. El diseño no define el estado de carga, así que se decidió lo siguiente (detalle en la sección 3.4 de `PLAN.md`):

- Un único skeleton con la forma de una card de producto mientras hay respuestas pendientes.
- Cada sección se pinta, con su título y sus cards, al llegar su respuesta.
- Una sección sin productos no se pinta.
- Sin manejo de errores en pantalla. El estado `error` del core se conserva solo para que una petición fallida cuente como terminada y el skeleton no quede visible para siempre; esa sección no se pinta.

El core no necesita cambios para esto: la pantalla calcula "hay algo cargando" a partir del `status` de cada `ProductGroup`.

## Conceptos

### Interceptor HTTP

Función que se ejecuta en cada petición de `HttpClient`, antes de que salga y cuando vuelve la respuesta. Recibe la petición y una función `next` para continuar la cadena. Las peticiones son inmutables: para cambiarlas se clonan (`req.clone(...)`). Se ejecutan en el orden en que se registran en `app.config.ts`.

### Guard de ruta

Función que el router llama antes de activar una ruta. Devuelve `true` para permitir o un `UrlTree` para redirigir. Los guards son comodidad de navegación, no seguridad: la protección real la da el servidor al exigir el token.

- `authGuard`: sin sesión, redirige a `/login`.
- `guestGuard`: con sesión, redirige a `/tabs/products` (evita ver el login estando autenticado).
- `onboardingGuard`: si el onboarding no se ha visto, redirige a `/onboarding`.

### Signals

Un signal es un valor que avisa cuando cambia. `signal()` crea uno modificable; `computed()` deriva un valor de otros y se recalcula solo. `SessionStore` guarda los tokens en un signal privado y expone `isAuthenticated` como `computed`.

### Inyección de dependencias

Angular crea los servicios y los entrega a quien los pide con `inject()`. Eso permite que una prueba entregue una versión falsa (`{ provide: StorageService, useClass: InMemoryStorageService }`) sin tocar el código probado.

### Capacitor Preferences

Almacén clave-valor persistente. En Android usa SharedPreferences, en iOS UserDefaults y en navegador `localStorage`. La API es la misma en las tres plataformas y siempre es asíncrona.

### Observable frío

`HttpClient` no envía nada hasta que alguien se suscribe. Por eso `signIn()` devuelve un `Observable`: la pantalla decide cuándo ejecutarlo.

## Credenciales del mock

Usuario `00100010321`, contraseña `1111` (las del enunciado). Están en `core/mocks/mock-data.ts`.

## Pruebas

68 pruebas en 13 archivos, todas en verde.

| Archivo | Qué comprueba |
|---|---|
| `product.model.spec.ts` | Últimos 4 dígitos, nombre visible, balances de cuenta y de tarjeta local e internacional, nombre completo del usuario |
| `product.factory.spec.ts` | AC → `Account`, TC → `CreditCard`, tipo desconocido lanza error |
| `session.store.spec.ts` | Inicio, usuario y limpieza de sesión |
| `storage.service.spec.ts` | Guardar, leer y borrar con Capacitor Preferences |
| `user-preferences.service.spec.ts` | Usuario recordado y onboarding visto |
| `api-services.spec.ts` | `AuthService`, `UserService`, `ProductsService`: URL, método y mapeo. Carga progresiva: estado inicial en carga, publicación por sección, orden fijo aunque las tarjetas respondan antes, error aislado por sección |
| `auth.interceptor.spec.ts` | Agrega el token; no lo agrega sin sesión ni en `/sign_in` |
| `error.interceptor.spec.ts` | 401 protegido cierra sesión y navega; 401 de login y otros errores no |
| `mock-api.interceptor.spec.ts` | Login válido e inválido, los tres GET con y sin token, rutas desconocidas pasan, elección aleatoria del retardo y espera efectiva del tiempo indicado |
| `guards.spec.ts` | Los tres guards, permitiendo y redirigiendo |
| `core-integration.spec.ts` | Cadena completa: login → usuario → grupos de productos; credenciales erróneas; petición sin token |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 68 / 68 |
| `ng lint` | Sin errores |

## Pendientes

- Aplicar los guards cuando existan las rutas.
- Refresh token: se recibe y se guarda en la sesión, pero no hay endpoint para renovarlo en el enunciado.
- Búsqueda de un producto por tipo y número para la pantalla de detalle (fase 8).
