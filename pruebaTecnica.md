# Prueba tecnica frontend

## Objetivo de la prueba tecnica

El objetivo de esta prueba es validar el conocimiento que se tienen en las tecnologías que se usan en los desarrollos que estarías participando.

### Tecnologías:

- HTML
- CSS (SCSS)
- Javascript (Angular, Ionic)
- Capacitor

### Que se tendrá en cuenta:

- Estructuración del proyecto
- Performance
- Buenas prácticas de programación
- Programación orientada a objetos
- Pixel perfection (para nosotros como banco es muy importante que las interfaces que recibimos de los diseñadores se vean exactamente como ellos quieren (porque son nuestra cara hacia al cliente), por lo tanto esto es de lo más importante)
- Clean code
- Patrones de diseño
- Pruebas unitarias serán bien recibidas

### Diseño de la aplicación a construir

En el siguiente enlace podrás ver lo que tienes que desarrollar. Por favor, para esta prueba usar los servicios mock.

+ Enlace: https://xd.adobe.com/view/e03d0ca0-5699-4cca-5056-53b17ee7f402-960c/
+ Password: BHDLeonTest2020

### Backend

La aplicación va a requerir que consumas 4 servicios:

(Es una pequeña aplicación que retorna un token (siempre el mismo), y ese token debe inyectarse en el resto de peticiones para que el cliente pueda acceder al resto de información)

Servicios:

- Login
- Obtener la información del usuario
- Obtener sus cuentas
- Obtener sus tarjetas de crédito

A continuación la documentación del dichos servicios:

1. Login

+ Path: /sign_in
+ Method: POST
+ NOTA: No requiere token. Sin headers personalizados.
+ Request body:
```json
{
"userId": "00100010321",
"password": "1111"
}
```
+ Response:
```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIwMDEwMDAxMDMyMSJ9.kPLjCmPJaw8jaOu0cEp4sXR0e52YRrj97OCGJaQGzA4",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.e30.dw8gT8j5mKQbY8dVK453_dRICvBSg2oAQYFfzO1rpoY"
}
```

2. Obtener la información del usuario


+ Path: /user
+ Method: GET
+ NOTA: Requiere token.
+ Headers:
```json
{
"authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIwMDEwMDAxMDMyMSJ9.kPLjCmPJaw8jaOu0cEp4sXR0e52YRrj97OCGJaQGzA4"
}
```
+ Response:
```json
{
    "name": "José",
    "lastName": "Pérez",
    "photo": "https://images.pexels.com/photos/220453/pexels-photo-220453.jpeg?auto=compress&cs=tinysrgb&dpr=1&w=500"
}
```


3. Obtener sus cuentas

+ Path: /accounts
+ Method: GET
+ NOTA: Requiere token.
+ Headers:
```json
{
"authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIwMDEwMDAxMDMyMSJ9.kPLjCmPJaw8jaOu0cEp4sXR0e52YRrj97OCGJaQGzA4"
}
```
+ Response:
```json
[
    {
        "alias": "Cuenta de ahorros 1",
        "number": "1234567890001",
        "availableAmount": 100000,
        "productType": "AC"
    },
    {
        "alias": "Cuenta de ahorros 2",
        "number": "1234567890002",
        "availableAmount": 200000,
        "productType": "AC"
    }
]
```

4. Obtener sus tarjetas de crédito

+ Path: /credit_cards
+ Method: GET
+ NOTA: Requiere token.
+ Headers:
```json
{
"authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIwMDEwMDAxMDMyMSJ9.kPLjCmPJaw8jaOu0cEp4sXR0e52YRrj97OCGJaQGzA4"
}
```
+ Response:
```json
[
    {
        "alias": "Tarjeta de crédito 1",
        "number": "0987 6543 2109 0001",
        "availableAmountRD": 1000000,
        "availableAmountUS": 1000,
        "isInternational": true,
        "productType": "TC"
    },
    {
        "alias": "Tarjeta de crédito 2",
        "number": "0987 6543 2109 0002",
        "availableAmountRD": 2000000,
        "availableAmountUS": 0,
        "isInternational": false,
        "productType": "TC"
    }
]
```