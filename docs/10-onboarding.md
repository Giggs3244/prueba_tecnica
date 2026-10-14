# 10 — Onboarding de primer arranque

Fecha: 2026-10-13 · Fase 9 del plan

## Qué se hizo

Las tres pantallas de presentación del diseño (`demo_1`, `demo_2`, `demo_3`), que se muestran la primera vez que se abre la app.

| Archivo | Cambio |
|---|---|
| `src/app/features/onboarding/onboarding.page.{ts,html,scss}` | Nueva página con las tres diapositivas |
| `src/app/features/onboarding/onboarding.page.spec.ts` | 9 pruebas |
| `src/app/app.routes.ts` | Ruta `/onboarding`; `onboardingGuard` añadido a `/login` |
| `src/app/features/tabs/tabs.page.ts` | Un fallo al pedir el usuario ya no produce un error sin manejar |

## Flujo

1. La app arranca en `/`, que redirige a `/login`.
2. `onboardingGuard` consulta en el almacenamiento si el onboarding ya se vio. Si no, redirige a `/onboarding`.
3. El usuario avanza con "CONTINUAR", tocando la diapositiva o deslizando.
4. En la última diapositiva, "CONTINUAR" guarda la marca "visto" y abre el login.
5. En los siguientes arranques el guard deja pasar directamente al login.

## Rutas actuales

| Ruta | Guards | Página |
|---|---|---|
| `/onboarding` | `guestGuard` | `OnboardingPage` |
| `/login` | `guestGuard`, `onboardingGuard` | `LoginPage` |
| `/tabs/...` | `authGuard` | Pestañas |
| `/product/:type/:digits` | `authGuard`, `productExistsGuard` | Detalle |
| `/contact`, `/branches` | — | Relleno con botón atrás |

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | Se muestra una sola vez, en el primer arranque | En el prototipo solo se llega tocando el logo del login, que es un atajo del diseñador para poder mostrar las pantallas. El uso habitual de un onboarding es el primer arranque (inconsistencia 8 del plan) |
| 2 | Carrusel con **CSS scroll-snap**, sin librería | Ionic retiró su componente `ion-slides` en la versión 7 y recomienda Swiper. Para tres diapositivas fijas basta el desplazamiento nativo del navegador: el gesto de deslizar y la inercia son los del sistema y no se añade ninguna dependencia |
| 3 | La diapositiva activa se deduce de la posición de scroll | Una sola fuente de verdad: da igual si el usuario desliza, toca o pulsa el botón, los puntos indicadores siempre reflejan lo que se ve |
| 4 | Tocar la diapositiva también avanza | Así está definido en el prototipo |
| 5 | La marca "visto" se guarda al terminar, no al entrar | Si el usuario cierra la app a mitad, lo vuelve a ver completo |
| 6 | Marca guardada con Capacitor Preferences | Es una preferencia no sensible y debe sobrevivir al cierre de la app |
| 7 | Menú lateral desactivado mientras se ve el onboarding | El gesto de deslizar desde el borde izquierdo abriría el menú en lugar de retroceder de diapositiva |
| 8 | Posición vertical de cada ilustración tomada del XD por diapositiva | En el diseño las tres ilustraciones no comparten centro (difieren hasta 5 px). Se respetó la posición de cada una |
| 9 | `guestGuard` en `/onboarding` | Un usuario con sesión no debe ver la presentación |
| 10 | Navegación al login con `navigateRoot` y animación de avance | No se puede volver al onboarding con "atrás", y la transición es la del prototipo (deslizar a la izquierda) |

## Verificación contra el diseño

Medido en el navegador a 375 × 812.

| Elemento | Diseño (XD) | Medido |
|---|---|---|
| Fondo | `#F5F5F5` | `rgb(245, 245, 245)` |
| Título 1 | (110, 169) 156 × 56, 24/28, `#666` | (109.2, 169) 156.8 × 56, 24/28, `rgb(102, 102, 102)` |
| Título 2 | (51, 169) 274 × 84 | (50.7, 169) 273.8 × 84 |
| Título 3 | (72, 169) 232 × 56 | (71.6, 169) 232.1 × 56 |
| Ilustración 1 | (110.4, 363.8) 155.1 × 144.4 | (110.0, 363.8) 155.1 × 144.4 |
| Ilustración 2 | (117.7, 353.9) 139.7 × 162 | (117.8, 353.9) 139.7 × 162 |
| Ilustración 3 | (116, 364.1) 144 × 151.7 | (115.6, 364.1) 144 × 151.7 |
| Puntos | y = 706; x = 156, 180, 204; 16 × 16 | y = 706; x = 155.6, 179.6, 203.6; 16 × 16 |
| Punto activo | Verde, cambia con cada diapositiva | Verde, cambia con cada diapositiva |
| Pie | (0, 752) 375 × 60, blanco | (0, 752) 375 × 60, blanco |
| "CONTINUAR" | (266, 774) 77 × 16, Bold 14, verde | (266.7, 774) 76.5 × 16, 700 14, `rgb(73, 183, 66)` |

Todas las diferencias son inferiores a 1 px.

Comprobado en el navegador:

- Sin la marca "visto", abrir `/` lleva a `/onboarding`.
- Al pasar de diapositiva el punto activo cambia.
- "CONTINUAR" en la última guarda la marca y abre `/login`.
- En el siguiente arranque `/` lleva directamente a `/login`.

**No verificado en el navegador:** el desplazamiento suave entre diapositivas, el gesto de deslizar con el dedo y la transición hacia el login. El panel de vista previa no dibujaba fotogramas; el cambio de diapositiva se probó moviendo el scroll directamente.

## Cómo volver a ver el onboarding

La marca se guarda en el almacenamiento del navegador con la clave `CapacitorStorage.onboardingSeen`. Para repetirlo, borrarla desde las herramientas de desarrollo (Application → Local Storage) o ejecutar en la consola:

```js
localStorage.removeItem('CapacitorStorage.onboardingSeen');
```

En un dispositivo, borrar los datos de la app o reinstalarla.

## Conceptos

### CSS scroll-snap

```scss
.onboarding__track {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
}

.slide {
  flex: 0 0 100%;
  scroll-snap-align: center;
  scroll-snap-stop: always;
}
```

El contenedor se desplaza en horizontal y el navegador detiene el scroll siempre en una diapositiva completa. `scroll-snap-stop: always` impide saltarse una diapositiva con un gesto rápido.

### Guard asíncrono

`onboardingGuard` devuelve una promesa porque leer el almacenamiento es asíncrono. El router espera el resultado antes de activar la ruta.

### Varios guards en una ruta

`canActivate: [guestGuard, onboardingGuard]`. Los dos se evalúan; si alguno redirige, se aplica el del primero en el arreglo. Con sesión manda a productos; sin sesión y sin onboarding visto, al onboarding.

### `viewChild` con referencia de plantilla

`viewChild.required<ElementRef<HTMLElement>>('track')` obtiene el elemento marcado como `#track` en la plantilla, para leer su `scrollLeft` y llamar a `scrollTo`.

### Activar y desactivar el menú

`MenuController.enable(false)` deshabilita el menú y su gesto. Se llama en `ionViewWillEnter` y se revierte en `ionViewWillLeave`, los eventos de ciclo de vida de Ionic que sí se ejecutan en cada visita.

## Pruebas

158 pruebas en 25 archivos, todas en verde (10 nuevas).

| Grupo | Qué comprueba |
|---|---|
| `OnboardingPage` | Las tres diapositivas con sus textos y la primera activa; avanzar con el botón; avanzar tocando la diapositiva; seguir la diapositiva a la que desliza el usuario; al final guarda la marca y abre el login con animación de avance; desactiva el menú al entrar y lo restaura al salir |
| Primer arranque | `/` abre el onboarding la primera vez; abre el login si ya se vio; un usuario con sesión no llega al onboarding |
| Pestañas | La app sigue funcionando si falla la carga del usuario |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 158 / 158 |
| `ng lint` | Sin errores |
