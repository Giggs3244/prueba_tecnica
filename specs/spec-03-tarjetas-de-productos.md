# Spec 03 — Tarjetas de productos en el inicio

| | |
|---|---|
| Ruta | `/tabs/products` |
| Diseño de referencia | Artboard `dashboard` |
| Servicios | `GET /accounts`, `GET /credit_cards` |
| Depende de | Spec 05 (tarjeta de producto, tarjeta de carga, encabezado) y Spec 04 (pestañas, detalle) |

## 1. Contexto

Es la pantalla a la que llega el usuario después de iniciar sesión. Muestra sus productos agrupados por tipo: primero las cuentas y después las tarjetas de crédito. Cada producto se ve como una tarjeta con su nombre y su saldo disponible.

En esta especificación, "tarjeta" es el recuadro que muestra un producto en pantalla. Cuando se habla del producto bancario se dice "tarjeta de crédito".

Hoy hay dos tipos de producto. En una app real habrá más (préstamos, certificados, inversiones), y la solución tiene que aguantar ese crecimiento sin reescribirse.

## 2. Historias de usuario

**HU-01. Ver mis productos**
Como cliente, quiero ver mis cuentas y mis tarjetas de crédito con su saldo disponible apenas entro.

**HU-02. No esperar de más**
Como cliente, quiero ir viendo mis productos a medida que cargan, para no quedarme esperando a que llegue todo.

**HU-03. Saber que falta algo por cargar**
Como cliente, quiero que la app me muestre que todavía está trayendo información.

**HU-04. Ver el detalle**
Como cliente, quiero tocar un producto para ver su detalle.

## 3. Requisitos funcionales

### Carga

| ID | Requisito |
|---|---|
| RF-01 | Al entrar a la pantalla se piden las cuentas y las tarjetas de crédito al mismo tiempo, cada una a su servicio. |
| RF-02 | Las dos peticiones llevan el token de la sesión en el encabezado `Authorization`. |
| RF-03 | Cada grupo se muestra apenas llega su respuesta, sin esperar al otro. |
| RF-04 | Los productos se piden una sola vez al entrar; cambiar de pestaña y volver no los vuelve a pedir. |

### Orden

| ID | Requisito |
|---|---|
| RF-05 | El grupo de cuentas va siempre de primero y el de tarjetas de crédito de segundo. |
| RF-06 | El orden no depende de cuál servicio responda antes. |
| RF-07 | Dentro de cada grupo, los productos van en el orden en que los entrega el servicio. |

### Estado de carga

| ID | Requisito |
|---|---|
| RF-08 | Mientras falte por responder algún servicio, se muestra una sola tarjeta de carga con la misma forma de una tarjeta de producto. |
| RF-09 | Es una sola porque no se sabe cuántos tipos de producto tiene el usuario. |
| RF-10 | La tarjeta de carga se ubica en el lugar del primer grupo que todavía no ha respondido. |
| RF-11 | Si las tarjetas de crédito llegan primero, la tarjeta de carga queda arriba, en el lugar de las cuentas, y las tarjetas de crédito se pintan debajo. |
| RF-12 | La tarjeta de carga queda en la posición exacta de la primera tarjeta real, dejando libre el espacio del título, para que no haya salto cuando lleguen los datos. |
| RF-13 | Cuando ya respondieron todos los servicios, la tarjeta de carga desaparece. |
| RF-14 | Mientras un grupo carga no se muestra su título. |

### Contenido de un grupo

| ID | Requisito |
|---|---|
| RF-15 | Cada grupo tiene un título con el nombre del tipo y la cantidad entre paréntesis: `CUENTAS (2)`, `TARJETAS (2)`. |
| RF-16 | El título de cuentas va en verde y el de tarjetas de crédito en azul. |
| RF-17 | Un grupo sin productos no se muestra: ni título ni tarjetas. |

### Contenido de una tarjeta

| ID | Requisito |
|---|---|
| RF-18 | Cada tarjeta muestra el icono del tipo, el nombre del producto, el saldo disponible y el texto "Balance disponible". |
| RF-19 | El nombre es el alias seguido de los últimos cuatro dígitos del número: `Cuenta de ahorros 1 - 0001`. |
| RF-20 | Para sacar los últimos cuatro dígitos se ignoran los espacios del número. |
| RF-21 | Una cuenta muestra un saldo en pesos dominicanos (`RD$`). |
| RF-22 | Una tarjeta de crédito internacional muestra dos saldos: `RD$` y `US$`, en ese orden. |
| RF-23 | Una tarjeta de crédito que no es internacional muestra solo el saldo en `RD$`. |
| RF-24 | Los saldos llevan separador de miles con coma y dos decimales con punto: `1,000,000.00`. |
| RF-25 | Las cuentas llevan una franja verde al lado izquierdo; las tarjetas de crédito, una azul. |

### Navegación

| ID | Requisito |
|---|---|
| RF-26 | Tocar cualquier tarjeta abre el detalle de ese producto (ver Spec 04). |
| RF-27 | La dirección del detalle lleva el tipo de producto y los últimos cuatro dígitos, nunca el número completo. |

### Errores

| ID | Requisito |
|---|---|
| RF-28 | Si falla el servicio de un grupo, ese grupo no se muestra y los demás se muestran normalmente. |
| RF-29 | Un servicio que falla cuenta como que ya respondió: no deja la tarjeta de carga pegada. |
| RF-30 | No se muestra mensaje de error en esta pantalla. |
| RF-31 | Si un servicio responde 401, se cierra la sesión y se manda al usuario al login. |

### Crecimiento

| ID | Requisito |
|---|---|
| RF-32 | Agregar un tipo de producto nuevo no debe obligar a cambiar la pantalla ni la forma en que se cargan los grupos. |
| RF-33 | El orden de los grupos se define en un solo lugar. |

## 4. Criterios de aceptación

**CA-01. Recién llegado**
- Dado que acabo de iniciar sesión
- Cuando aparece "Mis productos" y todavía no ha respondido ningún servicio
- Entonces veo una sola tarjeta de carga y ningún título

**CA-02. Llegan primero las cuentas**
- Dado que estoy viendo la tarjeta de carga
- Cuando responde el servicio de cuentas
- Entonces veo `CUENTAS (2)` con sus dos tarjetas y, debajo, la tarjeta de carga

**CA-03. Llegan primero las tarjetas de crédito**
- Dado que estoy viendo la tarjeta de carga
- Cuando responde el servicio de tarjetas de crédito antes que el de cuentas
- Entonces la tarjeta de carga sigue arriba y debajo aparece `TARJETAS (2)` con sus dos tarjetas

**CA-04. Todo cargado**
- Dado que ya respondieron los dos servicios
- Entonces veo `CUENTAS (2)` y después `TARJETAS (2)`, y no hay tarjeta de carga

**CA-05. Sin salto**
- Dado que la tarjeta de carga está arriba
- Cuando llegan las cuentas
- Entonces la primera tarjeta de cuenta queda en el mismo lugar donde estaba la tarjeta de carga

**CA-06. Tarjeta de crédito internacional**
- Dado un producto con `isInternational` en verdadero
- Entonces su tarjeta muestra el saldo en `RD$` y, debajo, el saldo en `US$`

**CA-07. Tarjeta de crédito local**
- Dado un producto con `isInternational` en falso
- Entonces su tarjeta muestra solo el saldo en `RD$`, aunque el servicio mande un valor en dólares

**CA-08. Cliente sin tarjetas de crédito**
- Dado que el servicio de tarjetas de crédito responde una lista vacía
- Entonces solo veo el grupo de cuentas y no aparece el título `TARJETAS`

**CA-09. Un servicio falla**
- Dado que el servicio de cuentas falla y el de tarjetas de crédito responde bien
- Entonces veo solo `TARJETAS (2)` y no queda tarjeta de carga

**CA-10. Ir al detalle**
- Dado que veo mis productos
- Cuando toco la tarjeta "Tarjeta de crédito 1 - 0001"
- Entonces se abre el detalle con ese mismo producto y sus dos saldos

## 5. Requisitos de diseño

Medidas a 375 × 812, en píxeles.

### Pantalla

| Elemento | Medida |
|---|---|
| Fondo | Gris claro `#F5F5F5` |
| Margen lateral | 24 |
| Ancho del contenido | 326 |
| Del encabezado al primer título | 25 |
| Del título a la primera tarjeta | 16 |
| Entre tarjetas | 16 |
| Entre grupos | 33 |
| Debajo de la última tarjeta | 53 |
| Título de grupo | Negrilla 14/16; verde `#49B742` o azul `#00477A` |

El contenido tiene desplazamiento vertical; el encabezado y las pestañas se quedan fijos.

### Tarjeta

| Elemento | Medida |
|---|---|
| Ancho | 326 |
| Alto | 128 con un saldo; 156 con dos |
| Fondo y borde | Blanco; borde de 1 en `#CCCCCC` al 60 %; esquinas de 4 |
| Franja izquierda | 6 de ancho |
| Relleno interno | 16 |
| Icono | 40 × 40, gris `#666666` |
| Nombre | A 72 del borde izquierdo, centrado con el icono; 14/16 |
| Saldo | A 72 del borde superior; renglón de 28 de alto |
| Tamaños del saldo | Moneda y decimales a 14; parte entera a 24 |
| "Balance disponible" | 1 px debajo del último saldo; 10/11; gris `#999999` |

### Tarjeta de carga

Misma caja que una tarjeta con un saldo (326 × 128), con franja gris y bloques grises animados en el lugar del icono, el nombre, el saldo y la etiqueta.

## 6. Requisitos no funcionales

| ID | Requisito |
|---|---|
| RNF-01 | El tiempo para ver el primer grupo es el del servicio más rápido, no el del más lento. |
| RNF-02 | La regla de cuándo se muestra el saldo en dólares vive en el modelo de la tarjeta de crédito, no en la pantalla. |
| RNF-03 | La pantalla no conoce las direcciones de los servicios ni los tipos de producto que existen. |
| RNF-04 | Al cerrar sesión, los productos se borran de la memoria. |
| RNF-05 | El lector de pantalla anuncia que hay productos cargando. |
| RNF-06 | La pantalla se carga de forma diferida. |

## 7. Casos borde

| Caso | Comportamiento esperado |
|---|---|
| El cliente no tiene ningún producto | Pantalla sin grupos y sin tarjeta de carga |
| Fallan los dos servicios | Pantalla sin grupos y sin tarjeta de carga |
| El servicio manda un tipo de producto desconocido | La respuesta se rechaza y ese grupo se trata como fallido: no se muestra. No se pinta una tarjeta a medias |
| Dos productos del mismo tipo con los mismos últimos cuatro dígitos | Limitación conocida: el detalle no los distingue. Se resuelve cuando la API entregue un identificador propio |
| Saldo en cero | Se muestra `0.00` |

## 8. Fuera de alcance

- Mensajes de error y botón de reintentar en esta pantalla.
- Mensaje para el cliente sin productos.
- Deslizar hacia abajo para recargar.
- Movimientos del producto.
- Ocultar o mostrar los saldos.

## 9. Decisiones tomadas al aclarar

| Duda | Decisión |
|---|---|
| ¿Se espera a los dos servicios o se va mostrando? | Se va mostrando a medida que llegan |
| ¿Qué va primero? | Siempre las cuentas |
| ¿Cómo se ve la carga? | Una sola tarjeta de carga con forma de tarjeta de producto |
| ¿Cuándo desaparece la tarjeta de carga? | Cuando ya respondieron todos los servicios; mientras tanto ocupa el lugar del primer grupo pendiente |
| ¿Se manejan errores en pantalla? | No, para la prueba. El grupo que falla no se muestra |
| Los saldos del diseño no son los de la API | Se usan los de la API con el formato del diseño |
| El diseño separa las tarjetas de crédito 22 y las cuentas 16 | Se usa 16 en ambos |
| El diseño dibuja la tarjeta de crédito de un saldo de 122 de alto | Se usa 128, igual a la cuenta, con el icono en la misma posición |
| ¿Solo la primera cuenta lleva al detalle, como en el prototipo? | No; todas las tarjetas llevan al detalle |

## 10. Cómo se verifica

- Pruebas del servicio: estado inicial, publicación por grupo, orden fijo con las tarjetas de crédito llegando primero y error aislado.
- Pruebas de los modelos: últimos cuatro dígitos, nombre visible y saldos según el tipo.
- Pruebas de la pantalla: cada uno de los criterios CA-01 a CA-09.
- Pruebas de la tarjeta: cuenta, tarjeta de crédito internacional y local, formato del saldo.
- Medición en el navegador de las tablas de la sección 5.
- El servicio simulado tarda 1, 2 o 3 segundos al azar en cada petición, para que la carga progresiva se pueda ver.
