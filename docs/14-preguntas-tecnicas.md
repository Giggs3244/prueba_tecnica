# 14 — Preguntas técnicas de Ionic y Angular

Respuestas cortas para la entrevista. Cuando aplica, se indica qué se hizo en este proyecto.

---

## 1. ¿Cómo agrego el tema oscuro?

Ionic ya trae una paleta oscura. Se importa en `global.scss` una de estas tres variantes:

| Archivo | Cuándo se activa |
|---|---|
| `@ionic/angular/css/palettes/dark.system.css` | Según la configuración del celular (`prefers-color-scheme`) |
| `@ionic/angular/css/palettes/dark.class.css` | Cuando el `<html>` tiene la clase `ion-palette-dark` (sirve para un botón de "modo oscuro" en la app) |
| `@ionic/angular/css/palettes/dark.always.css` | Siempre |

```scss
@use '@ionic/angular/css/palettes/dark.class.css';
```

```ts
document.documentElement.classList.toggle('ion-palette-dark', isDark);
```

Para los colores propios, se redefinen las variables de Ionic (`--ion-background-color`, `--ion-text-color`, `--ion-color-primary`, etc.) dentro de ese selector.

**En este proyecto:** los colores están en variables SCSS (`_tokens.scss`), que se resuelven al compilar y no pueden cambiar en tiempo de ejecución. Para tener tema oscuro habría que pasarlos a variables CSS (`--bhd-...`) y redefinirlas en `.ion-palette-dark`.

---

## 2. ¿A qué aplica el selector `:host`?

Al **elemento raíz del propio componente**, es decir, a la etiqueta con la que se usa (`<app-button>`, `<app-input>`). Por fuera de `:host`, los estilos del componente solo alcanzan lo que hay dentro de su plantilla.

```scss
:host { display: block; }
:host(.is-disabled) { opacity: .5; }
:host-context(.ion-palette-dark) { color: white; }
```

- `:host(.clase)`: aplica cuando el propio componente tiene esa clase.
- `:host-context(.clase)`: aplica cuando algún ancestro tiene esa clase.

Es útil porque un elemento personalizado es `inline` por defecto, y con `:host` se le define el `display`, el tamaño o el margen.

---

## 3. ¿Qué plugin uso para guardar el token de forma segura?

Uno que use el almacenamiento seguro del sistema operativo: **Keychain** en iOS y **Keystore** en Android.

| Opción | Tipo |
|---|---|
| `@aparajita/capacitor-secure-storage` | Comunitario, gratuito |
| `capacitor-secure-storage-plugin` | Comunitario, gratuito |
| Ionic Identity Vault | De pago, agrega desbloqueo con huella o rostro y cierre por inactividad |

`@capacitor/preferences` **no** sirve para esto: guarda en texto plano.

Buena práctica:
- El **access token** en memoria.
- El **refresh token** en almacenamiento seguro, solo si se quiere mantener la sesión al cerrar la app.
- En la versión web no hay almacenamiento seguro real; ahí lo correcto es una cookie `HttpOnly` que pone el servidor.

**En este proyecto:** el token está solo en memoria (`SessionStore`) y se pierde al cerrar la app. En Preferences solo se guardan el documento recordado y la marca del demo.

---

## 4. ¿Cómo implementaría el refresh token?

Con un interceptor que reacciona al **401**:

1. Llega un 401 en una petición que no es la de login ni la de refresco.
2. Se llama al servicio de refresco con el refresh token.
3. Se guarda el nuevo access token y se **repite la petición original** con él.
4. Si el refresco falla, se cierra la sesión y se manda al login.

El detalle importante es que, si llegan varios 401 al mismo tiempo, se haga **un solo refresco** y las demás peticiones lo esperen:

```ts
export const refreshInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  return next(req).pipe(
    catchError((error: HttpErrorResponse) =>
      error.status === 401 && !auth.isAuthRequest(req)
        ? auth.refresh().pipe(
            switchMap(token => next(req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }))),
          )
        : throwError(() => error),
    ),
  );
};
```

```ts
refresh(): Observable<string> {
  this.refreshing ??= this.http.post<AuthTokens>(REFRESH_URL, { refreshToken: this.session.refreshToken() }).pipe(
    map(tokens => this.session.updateTokens(tokens)),
    catchError(error => { this.signOut(); return throwError(() => error); }),
    finalize(() => (this.refreshing = undefined)),
    shareReplay(1),
  );
  return this.refreshing;
}
```

`shareReplay(1)` hace que todas las peticiones que fallaron a la vez compartan el mismo refresco. Como mejora, se puede refrescar **antes** de que venza el token, leyendo el campo `exp`.

**En este proyecto:** el `SessionStore` ya guarda el refresh token, pero la API simulada no tiene servicio de refresco. Está en la lista de mejoras.

---

## 5. ¿Cómo evito que el notch y las barras del sistema tapen el diseño?

Son las **áreas seguras** (*safe areas*).

| Pieza | Para qué |
|---|---|
| `<meta name="viewport" content="viewport-fit=cover, ...">` | Deja que la app ocupe toda la pantalla, incluida la zona del notch. Sin esto, `env()` vale 0 en iOS |
| `env(safe-area-inset-top)`, `-bottom`, `-left`, `-right` | Variables CSS del navegador con el tamaño del notch, la barra de estado y la barra de gestos |
| `--ion-safe-area-top`, `-bottom`, `-left`, `-right` | Las mismas medidas expuestas por Ionic; se pueden sobrescribir |
| `ion-header`, `ion-toolbar`, `ion-tab-bar`, `ion-content`, `ion-menu` | Ya aplican el área segura solos |
| Plugin `SystemBars` / `StatusBar` de Capacitor | Controla si la app se dibuja por debajo de las barras del sistema y su estilo (claro u oscuro) |

En Android 15 en adelante las apps se dibujan de borde a borde de forma obligatoria, así que el área segura importa también ahí, no solo en iPhone.

```scss
.footer {
  padding-bottom: calc(16px + var(--ion-safe-area-bottom, 0px));
}
```

**En este proyecto:** el `viewport-fit=cover` está en `index.html`; `capacitor.config.ts` configura `SystemBars` con `insetsHandling: 'css'`; y el login, el demo y el menú lateral usan `--ion-safe-area-top` y `--ion-safe-area-bottom` en los elementos que no son componentes de Ionic.

---

## 6. ¿Qué es el Shadow DOM y por qué lo usa Ionic?

Es un **DOM aislado dentro de un elemento**. Lo que hay adentro no se ve afectado por el CSS de afuera, y su CSS no se sale.

Ionic lo usa porque sus componentes son *web components* (hechos con Stencil) y deben funcionar igual en Angular, React, Vue o sin framework. El Shadow DOM:

- Evita que los estilos de la app dañen por accidente los componentes.
- Evita que los estilos de los componentes se filtren a la app.
- Define una forma oficial de personalizar: **variables CSS** (`--background`, `--color`) y **`::part()`**.

```scss
ion-button { --background: #49B742; }
ion-toggle::part(track) { height: 24px; }
```

Por eso un selector normal como `ion-button .button-native { }` no funciona.

No confundirlo con el encapsulamiento de Angular: por defecto Angular **emula** el aislamiento agregando atributos (`_ngcontent-xxx`) a los selectores, sin usar Shadow DOM.

**En este proyecto:** el toggle, las pestañas y el toast se ajustaron con variables CSS y `::part`.

---

## 7. ¿Qué es `as` en un `@if`?

Guarda el resultado de la condición en una **variable local de la plantilla**.

```html
@if (icon(); as iconSrc) {
  <img [src]="iconSrc" alt="" />
}
```

- `icon()` se evalúa una sola vez.
- Dentro del bloque, `iconSrc` ya no puede ser `null` ni `undefined`: TypeScript lo sabe y no hay que usar `!`.
- Es útil con signals y con el pipe `async`: `@if (user$ | async; as user)`.

Para variables que no dependen de una condición existe `@let`: `@let total = price() * qty();`.

---

## 8. Diferencia entre `@Service` y `@Injectable`

`@Service` es un decorador nuevo de Angular (llegó en la versión 22) para el caso más común: un servicio único para toda la app.

| | `@Injectable({ providedIn: 'root' })` | `@Service()` |
|---|---|---|
| Disponible para toda la app | Hay que decirlo con `providedIn` | Sí, por defecto |
| Inyección por constructor | Sí | No; solo con `inject()` |
| Fábrica propia | Con `useFactory` en un provider | Opción `factory` |
| Proveerlo a mano en un componente | Sin `providedIn` | `@Service({ autoProvided: false })` |
| Compatibilidad | Todas las versiones | Angular 22 en adelante |

En resumen, `@Service()` es lo mismo que `@Injectable({ providedIn: 'root' })`, con menos código y obligando a usar `inject()`. `@Injectable` sigue siendo válido y sirve para casos más avanzados.

**En este proyecto:** se usa `@Injectable({ providedIn: 'root' })` con `inject()`. Se podría migrar a `@Service()` sin cambiar nada más.

---

## 9. ¿Cómo manejo varios toast al mismo tiempo desde cualquier pantalla?

Con un **servicio global** (`providedIn: 'root'`) que use `ToastController` y una **cola**: los mensajes se muestran uno detrás de otro y no se tapan entre sí.

```ts
@Injectable({ providedIn: 'root' })
export class NotificationService {
  private readonly toasts = inject(ToastController);
  private queue = Promise.resolve();

  show(message: string, color: 'danger' | 'success' = 'success'): void {
    this.queue = this.queue.then(async () => {
      const toast = await this.toasts.create({ message, color, duration: 3000, position: 'top' });
      await toast.present();
      await toast.onDidDismiss();
    });
  }
}
```

- Cualquier pantalla, servicio o interceptor llama `inject(NotificationService).show('...')`.
- Si de verdad deben verse **varios a la vez**, Ionic no los apila solo: se superponen en la misma posición. Ahí la opción es un componente propio en `app.component` que pinte una lista de mensajes guardada en un signal.
- Conviene descartar mensajes repetidos para no mostrar el mismo error cinco veces.

**En este proyecto:** solo hay un toast, el de error del login, puesto directamente en su pantalla con `<ion-toast [isOpen]>`. Si se necesitaran mensajes en otras pantallas, se haría con este servicio.

---

## 10. ¿Qué uso para internacionalizar la app?

| Opción | Cómo funciona | Para una app móvil |
|---|---|---|
| **Transloco** | Cambia el idioma en tiempo de ejecución; traducciones en JSON; soporta signals y carga por pantalla | Recomendada |
| **ngx-translate** | Igual, en tiempo de ejecución con JSON | Muy usada en proyectos Ionic existentes |
| `@angular/localize` | Genera una compilación por idioma | Poco práctica: obligaría a un APK por idioma o a cambiar de idioma recargando |

Para conocer el idioma del celular se usa el plugin **`@capacitor/device`** (`Device.getLanguageCode()`). Para fechas y monedas se usan los pipes de Angular con el locale registrado (`es-DO`, `es-CO`).

---

## 11. ¿`NavController` o `Router`?

`NavController` es de Ionic y usa el `Router` de Angular por debajo. La diferencia es que **le dice a Ionic qué animación hacer y cómo manejar la pila de pantallas**.

| Método | Efecto |
|---|---|
| `navigateForward(url)` | Animación de avance; la pantalla nueva se apila |
| `navigateBack(url)` | Animación de regreso |
| `navigateRoot(url)` | Reemplaza toda la pila; no se puede volver con atrás (login, cerrar sesión) |
| `back()` | Vuelve a la pantalla anterior de la pila de Ionic |

Cuándo usar cada uno:
- **En pantallas, al navegar por una acción del usuario:** `NavController`, o en la plantilla `routerLink` con `routerDirection="forward" | "back" | "root"`.
- **En guards y lógica sin interfaz:** el `Router` de Angular, devolviendo un `UrlTree` (`router.parseUrl('/login')` o `router.createUrlTree(['/login'])`).
- Con `router.navigate()` Ionic no sabe la dirección y puede animar en el sentido equivocado.

**En este proyecto:** el login y el demo usan `navigateRoot` para que el botón atrás no devuelva; cerrar sesión también usa `navigateRoot`, con animación de regreso; los guards devuelven un `UrlTree` con `router.parseUrl`; las tarjetas y el menú usan `routerLink` con `routerDirection`.
