# 08 — Menú lateral y cierre de sesión

Fecha: 2026-10-13 · Fase 7 del plan

## Qué se hizo

Un único menú lateral que cambia de contenido según haya o no sesión, y el cierre de sesión.

| Archivo | Cambio |
|---|---|
| `src/app/layout/side-menu/side-menu.component.{ts,html,scss}` | Nuevo. El menú (`ion-menu`) |
| `src/app/app.component.{ts,html}` | El menú se declara junto al `ion-router-outlet` principal |
| `src/app/shared/components/icon/` | Nuevo `app-icon`: icono SVG coloreable por CSS. Lo usan las pestañas y el menú |
| `src/app/shared/config/main-sections.ts` | Nuevos `MAIN_SECTION_LINKS` y `SUPPORT_LINKS` |
| `src/app/features/tabs/tabs.page.ts` | Pide los datos del usuario (`GET /user`) al entrar a la zona privada |
| `src/app/app.routes.ts` | Nuevas rutas `/contact` y `/branches` |

## Contenido del menú

| | Sin sesión (desde el login) | Con sesión |
|---|---|---|
| Cabecera | Logo del banco | Foto y nombre del usuario (de `GET /user`) |
| Enlaces | Contacto, Sucursales | Mis productos, Transacciones, Ofertas, Configuración · separador · Contacto, Sucursales |
| Pie | — | Botón "CERRAR SESIÓN" |

El menú lee `isAuthenticated` y `user` de `SessionStore`. Como son signals, el contenido cambia solo al iniciar o cerrar sesión.

## Cierre de sesión

```ts
protected async signOut(): Promise<void> {
  await this.menu().close();
  this.auth.signOut();
  await this.navigation.navigateRoot(APP_ROUTES.login);
}
```

1. Se cierra el menú y se espera a que termine la animación.
2. Se borra la sesión (token y usuario, que solo viven en memoria).
3. Se navega al login como nueva raíz: no se puede volver atrás a una pantalla privada.

El orden importa: si la sesión se borrara antes de cerrar el menú, su contenido cambiaría a la versión pública mientras todavía se está ocultando.

## Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | Un solo `ion-menu` con contenido condicional | El diseño muestra el mismo panel con dos contenidos. Dos menús obligarían a activar y desactivar cada uno según la ruta |
| 2 | El componente vive en `layout/`, no en `shared/` (cambio respecto al plan) | Usa la sesión y el servicio de autenticación. `shared/` queda reservado a componentes sin lógica de negocio |
| 3 | Los datos del usuario se piden en `TabsPage` | Es el contenedor de toda la zona privada: se crea una vez al iniciar sesión y se destruye al cerrarla. No retrasa la aparición de los productos, porque la petición corre en paralelo |
| 4 | Avatar gris mientras llega la foto | Coincide con el círculo `#666666` que dibuja el diseño |
| 5 | Enlaces de sección con `routerDirection="root"`; Contacto y Sucursales con `"forward"` | Reproduce el prototipo: cambiar de sección no apila páginas; Contacto y Sucursales entran deslizando y tienen botón de retroceso |
| 6 | Cada enlace va dentro de `ion-menu-toggle` | Cierra el menú al elegir una opción, sin código |
| 7 | Filas de 48 px de alto | En el diseño el icono mide 24 y las filas están separadas 24. La fila ocupa los 48 completos para que el área táctil sea cómoda, sin mover nada de su posición |
| 8 | Separación uniforme de 48 entre Contacto y Sucursales | En el menú privado el XD separa esas dos filas 52 px y en el menú público 48. Se usó 48 en ambos; Sucursales queda 4 px más arriba que en el artboard privado |
| 9 | Rutas `/contact` y `/branches` creadas en esta fase | El menú necesita destinos reales. Reutilizan `PlaceholderPage` con botón de retroceso. No llevan guard porque se abren con y sin sesión |
| 10 | Menú cerrado mediante la referencia al `ion-menu` de la plantilla (`viewChild`) | El menú está en la propia plantilla del componente; no hace falta el controlador global |
| 11 | Fondo del overlay calculado | Ionic anima la opacidad del fondo hasta 0.25. Para obtener el 16% de negro del diseño, el color del fondo es negro al 64% (0.16 ÷ 0.25) |
| 12 | `app-icon` con máscara CSS | Un SVG por icono para cualquier tamaño y color: 30 px gris o verde en las pestañas, 24 px verde en el menú |

## Verificación contra el diseño

Medido en el navegador a 375 × 812.

| Elemento | Diseño (XD) | Medido |
|---|---|---|
| Panel | 269 × 812, blanco, sin sombra | 269 × 812, sin sombra |
| Barra verde | y = 62, alto 4 | y = 62, alto 4 |
| Logo (público) | (60, 12) 149 × 46 | (60, 12) 149 × 46 |
| Avatar (privado) | (24, 12) 42 × 42 | (24, 12) 42 × 42 |
| Nombre | x = 82, y = 26 | x = 82, centrado con el avatar |
| Iconos de sección | x = 24; y = 106, 154, 202, 250; 24 × 24, verdes | x = 24; y = 106, 154, 202, 250; 24 × 24, `rgb(73, 183, 66)` |
| Texto de enlace | 12/14 | 12/14 |
| Separador | y = 314, alto 1 | y = 314, alto 1 |
| Contacto (privado) | y = 354 | y = 354 |
| Sucursales (privado) | y = 406 | y = 402 |
| Contacto / Sucursales (público) | y = 106 / 154 | y = 106 / 154 |
| Botón cerrar sesión | (16, 744) 237 × 44 | (16, 744) 237 × 44 |
| Overlay | Negro al 16% | Negro al 64% × opacidad 0.25 de Ionic |

Diferencias aceptadas:

- Sucursales 4 px más arriba en el menú privado (decisión 8).
- El nombre mostrado es el de la API ("José Pérez"), no el del dibujo ("Juan Pérez").

Comprobado en el navegador:

- Tras iniciar sesión el menú muestra la foto, el nombre y los seis enlaces.
- "CERRAR SESIÓN" lleva a `/login` y el menú pasa a mostrar el logo y los dos enlaces públicos.
- Después de cerrar sesión, abrir `/tabs/products` redirige a `/login`.
- El enlace Contacto abre `/contact` con su título, su texto y el botón de retroceso.

**No verificado en el navegador:** la apertura del menú con la hamburguesa o deslizando, su animación, el oscurecimiento del fondo y el cierre al tocar fuera. El panel de vista previa seguía oculto y sin dibujar fotogramas; para medir el menú hubo que forzar su visibilidad. Son comportamientos propios de `ion-menu`, pero conviene revisarlos a ojo.

## Conceptos

### `ion-menu`

Panel lateral de Ionic. Se asocia al contenido principal con `contentId`, que debe coincidir con el `id` del `ion-router-outlet`. Tipos: `overlay` (se desliza por encima), `push` y `reveal`. Trae incluido el gesto de deslizar desde el borde, el fondo oscurecido y el cierre al tocar fuera.

### `ion-menu-button` e `ion-menu-toggle`

- `ion-menu-button`: botón que abre el menú. Es el que usa `app-menu-button`.
- `ion-menu-toggle`: envoltorio que abre o cierra el menú cuando se pulsa su contenido.

Ninguno necesita código: localizan el menú por sí mismos.

### `routerDirection`

Atributo de Ionic que acompaña a `routerLink` e indica la animación y qué pasa con la pila de páginas: `forward` apila, `back` desapila, `root` reemplaza la pila.

### `viewChild`

Consulta, como signal, un elemento o componente de la propia plantilla. `viewChild.required(IonMenu)` devuelve el `ion-menu` y permite llamar a sus métodos (`open()`, `close()`).

### Estado compartido con signals

`SessionStore` es el único dueño del token y del usuario. El login escribe en él, `TabsPage` le entrega el usuario, y el menú, los guards y el interceptor solo leen. Ningún componente se comunica directamente con otro.

## Pruebas

134 pruebas en 22 archivos, todas en verde (12 nuevas).

| Archivo | Qué comprueba |
|---|---|
| `side-menu.component.spec.ts` | Tipo de menú y enlace con el contenido; sin sesión: logo, dos enlaces, sin botón de salida; con sesión: avatar vacío hasta cargar el usuario, foto y nombre, seis enlaces con su destino y dirección, separador, cada enlace cierra el menú; cerrar sesión: cierra el menú con la sesión aún activa, la borra y navega al login |
| `app.component.spec.ts` | El menú apunta al `ion-router-outlet` principal |
| `tabs-navigation.spec.ts` | El usuario se pide una sola vez al entrar a las pestañas; `/contact` y `/branches` abren sin sesión, con botón de retroceso y sin pedir el usuario |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 134 / 134 |
| `ng lint` | Sin errores |

## Pendientes

- Detalle de producto y comprobación de Contacto y Sucursales contra el diseño: fase 8.
- En el onboarding (fase 9) habrá que desactivar el gesto del menú.
