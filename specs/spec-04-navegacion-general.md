# Spec 04 — Navegación general

| | |
|---|---|
| Archivo de rutas | `demo-bhd-app/src/app/app.routes.ts` |
| Diseño de referencia | Artboards `transactions`, `offers`, `configuration`, `contact`, `branches`, `product_detail` y todos los terminados en `– 1` (menú abierto) |
| Servicio | `GET /user` |
| Depende de | Spec 05 (encabezado, botón de menú, icono, botón, contenido de relleno, tarjeta de producto) |

## 1. Contexto

Esta especificación cubre todo lo que le permite al usuario moverse por la app: el mapa de rutas, quién puede entrar a cada una, la barra de pestañas, el menú lateral, las pantallas de relleno y el detalle de producto.

El diseño deja claro que varias pantallas existen solo para validar cómo se arma la navegación y cómo se pasan datos de una pantalla a otra; su contenido es un texto de relleno.

## 2. Mapa de rutas

| Ruta | Pantalla | Quién entra | Si no cumple |
|---|---|---|---|
| `/` | — | Todos | Siempre redirige a `/login` |
| `/onboarding` | Demo de bienvenida (Spec 02) | Solo sin sesión | Va a "Mis productos" |
| `/login` | Inicio de sesión (Spec 01) | Solo sin sesión y con el demo ya visto | Con sesión va a "Mis productos"; sin demo visto va al demo |
| `/tabs` | Contenedor de pestañas | Solo con sesión | Va al login |
| `/tabs/products` | Mis productos (Spec 03) | Hereda de `/tabs` | — |
| `/tabs/transactions` | Relleno "Transacciones" | Hereda de `/tabs` | — |
| `/tabs/offers` | Relleno "Ofertas" | Hereda de `/tabs` | — |
| `/tabs/configuration` | Relleno "Configuración" | Hereda de `/tabs` | — |
| `/product/:type/:digits` | Detalle de producto | Con sesión y con ese producto cargado | Sin sesión va al login; sin el producto va a "Mis productos" |
| `/contact` | Relleno "Contacto" | Todos | — |
| `/branches` | Relleno "Sucursales" | Todos | — |

`/tabs` sin nada más redirige a `/tabs/products`.

## 3. Historias de usuario

**HU-01. Moverme entre secciones**
Como cliente, quiero pasar de una sección a otra con la barra de abajo, sin perder de vista dónde estoy.

**HU-02. Tener un menú a la mano**
Como usuario, quiero abrir un menú lateral desde cualquier pantalla principal para ir a donde necesito.

**HU-03. Contactar al banco sin ingresar**
Como persona que todavía no ha iniciado sesión, quiero llegar a Contacto y a Sucursales desde el login.

**HU-04. Ver el detalle de un producto**
Como cliente, quiero abrir un producto y ver ese mismo producto en la pantalla de detalle.

**HU-05. Salir**
Como cliente, quiero cerrar sesión y que nadie más pueda ver mis datos en ese celular.

**HU-06. Que mis datos estén protegidos**
Como cliente, quiero que nadie entre a mis productos escribiendo una dirección a mano.

## 4. Requisitos funcionales

### 4.1 Control de acceso

| ID | Requisito |
|---|---|
| RF-01 | Sin sesión, cualquier intento de abrir una ruta de `/tabs` o el detalle de un producto termina en el login. |
| RF-02 | Con sesión, cualquier intento de abrir el login o el demo termina en "Mis productos". |
| RF-03 | El control de acceso de `/tabs` se define una sola vez y cubre todas sus pantallas hijas. |
| RF-04 | Si un servicio protegido responde 401, se cierra la sesión y se manda al usuario al login. Un 401 del servicio de login no cierra nada. |
| RF-05 | Todas las pantallas se cargan de forma diferida. |

### 4.2 Barra de pestañas

| ID | Requisito |
|---|---|
| RF-06 | Las pantallas de `/tabs` muestran abajo una barra fija con cuatro pestañas, en este orden: Mis productos, Transacciones, Ofertas, Configuración. |
| RF-07 | Cada pestaña tiene un icono y un texto. |
| RF-08 | La pestaña de la pantalla actual va en verde; las demás en gris. |
| RF-09 | Tocar una pestaña muestra su pantalla con un fundido de 0.3 segundos. |
| RF-10 | Cada pestaña conserva su estado al cambiar a otra y volver. |
| RF-11 | Las cuatro pestañas tienen el mismo ancho. |

### 4.3 Encabezado de las pantallas

| ID | Requisito |
|---|---|
| RF-12 | Las pantallas de `/tabs` llevan encabezado con botón de menú y el nombre de la sección. |
| RF-13 | Detalle de producto, Contacto y Sucursales llevan encabezado con flecha para volver y el nombre de la pantalla. |
| RF-14 | El login muestra solo el botón de menú, sin barra de encabezado. |
| RF-15 | El demo no lleva encabezado ni botón de menú. |

### 4.4 Menú lateral

| ID | Requisito |
|---|---|
| RF-16 | El menú se abre con el botón de menú o deslizando el dedo desde el borde izquierdo. |
| RF-17 | Se abre por encima del contenido, que queda oscurecido. |
| RF-18 | Se cierra al tocar por fuera, al deslizarlo de vuelta o al escoger una opción. |
| RF-19 | **Sin sesión** muestra: el logo del banco, y las opciones Contacto y Sucursales. |
| RF-20 | **Con sesión** muestra: la foto y el nombre completo del usuario; las opciones Mis productos, Transacciones, Ofertas y Configuración; una línea divisoria; las opciones Contacto y Sucursales; y el botón "CERRAR SESIÓN" al fondo. |
| RF-21 | Cada opción tiene un icono verde y un texto. |
| RF-22 | Mientras llega la foto del usuario se muestra un círculo gris. |
| RF-23 | Las opciones de sección llevan a su pestaña sin apilar pantallas. |
| RF-24 | Contacto y Sucursales se abren con transición de avance y permiten volver. |
| RF-25 | El contenido del menú cambia solo al iniciar o cerrar sesión, sin recargar la app. |
| RF-26 | Las opciones del menú y las pestañas salen de una misma lista, para que no se desincronicen. |

### 4.5 Datos del usuario

| ID | Requisito |
|---|---|
| RF-27 | Al entrar a la zona de pestañas se piden los datos del usuario a `GET /user`, con el token de la sesión. |
| RF-28 | Se piden una sola vez por sesión, al mismo tiempo que los productos, sin retrasarlos. |
| RF-29 | El nombre que se muestra es el nombre y el apellido que entrega el servicio. |
| RF-30 | Si la petición falla, la app sigue funcionando y el menú queda con el círculo gris y sin nombre. |

### 4.6 Cierre de sesión

| ID | Requisito |
|---|---|
| RF-31 | "CERRAR SESIÓN" cierra el menú, borra la sesión y lleva al login. |
| RF-32 | Primero se cierra el menú y después se borra la sesión, para que no se vea el cambio de contenido mientras se esconde. |
| RF-33 | Al cerrar sesión se borran de la memoria el token, los datos del usuario y los productos. |
| RF-34 | Después de cerrar sesión no se puede volver a una pantalla privada con el botón atrás. |
| RF-35 | El documento recordado no se borra al cerrar sesión. |

### 4.7 Pantallas de relleno

| ID | Requisito |
|---|---|
| RF-36 | Transacciones, Ofertas, Configuración, Contacto y Sucursales muestran un título "Pantalla de …" y un párrafo que explica que la pantalla solo valida la navegación. |
| RF-37 | Las cinco se resuelven con una sola pantalla que recibe sus textos desde la definición de la ruta. |
| RF-38 | Transacciones, Ofertas y Configuración muestran la barra de pestañas. Contacto y Sucursales no. |
| RF-39 | Contacto y Sucursales se pueden abrir con sesión y sin sesión. |

### 4.8 Detalle de producto

| ID | Requisito |
|---|---|
| RF-40 | Al tocar un producto en "Mis productos" se abre su detalle con transición de avance. |
| RF-41 | El detalle muestra la tarjeta del producto escogido, con el mismo nombre y los mismos saldos de la lista, y debajo el texto de relleno. |
| RF-42 | La dirección lleva solo el tipo de producto y los últimos cuatro dígitos. |
| RF-43 | El detalle no vuelve a llamar a ningún servicio: usa el producto que ya está cargado. |
| RF-44 | El detalle no muestra la barra de pestañas. |
| RF-45 | La flecha de volver regresa a "Mis productos" con transición de retroceso. |
| RF-46 | Si el producto pedido no está cargado o no existe, se redirige a "Mis productos". |

### 4.9 Transiciones

| ID | Requisito |
|---|---|
| RF-47 | Avanzar a otra pantalla: la nueva entra desde la derecha y la anterior sale hacia la izquierda, en 0.3 segundos y a ritmo parejo. |
| RF-48 | Volver: el movimiento contrario. |
| RF-49 | Del login a "Mis productos" se avanza; al cerrar sesión se retrocede. |

### 4.10 Botón atrás del celular

| ID | Requisito |
|---|---|
| RF-50 | Si hay un mensaje emergente o el menú abierto, el botón atrás lo cierra. |
| RF-51 | Si hay una pantalla anterior, vuelve a ella. |
| RF-52 | Si no hay a dónde volver (login o pestañas), cierra la app. |

## 5. Criterios de aceptación

**CA-01. Entrar sin permiso**
- Dado que no he iniciado sesión
- Cuando abro `/tabs/products` escribiendo la dirección
- Entonces termino en el login

**CA-02. Cambiar de pestaña**
- Dado que estoy en "Mis productos"
- Cuando toco "Ofertas"
- Entonces veo la pantalla de ofertas, el encabezado dice "Ofertas" y esa pestaña queda en verde

**CA-03. Menú sin sesión**
- Dado que estoy en el login
- Cuando abro el menú
- Entonces veo el logo y solo las opciones Contacto y Sucursales

**CA-04. Menú con sesión**
- Dado que inicié sesión
- Cuando abro el menú
- Entonces veo mi foto, mi nombre, las seis opciones y el botón "CERRAR SESIÓN"

**CA-05. Ir a Contacto y volver**
- Dado que tengo el menú abierto
- Cuando toco "Contacto"
- Entonces el menú se cierra y veo la pantalla de contacto con flecha de volver
- Y cuando toco la flecha vuelvo a donde estaba

**CA-06. Detalle del producto correcto**
- Dado que veo mis productos
- Cuando toco "Cuenta de ahorros 2 - 0002"
- Entonces el detalle muestra "Cuenta de ahorros 2 - 0002" con su saldo

**CA-07. Detalle que no existe**
- Dado que tengo sesión
- Cuando abro `/product/AC/9999`
- Entonces termino en "Mis productos"

**CA-08. Cerrar sesión**
- Dado que tengo el menú abierto con sesión
- Cuando toco "CERRAR SESIÓN"
- Entonces veo el login, el menú pasa a su versión sin sesión y no puedo volver atrás a mis productos

**CA-09. Sesión vencida**
- Dado que tengo sesión
- Cuando un servicio protegido responde 401
- Entonces la sesión se cierra y termino en el login

**CA-10. Salir con el botón atrás**
- Dado que estoy en "Mis productos" en un celular Android
- Cuando oprimo el botón atrás
- Entonces la app se cierra

## 6. Requisitos de diseño

Medidas a 375 × 812, en píxeles.

### Barra de pestañas

| Elemento | Medida |
|---|---|
| Barra | (0, 732), 375 × 80, blanca, con línea verde de 4 arriba |
| Icono | 30 × 30, a 15 del borde superior de la barra |
| Texto | 10/11, a 6 del icono |
| Colores | Activa verde `#49B742`; inactiva gris `#666666` |

### Menú lateral

| Elemento | Medida |
|---|---|
| Panel | 269 de ancho, alto completo, blanco, sin sombra |
| Fondo oscurecido | Negro al 16 % |
| Línea verde | y = 62, alto 4 |
| Logo (sin sesión) | (60, 12), 149 × 46 |
| Foto (con sesión) | (24, 12), 42 × 42, redonda |
| Nombre | x = 82, alineado al centro de la foto, 14/16 |
| Opciones | Icono de 24 en x = 24; texto en x = 64, 12/14; una cada 48 |
| Primera opción | y = 106 |
| Línea divisoria | y = 314, alto 1 |
| Contacto (con sesión) | y = 354 |
| Botón cerrar sesión | (16, 744), 237 × 44, borde verde, texto verde en negrilla |

### Pantallas de relleno

| Elemento | Medida |
|---|---|
| Título | y = 261, 24/28, gris `#CCCCCC`, centrado, en dos renglones |
| Párrafo | 30 debajo del título, 294 de ancho, 14/16, gris `#CCCCCC`, centrado |

### Detalle de producto

| Elemento | Medida |
|---|---|
| Tarjeta | (24, 90), 326 de ancho |
| Título de relleno | 100 debajo de la tarjeta |
| Párrafo | 30 debajo del título |

## 7. Requisitos no funcionales

| ID | Requisito |
|---|---|
| RNF-01 | El número completo de una cuenta o tarjeta de crédito no aparece en ninguna dirección. |
| RNF-02 | El estado de la sesión vive en un solo lugar; el menú, los controles de acceso y las peticiones lo leen de ahí. |
| RNF-03 | El token se agrega a las peticiones en un solo punto, no en cada servicio. |
| RNF-04 | La petición de login no lleva token. |
| RNF-05 | El botón de menú y la flecha de volver tienen nombre accesible y zona de toque de 48 × 48. |
| RNF-06 | El fundido de las pestañas se desactiva si el sistema pide reducir movimiento. |
| RNF-07 | En celulares con muesca o barra de gestos, encabezado, pestañas y menú respetan las zonas seguras. |

## 8. Casos borde

| Caso | Comportamiento esperado |
|---|---|
| Se abre el detalle antes de que carguen los productos | Redirección a "Mis productos" |
| Se recarga el navegador con sesión | La sesión se pierde (vive en memoria) y se vuelve al login |
| Falla `GET /user` | El menú queda sin foto ni nombre; el resto funciona |
| Se toca la opción del menú de la sección donde ya se está | El menú se cierra y se sigue en la misma pantalla |
| Se abre una dirección que no existe | **Sin definir.** Hoy no hay una ruta comodín; ver sección 10 |

## 9. Fuera de alcance

- Contenido real de Transacciones, Ofertas, Configuración, Contacto y Sucursales.
- Información adicional en el detalle de producto.
- Enlaces profundos desde fuera de la app.
- Cierre de sesión por inactividad.
- Renovar la sesión con el refresh token.

## 10. Decisiones tomadas al aclarar

| Duda | Decisión |
|---|---|
| ¿Cinco pantallas de relleno o una? | Una sola, con los textos definidos en la ruta |
| ¿El detalle va dentro de las pestañas? | No; el diseño lo muestra sin la barra |
| ¿Cómo se pasan los datos al detalle? | La dirección lleva el identificador y el producto sale de la memoria |
| ¿Qué identificador va en la dirección? | Tipo y últimos cuatro dígitos; la API no entrega uno propio |
| El diseño de Sucursales dice "Pantalla de transacciones" | Es un error del diseño; se escribe "Pantalla de sucursales" |
| El diseño muestra "Juan Pérez" y la API responde "José Pérez" | Se muestra el dato de la API |
| En el menú con sesión, Sucursales está a 52 de Contacto; sin sesión, a 48 | Se usa 48 en los dos |
| El prototipo abre el menú con un fundido | Se deja el deslizamiento del panel, que acompaña el dedo |
| El prototipo cierra sesión con un fundido | Se usa la transición de retroceso, para tener un solo tipo de movimiento |
| Los iconos de las pestañas no están repartidos parejo en el diseño | Se reparten parejo; quedan a 1 px o menos del diseño |

**Pendiente por decidir:** qué mostrar cuando se abre una dirección que no existe. Lo usual es una ruta comodín que redirija al inicio.

## 11. Cómo se verifica

- Pruebas de los controles de acceso: cada uno permitiendo y redirigiendo.
- Pruebas de navegación con las rutas reales: sin sesión, pestaña por defecto, las cuatro pestañas, las pantallas de relleno, Contacto y Sucursales, detalle con y sin producto.
- Pruebas del menú: versión sin sesión, versión con sesión, opciones con su destino y cierre de sesión en el orden correcto.
- Pruebas de la transición entre pantallas: duración, ritmo y dirección.
- Pruebas del botón atrás: cierra la app solo cuando no hay a dónde volver.
- Medición en el navegador de las tablas de la sección 6.
