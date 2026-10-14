# 15 — Patrones de diseño usados en la app

Referencia principal: *Sumérgete en los patrones de diseño*, de Alexander Shvets (Refactoring.Guru). Las páginas son las del índice del libro. Los patrones que el libro no trae (Store, Data Mapper y Guard) se explican a partir de otras fuentes, listadas al final.

Cada patrón tiene cuatro partes:
- **Qué es:** una explicación corta.
- **Reglas:** las clases o interfaces que se necesitan para implementarlo.
- **En la app:** dónde se usa.
- **Ejemplo:** un fragmento corto del código del proyecto.

## Resumen

| # | Patrón | Tipo | Fuente | Dónde |
|---|---|---|---|---|
| 1 | Factory Method (variante simple) | Creacional | Libro, p. 78 | `ProductFactory` |
| 2 | Singleton | Creacional | Libro, p. 148 | Servicios con `providedIn: 'root'` |
| 3 | Adapter | Estructural | Libro, p. 161 | `PreferencesStorageService` |
| 4 | Facade | Estructural | Libro, p. 226 | `UserPreferencesService` |
| 5 | Chain of Responsibility | Comportamiento | Libro, p. 270 | Interceptores HTTP |
| 6 | Observer | Comportamiento | Libro, p. 358 | Signals y RxJS |
| 7 | Strategy | Comportamiento | Libro, p. 391 | `MOCK_API_DELAY` |
| — | Inversión de dependencias | Principio SOLID | Libro, p. 71 | `StorageService` |
| 8 | Store | Arquitectura del estado | Investigación | `SessionStore`, `ProductsStore` |
| 9 | Data Mapper | Arquitectura de datos | Investigación | DTO → modelo en `ProductFactory` |
| 10 | Guard | Propio de Angular | Investigación | `authGuard`, `guestGuard`, `onboardingGuard`, `productExistsGuard` |

---

## 1. Factory Method (variante simple)

**Qué es:** se centraliza la creación de objetos en un método que decide qué clase concreta instanciar. Quien lo usa recibe el tipo general (`Product`) y no sabe si le llegó una `Account` o una `CreditCard`.

**Reglas según el libro:**
| Participante | Qué es |
|---|---|
| Producto | Interfaz o clase abstracta común a todo lo que se puede crear |
| Productos concretos | Las implementaciones de esa interfaz |
| Creador | Clase con el método fábrica, que devuelve el tipo Producto |
| Creadores concretos | Subclases que sobrescriben el método fábrica para devolver otro producto |

**En la app:**
| Participante | Clase |
|---|---|
| Producto | `Product` (abstracta) |
| Productos concretos | `Account`, `CreditCard` |
| Creador | `ProductFactory.create(dto)` |

**Ejemplo:**
```ts
static create(dto: ProductDto): Product {
  switch (dto.productType) {
    case 'AC': return ProductFactory.createAccount(dto);
    case 'TC': return ProductFactory.createCreditCard(dto);
    default: throw new Error(`Unsupported product type`);
  }
}
```

> **Para la entrevista:** esta es una **fábrica simple** (un método estático con `switch`), no el Factory Method completo del libro, que usa subclases creadoras. El libro muestra este `switch` como paso intermedio antes de dividirlo en subclases. Para este caso se escogió la versión simple: con dos tipos de producto que solo cambian en sus datos, crear una subclase por cada uno sería complejidad sin beneficio. Si cada tipo tuviera reglas de creación propias, se pasaría a subclases.

---

## 2. Singleton

**Qué es:** una clase de la que existe una sola instancia en toda la app, con un único punto de acceso.

**Reglas según el libro:**
- Un campo estático que guarda la instancia.
- Un constructor oculto (privado).
- Un método estático, por ejemplo `getInstance()`, que crea la instancia la primera vez y luego devuelve siempre la misma.

**En la app:** no se escribe a mano. Lo resuelve Angular: un servicio con `providedIn: 'root'` tiene una sola instancia, que el inyector crea la primera vez que alguien la pide. Así funcionan `SessionStore`, `ProductsStore`, `AuthService` y los demás servicios.

**Ejemplo:**
```ts
@Injectable({ providedIn: 'root' })
export class SessionStore { }

private readonly session = inject(SessionStore);
```

> **Diferencia con el libro:** el Singleton clásico es difícil de probar porque la instancia es global y no se puede reemplazar. Con el inyector de Angular cada prueba tiene su propia instancia y se puede cambiar por una falsa (`{ provide: X, useClass: Y }`). Por eso en Angular no se escribe un `getInstance()`.

---

## 3. Adapter

**Qué es:** una clase intermedia que permite usar un objeto cuya interfaz no es la que el código espera. Traduce las llamadas de un formato al otro.

**Reglas según el libro:**
| Participante | Qué es |
|---|---|
| Cliente | El código de la app |
| Interfaz con el cliente | El contrato que el cliente conoce |
| Servicio | La clase externa con una interfaz distinta |
| Adaptador | Implementa la interfaz con el cliente y envuelve al servicio, traduciendo cada llamada |

**En la app:**
| Participante | Clase |
|---|---|
| Cliente | `UserPreferencesService` |
| Interfaz | `StorageService` (abstracta), con `get(key)` que devuelve `string \| null` |
| Servicio | `Preferences` de Capacitor, con `get({ key })` que devuelve `{ value }` |
| Adaptador | `PreferencesStorageService` |

**Ejemplo:**
```ts
export class PreferencesStorageService extends StorageService {
  async get(key: string): Promise<string | null> {
    const { value } = await Preferences.get({ key });
    return value;
  }
}
```

`NativeAppService` sigue la misma idea con el plugin `App`: envolver el plugin permite simularlo en las pruebas.

---

## 4. Facade

**Qué es:** una clase que ofrece una forma sencilla de usar un subsistema más complejo y esconde sus detalles.

**Reglas según el libro:**
| Participante | Qué es |
|---|---|
| Fachada | Expone pocos métodos claros y sabe cómo usar el subsistema |
| Subsistema | Las clases con los detalles; no saben que existe la fachada |
| Cliente | Usa la fachada en lugar del subsistema |

Opcionalmente puede haber fachadas adicionales para no mezclar responsabilidades.

**En la app:** `UserPreferencesService` es la fachada del almacenamiento. Las pantallas llaman `rememberUserId()` o `hasSeenOnboarding()` y no conocen las claves (`'rememberedUserId'`, `'onboardingSeen'`), ni que los valores se guardan como texto, ni que detrás está Capacitor.

**Ejemplo:**
```ts
async hasSeenOnboarding(): Promise<boolean> {
  return (await this.storage.get(ONBOARDING_SEEN_KEY)) === 'true';
}
```

---

## 5. Chain of Responsibility

**Qué es:** una petición pasa por una cadena de manejadores. Cada uno decide si la procesa, la modifica o la pasa al siguiente.

**Reglas según el libro:**
| Participante | Qué es |
|---|---|
| Interfaz manejadora | Método común para atender la petición |
| Manejadora base (opcional) | Guarda la referencia al siguiente manejador |
| Manejadores concretos | Cada uno decide si procesa y si continúa la cadena |
| Cliente | Arma la cadena |

**En la app:**
| Participante | Pieza |
|---|---|
| Interfaz | `HttpInterceptorFn` de Angular |
| Siguiente manejador | La función `next` |
| Manejadores concretos | `authInterceptor` (agrega el token), `errorInterceptor` (cierra la sesión si llega un 401), `mockApiInterceptor` (responde con datos simulados y corta la cadena) |
| Cliente | `app.config.ts` con `withInterceptors([...])` |

**Ejemplo:**
```ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const accessToken = inject(SessionStore).accessToken();
  if (req.url === API_ENDPOINTS.signIn || accessToken === null) return next(req);
  return next(req.clone({ setHeaders: { Authorization: `Bearer ${accessToken}` } }));
};
```

El mock es el último de la cadena y no llama a `next`. Por eso la petición nunca sale a la red, y para usar la API real basta con quitarlo de la lista.

---

## 6. Observer

**Qué es:** un objeto (notificador) avisa automáticamente a los que están suscritos cuando cambia su estado, sin conocer sus clases concretas.

**Reglas según el libro:**
| Participante | Qué es |
|---|---|
| Notificador | Guarda la lista de suscriptores y les avisa |
| Interfaz suscriptora | El método de aviso, por ejemplo `update()` |
| Suscriptores concretos | Reaccionan al aviso |
| Cliente | Registra los suscriptores |

**En la app:** no se implementa a mano; lo dan Angular y RxJS.
| Participante | Pieza |
|---|---|
| Notificador | `signal()` y `Observable` |
| Suscriptores | `computed()`, `effect()`, la plantilla y `subscribe()` |

Un ejemplo concreto: `ProductsStore` observa la sesión y borra los productos cuando el usuario cierra sesión, sin que `SessionStore` sepa que `ProductsStore` existe.

**Ejemplo:**
```ts
effect(() => {
  if (!this.session.isAuthenticated()) this.clear();
});
```

---

## 7. Strategy

**Qué es:** se separa un algoritmo detrás de una interfaz para poder cambiarlo por otro sin tocar el código que lo usa.

**Reglas según el libro:**
| Participante | Qué es |
|---|---|
| Contexto | Usa la estrategia solo a través de su interfaz |
| Interfaz estrategia | Declara el método que se ejecuta |
| Estrategias concretas | Variantes del algoritmo |
| Cliente | Escoge la estrategia y se la entrega al contexto |

**En la app:**
| Participante | Pieza |
|---|---|
| Contexto | `mockApiInterceptor` |
| Interfaz | `type MockDelayStrategy = () => number` |
| Estrategias | La de producción, que da 1, 2 o 3 s al azar; las de las pruebas: `() => 0` y `() => 2000` |
| Cliente | El inyector, con el token `MOCK_API_DELAY` |

**Ejemplo:**
```ts
export const MOCK_API_DELAY = new InjectionToken<MockDelayStrategy>('MOCK_API_DELAY', {
  providedIn: 'root',
  factory: () => () => pickRandomDelay(environment.mockDelaysMs),
});

{ provide: MOCK_API_DELAY, useValue: () => 0 }
```

En TypeScript la estrategia puede ser una función en lugar de una clase. La idea del patrón es la misma.

---

## Principio: Inversión de dependencias (la D de SOLID)

No es un patrón, sino uno de los principios del libro. Se incluye porque es lo que permite el Adapter y las pruebas sin Capacitor.

**Qué es:** el código de alto nivel (la lógica de la app) no depende del de bajo nivel (el plugin). Los dos dependen de una abstracción.

**Reglas:**
1. Definir una interfaz o clase abstracta con operaciones pensadas para el negocio.
2. Hacer que la lógica dependa solo de esa abstracción.
3. Implementar la abstracción con la tecnología concreta.

**En la app:** `StorageService` es la abstracción; `PreferencesStorageService` la implementa en la app real, y `InMemoryStorageService` en las pruebas.

**Ejemplo:**
```ts
providers: [{ provide: StorageService, useClass: InMemoryStorageService }]
```

---

## 8. Store (no está en el libro)

**Qué es:** un único lugar que guarda una parte del estado de la app. Lo leen todas las pantallas y solo se modifica con métodos definidos. Viene de la arquitectura Flux de Facebook y se hizo popular con Redux y NgRx.

**Reglas** (basadas en los principios de Redux, adaptados a signals):
1. **Una sola fuente de verdad:** cada dato vive en un solo store.
2. **Solo lectura hacia afuera:** se expone el estado, pero no se deja modificar directamente.
3. **Cambios por métodos con nombre:** `start()`, `clear()`, `load()`. En Redux esto se hace con acciones y reducers.
4. **Estado derivado calculado, no guardado:** `isAuthenticated` se calcula a partir del token.

**Cómo se implementa con signals:**
- Un servicio con `providedIn: 'root'`.
- Un `signal()` **privado** con el estado.
- `asReadonly()` o `computed()` como la parte pública.
- Métodos públicos para cambiar el estado.

**En la app:** `SessionStore` (token y usuario) y `ProductsStore` (los productos cargados, que usa la pantalla de detalle).

**Ejemplo:**
```ts
@Injectable({ providedIn: 'root' })
export class SessionStore {
  private readonly tokensState = signal<AuthTokens | null>(null);
  readonly accessToken = computed(() => this.tokensState()?.accessToken ?? null);
  readonly isAuthenticated = computed(() => this.accessToken() !== null);

  start(tokens: AuthTokens): void { this.tokensState.set(tokens); }
  clear(): void { this.tokensState.set(null); }
}
```

**Cuándo usar una librería:** para una app pequeña basta con un servicio con signals, como aquí. En apps grandes se usa **NgRx SignalStore** (`signalStore`, `withState`, `withComputed`, `patchState`) o NgRx Store con acciones y reducers, que agregan herramientas de depuración y más estructura.

---

## 9. Data Mapper (no está en el libro)

**Qué es:** una capa que convierte los datos tal como llegan de afuera (API o base de datos) en los objetos de la app, para que el modelo no dependa del formato externo. Lo nombró Martin Fowler en *Patterns of Enterprise Application Architecture*.

**Reglas:**
| Participante | Qué es |
|---|---|
| DTO | La forma exacta de los datos externos |
| Modelo de dominio | Las clases de la app, con su comportamiento |
| Mapper | Transforma uno en otro; ni el DTO ni el modelo saben cómo se hace la conversión |

**En la app:**
| Participante | Pieza |
|---|---|
| DTO | `AccountDto`, `CreditCardDto` en `api.dto.ts` |
| Modelo | `Account`, `CreditCard`, con reglas como `balances` y `lastDigits` |
| Mapper | `ProductFactory`, que además de convertir decide qué clase crear |

**Ejemplo:**
```ts
static createCreditCard(dto: CreditCardDto): CreditCard {
  return new CreditCard(dto.alias, dto.number, dto.availableAmountRD, dto.availableAmountUS, dto.isInternational);
}
```

> **Corrección a `docs/03`:** allí la conversión de DTO a modelo aparece como "Adapter / Mapper". El nombre más preciso es **Data Mapper**. El Adapter del libro envuelve un objeto para cambiar su interfaz (como `PreferencesStorageService`), no transforma datos.

---

## 10. Guard (no está en el libro)

**Qué es:** una función que el router de Angular ejecuta antes de abrir una ruta para decidir si se permite la entrada. Es un mecanismo propio de Angular, no un patrón clásico; la idea es la misma de una "cláusula de guarda": comprobar una condición al inicio y salir temprano si no se cumple.

**Reglas:**
- Una función de tipo `CanActivateFn` (también existen `CanMatchFn`, `CanDeactivateFn` y `CanActivateChildFn`).
- Devuelve `true` para dejar pasar, o un `UrlTree` para redirigir. Se recomienda redirigir con `UrlTree` y no con `router.navigate()` dentro del guard.
- Puede ser asíncrona (`Promise` u `Observable`).
- Se registra en la ruta con `canActivate: [...]`.

**En la app:**
| Guard | Qué decide |
|---|---|
| `authGuard` | Hay sesión; si no, va al login |
| `guestGuard` | No hay sesión; si la hay, va a Mis productos |
| `onboardingGuard` | Ya vio el demo; es asíncrono porque lee el almacenamiento |
| `productExistsGuard` | El producto de la URL existe en el store; si no, vuelve a Mis productos |

**Ejemplo:**
```ts
export const authGuard: CanActivateFn = () =>
  inject(SessionStore).isAuthenticated() || inject(Router).parseUrl(APP_ROUTES.login);
```

---

## Lo que no es un patrón, aunque se mencione

- **Herencia y polimorfismo** (`Product` → `Account`, `CreditCard`): son pilares de la POO (libro, p. 13). Son la base del Factory Method, pero no son un patrón por sí mismos.
- **Template Method:** podría parecer que `Product` lo usa, pero no es así. Para que lo fuera, un método de la clase base tendría que armar un algoritmo llamando pasos que las subclases redefinen. Aquí las subclases solo implementan `balances`.

## Fuentes

- Shvets, A. *Sumérgete en los patrones de diseño*, Refactoring.Guru, v2023-1.23.
- [Redux — Three Principles](https://redux.js.org/introduction/threeprinciples)
- [Decision guide: Angular state management (Telerik)](https://www.telerik.com/blogs/decision-guide-angular-state-management-signals-services-signalstore-ngrx)
- [Data mapper pattern (Wikipedia)](https://en.wikipedia.org/wiki/Data_mapper_pattern)
- [Angular — Route guards](https://angular.dev/guide/routing/route-guards)
