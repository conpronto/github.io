# Cliente

## Objeto Cliente

```json
{
  "id": 123445,
  "identificacion": "099999999001",
  "nombre": "NOMBRE CLIENTE",
  "apellido": "",
  "razon_social": "NOMBRE CLIENTE",
  "nombre_comercial": "NOMBRE CLIENTE",
  "direccion": "PICHINCHA / QUITO / SAN ISIDRO DEL INCA",
  "silenciado": false,
  "metadatos": {
    "segmento": "corporativo",
    "zona": "norte"
  },
  "contactos": [
    {
      "id_contacto": 12345,
      "id_cliente": 12345,
      "nombre": "PRUEBA",
      "apellido": "",
      "movil": "+593999999999",
      "correo_electronico": "prueba@mail.com",
      "tipo_contacto_email": "TO"
    },
    {
      "id_contacto": 12346,
      "id_cliente": 12345,
      "nombre": "PRUEBA",
      "apellido": "",
      "movil": "+593999999999",
      "correo_electronico": null,
      "tipo_contacto_email": "CC"
    }
  ]
}
```

### Atributos del objeto Cliente
----------------------------------------------------------------

| Parámetro        | Tipo    | Longitud | Descripción                                                     |
|------------------|---------|----------|-----------------------------------------------------------------|
| id               | int     |          | Identificador interno del cliente                               |
| identificacion   | varchar | 25       | Cédula o RUC del cliente                                        |
| nombre           | varchar | 300      | Nombre (persona natural)                                        |
| apellido         | varchar | 300      | Apellido (persona natural)                                      |
| razon_social     | varchar | 300      | Razón social (empresa)                                          |
| nombre_comercial | varchar | 300      | Nombre comercial (empresa)                                      |
| direccion        | varchar | 300      | Dirección del cliente                                           |
| silenciado       | boolean |          | Indica si el cliente está silenciado                            |
| metadatos        | object  |          | Objeto JSON con información adicional estructurada              |
| contactos        | list    |          | Lista de [objectos de contacto](#atributos-del-objeto-contacto) |                      

#### Atributos del objeto contacto

| Parámetro             | Tipo    | Longitud | Obligatorio | Descripción                                                   |
|-----------------------|---------|----------|-------------|---------------------------------------------------------------|
| `id_contacto`         | int     |          | Si          | Identificador del cliente al que se desea agregar un contacto |
| `id_cliente`          | int     |          | Si          | Identificador del cliente al que se desea agregar un contacto |
| `nombre`              | varchar | 300      | Si          | Nombre del contacto                                           |
| `apellido`            | varchar | 300      | No          | Apellido del contacto                                         |
| `movil`               | varchar | 30       | Si          | Móvil del contacto                                            |
| `correo_electronico`  | varchar | 254      | Si          | Email del contacto                                            |
| `tipo_contacto_email` | varchar | 2        | No          | Tipo de contacto: TO o CC                                     |

## Obtener un Cliente (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v1/customers/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

Respuesta 200
[Devuelve el objeto Cliente.](#objeto-cliente)

## Obtener listado de Clientes (GET)

Permite obtener el listado de clientes asociados al partner autenticado.

```bash title="URL"
GET https://app.conpronto.com/api/v1/customers/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

---

### Parámetros de Consulta (Opcionales)

| Parámetro | Tipo    | Descripción                                                            |
|-----------|---------|------------------------------------------------------------------------|
| p         | integer | Número de página                                                       |
| page_size | integer | Cantidad de registros por página                                       |
| search    | string  | Permite buscar por nombre, apellido, identificación o nombre comercial |

---

### Respuesta

```json title="Respuesta paginada"
{
  "count": 713,
  "next": "https://app.conpronto.com/api/v1/customers/?p=2&page_size=50",
  "previous": null,
  "results": [
    {
      "id": 123445,
      "identificacion": "099999999001",
      "nombre": "NOMBRE CLIENTE",
      "apellido": "",
      "razon_social": "NOMBRE CLIENTE",
      "nombre_comercial": "NOMBRE CLIENTE",
      "direccion": "Quito",
      "silenciado": false,
      "metadatos": null
    }
  ]
}
```

## Crear Cliente (POST)

```bash title="URL"
POST https://app.conpronto.com/api/v1/customers/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body"
{
  "identificacion": "0999999999001",
  "nombre": "Andres",
  "apellido": "Lopez",
  "razon_social": null,
  "nombre_comercial": null,
  "direccion": "Guayaquil",
  "silenciado": false,
  "metadatos": {
    "segmento": "retail",
    "origen": "landing_web"
  },
  "contactos": [
    {
      "nombre": "PRUEBA",
      "apellido": "",
      "movil": "+593999999999",
      "correo_electronico": "prueba@mail.com",
      "tipo_contacto_email": "TO"
    },
    {
      "nombre": "PRUEBA",
      "apellido": "",
      "movil": "+593999999999",
      "correo_electronico": null,
      "tipo_contacto_email": "CC"
    }
  ]
}
```

### Comportamiento

- No pueden existir dos clientes con la misma identificación dentro del mismo partner.
- Para persona natural se usa nombre y apellido.
- Para empresa se usa razon_social y nombre_comercial.
- metadatos es opcional.
- silenciado es opcional (por defecto false).

### Respuesta

```json title="Respuesta 201"
{
  "id": 12342,
  "identificacion": "0999999999001",
  "nombre": "Andres",
  "apellido": "Lopez",
  "razon_social": null,
  "nombre_comercial": null,
  "direccion": "Guayaquil",
  "silenciado": false,
  "metadatos": {
    "segmento": "retail",
    "origen": "landing_web"
  },
  "contactos": [
    {
      "id_contacto": 12345,
      "id_cliente": 12345,
      "nombre": "PRUEBA",
      "apellido": "",
      "movil": "+593999999999",
      "correo_electronico": null,
      "tipo_contacto_email": "TO"
    },
    {
      "id_contacto": 12346,
      "id_cliente": 12345,
      "nombre": "PRUEBA",
      "apellido": "",
      "movil": "+593999999999",
      "correo_electronico": null,
      "tipo_contacto_email": "CC"
    }
  ]
}
```

## Modificar Cliente (PUT)

```bash title="URL"
PUT https://app.conpronto.com/api/v1/customers/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body"
{
  "identificacion": "0999999999001",
  "nombre": "Andres",
  "apellido": "Lopez",
  "razon_social": null,
  "nombre_comercial": null,
  "direccion": "Guayaquil",
  "silenciado": false,
  "metadatos": {
    "segmento": "retail",
    "origen": "landing_web"
  },
  "contactos": [
    {
      "nombre": "PRUEBA",
      "apellido": "",
      "movil": "+593999999999",
      "correo_electronico": "prueba@mail.com",
      "tipo_contacto_email": "TO"
    },
    {
      "nombre": "PRUEBA",
      "apellido": "",
      "movil": "+593999999999",
      "correo_electronico": null,
      "tipo_contacto_email": "CC"
    }
  ]
}
```

### Comportamiento

- Solo se actualizan los campos enviados.
- Si se envía metadatos reemplaza completamente el objeto anterior.
- El cliente debe pertenecer al partner autenticado.
- Nota: Si no se pasa el arreglo de contactos se mantendrá los contactos existentes y en caso de pasar el arrelgo de
  contactos, eliminará los contactos existentes y agregará los nuevos contactos

### Respuesta

[Devuelve el objeto Cliente.](#objeto-cliente)

```bash title="Respuesta"
201 Created
```

## Eliminar Cliente (DELETE)

```bash title="URL"
DELETE https://app.conpronto.com/api/v1/customers/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Comportamiento

- No se podrá eliminar si el cliente tiene:
- Documentos asociados
- Pagos asociados
- Contactos asociados
- En caso de tener dependencias, se retornará error 400.

### Respuesta

```bash title="Respuesta"
204 Not Content
```

## Códigos de Respuesta

| Código | Descripción                        |
|--------|------------------------------------|
| 204    | Contactos eliminados correctamente |
| 400    | Cliente no existe                  |
| 401    | Token inválido o expirado          |
| 403    | Sin permisos                       |