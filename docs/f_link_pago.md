# Link de Pago

Permite generar un link de pago unico para uso externo y recibir el
resultado final del procesamiento mediante webhook.

## Requisitos

- Autenticacion JWT: ver [Introduccion](index.md#autenticacion).
- Cliente existente (o creado previamente): ver
  [Clientes](a_cliente.md).
- Webhook (opcional): configuracion `url_webhook` en la Cuenta.

## Crear Link de Pago (POST)

Genera un link unico para que el usuario final realice el pago desde un
entorno externo.

```bash title="URL"
POST https://app.conpronto.com/api/v1/paylinks/
```

> Nota: si su ambiente utiliza una ruta distinta, este endpoint puede
> variar segun configuracion del Partner.

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body (ejemplo)"
{
  "customer_id": 123445,
  "monto": 10.5,
  "descripcion": "Pago unico desde API externa"
}
```

### Atributos

| Parametro     | Tipo    | Obligatorio | Descripcion                            |
|---------------|---------|-------------|----------------------------------------|
| `customer_id` | integer | Si          | Identificador del cliente en Pronto    |
| `monto`       | number  | Si          | Valor del pago (debe ser mayor a 0)    |
| `descripcion` | string  | No          | Descripcion visible/operativa del pago |

[//]: # (| `metadatos`   | object  | No          | Objeto JSON para informacion adicional. Debe incluir `origen = external_api` para registrar el origen |)

### Comportamiento

- `monto` debe ser mayor a `0`.
- Cada solicitud crea un link unico.

### Respuesta

```json title="Respuesta (ejemplo)"
{
  "paylink_id": 98765,
  "payment_link": "https://app.conpronto.com/pay/xxxxxxxx",
  "estado": "created"
}
```

| Campo          | Tipo    | Descripcion                             |
|----------------|---------|-----------------------------------------|
| `paylink_id`   | integer | Identificador del link de pago          |
| `payment_link` | string  | URL publica para ejecutar el pago       |
| `estado`       | string  | Estado inicial del link (ej: `created`) |

## Ejecucion del pago

El usuario accede al `payment_link` y realiza el pago mediante la pasarela. Al finalizar el procesamiento, Pronto
notifica el resultado al webhook (si aplica).

## Webhook

El webhook se envia **solo** si `url_webhook` esta configurado en la Cuenta. Si no esta configurado, no se envian
eventos.

### Estados

- `done`: pago procesado correctamente
- `error`: pago fallido o no procesado

### Evento (payload)

Se envia un `POST` al `url_webhook` del Partner con `Content-Type: application/json`.

```json title="Webhook payload (ejemplo)"
{
  "estado": "done",
  "paylink_id": 98765,
  "data": {
    "codigo": "xxx",
    "customer_id": 123445,
    "estado": "paid",
    "monto": 10.5,
    "no_autorizacion": "2213",
    "cuotas": 0,
    "tipo_credito": "Corriente"
  }
}
```

En caso de error:

```json title="Webhook payload error (ejemplo)"
{
  "estado": "error",
  "paylink_id": 98765,
  "data": {
    "codigo": "xxxxxx",
    "mensaje": "Detalle del error"
  }
}
```

### Atributos del webhook

| Campo        | Tipo    | Descripcion                                                                    |
|--------------|---------|--------------------------------------------------------------------------------|
| `estado`     | string  | `done` o `error`                                                               |
| `paylink_id` | integer | Identificador del link de pago                                                 |
| `data`       | object  | Datos finales del procesamiento (contenido segun configuracion/implementacion) |

## Reglas generales

- El monto debe ser mayor a 0.
- Cada pago genera un link unico.

------------------------------------------------------------------------

## Consulta de Links de Pago

## Listar Links de Pago (GET)

Permite consultar el historial de links de pago generados para la cuenta
autenticada.

```bash title="URL" 
GET https://app.conpronto.com/api/v1/paylinks/
```

```bash title="Curl" 
 curl --location '{{base_url}}/api/v1/paylinks/?search=Consumidor&status=2&tipo_pago=1&ordering=-created_date&p=1&page_size=1&id_cliente=790224' \ --header 'Authorization: Bearer {{token}}'
```

### Parámetros de consulta

| Parámetro    | Tipo    | Obligatorio                 | Descripción                                                            |
|--------------|---------|-----------------------------|------------------------------------------------------------------------| 
| `search`     | string  | No                          | Busca por cliente, descripción, checkout o transacción.                |
| `status`     | integer | No Estado del link de pago. |
| `tipo_pago`  | integer | No                          | Tipo de uso del link de pago.                                          |
| `id_cliente` | integer | No                          | Filtra por cliente.                                                    |
| `ordering`   | string  | No                          | Ordenamiento(`-created_date`,`created_date`,`amount`, `status`, etc.). 
| `p`          | integer | No                          | Número de página.                                                      |
| `page_size`  | integer | No                          | Cantidad de registros por página.                                      |

### Estados

| Código | Descripción     |
|--------|-----------------|
| 0      | Created         |
| 1      | Canceled        |
| 2      | Generated Link  |
| 3      | Sent Link       |
| 4      | Paid            |
| 5      | Retired         |
| 6      | Error Generated |
| 7      | Error Sent      |

### Tipos de pago

| Código | Descripción      |
|--------|------------------|
| 0      | Pago por Campaña |
| 1      | Pago Directo     |
| 2      | Pago vía API     |

### Respuesta

``` json
{
  "count":1,
  "next":null,
  "previous":null,
  "results":[
    {
      "id":9694,
      "cliente":"SAIRY ISRAEL REMACHE TERAN",
      "monto":"788.59",
      "tipo_pago":"Pago por Campaña",
      "estado":"Paid",
      "descripcion":null,
      "fecha_creacion":"2026-07-06T18:54:36.689870",
      "fecha_pago":"2026-07-06T18:56:12.640304",
      "documentos":[
        {
          "id_documento":5544567,
          "numero_documento":"FDIS4326",
          "tipo_documento":0,
          "monto":"788.59"
        }
      ]
    }
  ]
}
```

### Objeto de respuesta

| Campo          | Tipo     | Longitud | Descripción                    |
|----------------|----------|----------|--------------------------------|
| id             | integer  | -        | Identificador del link de pago |
| cliente        | string   | -        | Nombre del cliente             |
| monto          | decimal  | 12,2     | Monto total                    |
| tipo_pago      | string   | 30       | Tipo de pago                   |
| estado         | string   | 30       | Estado actual                  |
| descripcion    | string   | 255      | Descripción del pago           |
| fecha_creacion | datetime | -        | Fecha de creación              |
| fecha_pago     | datetime | -        | Fecha de pago                  |
| documentos     | array    | -        | Documentos asociados           |

### Objeto documentos

| Campo            | Tipo    | Longitud | Descripción          
|------------------|---------|----------|----------------------|
| id_documento     | integer | -        | Id del documento     |
| numero_documento | string  | 100      | Número del documento |
| monto            | decimal | 12,2     | Monto pagado         |

## Obtener un Link de Pago (GET)

```bash title="URL" 
GET https://app.conpronto.com/api/v1/paylinks/{id}/
```

```bash title="Curl" 
curl --location '{{base_url}}/api/v1/paylinks/9694/' \ --header 'Authorization: Bearer {{token}}'
```

La respuesta corresponde al mismo objeto descrito anteriormente, sin paginación.
