# Registro de Tarjeta y Pagos Recurrentes

Permite generar un enlace público para que un cliente registre su tarjeta y consultar las tarjetas registradas junto con los pagos recurrentes asociados.

## Requisitos

- Autenticación JWT mediante Bearer Token.
- Cliente existente en Pronto.
- El cliente debe pertenecer al partner autenticado.
- El partner debe tener habilitados los débitos recurrentes en Datafast.

---

## Generar Link de Registro de Tarjeta (POST)

Genera un enlace público para que el cliente acceda al formulario de registro de tarjeta.

```bash title="URL"
POST https://app.conpronto.com/api/v1/recurrent/
```

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body (ejemplo)"
{
  "customer_id": 297453
}
```

### Atributos

| Parámetro     | Tipo    | Obligatorio | Descripción                                      |
|---------------|---------|-------------|--------------------------------------------------|
| `customer_id` | integer | Sí          | Identificador del cliente en Pronto.             |

### Comportamiento

- No crea todavía una tarjeta ni un pago recurrente.
- Valida que el cliente pertenezca al partner autenticado.
- Valida que el partner tenga habilitados los débitos recurrentes.
- Devuelve una URL pública para compartir con el cliente.

### Respuesta

```json title="Respuesta (ejemplo)"
{
  "payment_link": "https://console.conpronto.com/register_card/68040?t=null",
  "customer_id": 297453,
  "estado": "Disponible"
}
```

### Atributos de respuesta

| Campo          | Tipo    | Descripción                                      |
|----------------|---------|--------------------------------------------------|
| `payment_link` | string  | URL pública para registrar la tarjeta.           |
| `customer_id`  | integer | Identificador del cliente.                       |
| `estado`       | string  | Estado de disponibilidad del enlace.             |

---

## Consultar Registros por Cliente (GET)

Consulta las tarjetas registradas de un cliente y los pagos recurrentes asociados a cada tarjeta.

```bash title="URL"
GET https://app.conpronto.com/api/v1/recurrent/?customer_id=297453
```

### Ejemplo

```bash
curl --location '{{base_url}}/api/v1/recurrent/?customer_id=297453' \
--header 'Authorization: Bearer {{token}}' \
--header 'Accept: application/json'
```

### Parámetros de consulta

| Parámetro     | Tipo    | Obligatorio | Descripción                         |
|---------------|---------|-------------|-------------------------------------|
| `customer_id` | integer | Sí          | Identificador del cliente.          |

Si el cliente no tiene tarjetas registradas, la API devuelve un arreglo vacío:

```json
[]
```

### Respuesta

```json
[
  {
    "id": 10,
    "id_cliente": 297453,
    "cliente": "Nombre del cliente",
    "tipo_tarjeta": "VISA",
    "ultimos_digitos": "8211",
    "exp_month": 1,
    "exp_year": 2031,
    "cardholder_name": "NOMBRE DEL TITULAR",
    "months": 12,
    "amount": 39.71,
    "start_date": "2026-08-04",
    "estado": "PAGADO",
    "tiene_pagos": true,
    "cantidad_pagos": 3,
    "pagos_exitosos": 2,
    "pagos_fallidos": 1,
    "ultimo_pago": {
      "id": 47,
      "cycle_number": 11,
      "scheduled_date": "2026-08-04",
      "fecha_pago": "2026-08-04T09:10:16.593553",
      "monto": "39.71",
      "estado": "PAGADO",
      "referencia": "8acda49f9fa80a12019fcd1c56e901af",
      "error_message": ""
    },
    "pagos": []
  }
]
```

### Estados

| Valor      | Descripción                                                    | 
|------------|----------------------------------------------------------------|
| `GENERADO` | La tarjeta está registrada, pero aún no tiene un pago exitoso. |
| `PAGADO`   | Existe al menos un pago recurrente procesado correctamente.    |
| `ERROR`    | El registro presenta pagos fallidos y no pagos exitosos.       |

### Objeto de respuesta

| Campo               | Tipo          | Descripción                                       |
|---------------------|---------------|---------------------------------------------------|
| `id`                | integer       | Identificador del registro de tarjeta.            |
| `id_cliente`        | integer       | Identificador del cliente.                        |
| `cliente`           | string        | Nombre completo del cliente.                      |
| `tipo_tarjeta`      | string        | Marca de la tarjeta.                              |
| `ultimos_digitos`   | string        | Últimos cuatro dígitos de la tarjeta.             |
| `exp_month`         | integer       | Mes de expiración.                                |
| `exp_year`          | integer       | Año de expiración.                                |
| `cardholder_name`   | string        | Nombre del titular.                               |
| `months`            | integer/null  | Periodicidad configurada en meses.                |
| `amount`            | decimal/null  | Monto configurado para el cobro recurrente.       |
| `start_date`        | date/null     | Fecha de inicio de la recurrencia.                |
| `estado`            | string        | Estado general del registro.                      |
| `tiene_pagos`       | boolean       | Indica si existen cobros recurrentes.             |
| `cantidad_pagos`    | integer       | Total de cobros registrados.                      |
| `pagos_exitosos`    | integer       | Cantidad de cobros procesados correctamente.      |
| `pagos_fallidos`    | integer       | Cantidad de cobros fallidos.                      |
| `ultimo_pago`       | object/null   | Último cobro procesado o registrado.              |
| `pagos`             | array         | Detalle de los cobros recurrentes.                |

---

## Consultar un Registro por ID (GET)

Consulta una tarjeta registrada junto con todos sus pagos recurrentes asociados.

```bash title="URL"
GET https://app.conpronto.com/api/v1/recurrent/{id}/
```

### Ejemplo

```bash
curl --location '{{base_url}}/api/v1/recurrent/10/' \
--header 'Authorization: Bearer {{token}}' \
--header 'Accept: application/json'
```

### Parámetros de ruta

| Parámetro | Tipo    | Obligatorio | Descripción                              |
|-----------|---------|-------------|------------------------------------------|
| `id`      | integer | Sí          | ID del registro en `CustomerDatafastCard`. |

La respuesta tiene la misma estructura del objeto incluido en `results` de la consulta por cliente, pero devuelve un único registro.

Si el registro no existe o pertenece a otro partner:

```json
{
  "error": "No se encontró el registro de tarjeta o no pertenece al partner autenticado"
}
```

---

## Pagos recurrentes

Cada elemento de `pagos` corresponde a un registro de `DatafastRecurrentPayment`.

| Campo              | Tipo          | Descripción                                   |
|--------------------|---------------|-----------------------------------------------|
| `id`               | integer       | Identificador del cobro.                      |
| `cycle_number`     | integer       | Número de cuota o ciclo.                      |
| `scheduled_date`   | date/null     | Fecha programada para el cobro.               |
| `fecha_pago`       | datetime/null | Fecha y hora de procesamiento.                |
| `monto`            | decimal       | Monto cobrado.                                |
| `estado`           | string        | Estado general del cobro.                     |
| `referencia`       | string/null   | Referencia devuelta por Datafast.             |
| `error_message`    | string        | Mensaje de error, si el cobro falló.          |

### Estados de los pagos recurrentes

| Estado interno | Estado expuesto | Descripción                         |
|----------------|-----------------|-------------------------------------|
| `pending`      | `GENERADO`      | Cobro pendiente de ejecución.       |
| `success`      | `PAGADO`        | Cobro procesado correctamente.      |
| `failed`       | `ERROR`         | Cobro procesado con error.          |

