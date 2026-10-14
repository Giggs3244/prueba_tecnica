# Spec 05 — Componentes compartidos

| | |
|---|---|
| Carpeta | `demo-bhd-app/src/app/shared/` |
| Diseño de referencia | Todos los artboards; cada componente indica de cuál sale |
| Servicio | Ninguno |
| La usan | Specs 01, 02, 03 y 04 |

## 1. Contexto

Son las piezas de interfaz que se repiten en varias pantallas. Se construyen una sola vez, con las medidas del diseño, para que todas las pantallas se vean iguales y un cambio de diseño se haga en un solo sitio.

## 2. Reglas para todos los componentes

| ID | Regla |
|---|---|
| RG-01 | No tienen lógica de negocio. No llaman servicios, no leen la sesión ni el almacenamiento. |
| RG-02 | Reciben lo que necesitan por entradas y avisan lo que pasa por salidas. |
| RG-03 | Sus medidas, colores y letras salen de los tokens del tema. |
| RG-04 | Usan detección de cambios `OnPush`. |
| RG-05 | Cuando existe un componente de Ionic que resuelve el comportamiento, se usa por dentro y se le ajusta el estilo. |
| RG-06 | Las entradas obligatorias se declaran como obligatorias, para que olvidarlas sea un error de compilación. |
| RG-07 | Todo control de solo icono lleva nombre accesible. |
| RG-08 | Las imágenes decorativas llevan texto alternativo vacío. |
| RG-09 | Cada componente tiene pruebas unitarias. |

## 3. Inventario

| Componente | Selector | Para qué |
|---|---|---|
| Botón | `app-button` | Acción principal y acción secundaria |
| Campo de texto | `app-input` | Entrada de una línea con icono |
| Interruptor | `app-toggle` | Opción de sí o no con texto |
| Encabezado | `app-header` | Barra superior con título |
| Botón de menú | `app-menu-button` | Abre el menú lateral |
| Icono | `app-icon` | Icono de un color que se puede cambiar |
| Tarjeta de producto | `app-product-card` | Muestra un producto |
| Tarjeta de carga | `app-product-card-skeleton` | Ocupa el lugar de una tarjeta mientras carga |
| Indicador de carga | `app-loading-overlay` | Bloquea la pantalla mientras se espera |
| Contenido de relleno | `app-placeholder-content` | Título y párrafo de las pantallas de relleno |

Además, en `shared/` viven la transición entre pantallas y dos listas de configuración: la presentación de cada tipo de producto y las secciones de la app.

---

## 4. Botón — `app-button`

Diseño: `btn_main` (login) y `btn_outline` (menú).

| ID | Requisito |
|---|---|
| BT-01 | Tiene dos variantes: **sólida** (fondo verde, texto blanco) y **de borde** (sin fondo, borde y texto verdes). La sólida es la de por defecto. |
| BT-02 | El texto se escribe entre las etiquetas del componente. |
| BT-03 | Puede ser de tipo botón o de tipo envío de formulario. |
| BT-04 | Se puede deshabilitar. |
| BT-05 | Ocupa todo el ancho de su contenedor. |
| BT-06 | El texto se muestra tal como se escribe; el componente no lo pasa a mayúsculas. |

| Medida | Valor |
|---|---|
| Alto | 44 |
| Esquinas | 8 |
| Texto | Negrilla 14/16, sin espacio extra entre letras |
| Borde (variante de borde) | 1 px verde `#49B742` |
| Sombra | Ninguna |

**Aceptación:** dado un botón de variante de borde, entonces no tiene fondo y su borde y su texto son verdes.

---

## 5. Campo de texto — `app-input`

Diseño: `input` (login).

| ID | Requisito |
|---|---|
| IN-01 | Recibe un texto de ayuda obligatorio, que se muestra mientras el campo está vacío. |
| IN-02 | Puede llevar un icono a la izquierda. Si no se le pasa, no se muestra ni se reserva el espacio. |
| IN-03 | Puede ser de texto o de contraseña. |
| IN-04 | Puede pedir teclado de texto o numérico. |
| IN-05 | Acepta una pista de autocompletado (usuario, contraseña actual o ninguna). |
| IN-06 | Su valor es de doble vía: se le puede poner un valor desde afuera y avisa cuando el usuario lo cambia. |
| IN-07 | Si el usuario borra todo, el valor es una cadena vacía, nunca nulo. |
| IN-08 | Avisa cuando pierde el foco. |
| IN-09 | Se puede deshabilitar; deshabilitado se ve atenuado. |
| IN-10 | Se conecta a un formulario de Signal Forms sin código adicional. |
| IN-11 | Como no tiene etiqueta visible, usa el texto de ayuda como nombre accesible. |

| Medida | Valor |
|---|---|
| Caja | Alto 44, esquinas de 8, fondo blanco, borde de 1 en `#CCCCCC` al 60 % |
| Icono | A 16 del borde izquierdo, gris `#CCCCCC` |
| Texto | A 47 del borde izquierdo, 14 |
| Texto de ayuda | Gris `#CCCCCC` |
| Texto escrito | Gris `#666666` |

**Aceptación:** dado un campo conectado a un formulario, cuando el usuario escribe `1111`, entonces el formulario tiene `1111` en ese campo.

---

## 6. Interruptor — `app-toggle`

Diseño: `toggle` (login, apagado y encendido).

| ID | Requisito |
|---|---|
| TG-01 | Su estado es de doble vía. |
| TG-02 | El texto se escribe entre las etiquetas del componente y se muestra a la derecha. |
| TG-03 | Tocar el texto también cambia el estado. |
| TG-04 | Avisa cuando pierde el foco. |
| TG-05 | Se puede deshabilitar. |
| TG-06 | Se conecta a un formulario de Signal Forms sin código adicional. |

| Medida | Valor |
|---|---|
| Pista | 40 × 24, esquinas de 12 |
| Perilla | 22 × 22, blanca, a 1 del borde, con sombra `1px 1px 1.5px` negra al 16 % |
| Apagado | Pista blanca al 60 % con borde de 1; perilla a la izquierda |
| Encendido | Pista verde `#49B742` sin borde; perilla a la derecha |
| Texto | A 8 de la pista, 12/14, gris `#666666` |

**Aceptación:** dado un interruptor apagado, cuando el usuario lo toca, entonces queda encendido, la pista se pone verde y el valor enlazado pasa a verdadero.

---

## 7. Encabezado — `app-header`

Diseño: `header` (todas las pantallas internas).

| ID | Requisito |
|---|---|
| HD-01 | Recibe un título obligatorio. |
| HD-02 | A la izquierda puede llevar: botón de menú (por defecto), flecha de volver o nada. |
| HD-03 | La flecha de volver regresa a la pantalla anterior. Si no hay pantalla anterior, va a una dirección de respaldo que se le pasa. |
| HD-04 | El título queda centrado respecto a toda la pantalla, tenga o no botón a la izquierda. |
| HD-05 | En celulares con muesca, el contenido baja lo que ocupe la zona segura. |

| Medida | Valor |
|---|---|
| Alto total | 66: 62 blancos más una línea verde de 4 abajo |
| Título | y = 23, 16/19, gris `#666666` |
| Botón de menú | Dibujo en (24, 23) |
| Flecha | 19.2 × 19.2 en (26.4, 23.4), verde |
| Sombra | Ninguna |

**Aceptación:** dado un encabezado con la opción de volver, entonces muestra la flecha y no muestra el botón de menú.

---

## 8. Botón de menú — `app-menu-button`

Diseño: `menu_icon`.

| ID | Requisito |
|---|---|
| MB-01 | Al tocarlo abre el menú lateral. No necesita que se le indique cuál. |
| MB-02 | Se muestra siempre, aunque el menú esté desactivado. |
| MB-03 | Se usa dentro del encabezado y también suelto en el login. |
| MB-04 | Tiene nombre accesible "Abrir menú". |

| Medida | Valor |
|---|---|
| Dibujo | 25 × 19: tres barras verdes de 3 de alto, separadas 5 |
| Zona de toque | 48 × 48, con el dibujo centrado |

---

## 9. Icono — `app-icon`

| ID | Requisito |
|---|---|
| IC-01 | Recibe la ubicación de un archivo SVG, obligatoria. |
| IC-02 | Recibe un tamaño en píxeles; por defecto 24. Es cuadrado. |
| IC-03 | Se pinta con el color de texto del lugar donde está puesto. El mismo archivo sirve para cualquier color. |
| IC-04 | Es decorativo: el lector de pantalla no lo anuncia. |

**Aceptación:** dado un mismo icono puesto en una pestaña activa y en una inactiva, entonces se ve verde en la primera y gris en la segunda sin cambiar de archivo.

---

## 10. Tarjeta de producto — `app-product-card`

Diseño: `account` y `credit_card` (dashboard y detalle).

| ID | Requisito |
|---|---|
| PC-01 | Recibe un producto, obligatorio. Sirve para cualquier tipo de producto. |
| PC-02 | Muestra el icono del tipo, el nombre para mostrar, un renglón por cada saldo y el texto "Balance disponible". |
| PC-03 | No decide cuántos saldos mostrar: muestra los que el producto entrega. |
| PC-04 | El icono y el color de la franja salen de la lista de presentación por tipo de producto. |
| PC-05 | Cada saldo se escribe así: símbolo de moneda, espacio, parte entera con comas de miles, punto y dos decimales. |
| PC-06 | La parte entera va en letra grande; el símbolo y los decimales en letra pequeña. |
| PC-07 | El componente solo muestra. Si la tarjeta debe llevar a otra pantalla, eso lo resuelve quien la usa. |

Las medidas están en la Spec 03, sección 5.

**Aceptación:**
- Dada una cuenta con saldo 100000, entonces se lee `RD$ 100,000.00` y la franja es verde.
- Dada una tarjeta de crédito con dos saldos, entonces se leen dos renglones y la franja es azul.
- Dado el saldo 1234567.5, entonces la parte grande es `1,234,567` y se lee completo `RD$ 1,234,567.50`.

---

## 11. Tarjeta de carga — `app-product-card-skeleton`

No está en el diseño. Se definió al aclarar la Spec 03.

| ID | Requisito |
|---|---|
| SK-01 | Tiene la misma caja que una tarjeta de producto con un saldo: 326 × 128, esquinas de 4, borde y franja izquierda de 6. |
| SK-02 | En vez de contenido muestra bloques grises animados en el lugar del icono, el nombre, el saldo y la etiqueta. |
| SK-03 | La franja es gris, porque todavía no se sabe de qué tipo es el producto. |
| SK-04 | Comparte con la tarjeta de producto las mismas medidas de base, definidas en un solo lugar. |
| SK-05 | Se anuncia al lector de pantalla como "Cargando productos"; los bloques no se anuncian. |

**Aceptación:** dada una tarjeta de carga y una tarjeta de cuenta en el mismo lugar, entonces ocupan exactamente el mismo espacio.

---

## 12. Indicador de carga — `app-loading-overlay`

Diseño: `busy_indicator` (`login – 3`).

| ID | Requisito |
|---|---|
| LO-01 | Cubre toda la pantalla y no deja tocar lo que hay debajo. |
| LO-02 | Muestra en el centro una caja blanca con un indicador que gira. |
| LO-03 | Se muestra y se oculta desde la pantalla que lo usa; el componente no maneja tiempos. |
| LO-04 | Se anuncia al lector de pantalla como "Cargando". |
| LO-05 | Si el sistema pide reducir movimiento, el indicador no gira. |

| Medida | Valor |
|---|---|
| Fondo | Negro al 40 % |
| Caja | 103 × 109, blanca, esquinas de 4, centrada |
| Indicador | 28 × 28, negro, puntos dispuestos en círculo; da una vuelta por segundo en ocho pasos |

---

## 13. Contenido de relleno — `app-placeholder-content`

Diseño: textos de `transactions`, `offers`, `configuration`, `contact`, `branches`, `product_detail`.

| ID | Requisito |
|---|---|
| PH-01 | Recibe un título y una descripción, los dos obligatorios. |
| PH-02 | El título respeta los saltos de línea que traiga el texto. |
| PH-03 | No define su posición en la pantalla; eso lo decide quien lo usa. |

| Medida | Valor |
|---|---|
| Título | 24/28, gris `#CCCCCC`, centrado |
| Descripción | 30 debajo del título, 294 de ancho, 14/16, gris `#CCCCCC`, centrada |

---

## 14. Transición entre pantallas

Diseño: transiciones del prototipo.

| ID | Requisito |
|---|---|
| TR-01 | Se aplica a toda la navegación de la app desde un solo punto de configuración. |
| TR-02 | Al avanzar, la pantalla nueva entra desde la derecha y la anterior sale hacia la izquierda. |
| TR-03 | Al volver, el movimiento es el contrario. |
| TR-04 | Dura 0.3 segundos, a ritmo parejo. |
| TR-05 | Si no hay pantalla anterior, solo se anima la que entra. |
| TR-06 | Al terminar no deja estilos pegados en las pantallas. |

---

## 15. Listas de configuración

| ID | Requisito |
|---|---|
| CF-01 | **Presentación por tipo de producto:** para cada tipo define el título del grupo, el icono y el color. Agregar un tipo es agregar una entrada. |
| CF-02 | **Secciones de la app:** define, en orden, las cuatro secciones con su ruta, su texto y su icono. La usan la barra de pestañas y el menú. |
| CF-03 | **Enlaces de ayuda:** define Contacto y Sucursales con su dirección, texto e icono. |

---

## 16. Fuera de alcance

- Tema oscuro.
- Tamaños de botón distintos al del diseño.
- Mensajes de error por campo en el campo de texto.
- Componente propio para el mensaje emergente de error: se usa el de Ionic, con estilo, desde la pantalla de login.
- Soporte de Reactive Forms en el campo de texto y el interruptor. Se construyó y se probó, pero se retiró para mantener un solo sistema de formularios.

## 17. Decisiones tomadas al aclarar

| Duda | Decisión |
|---|---|
| ¿Componentes propios o los de Ionic directamente? | Propios, que por dentro usan los de Ionic. Se gana el comportamiento de Ionic y el estilo se escribe una vez |
| ¿Indicador de carga de Ionic o propio? | Propio: la caja y el indicador del diseño no los trae Ionic |
| ¿Cómo se pintan los iconos de dos colores? | Un solo archivo por icono, pintado con el color del texto |
| El dibujo del botón de menú es más pequeño que una zona de toque cómoda | La zona de toque es de 48 × 48 y el dibujo queda en su posición |
| ¿La tarjeta de producto maneja el toque? | No; solo muestra |
| ¿Hacen falta transformadores de texto para saldo y últimos dígitos? | No. Los últimos dígitos los da el modelo y el saldo se arma en la tarjeta, porque lleva dos tamaños de letra |
| El título del encabezado usa otra tipografía en algunas pantallas del diseño | Se usa la misma del resto de la app |

## 18. Cómo se verifica

- Pruebas unitarias por componente, sobre lo que se ve y lo que avisa.
- Medición en el navegador, a 375 × 812, de cada tabla de medidas.
- Prueba de interacción: escribir en el campo, cambiar el interruptor y mostrar el indicador de carga.
