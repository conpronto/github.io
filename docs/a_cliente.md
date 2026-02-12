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
  }
}
```

Atributos del objeto Cliente
----------------------------------------------------------------

| Parámetro        | Tipo    | Descripción                                        
|------------------|---------|----------------------------------------------------
| id               | integer | Identificador interno del cliente                  
| identificacion   | string  | Cédula o RUC del cliente                           
| nombre           | string  | Nombre (persona natural)                           
| apellido         | string  | Apellido (persona natural)                         
| razon_social     | string  | Razón social (empresa)                             
| nombre_comercial | string  | Nombre comercial (empresa)                         
| direccion        | string  | Dirección del cliente                              
| silenciado       | boolean | Indica si el cliente está silenciado               
| metadatos        | object  | Objeto JSON con información adicional estructurada 

## Obtener un Cliente (GET)

```bash title="URL"
GET https://app.conpronto.com/api/v2/customers/{id}/
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
GET https://app.conpronto.com/api/v2/customers/
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

### Ejemplo con paginación

```json title="Respuesta paginada"
{
  "count": 713,
  "next": "https://app.conpronto.com/api/v2/customers/?p=2&page_size=50",
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
POST https://app.conpronto.com/api/v2/customers/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

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
  }
}
```

Reglas

- No pueden existir dos clientes con la misma identificación dentro del mismo partner.
- Para persona natural se usa nombre y apellido.
- Para empresa se usa razon_social y nombre_comercial.
- metadatos es opcional.
- silenciado es opcional (por defecto false).

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
  }
}
```

## Modificar Cliente (PUT)

```bash title="URL"
PUT https://app.conpronto.com/api/v2/customers/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

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
  }
}
```

Reglas

- Solo se actualizan los campos enviados.
- Si se envía metadatos reemplaza completamente el objeto anterior.
- El cliente debe pertenecer al partner autenticado.

Respuesta 200:

[Devuelve el objeto Cliente.](#objeto-cliente)

## Eliminar Cliente (DELETE)

```bash title="URL"
DELETE https://app.conpronto.com/api/v2/customers/{id}/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

```bash title="Respuesta"
204 Not Content
```

Consideraciones

- No se podrá eliminar si el cliente tiene:
- Documentos asociados
- Pagos asociados
- Contactos asociados
- En caso de tener dependencias, se retornará error 400.