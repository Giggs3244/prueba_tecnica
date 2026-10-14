# 13 — Cierre y guion para la entrevista

Fecha: 2026-10-13 · Fase 12 del plan

## Qué se hizo en esta fase

| Archivo | Cambio |
|---|---|
| `demo-bhd-app/README.md` | Nuevo. Puesta en marcha, comandos, estructura, decisiones y limitaciones |
| `package.json` | Autor y descripción propios; scripts `test:ci`, `test:coverage` y `android:sync`; dependencia de desarrollo `@vitest/coverage-v8` |
| `src/assets/icons/` | Eliminados 8 iconos sin uso (las variantes verdes de pestañas y menú, sustituidas por la máscara CSS) |
| `.browserslistrc` | Eliminado. Era el de la plantilla de Ionic y provocaba un aviso de navegadores no admitidos en cada build; ahora se usa la lista por defecto de Angular |
| `src/test-setup.ts` | Quitados los comentarios de la plantilla |

## Estado final

| Comprobación | Resultado |
|---|---|
| `npm run build` | Sin errores ni advertencias |
| `npm run test:ci` | 165 pruebas en 26 archivos, todas en verde |
| `npm run lint` | Sin errores |
| Cobertura | 98.9 % sentencias · 98.2 % ramas · 96.4 % funciones · 99.6 % líneas |
| Tamaño inicial del build | 792 kB sin comprimir, 182 kB transferidos |
| APK de depuración | Compila (4.42 MB) |

Las pruebas del menú y del componente raíz escriben en la salida un aviso de Ionic (`[ion-menu] - Must have a "content" element`). Es ruido del entorno de pruebas (jsdom); no aparece en el navegador y no afecta al resultado.

### Lo que no está verificado

- La app ejecutándose en Android: el emulador necesita aceleración por hardware que no está activa en la máquina. Sin eso no se han visto la pantalla de arranque, las barras del sistema, las áreas seguras, el botón atrás físico ni la fuente Roboto.
- Las animaciones fotograma a fotograma. Se comprobó que existen con los valores correctos y que el estado final es el esperado.

## Fases del plan

| # | Fase | Documento | Estado |
|---|---|---|---|
| 0 | Assets del diseño | `PLAN.md` | Hecho |
| 1 | Creación del proyecto | `01` | Hecho |
| 2 | Tema y tokens | `02` | Hecho |
| 3 | Core | `03` | Hecho |
| 4 | Componentes compartidos | `04`, `05` | Hecho |
| 5 | Login | `06` | Hecho |
| 6 | Pestañas y "Mis productos" | `07` | Hecho |
| 7 | Menú lateral | `08` | Hecho |
| 8 | Detalle de producto | `09` | Hecho |
| 9 | Onboarding | `10` | Hecho |
| 10 | Pixel perfect y transiciones | `11` | Hecho |
| 11 | Capacitor y Android | `12` | APK compila; ejecución en dispositivo pendiente |
| 12 | Cierre | `13` | Hecho |

## Guion para la entrevista

### Recorrido de demostración (unos 5 minutos)

Antes de empezar: navegador en modo móvil a 375 × 812 y, en la consola, `localStorage.clear()` para que aparezca el onboarding.

1. **Onboarding.** Avanzar con "CONTINUAR" y deslizando. Mencionar que es CSS scroll-snap, sin librería.
2. **Login con error.** Contraseña incorrecta: indicador de carga y toast rojo.
3. **Login correcto** con "Recuerda mi usuario" activado.
4. **Mis productos.** Señalar el skeleton y cómo aparece cada sección al llegar su respuesta. Recargar e iniciar sesión otra vez para que se vea el caso en que las tarjetas llegan primero.
5. **Detalle.** Tocar una tarjeta de crédito; se ve con sus dos balances. Volver.
6. **Pestañas.** Cambiar entre las cuatro.
7. **Menú.** Abrirlo, entrar a Contacto, volver, cerrar sesión.
8. **Login de nuevo.** El documento aparece relleno y la contraseña vacía.
9. **Código.** Mostrar `core/` (modelos, fábrica, interceptores) y `theme/_tokens.scss`.
10. **Pruebas.** `npm run test:coverage`.

### Lo que evalúa el enunciado y dónde está

| Criterio | Dónde mostrarlo |
|---|---|
| Estructuración del proyecto | `core/`, `shared/`, `layout/`, `features/`; una carpeta por pantalla |
| Performance | Rutas con carga diferida, `OnPush` y signals, peticiones en paralelo con pintado progresivo, iconos SVG, un solo SVG por icono, sin librerías extra |
| Buenas prácticas | TypeScript estricto, ESLint sin errores, sin valores sueltos en los estilos, guards e interceptores como funciones |
| Programación orientada a objetos | `Product` abstracta, `Account` y `CreditCard`; la regla del balance en dólares vive en `CreditCard` |
| Pixel perfection | Tokens tomados del XD; tablas de medidas en los documentos; lista de 14 diferencias aceptadas en `11` |
| Clean code | Sin comentarios, nombres que explican la intención, funciones cortas, constantes con nombre |
| Patrones de diseño | Fábrica, adaptador, cadena de responsabilidad (interceptores), store, inversión de dependencias (`StorageService`), guard |
| Pruebas unitarias | 165 pruebas, 98.9 % de cobertura |

### Preguntas probables y respuesta corta

**¿Cómo sacaste las medidas del diseño?**
El enlace de XD estaba compartido en modo Specs. Leí las coordenadas, colores y tipografías de los datos del archivo y las convertí en tokens. Luego medí cada pantalla en el navegador con `getBoundingClientRect` y comparé.

**¿Encontraste problemas en el diseño?**
Sí, varias inconsistencias: dos familias tipográficas mezcladas, separaciones distintas entre cards equivalentes, una pantalla con el texto de otra, y datos que no coinciden con la API. Están listadas con la decisión tomada en cada caso (`11`). No las corregí en silencio: las documenté.

**¿Por qué px y no rem?**
Para igualar el diseño, que está en px. La contrapartida es que el texto no escala con la configuración de accesibilidad del sistema. En un producto real lo hablaría con diseño.

**¿Cómo simulaste la API?**
Con un interceptor HTTP. Los servicios hacen peticiones reales con `HttpClient`; el interceptor las responde. Valida credenciales y token, así que los flujos de error son reales. Para usar un backend basta cambiar `useMocks` y `apiUrl`.

**¿Dónde guardas el token?**
En memoria. Capacitor Preferences no cifra, así que solo guardo ahí el documento recordado y la marca del onboarding. En producción usaría almacenamiento seguro del sistema (Keychain, Keystore).

**¿Por qué los productos no llegan todos a la vez?**
Cada tipo se pide en paralelo y se pinta en cuanto responde, con `merge` en lugar de `forkJoin`. El usuario no espera al servicio más lento, y si uno falla los demás se muestran. El orden en pantalla es fijo aunque cambie el orden de llegada.

**¿Y si hubiera diez tipos de producto?**
Se añade una entrada al registro de tipos, su clase y su caso en la fábrica. Ni el servicio ni la pantalla cambian.

**¿Cómo pasas datos entre pantallas?**
La URL lleva el identificador y el objeto sale de un store en memoria. No uso el estado de navegación porque el navegador lo serializa y se pierde la clase. Uso los últimos cuatro dígitos y no el número completo porque un número de cuenta no debe quedar en una URL. Limitación: la API no da un id propio.

**¿Signal Forms o Reactive Forms?**
Signal Forms, que es lo recomendado desde Angular 22. Construí y probé también la variante con Reactive Forms y la retiré para no mantener dos; está documentada en `05`, con el `ControlValueAccessor` que necesitaba.

**¿Qué aporta Ionic?**
Los componentes móviles y su comportamiento: pestañas con pila propia, menú con gesto, navegación con animación y caché de páginas, toast, botón atrás. Los estilos los ajusté con variables CSS y `::part`.

**¿Por qué `ionViewWillEnter`?**
Ionic conserva las páginas en memoria. `ngOnInit` solo corre una vez; lo que debe refrescarse en cada visita va en los eventos de ciclo de vida de Ionic.

**¿Qué hace Capacitor aquí?**
Empaqueta la web en un proyecto Android y da acceso a almacenamiento, botón atrás, barras del sistema y pantalla de arranque. Los plugins que uso funcionan también en el navegador, así que no hay código condicional por plataforma.

**¿Qué mejorarías con más tiempo?**
Fuente propia para igualar el diseño en Android, renovación de sesión con el refresh token, manejo visible de errores en "Mis productos", pruebas de extremo a extremo, accesibilidad con lector de pantalla, internacionalización, e icono y pantalla de arranque propios.

### Limitaciones que conviene decir antes de que las pregunten

- En Android la fuente es Roboto, no Trebuchet MS: ahí el diseño no es pixel perfect.
- El APK compila pero no se ha ejecutado en un dispositivo.
- No hay estado de error en "Mis productos"; se acordó dejarlo fuera.
- El identificador de producto se deriva del número porque la API no da uno.

## Cómo se trabajó

El proyecto se construyó por fases siguiendo `PLAN.md`, con Claude Code y las skills de Ionic, Angular y Capacitor. Cada fase terminó con build, pruebas y lint en verde, una comprobación en el navegador contra el diseño y un documento con las decisiones y los conceptos. Las decisiones de producto (carga progresiva, skeleton único, solo Signal Forms, fuente del sistema en Android) las tomó el autor.
