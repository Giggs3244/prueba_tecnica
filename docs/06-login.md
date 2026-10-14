# 06 — Login

Fecha: 2026-10-13 · Fase 5 del plan

## Qué se hizo

Primera pantalla real de la aplicación: `/login`, con sus cuatro estados del diseño (`login`, `login – 2`, `login – 3`, `login – 4`).

| Archivo | Cambio |
|---|---|
| `src/app/features/auth/login/login.page.{ts,html,scss}` | Nueva página de login |
| `src/app/features/auth/login/login.page.spec.ts` | 12 pruebas |
| `src/app/shared/components/menu-button/` | Nuevo `app-menu-button`: el botón hamburguesa, extraído del header para usarlo también en el login |
| `src/app/shared/components/header/` | Ahora usa `app-menu-button` |
| `src/app/app.routes.ts` | Rutas `login` y `tabs/products` con sus guards; la raíz redirige a `login` |
| `src/theme/_tokens.scss` | Nuevos `$touch-target`, `$menu-button-offset`, `$content-inset-end` |

## Rutas actuales

| Ruta | Guard | Página |
|---|---|---|
| `/` | — | Redirige a `/login` |
| `/login` | `guestGuard` (con sesión, redirige a productos) | `LoginPage` |
| `/tabs/products` | `authGuard` (sin sesión, redirige a login) | Galería temporal de componentes, hasta la fase 6 |

## Flujo

1. Al entrar a la página se lee el usuario recordado. Si existe, se rellena el campo y se enciende el toggle. La contraseña siempre empieza vacía.
2. Al enviar el formulario:
   - Si falta el documento o la contraseña, se muestra el toast de error sin llamar a la API.
   - Si está completo, aparece el indicador de carga y se llama a `POST /sign_in`.
3. Respuesta correcta: se guarda o se olvida el usuario según el toggle y se navega a `/tabs/products` como nueva raíz (no se puede volver al login con "atrás").
4. Respuesta 401: toast "Usuario y/o contraseña incorrectos". Cualquier otro fallo: "No fue posible iniciar sesión. Intenta de nuevo".
5. Un segundo envío mientras el primero está en curso se ignora.

Credenciales del mock: `00100010321` / `1111`.

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | Formulario con **Signal Forms** y validación `required` en documento y contraseña | Decisión del usuario (ver `05`). El diseño no muestra mensajes por campo, así que la validación solo evita una llamada innecesaria |
| 2 | Formulario incompleto muestra el mismo toast que unas credenciales erróneas | Es el único estado de error que define el diseño (`login – 4`) |
| 3 | Lectura del usuario recordado en `ionViewWillEnter`, no en `ngOnInit` | Ionic conserva las páginas en memoria. Al cerrar sesión y volver al login, `ngOnInit` no se ejecuta otra vez y la contraseña anterior seguiría escrita. `ionViewWillEnter` se ejecuta en cada visita |
| 4 | Navegación con `NavController.navigateRoot` | Reemplaza la pila de navegación: tras iniciar sesión, el botón atrás no regresa al login |
| 5 | Toast con `ion-toast` declarado en la plantilla (`[isOpen]`) | Se controla con un signal, igual que el resto del estado de la página, y sus estilos viven en el SCSS de la página |
| 6 | El usuario recordado se guarda solo tras un inicio de sesión correcto | Un documento mal escrito no queda guardado |
| 7 | Nunca se guarda la contraseña | Solo el documento, que no es una credencial por sí solo |
| 8 | Fondo con `background-size: cover` y foco horizontal en 77.35% | Reproduce el encuadre exacto del XD a 375 × 812 (imagen desplazada 181 px a la izquierda) y se adapta a otros tamaños de pantalla sin deformarse |
| 9 | `ion-content` sin `fullscreen` | Con `fullscreen` Ionic desplazaba el contenido 1 px. Esa opción es para encabezados translúcidos, y esta pantalla no tiene encabezado |
| 10 | Dos ajustes de 1 a 2 px para igualar el XD: el formulario termina 1 px antes del margen derecho y el toggle va 1.5 px a la izquierda del centro | En el diseño el contenido mide 326 (deja 25 a la derecha) y el grupo del toggle no está centrado exactamente. Se replicó tal cual y quedó como token y variable con nombre |
| 11 | `app-menu-button` como componente propio | El login muestra la hamburguesa sin encabezado; así header y login comparten el mismo botón |
| 12 | La hamburguesa del login aún no abre nada | El menú lateral es la fase 7 |
| 13 | Posiciones superiores sumadas a `--ion-safe-area-top` | En teléfonos con muesca, el botón de menú y el logo bajan lo necesario |

## Verificación contra el diseño

Medido en el navegador a 375 × 812.

| Elemento | Diseño (XD) | Medido |
|---|---|---|
| Hamburguesa | (24, 23) 25 × 19 | (24, 23) 25 × 19 |
| Logo | (80, 120) 215 × 66 | (80, 120) 215 × 66 |
| Campo documento | (24, 263) 326 × 44 | (24, 263) 326 × 44 |
| Campo contraseña | (24, 331) 326 × 44 | (24, 331) 326 × 44 |
| Toggle | (107, 399) 40 × 24 | (106.92, 399) 40 × 24 |
| Etiqueta del toggle | (155, 405) | (154.92, 404) |
| Botón | (24, 447) 326 × 44 | (24, 447) 326 × 44 |
| Fondo | Cubre 375 × 812 | Cubre 375 × 812, `cover`, foco 77.35% |
| Toast | (10, 12) 356 × 61, `#C92340`, radio 4 | (10, 12) 356.2 × 61, `rgb(201, 35, 64)`, radio 4 |
| Texto del toast | 14 px, blanco, centrado | 14/16, blanco, centrado |
| Indicador de carga | Caja 103 × 109 centrada | Verificado en la fase 4 |

Diferencias aceptadas:

- Etiqueta del toggle 1 px más arriba (se centra con la pista).
- Texto del toast en Trebuchet MS, no Helvetica Neue (inconsistencia 1 del plan).

Comprobado en el navegador:

- Credenciales incorrectas: aparece el indicador de carga, luego el toast, y se permanece en `/login`.
- Credenciales correctas con el toggle activo: se navega a `/tabs/products` y el documento queda guardado.
- Entrar a `/tabs/products` sin sesión redirige a `/login`.

**No verificado en el navegador:** las animaciones (transición entre páginas, entrada y cierre automático del toast, giro del spinner) y el relleno del usuario recordado al entrar. El panel de vista previa estaba oculto y no dibujaba fotogramas, y Ionic espera a la transición para ejecutar `ionViewWillEnter`. La lógica sí está cubierta por pruebas unitarias; conviene revisarlo a ojo con la app abierta.

## Conceptos

### Ciclo de vida de una página Ionic

`ion-router-outlet` no destruye la página anterior al navegar hacia adelante: la deja en memoria para poder volver con su estado y su posición de scroll. Por eso los eventos de Angular no bastan:

| Evento | Cuándo ocurre |
|---|---|
| `ngOnInit` | Una vez, al crear la página |
| `ionViewWillEnter` | Cada vez que la página va a mostrarse |
| `ionViewDidEnter` | Cada vez, al terminar la animación de entrada |
| `ionViewWillLeave` / `ionViewDidLeave` | Cada vez que la página se oculta |
| `ngOnDestroy` | Cuando la página sale de la pila |

Lo que debe refrescarse en cada visita va en `ionViewWillEnter`.

### `NavController`

Envoltorio de Ionic sobre el router de Angular que añade dirección y animación a la navegación:

- `navigateForward(url)`: animación de avance, apila la página.
- `navigateBack(url)`: animación de retroceso.
- `navigateRoot(url)`: reemplaza la pila; no hay página a la que volver.

### Overlays de Ionic

Toast, modal, alert, loading y popover son overlays: Ionic los mueve al final de `ion-app` al abrirlos para que queden encima de todo. Se pueden crear desde código (`ToastController`) o declarar en la plantilla y controlarlos con `isOpen`. El evento `didDismiss` avisa cuando se cierran.

### Signal Forms con validación

```ts
protected readonly model = signal({ userId: '', password: '', rememberUser: false });
protected readonly loginForm = form(this.model, (path) => {
  required(path.userId);
  required(path.password);
});
```

`this.loginForm().invalid()` es un signal con la validez del formulario completo. `this.model()` devuelve los valores.

### Áreas seguras

`--ion-safe-area-top` y `--ion-safe-area-bottom` son variables que Ionic rellena con el espacio que ocupan la muesca y la barra de gestos. En navegador valen 0.

## Pruebas

107 pruebas en 20 archivos, todas en verde (12 nuevas).

| Grupo | Qué comprueba |
|---|---|
| Al entrar | Formulario vacío sin usuario recordado; documento y toggle rellenados con usuario recordado; la contraseña de una visita anterior se borra |
| Formulario incompleto | Muestra el error y no llama a la API |
| Envío | Envía las credenciales y muestra el indicador de carga; ignora el doble envío; navega a productos como raíz; guarda el usuario con el toggle encendido y lo olvida con el toggle apagado; 401 muestra el error y no navega ni guarda; otro fallo muestra el mensaje genérico; cerrar el toast limpia el error |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 107 / 107 |
| `ng lint` | Sin errores |

## Pendientes

- Menú lateral público (Contacto, Sucursales) al tocar la hamburguesa: fase 7.
- Guard de onboarding en la ruta de login: fase 9.
- `/tabs/products` apunta a la galería temporal hasta la fase 6.
