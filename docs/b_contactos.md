# Contacto

## Objeto Contacto

```json title="Objeto Contacto:"
[
  {
    "id_contacto": "1224",
    "id_cliente": "4321",
    "nombre": "Andrés",
    "apellido": "López",
    "movil": "+593942985587",
    "email ": "alopez@gmail.com",
    "tipo_contacto_email": "TO"
  },
  {
    "id_contacto": "1224",
    "id_cliente": "4321",
    "nombre": "Juan",
    "apellido": "López",
    "movil": "+593942985587",
    "email ": "jlopez@gmail.com",
    "tipo_contacto_email": "CC"
  }
]
```

Atributos del objeto contacto:

| Parámetro             | Tipo    | Longitud | Obligatorio | Descripción                                                   |
|-----------------------|---------|----------|-------------|---------------------------------------------------------------|
| `id_contacto`         | int     |          | Si          | Identificador del cliente al que se desea agregar un contacto |
| `id_cliente`          | int     |          | Si          | Identificador del cliente al que se desea agregar un contacto |
| `nombre`              | varchar | 200      | Si          | Nombre del contacto                                           |
| `apellido`            | varchar | 200      | Si          | Apellido del contacto                                         |
| `movil`               | varchar | 20       | Si          | Móvil del contacto                                            |
| `email`               | varchar | 200      | Si          | Email del contacto                                            |
| `tipo_contacto_email` | varchar | 3        | No          | Tipo de contacto: TO o CC                                     |

Reglas Funcionales

- Un cliente puede tener uno o más contactos.
- Se debe enviar al menos uno de los siguientes campos:
    - movil
    - correo_electronico
- No pueden enviarse ambos vacíos.
- tipo_contacto_email puede ser:
    - TO (contacto principal)
    - CC (contacto copia)
- La API recibe una lista de contactos para crear.
- Para actualizar contactos se debe:
    - Eiminar los contactos existentes.
    - Enviar nuevamente la lista completa.

## Obtener Contactos (GET)

Permite obtener el listado de contactos registrados.

```bash
GET https://api.conpronto.com/contacts/
```

Headers

```bash
Authorization: Bearer <access_token>
```

---

### Parámetros de Consulta (Opcionales)

| Parámetro  | Tipo    | Descripción                             |
|------------|---------|-----------------------------------------|
| p          | integer | Número de página                        |
| page_size  | integer | Cantidad de registros por página        |
| id_cliente | integer | Filtra los contactos por ID del cliente |

---

### Ejemplo con filtros y paginación

---

```bash
GET https://app.conpronto.com/api/v2/contacts/?p=1&page_size=25&id_cliente=<ID_CLIENTE>
```

### Respuesta

La respuesta es paginada y contiene:

- count → total de registros
- next → siguiente página
- previous → página anterior
- results → lista de contactos

```json
{
  "count": 2,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1224,
      "id_cliente": 4321,
      "nombre": "Andrés",
      "apellido": "Lopez",
      "movil": "+593942985587",
      "correo_electronico": "alopez@gmail.com",
      "tipo_contacto_email": "TO"
    },
    {
      "id": 1225,
      "id_cliente": 4321,
      "nombre": "Juan",
      "apellido": "Lopez",
      "movil": "+593957235284",
      "correo_electronico": "jlopez@gmail.com",
      "tipo_contacto_email": "CC"
    }
  ]
}
```

## Obtener un Contacto (GET)

Permite obtener la información de un contacto específico.

`GET https://app.conpronto.com/api/v2/contacts/{id}/`

Headers

`Authorization: Bearer <access_token>`

### Parámetros

| Parámetro | Tipo    | Descripción                      |
|-----------|---------|----------------------------------|
| id        | integer | Identificador único del contacto |

---

### Ejemplo

`GET https://app.conpronto.com/api/v2/contacts/<ID_CONTACTO>/`

---

### Respuesta

```json
{
  "id_contacto": 231,
  "id_cliente": 321,
  "nombre": "Andrés",
  "apellido": "López",
  "movil": "+593942985587",
  "correo_electronico": "alopez@gmail.com",
  "tipo_contacto_email": "TO"
}
```

## Crear Contactos (POST)

Este servicio permite registrar uno o varios contactos asociados a un cliente en una sola solicitud.

`POST https: //api.conpronto.com/api/v2/contacts/`

Por medio del método POST enviando en el cuerpo del requerimiento los datos del contacto.
Headers

```bash
Authorization: Bearer <access_token>
Content-Type: application/json
```

---

### Estructura del JSON

El cuerpo del requerimiento debe enviarse como una **lista de objetos**.  
Cada objeto representa un contacto que será creado.

```json title="Body"
[
  {
    "id_cliente": 122,
    "nombre": "Andrés",
    "apellido": "Lopez",
    "movil": "+593942985587",
    "correo_electronico": "alopez@gmail.com",
    "tipo_contacto_email": "TO"
  },
  {
    "id_cliente": 122,
    "nombre": "Juan",
    "apellido": "Perez",
    "movil": "+593999999999",
    "correo_electronico": "juan@email.com",
    "tipo_contacto_email": "CC"
  }
]
```

Comportamiento

- Se pueden enviar uno o varios contactos en la misma solicitud.
- Todos los contactos enviados serán creados y asociados al id_cliente indicado.
- Si alguno de los registros contiene errores de validación, la operación podrá ser rechazada.
- Cada contacto creado quedará registrado como activo.
- El cliente debe pertenecer al partner autenticado.

Validaciones

- id_cliente debe existir.
- Se debe enviar al menos uno de los siguientes campos:
    - movil
    - correo_electronico
- No se permiten ambos campos vacíos.
- tipo_contacto_email debe ser:
    - TO (contacto principal)
    - CC (contacto copia)

### Respuesta

Devuelve la lista de contactos creados:

```json
[
  {
    "id_contacto": 1234,
    "id_cliente": 122,
    "nombre": "Andrés",
    "apellido": "Lopez",
    "movil": "+593942985587",
    "correo_electronico": "alopez@gmail.com",
    "tipo_contacto_email": "TO"
  },
  {
    "id_contacto": 1243,
    "id_cliente": 122,
    "nombre": "Juan",
    "apellido": "Perez",
    "movil": "+593999999999",
    "correo_electronico": "juan@email.com",
    "tipo_contacto_email": "CC"
  }
]
```

## Eliminar Contactos por Cliente (DELETE)

Este servicio permite eliminar todos los contactos asociados a un cliente específico.

El ID que se envía en la URL corresponde al **ID del cliente**, no al ID del contacto.

`DELETE http://app.conpronto.com/api/v2/contacts/<ID_CLIENTE>/`

Headers

```bash
Authorization: Bearer <access_token>
```

---

### Comportamiento

- Se eliminarán todos los contactos asociados al cliente indicado.
- El cliente debe pertenecer al partner autenticado.
- Si el cliente no existe, se retornará error.
- Este endpoint no devuelve contenido en la respuesta.

---

### Respuesta Exitosa

```bash
204 No Content
```

No retorna cuerpo en la respuesta.

---

### Códigos de Respuesta

| Código | Descripción                        |
|--------|------------------------------------|
| 204    | Contactos eliminados correctamente |
| 400    | Cliente no existe                  |
| 401    | Token inválido o expirado          |
| 403    | Sin permisos                       |