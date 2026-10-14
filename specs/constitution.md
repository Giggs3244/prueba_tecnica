# Constitución del proyecto — Demo BHD

Este documento reúne los principios que mandan en el proyecto. Cualquier especificación, plan o línea de código tiene que respetarlos. Si una especificación choca con la constitución, gana la constitución; si hay que cambiar un principio, se cambia aquí primero y se deja escrito el porqué.

## 1. Propósito

Construir una aplicación bancaria de demostración para la prueba técnica de frontend, a partir del diseño entregado en Adobe XD y de los cuatro servicios descritos en el enunciado.

La prueba evalúa: estructura del proyecto, rendimiento, buenas prácticas, programación orientada a objetos, fidelidad al diseño, código limpio, patrones de diseño y pruebas unitarias. Los principios de abajo salen de ahí.

## 2. Tecnologías acordadas

| Tema | Decisión |
|---|---|
| Framework | Angular 22, con componentes standalone |
| Interfaz móvil | Ionic 9, en modo `md` para todas las plataformas |
| Empaquetado nativo | Capacitor 8, plataforma Android |
| Estilos | SCSS |
| Formularios | Signal Forms |
| Estado | Signals y stores inyectables |
| Pruebas | Vitest |
| Calidad | ESLint y TypeScript en modo estricto |

No se agrega una librería nueva sin justificar qué problema resuelve que no se pueda resolver con lo que ya hay.

## 3. Principios

### P1. El diseño manda

La interfaz debe verse igual al diseño del XD. Para un banco, la pantalla es la cara frente al cliente.

- Toda medida, color y tamaño de letra sale del XD, no del ojo.
- Los valores viven en un solo archivo de tokens. En los estilos de los componentes no se escriben colores ni medidas sueltas sin nombre.
- La pantalla de referencia es de 375 × 812.
- Cada pantalla se mide en el navegador contra el XD antes de darla por terminada.
- Cuando el diseño se contradice a sí mismo o contradice a la API, no se arregla calladito: se anota la diferencia, la decisión y el motivo.

### P2. Cada cosa en su puesto

| Carpeta | Qué va ahí | Qué no va ahí |
|---|---|---|
| `core/` | Modelos, servicios, estado, interceptores, guards | Nada que pinte en pantalla |
| `shared/` | Componentes de interfaz reutilizables | Lógica de negocio, llamados a la API |
| `layout/` | Piezas de estructura que usan la sesión, como el menú | Pantallas |
| `features/` | Una carpeta por pantalla | Código que otra pantalla necesite reutilizar |

Una pantalla no conoce las URL de la API ni las claves del almacenamiento. Un componente compartido no conoce la sesión.

### P3. Orientación a objetos donde aporta

- Los datos que llegan de la API se convierten en modelos propios. La app no trabaja con la forma cruda de la respuesta.
- Las reglas de negocio viven en el modelo, no en la plantilla. Ejemplo: si una tarjeta muestra saldo en dólares lo decide la tarjeta.
- La interfaz trabaja contra la abstracción (`Product`), no contra el tipo concreto.
- Se depende de abstracciones cuando eso permite probar o cambiar una pieza sin tocar las demás.

### P4. Código limpio

- Sin comentarios en el código. Si algo necesita explicación, se mejora el nombre o se parte la función. Las explicaciones largas van en la documentación.
- Nombres que digan la intención.
- Sin números ni textos mágicos: constantes con nombre.
- Funciones cortas que hacen una sola cosa.
- El código nuevo se escribe con el mismo estilo del que ya existe.
- No se deja código muerto ni archivos sin uso.

### P5. Rendimiento desde el diseño

- Cada pantalla se carga solo cuando se necesita (rutas con carga diferida).
- Detección de cambios `OnPush` y estado con signals.
- Lo que se puede pedir en paralelo se pide en paralelo.
- El usuario no espera por lo que no necesita ver todavía: lo que va llegando se va mostrando.
- Iconos en SVG; un solo archivo por icono.

### P6. Seguridad básica de una app bancaria

- La contraseña nunca se guarda.
- El token de sesión vive solo en memoria. No se escribe en almacenamiento sin cifrar.
- En el almacenamiento del dispositivo solo van datos que no son sensibles.
- Un número de cuenta o de tarjeta completo no viaja en una URL.
- Al cerrar sesión no queda en memoria ningún dato del usuario.
- Los guards son comodidad de navegación; la seguridad de verdad la pone el servidor al exigir el token.

### P7. Lo que no está probado no está terminado

- Toda regla de negocio, servicio, interceptor y guard lleva pruebas unitarias.
- Toda pantalla lleva pruebas de sus comportamientos principales.
- Las pruebas describen lo que hace el sistema desde afuera, no cómo está armado por dentro.
- Una funcionalidad se da por terminada cuando compila sin advertencias, pasan todas las pruebas y el linter no reporta errores.

### P8. Los servicios se simulan sin que la app se entere

- El enunciado pide usar servicios simulados. La simulación no puede contaminar el código de la app.
- Los servicios de la app hacen peticiones HTTP normales a las rutas del enunciado.
- La simulación valida credenciales y token, para que los caminos de error sean de verdad.
- Pasar a un backend real debe ser un cambio de configuración, no de código.

### P9. Accesibilidad mínima

- Todo control que solo tiene icono lleva un nombre accesible.
- Los botones que solo tienen icono tienen una zona de toque de por lo menos 44 × 44, así el dibujo sea más pequeño.
- Las imágenes decorativas no las anuncia el lector de pantalla.
- Los estados de carga se anuncian.
- Se respeta la preferencia del sistema de reducir movimiento.

### P10. Decisiones por escrito

- Cada decisión que no sea obvia se documenta con su motivo, en lenguaje sencillo.
- Cuando algo no se pudo verificar, se dice claramente qué fue y por qué.
- Cuando una petición no cuadra con el plan o con las buenas prácticas, se avisa antes de hacerla.

## 4. Cómo se trabaja

1. **Especificar.** Se escribe qué debe hacer la funcionalidad y cómo se sabe que quedó bien, sin hablar todavía de código.
2. **Aclarar.** Las dudas y las contradicciones del diseño se resuelven con quien toma la decisión, y la respuesta queda en la especificación.
3. **Planear.** Se decide cómo se va a construir respetando esta constitución.
4. **Construir por partes.** Una funcionalidad a la vez, de adentro hacia afuera: primero datos y reglas, después componentes, después la pantalla.
5. **Verificar.** Compilación, pruebas, linter y comparación contra el diseño.
6. **Documentar.** Decisiones, diferencias aceptadas y lo que quedó pendiente.

## 5. Definición de terminado

Una funcionalidad está terminada cuando cumple todo esto:

- [ ] Cumple todos los criterios de aceptación de su especificación.
- [ ] Compila sin errores ni advertencias.
- [ ] Todas las pruebas pasan.
- [ ] El linter no reporta errores.
- [ ] Se midió contra el diseño a 375 × 812 y las diferencias quedaron resueltas o anotadas.
- [ ] No tiene comentarios, código muerto ni valores sueltos.
- [ ] Sus decisiones quedaron documentadas.

## 6. Lo que queda por fuera de la prueba

- Backend real.
- Renovación de la sesión con el refresh token (el enunciado no trae servicio para eso).
- Contenido real de Transacciones, Ofertas, Configuración, Contacto y Sucursales: el diseño las define como pantallas de relleno para validar la navegación.
- Versión para iOS.
- Varios idiomas.

## 7. Especificaciones

| Archivo | Tema |
|---|---|
| `spec-01-login.md` | Inicio de sesión |
| `spec-02-demo.md` | Demo de bienvenida del primer ingreso |
| `spec-03-tarjetas-de-productos.md` | Tarjetas de productos en el inicio |
| `spec-04-navegacion-general.md` | Rutas, pestañas, menú lateral y pantallas generales |
| `spec-05-componentes-compartidos.md` | Componentes de `shared` |
