# 11 — Repaso de pixel perfect y transiciones

Fecha: 2026-10-13 · Fase 10 del plan

## Qué se hizo

1. Recorrido visual de todas las pantallas a 375 × 812 con el navegador dibujando con normalidad, para cerrar lo que en fases anteriores no se pudo comprobar.
2. Transiciones entre páginas iguales a las del prototipo.
3. Fundido al cambiar de pestaña.
4. Lista única de las diferencias aceptadas frente al diseño.

| Archivo | Cambio |
|---|---|
| `src/app/shared/animations/slide-page-transition.ts` | Nuevo. Transición de página: deslizamiento horizontal de 0.3 s |
| `src/app/app.config.ts` | `navAnimation: slidePageTransition` en la configuración de Ionic |
| `src/global.scss` | Fundido de 0.3 s al mostrar una pestaña |
| `login.page.ts` | La entrada a "Mis productos" se anima hacia delante |
| `side-menu.component.ts` | La vuelta al login al cerrar sesión se anima hacia atrás |

## Transiciones

| Acción | Prototipo (XD) | Antes | Ahora |
|---|---|---|---|
| Card → detalle | Deslizar a la izquierda, 0.3 s | Transición `md` de Ionic (aparece desde abajo con fundido) | Deslizar a la izquierda, 0.3 s lineal |
| Detalle → atrás | Deslizar a la derecha, 0.3 s | Transición `md` inversa | Deslizar a la derecha, 0.3 s lineal |
| Menú → Contacto / Sucursales | Deslizar a la izquierda | Transición `md` | Deslizar a la izquierda |
| Onboarding → login | Deslizar a la izquierda | Transición `md` | Deslizar a la izquierda |
| Login → Mis productos | Deslizar a la izquierda | Sin animación | Deslizar a la izquierda |
| Cerrar sesión → login | Fundido | Sin animación | Deslizar a la derecha |
| Cambio de pestaña | Fundido, 0.3 s | Sin animación | Fundido, 0.3 s lineal |
| Abrir menú | Fundido | Panel deslizante de Ionic | Panel deslizante de Ionic |

### Decisiones

| # | Decisión | Motivo |
|---|---|---|
| 1 | Transición de página propia, registrada como `navAnimation` | El modo `md` se fijó para que los componentes se vean igual en todas las plataformas, pero su transición de página no es la del prototipo. Ionic permite sustituirla por una función; así se aplica a toda la navegación sin tocar ninguna página |
| 2 | 300 ms y ritmo lineal | Son los valores del prototipo |
| 3 | Fundido de pestañas con una animación CSS | Ionic muestra y oculta las páginas de pestaña sin animar. Cuando un elemento pasa de oculto a visible, el navegador reinicia sus animaciones CSS, así que basta una regla global |
| 4 | El menú conserva la animación de Ionic | Un panel lateral que se desliza es el comportamiento que el usuario espera y sigue el gesto del dedo. Un fundido no puede acompañar el arrastre |
| 5 | Cerrar sesión desliza hacia atrás en lugar de fundir | Volver al login es retroceder en la app; se usó la misma transición del resto para no tener dos lenguajes de movimiento |
| 6 | Sin animación con `prefers-reduced-motion` en el fundido de pestañas y el spinner | Respeta la preferencia de accesibilidad del sistema |

## Comprobaciones que estaban pendientes

En las fases 5 a 9 el panel de vista previa estuvo oculto y no dibujaba fotogramas. En esta fase sí, y se comprobó:

| Pendiente | Resultado |
|---|---|
| Onboarding: avance con "CONTINUAR" y desplazamiento entre diapositivas | Correcto; el punto activo sigue a la diapositiva |
| Onboarding → login | Correcto; el menú vuelve a quedar habilitado |
| Hamburguesa del login abre el menú público | Correcto; panel de 269 px |
| Oscurecimiento del fondo del menú | Negro al 64% con opacidad 0.25, equivalente al 16% del diseño |
| Tocar fuera cierra el menú | Correcto |
| Toast de error: aparición y cierre automático | Aparece en (10, 12) 356 × 61 y se cierra solo a los 3.1 s |
| Los campos conservan su contenido tras un error | Correcto |
| Indicador de carga al iniciar sesión | Correcto; caja blanca centrada con el spinner |
| Skeleton único al entrar a "Mis productos" | Correcto |
| Pestaña activa en verde, las demás en gris | Correcto |
| Menú privado: foto, nombre, seis enlaces, botón de salida | Correcto |
| Usuario recordado al volver al login | El documento aparece relleno, el toggle encendido y la contraseña vacía |
| Card → detalle y vuelta | Correcto; la página de detalle se retira al volver |

Las transiciones se comprobaron leyendo las animaciones activas en el navegador:

| Momento | Animación observada |
|---|---|
| Entrar al detalle | Página nueva `translateX(100%) → 0`; página anterior `0 → translateX(-100%)`; 300 ms, lineal |
| Volver | Página que regresa `translateX(-100%) → 0`; detalle `0 → translateX(100%)` |
| Cambiar de pestaña | `tab-dissolve`: opacidad `0 → 1`, 300 ms, lineal |

No se pudo medir fotograma a fotograma: el panel dejó de dibujar de forma intermitente durante la prueba. Lo que se verificó es que las animaciones existen con los valores correctos y que el estado final es el esperado. Conviene mirarlas una vez a ojo.

## Diferencias aceptadas frente al diseño

Lista completa, reunida de todas las fases. Todo lo demás coincide con el XD con menos de 1 px de diferencia.

| # | Elemento | Diseño (XD) | App | Motivo |
|---|---|---|---|---|
| 1 | Tipografía de títulos de header, toast y un "Balance disponible" | Helvetica Neue Medium | Trebuchet MS | El XD mezcla dos familias; se unificó en la que usa el resto del diseño |
| 2 | Separación entre tarjetas de crédito | 22 px | 16 px | Igual que entre cuentas |
| 3 | Segunda tarjeta de crédito | y = 638, alto 122 | y = 632, alto 128 | Consecuencia de 2 y de colocar el icono como en las demás cards |
| 4 | Alias de la primera cuenta | x = 98 | x = 96 | El resto de cards usa 96 |
| 5 | Sucursales en el menú privado | y = 406 | y = 402 | Separación de 48 como en el menú público |
| 6 | Texto de la pantalla Sucursales | "Pantalla de transacciones" | "Pantalla de sucursales" | Errata del diseño |
| 7 | Nombre en el menú | "Juan Pérez" | "José Pérez" | Dato de la API |
| 8 | Montos | 1,000,000 · 2,000,000 · 100,000 · 200,000 | 100,000 · 200,000 · 1,000,000 · 2,000,000 | Datos de la API |
| 9 | Contraseña escrita | Asteriscos `****` | Puntos del campo de contraseña | Comportamiento nativo del navegador |
| 10 | Iconos de pestaña | x = 33, 126, 220, 314 | x = 31.9, 125.6, 219.4, 313.1 | Pestañas de ancho igual |
| 11 | Avatar del menú | Círculo gris | Foto del usuario | Se usa `photo` de la API; gris mientras carga |
| 12 | Alias de la card y etiqueta del toggle | — | 1 px más arriba | Centrado vertical con su icono |
| 13 | Animación del menú | Fundido | Deslizamiento | Decisión 4 de esta fase |
| 14 | Estado de carga de "Mis productos" | No existe en el diseño | Un skeleton con forma de card | Definido con el usuario |

## Cómo se midió

1. Viewport de 375 × 812, modo `md`, tema claro.
2. Posición y tamaño de cada elemento con `getBoundingClientRect()`, comparados con las coordenadas del XD.
3. Colores, fuentes y radios con `getComputedStyle()`.
4. Anchos de texto con `Range.getBoundingClientRect()`, que mide el texto y no su caja.
5. Capturas de pantalla para la comparación visual.

Las tablas de medidas de cada pantalla están en los documentos 04, 06, 07, 08, 09 y 10.

## Conceptos

### `navAnimation`

Opción de configuración de Ionic que sustituye la transición entre páginas de todos los `ion-router-outlet`. Recibe una función que crea la animación a partir de la página que entra, la que sale y la dirección (`forward` o `back`).

### `createAnimation`

Utilidad de Ionic sobre la Web Animations API del navegador. Permite describir una animación por pasos (`fromTo`), añadir o quitar clases antes y después, y agrupar varias animaciones para que corran sincronizadas.

```ts
createAnimation()
  .addElement(options.enteringEl)
  .beforeRemoveClass('ion-page-invisible')
  .fromTo('transform', 'translateX(100%)', 'translateX(0)')
  .afterClearStyles(['transform']);
```

Ionic crea cada página nueva con la clase `ion-page-invisible` para que no parpadee antes de animarse; la animación debe quitarla. Al terminar se limpia el `transform` para no dejar estilos en línea en la página.

### Animaciones CSS y `display`

Una animación CSS empieza cuando el elemento empieza a pintarse. Ionic oculta las pestañas inactivas con `display: none`; al volver a mostrarlas, la animación corre de nuevo.

### Dirección de navegación en Ionic

`NavController` y `routerDirection` indican si una navegación es `forward`, `back` o `root`. La dirección decide qué animación se usa y qué pasa con la pila de páginas. Una navegación `root` no anima salvo que se pida con `animated: true` y `animationDirection`.

## Pruebas

162 pruebas en 26 archivos, todas en verde (4 nuevas).

| Archivo | Qué comprueba |
|---|---|
| `slide-page-transition.spec.ts` | Duración de 300 ms y ritmo lineal; al avanzar la página nueva entra por la derecha y la anterior sale por la izquierda; al retroceder, al revés; sin página anterior solo anima la que entra |
| `login.page.spec.ts`, `side-menu.component.spec.ts` | Las navegaciones raíz piden animación en la dirección correcta |

| Comando | Resultado |
|---|---|
| `ng build` | Sin advertencias |
| `ng test --watch=false` | 162 / 162 |
| `ng lint` | Sin errores |

## Pendiente fuera de esta fase

- Fuente en Android: Trebuchet MS no viene instalada. Hoy la app cae a la siguiente fuente de la pila. Hay que decidirlo antes de mostrarla en un dispositivo Android (fase 11).
