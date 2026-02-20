# Introducción

¡Bienvenido a Pronto!

A continuación, encontrarás toda la documentación necesaria para integrarse a Pronto mediante nuestra API (Application
Programming Interface).

```html title="Url Base:"
https://app.conpronto.com/
```

## Autenticación

Pronto utiliza un sistema de autenticación basado en tokens JWT.

Para consumir los endpoints es necesario:

1. Solicitar un Access Token utilizando las credenciales:
    - app_key
    - token

2. Incluir el Access Token en el encabezado Authorization de cada solicitud.

---

### Obtener Access Token

POST https://app.conpronto.com/api/v1/auth/token/

```json title="Body"
{
  "app_key": "xxxxxxxxxxxx-xxxxxxx-xxxxxx",
  "token": "secrettokenvalue"
}
```

```json title="Respuesta autenticación"
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## Métodos Soportados

Los endpoints disponibles se deben invocar por alguno de los siguientes métodos HTTP.

| Metodos  | Uso          |
|----------|--------------|
| `GET`    | Consulta     |
| `POST`   | Inserción    |
| `PUT`    | Modificación |
| `DELETE` | Eliminación  |

## Códigos de respuesta

Las respuestas a las llamadas a la API de Pronto utilizan la convención de códigos de estado HTTP para indicar si la
llamada fue exitosa o fallida.

| Status | Significado                                                                        | Descripción                                                                                                                                                                                               |
|--------|------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 
| `200`  | [OK](https://datatracker.ietf.org/doc/html/rfc7231#section-6.3.1)                  | La llamada fue exitosa                                                                                                                                                                                    |
| `201`  | [Created](https://datatracker.ietf.org/doc/html/rfc7231#section-6.3.2)             | Creado. La llamada fue exitosa y un nuevo objeto o grupo de objetos fue creado                                                                                                                            |
| `401`  | [Unauthorized](https://datatracker.ietf.org/doc/html/rfc7235#section-3.1)          | El token JWT es inválido, expirado o no fue enviado en el encabezado Authorization.                                                                                                                       |
| `404`  | [Not Found](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.4)           | No encontrado. Según la API Key indicada en el Header, el recurso no existe. Esto también puede indicar que no tienes permitido realizar este tipo de llamada.                                            |

[//]: # (| `422`  | [Unprocessable Entity]&#40;https://datatracker.ietf.org/doc/html/rfc2518#section-10.3&#41; | No Procesable. La solicitud está correctamente estructurada, pero presenta errores en cuanto a su significado. Es posible que hayas omitido campos obligatorios o incluido campos que no son pertinentes. |)

[//]: # (| `429`  | [Too Many Requests]&#40;https://datatracker.ietf.org/doc/html/rfc6585#section-4&#41;       | Demasiadas solicitudes. Llamaste muchas veces seguidas a la API.                                                                                                                                          |)

## URL's

Las siguientes son url's válidas para uso del api:

### Url's de cliente

| URL                                                | Método |
|----------------------------------------------------|--------|
| `https://app.conpronto.com/api/v1/customers/`      | GET    |
| `https://app.conpronto.com/api/v1/customers/<ID>/` | GET    |
| `https://app.conpronto.com/api/v1/customers/`      | POST   |
| `https://app.conpronto.com/api/v1/customers/<ID>/` | PUT    |
| `https://app.conpronto.com/api/v1/customers/<ID>/` | DELETE |

### Url's de contacto

| URL                                                                 | Método |
|---------------------------------------------------------------------|--------|
| `https://app.conpronto.com/api/v1/contacts/`                        | GET    |
| `https://app.conpronto.com/api/v1/contacts/`                        | POST   |
| `https://app.conpronto.com/api/v1/customers/<ID_CLIENTE>/contacts/` | DELETE |

### Url's de documento

| URL                                                                                             | Método |
|-------------------------------------------------------------------------------------------------|--------|
| `https://app.conpronto.com/documento/`                                                          | POST   |
| `https://app.conpronto.com/documento/`                                                          | PUT    |
| `https://app.conpronto.com/documento/<ID>/`                                                     | GET    |
| `https://app.conpronto.com/documento/?fecha_inicial=<FECHA_INICIAL>&fecha_final=<FECHA_FINAL>/` | GET    |

### Url's de cobro

| URL                                                                                             | Método |
|-------------------------------------------------------------------------------------------------|--------|
| `https://app.conpronto.com/documento/<ID>/cobro/`                                               | POST   |
| `https://app.conpronto.com/documento/<ID>/cobro/`                                               | GET    |
| `GET https://app.conpronto.com/cobro/?fecha_inicial=<FECHA_INICIAL>&fecha_final=<FECHA_FINAL>/` | GET    |
| `https://app.conpronto.com/cobro/<PASARELA>/`                                                   | GET    |