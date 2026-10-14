# Spec 01 — Inicio de sesión

| | |
|---|---|
| Ruta | `/login` |
| Diseño de referencia | Artboards `login`, `login – 2`, `login – 3`, `login – 4` |
| Servicio | `POST /sign_in` |
| Depende de | Spec 05 (campo de texto, interruptor, botón, indicador de carga, botón de menú) |

## 1. Contexto

Es la puerta de entrada a la aplicación. El usuario escribe su documento de identidad y su contraseña; si son correctos, entra a ver sus productos. El servicio responde siempre con el mismo token, que después se manda en todas las demás peticiones.

## 2. Historias de usuario

**HU-01. Ingresar a la aplicación**
Como cliente del banco, quiero ingresar con mi documento y mi contraseña para consultar mis productos.

**HU-02. Que la app se acuerde de mi usuario**
Como cliente que entra seguido, quiero que la app recuerde mi documento para no tener que digitarlo cada vez.

**HU-03. Saber qué pasó cuando algo sale mal**
Como cliente, quiero que la app me avise si me equivoqué en los datos, para poder corregirlos.

**HU-04. Saber que la app está trabajando**
Como cliente, quiero ver que la app está validando mis datos, para no quedarme oprimiendo el botón.

## 3. Requisitos funcionales

### Formulario

| ID | Requisito |
|---|---|
| RF-01 | La pantalla muestra, en este orden: botón de menú, logo del banco, campo "Documento de identidad", campo "Contraseña", interruptor "Recuerda mi usuario" y botón "INICIAR SESIÓN". |
| RF-02 | El campo de documento abre el teclado numérico en el celular. |
| RF-03 | El campo de contraseña oculta lo que se escribe. |
| RF-04 | El documento y la contraseña son obligatorios. |
| RF-05 | El formulario se envía con el botón "INICIAR SESIÓN". |

### Envío

| ID | Requisito |
|---|---|
| RF-06 | Al enviar con los dos campos diligenciados, la app llama a `POST /sign_in` con `userId` y `password`. |
| RF-07 | Esa petición no lleva token ni encabezados propios. |
| RF-08 | Mientras se espera la respuesta se muestra el indicador de carga encima de toda la pantalla. |
| RF-09 | Si el usuario vuelve a oprimir el botón mientras hay una petición en curso, no se hace una segunda petición. |
| RF-10 | Si falta alguno de los dos campos, no se llama al servicio y se muestra el mensaje de error de credenciales. |

### Respuesta correcta

| ID | Requisito |
|---|---|
| RF-11 | Se guardan en memoria el token de acceso y el de refresco. |
| RF-12 | Si el interruptor está encendido, se guarda el documento en el dispositivo. Si está apagado, se borra el que hubiera guardado. |
| RF-13 | La app pasa a "Mis productos" con transición de avance. |
| RF-14 | Desde "Mis productos" no se puede volver al login con el botón atrás. |

### Respuesta con error

| ID | Requisito |
|---|---|
| RF-15 | Si el servicio responde 401, se muestra el mensaje "Usuario y/o contraseña incorrectos". |
| RF-16 | Si la petición falla por cualquier otro motivo, se muestra "No fue posible iniciar sesión. Intenta de nuevo". |
| RF-17 | El mensaje aparece arriba, en rojo, y se cierra solo a los 3 segundos. |
| RF-18 | Después de un error el usuario sigue en el login y los campos conservan lo que había escrito. |
| RF-19 | Un intento fallido no guarda ni borra el documento recordado. |

### Al entrar a la pantalla

| ID | Requisito |
|---|---|
| RF-20 | Cada vez que se muestra el login, se consulta si hay un documento recordado. |
| RF-21 | Si lo hay, el campo de documento aparece diligenciado y el interruptor encendido. |
| RF-22 | El campo de contraseña siempre aparece vacío, incluso si el usuario viene de cerrar sesión. |

### Acceso

| ID | Requisito |
|---|---|
| RF-23 | Un usuario con sesión activa que intente abrir `/login` es enviado a "Mis productos". |
| RF-24 | El botón de menú abre el menú lateral en su versión pública (ver Spec 04). |

## 4. Criterios de aceptación

**CA-01. Ingreso exitoso**
- Dado que estoy en el login
- Cuando escribo el documento `00100010321` y la contraseña `1111` y oprimo "INICIAR SESIÓN"
- Entonces veo el indicador de carga y después la pantalla "Mis productos"

**CA-02. Contraseña equivocada**
- Dado que estoy en el login
- Cuando escribo un documento válido con una contraseña que no es y envío
- Entonces veo el indicador de carga, después el mensaje "Usuario y/o contraseña incorrectos", y sigo en el login con mis datos todavía escritos

**CA-03. Formulario incompleto**
- Dado que dejé vacía la contraseña
- Cuando oprimo "INICIAR SESIÓN"
- Entonces veo el mensaje de error y no se hace ninguna petición al servicio

**CA-04. Recordar el usuario**
- Dado que ingresé con el interruptor encendido
- Cuando cierro sesión y vuelvo al login
- Entonces el documento aparece diligenciado, el interruptor encendido y la contraseña vacía

**CA-05. Dejar de recordar**
- Dado que mi documento estaba recordado
- Cuando ingreso con el interruptor apagado y después vuelvo al login
- Entonces el campo de documento aparece vacío y el interruptor apagado

**CA-06. Doble toque**
- Dado que ya envié el formulario y sigo viendo el indicador de carga
- Cuando vuelvo a enviar
- Entonces se hace una sola petición

**CA-07. El mensaje se va solo**
- Dado que apareció el mensaje de error
- Cuando pasan 3 segundos
- Entonces el mensaje desaparece sin que yo haga nada

## 5. Requisitos de diseño

Medidas a 375 × 812, en píxeles, desde la esquina superior izquierda.

| Elemento | Posición | Tamaño | Detalle |
|---|---|---|---|
| Fondo | Toda la pantalla | — | Foto de fondo, cubre sin deformarse; encuadre con foco horizontal en 77.35 % |
| Botón de menú | (24, 23) | 25 × 19 | Tres barras verdes |
| Logo | (80, 120) | 215 × 66 | Centrado |
| Campo documento | (24, 263) | 326 × 44 | Icono de usuario |
| Campo contraseña | (24, 331) | 326 × 44 | Icono de llave |
| Interruptor | (107, 399) | 40 × 24 | Texto a 8 px, 12/14, gris `#666666` |
| Botón | (24, 447) | 326 × 44 | Verde `#49B742`, texto blanco en negrilla 14 |
| Mensaje de error | (10, 12) | 356 × 61 | Rojo `#C92340`, esquinas de 4, texto blanco centrado 14 |
| Indicador de carga | Centrado | 103 × 109 | Fondo negro al 40 %, caja blanca, esquinas de 4 |

En celulares con muesca, el botón de menú y el logo bajan lo que ocupe la zona segura de arriba.

## 6. Requisitos no funcionales

| ID | Requisito |
|---|---|
| RNF-01 | La contraseña nunca se guarda en el dispositivo. |
| RNF-02 | El token no se guarda en el dispositivo; se pierde al cerrar la app. |
| RNF-03 | El documento recordado sobrevive al cierre de la app. |
| RNF-04 | Los campos tienen nombre accesible aunque no tengan etiqueta visible. |
| RNF-05 | La pantalla se carga de forma diferida. |

## 7. Casos borde

| Caso | Comportamiento esperado |
|---|---|
| No hay red | Mensaje genérico de error; no se queda cargando |
| El usuario entra, sale y vuelve a entrar | La contraseña anterior no aparece escrita |
| Sesión activa y se abre `/login` a mano | Redirección a "Mis productos" |

## 8. Fuera de alcance

- Recuperar la contraseña.
- Registro de usuarios nuevos.
- Ingreso con huella o reconocimiento facial.
- Bloqueo por intentos fallidos.
- Mensajes de error campo por campo: el diseño solo trae un mensaje general.

## 9. Decisiones tomadas al aclarar

| Duda | Decisión |
|---|---|
| El diseño no dice qué pasa si falta un campo | Se muestra el mismo mensaje de credenciales incorrectas, que es el único error que trae el diseño |
| El mensaje de error del diseño usa otra tipografía | Se usa la misma del resto de la app |
| El diseño muestra la contraseña con asteriscos | Se usan los puntos propios del campo de contraseña del sistema |
| El interruptor no queda centrado exacto en el diseño | Se respeta la posición del diseño |

## 10. Cómo se verifica

- Pruebas unitarias de la pantalla: estado al entrar, envío, doble envío, respuesta correcta, recordar y olvidar, error 401, error genérico y cierre del mensaje.
- Medición en el navegador de cada elemento de la tabla de la sección 5.
- Recorrido manual de los criterios CA-01 a CA-07.
