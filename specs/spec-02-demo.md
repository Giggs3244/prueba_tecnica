# Spec 02 — Demo de bienvenida

| | |
|---|---|
| Ruta | `/onboarding` |
| Diseño de referencia | Artboards `demo_1`, `demo_2`, `demo_3` |
| Servicio | Ninguno |
| Depende de | Spec 01 (es la pantalla a la que se llega al terminar) |

## 1. Contexto

La primera vez que alguien abre la app ve tres pantallas cortas que le cuentan para qué sirve. Al terminarlas llega al login y no las vuelve a ver.

En el prototipo del diseño, a estas pantallas solo se llega tocando el logo del login. Eso es un atajo del diseñador para poder mostrarlas; se acordó que en la app se muestren en el primer ingreso.

## 2. Historias de usuario

**HU-01. Conocer la app**
Como persona que abre la app por primera vez, quiero ver un resumen de lo que puedo hacer, para saber si me sirve.

**HU-02. Avanzar a mi ritmo**
Como usuario, quiero pasar las pantallas con un botón o deslizando el dedo, como me quede más cómodo.

**HU-03. No repetir**
Como usuario que ya vio el demo, quiero llegar directo al login las próximas veces.

## 3. Requisitos funcionales

### Contenido

| ID | Requisito |
|---|---|
| RF-01 | El demo tiene tres pantallas, cada una con un título y una ilustración. |
| RF-02 | Pantalla 1: "Puedes ver tus productos", con la ilustración de la alcancía. |
| RF-03 | Pantalla 2: "Ahorra tiempo realizando transacciones de forma rápida y segura", con la ilustración del cronómetro. |
| RF-04 | Pantalla 3: "Aprovecha las ofertas que tenemos para ti", con la ilustración de la etiqueta. |
| RF-05 | Debajo hay tres puntos indicadores; el de la pantalla que se está viendo va en verde. |
| RF-06 | Abajo hay una franja blanca con el botón "CONTINUAR" a la derecha. |

### Avance

| ID | Requisito |
|---|---|
| RF-07 | "CONTINUAR" pasa a la pantalla siguiente con desplazamiento suave. |
| RF-08 | Tocar cualquier parte de la pantalla también pasa a la siguiente. |
| RF-09 | El usuario puede deslizar el dedo a los lados para avanzar o devolverse. |
| RF-10 | Al soltar, la pantalla siempre queda acomodada completa; nunca a medias. |
| RF-11 | Un deslizamiento rápido no se salta una pantalla. |
| RF-12 | El punto indicador activo siempre corresponde a la pantalla que se ve, sin importar cómo se llegó a ella. |

### Final

| ID | Requisito |
|---|---|
| RF-13 | En la tercera pantalla, "CONTINUAR" guarda en el dispositivo que el demo ya se vio y abre el login. |
| RF-14 | El paso al login se hace con transición de avance. |
| RF-15 | Desde el login no se puede volver al demo con el botón atrás. |
| RF-16 | La marca de "ya visto" se guarda al terminar, no al entrar. Si el usuario cierra la app a la mitad, la próxima vez lo ve completo. |

### Cuándo se muestra

| ID | Requisito |
|---|---|
| RF-17 | Al abrir la app sin haber visto el demo, se muestra el demo en vez del login. |
| RF-18 | Al abrir la app habiéndolo visto, se muestra el login. |
| RF-19 | Un usuario con sesión activa no ve el demo; es enviado a "Mis productos". |
| RF-20 | Mientras se ve el demo, el menú lateral queda desactivado, incluido el gesto de deslizar desde el borde. Al salir se vuelve a activar. |

## 4. Criterios de aceptación

**CA-01. Primer ingreso**
- Dado que nunca he abierto la app
- Cuando la abro
- Entonces veo la primera pantalla del demo con el primer punto en verde

**CA-02. Avanzar con el botón**
- Dado que estoy en la primera pantalla
- Cuando oprimo "CONTINUAR"
- Entonces veo la segunda pantalla y el segundo punto en verde

**CA-03. Avanzar tocando**
- Dado que estoy en la primera pantalla
- Cuando toco la ilustración
- Entonces veo la segunda pantalla

**CA-04. Devolverse deslizando**
- Dado que estoy en la segunda pantalla
- Cuando deslizo el dedo hacia la derecha
- Entonces veo la primera pantalla y el primer punto en verde

**CA-05. Terminar**
- Dado que estoy en la tercera pantalla
- Cuando oprimo "CONTINUAR"
- Entonces veo el login

**CA-06. Segundo ingreso**
- Dado que ya terminé el demo
- Cuando cierro y vuelvo a abrir la app
- Entonces veo el login directamente

**CA-07. Abandonar a la mitad**
- Dado que voy en la segunda pantalla
- Cuando cierro la app y la vuelvo a abrir
- Entonces veo otra vez el demo desde la primera pantalla

**CA-08. El menú no estorba**
- Dado que estoy en el demo
- Cuando deslizo el dedo desde el borde izquierdo
- Entonces no se abre el menú lateral

## 5. Requisitos de diseño

Medidas a 375 × 812, en píxeles.

| Elemento | Posición | Tamaño | Detalle |
|---|---|---|---|
| Fondo | Toda la pantalla | — | Gris claro `#F5F5F5` |
| Título | y = 169, centrado | 24/28 | Gris `#666666`, con los saltos de línea del diseño |
| Ilustración 1 | (110.4, 363.8) | 155.1 × 144.4 | Alcancía |
| Ilustración 2 | (117.7, 353.9) | 139.7 × 162 | Cronómetro |
| Ilustración 3 | (116, 364.1) | 144 × 151.7 | Etiqueta |
| Puntos | y = 706; x = 156, 180, 204 | 16 × 16 | Activo verde `#49B742`; los demás blancos con borde |
| Franja inferior | (0, 752) | 375 × 60 | Blanca |
| "CONTINUAR" | (266, 774) | 77 × 16 | Verde, negrilla 14, a 32 del borde derecho |

Las tres ilustraciones no comparten el mismo centro en el diseño; se respeta la posición de cada una.

En celulares con muesca o barra de gestos, el título baja y la franja inferior crece lo que ocupen las zonas seguras.

## 6. Requisitos no funcionales

| ID | Requisito |
|---|---|
| RNF-01 | No se usa ninguna librería de carrusel: el desplazamiento es el nativo del navegador. |
| RNF-02 | La marca de "ya visto" sobrevive al cierre de la app. |
| RNF-03 | Los puntos indicadores son decorativos y no los anuncia el lector de pantalla. |
| RNF-04 | Solo la pantalla visible queda expuesta al lector de pantalla. |
| RNF-05 | La zona de toque de "CONTINUAR" ocupa todo el alto de la franja. |
| RNF-06 | La pantalla se carga de forma diferida. |

## 7. Casos borde

| Caso | Comportamiento esperado |
|---|---|
| Se borran los datos de la app | El demo se vuelve a mostrar |
| Se abre `/onboarding` a mano con sesión activa | Redirección a "Mis productos" |
| Se abre `/login` a mano sin haber visto el demo | Redirección al demo |
| El usuario desliza hacia la izquierda en la última pantalla | No pasa nada; solo el botón termina el demo |

## 8. Fuera de alcance

- Botón para saltarse el demo.
- Volver a ver el demo desde un menú de la app.
- Avance automático por tiempo.

## 9. Decisiones tomadas al aclarar

| Duda | Decisión |
|---|---|
| ¿Cuándo se muestra el demo? | Solo en el primer ingreso |
| ¿Qué pasa si se abandona a la mitad? | Se vuelve a mostrar completo |
| El prototipo avanza al tocar toda la pantalla | Se mantiene, además del botón y del deslizamiento |
| ¿Carrusel de terceros? | No; desplazamiento nativo con ajuste por pantalla |

## 10. Cómo se verifica

- Pruebas unitarias: contenido de las tres pantallas, avance con botón, avance tocando, seguimiento del punto activo al deslizar, guardado de la marca y paso al login al final, y desactivación del menú.
- Pruebas de rutas: primer ingreso, ingreso posterior y usuario con sesión.
- Medición en el navegador de la tabla de la sección 5.
- Para repetir el demo en el navegador: borrar la clave `CapacitorStorage.onboardingSeen` del almacenamiento local.
