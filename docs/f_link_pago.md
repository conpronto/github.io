# Link de Pago

Permite generar un link de pago unico para uso externo y recibir el resultado final del procesamiento mediante webhook.

## Requisitos

- Autenticacion JWT: ver [Introduccion](index.md#autenticacion).
- Cliente existente (o creado previamente): ver [Clientes](a_cliente.md).
- Webhook (opcional): configuracion `url_webhook` en la Cuenta.

## Crear Link de Pago (POST)

Genera un link unico para que el usuario final realice el pago desde un entorno externo.

```bash title="URL"
POST https://app.conpronto.com/api/v1/paylinks/
```

> Nota: si su ambiente utiliza una ruta distinta, este endpoint puede variar segun configuracion del Partner.

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body (ejemplo)"
{
  "customer_id": 123445,
  "monto": 10.5,
  "descripcion": "Pago unico desde API externa",
}
```
### Atributos

| Parametro     | Tipo    | Obligatorio | Descripcion |
|--------------|---------|-------------|-------------|
| `customer_id` | integer | Si          | Identificador del cliente en Pronto |
| `monto`       | number  | Si          | Valor del pago (debe ser mayor a 0) |
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

| Campo          | Tipo    | Descripcion |
|---------------|---------|-------------|
| `paylink_id`   | integer | Identificador del link de pago |
| `payment_link` | string  | URL publica para ejecutar el pago |
| `estado`       | string  | Estado inicial del link (ej: `created`) |

## Ejecucion del pago

El usuario accede al `payment_link` y realiza el pago mediante la pasarela. Al finalizar el procesamiento, Pronto notifica el resultado al webhook (si aplica).

## Webhook

El webhook se envia **solo** si `url_webhook` esta configurado en la Cuenta. Si no esta configurado, no se envian eventos.

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

| Campo        | Tipo    | Descripcion |
|-------------|---------|-------------|
| `estado`     | string  | `done` o `error` |
| `paylink_id` | integer | Identificador del link de pago |
| `data`       | object  | Datos finales del procesamiento (contenido segun configuracion/implementacion) |

## Reglas generales

- El monto debe ser mayor a 0.
- Cada pago genera un link unico.
