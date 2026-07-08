# Link de Pago

Permite generar un link de pago único para uso externo y consultar posteriormente el estado y detalle de los pagos
realizados.

## Requisitos

- Autenticación JWT: ver [Introducción](index.md#autenticacion).
- Cliente existente (o creado previamente): ver [Clientes](a_cliente.md).
- Webhook (opcional): configuración `url_webhook` en la Cuenta.

---

## Crear Link de Pago (POST)

Genera un link único para que el usuario final realice el pago desde un entorno externo.

```bash title="URL"
POST https://app.conpronto.com/api/v1/paylinks/
```

> Nota: si su ambiente utiliza una ruta distinta, este endpoint puede variar según configuración del Partner.

```bash title="Headers"
Authorization: Bearer <access_token>
Content-Type: application/json
```

### Estructura del JSON

```json title="Body (ejemplo)"
{
  "customer_id": 123445,
  "monto": 10.50,
  "descripcion": "Pago único desde API externa"
}
```

### Atributos

| Parámetro     | Tipo    | Obligatorio | Descripción                                            |
|---------------|---------|-------------|--------------------------------------------------------|
| `customer_id` | integer | Sí          | Identificador del cliente en Pronto.                   |
| `monto`       | decimal | Sí          | Valor del pago. Debe ser mayor a 0.                    |
| `descripcion` | string  | No          | Descripción del pago. Longitud máxima: 255 caracteres. |

### Comportamiento

- El monto debe ser mayor a 0.
- Cada solicitud genera un link único.
- El link podrá ser consultado posteriormente mediante la API.

### Respuesta

```json title="Respuesta (ejemplo)"
{
  "paylink_id": 98765,
  "payment_link": "https://app.conpronto.com/pay/xxxxxxxx",
  "estado": "Created"
}
```

### Atributos de respuesta

| Campo          | Tipo    | Longitud | Descripción                        |
|----------------|---------|----------|------------------------------------|
| `paylink_id`   | integer | -        | Identificador del link de pago.    |
| `payment_link` | string  | 500      | URL pública para realizar el pago. |
| `estado`       | string  | 30       | Estado inicial del link generado.  |

---

## Ejecución del Pago

El usuario accede al `payment_link` y realiza el pago mediante la pasarela configurada.

Una vez finalizado el proceso, Pronto notificará el resultado al Webhook configurado (si existe).

---

## Webhook

El webhook se enviará únicamente si la Cuenta posee configurada una `url_webhook`.

### Estados posibles

- `done`
- `error`

### Evento exitoso

```json
{
  "estado": "done",
  "paylink_id": 98765,
  "data": {
    "codigo": "000.100.110",
    "customer_id": 123445,
    "estado": "paid",
    "monto": 10.50,
    "no_autorizacion": "2213"
  }
}
```

### Evento con error

```json
{
  "estado": "error",
  "paylink_id": 98765,
  "data": {
    "codigo": "800.100.152",
    "mensaje": "Detalle del error"
  }
}
```

### Objeto del webhook

| Campo        | Tipo    | Descripción                                  |
|--------------|---------|----------------------------------------------|
| `estado`     | string  | Estado del procesamiento (`done` o `error`). |
| `paylink_id` | integer | Identificador del link de pago.              |
| `data`       | object  | Información del resultado del procesamiento. |

---

## Consulta de Links de Pago

## Listar Links de Pago (GET)

Permite consultar el historial de links de pago generados para la cuenta autenticada.

```bash title="URL"
GET https://app.conpronto.com/api/v1/paylinks/
```

### Ejemplo

```bash
curl --location '{{base_url}}/api/v1/paylinks/?search=Consumidor&estado=PAGADO&fecha=2026-07-08&origen_pago=2&ordering=-created_date&p=1&page_size=10&id_cliente=790224' \
--header 'Authorization: Bearer {{token}}'
```

### Parámetros de consulta

| Parámetro        | Tipo    | Obligatorio | Descripción                                                                                                                  |
|------------------|---------|-------------|------------------------------------------------------------------------------------------------------------------------------|
| `search`         | string  | No          | Busca por identificación, nombres, apellidos, razón social, nombre comercial, descripción, checkout o número de transacción. |
| `estado`         | string  | No          | Estado general del pago (`GENERADO`, `PAGADO`, `ERROR`).                                                                     |
| `fecha_inicio`   | date    | No          | Fecha de creación del link. Formato: `yyyy-mm-dd`.                                                                           |
| `fecha_fin`      | date    | No          | Fecha de creación del link. Formato: `yyyy-mm-dd`.                                                                           |
| `origen_pago`    | integer | No          | Origen desde donde fue generado el link.                                                                                     |
| `id_cliente`     | integer | No          | Identificador del cliente.                                                                                                   |
| `ordering`       | string  | No          | Ordenamiento (`created_date`, `completed_date`, `amount`, `status`). Anteponer `-` para descendente.                         |
| `p`              | integer | No          | Número de página.                                                                                                            |
| `page_size`      | integer | No          | Cantidad de registros por página.                                                                                            |

---

### Estados

| Valor      | Descripción                                             |
|------------|---------------------------------------------------------|
| `GENERADO` | El link fue generado y aún no registra un pago exitoso. |
| `PAGADO`   | El pago fue procesado correctamente.                    |
| `ERROR`    | El pago presentó algún error durante el procesamiento.  |

---

### Origen del Pago

| Código | Descripción |
|--------|-------------|
| `0`    | Campaña     |
| `1`    | Manual      |
| `2`    | API         |

---

### Respuesta

```json
{
  "count": 12,
  "next": "https://app.conpronto.com/api/v1/paylinks/?p=2",
  "previous": null,
  "results": [
    {
      "id": 9671,
      "cliente": "Consumidor Final",
      "monto": "0.12",
      "origen_pago": "Pago vía api",
      "tipo_tarjeta": null,
      "lote": null,
      "numero_autorizacion": null,
      "estado": "GENERADO",
      "descripcion": "Pago unico desde API externa",
      "fecha_creacion": "2026-07-08T08:05:17.813256",
      "fecha_pago": "2026-07-08T08:05:17.807783",
      "documentos": []
    }
  ]
}
```

---

### Objeto de Respuesta

### Respuesta Principal

| Campo      | Tipo          | Longitud | Descripción                     |
|------------|---------------|----------|---------------------------------|
| `count`    | integer       | -        | Total de registros encontrados. |
| `next`     | string / null | -        | URL de la siguiente página.     |
| `previous` | string / null | -        | URL de la página anterior.      |
| `results`  | array         | -        | Lista de links de pago.         |

---

### Objeto `results[]`

| Campo                 | Tipo          | Longitud | Descripción                                                                                                                         |
|-----------------------|---------------|----------|-------------------------------------------------------------------------------------------------------------------------------------|
| `id`                  | integer       | -        | Identificador del link de pago.                                                                                                     |
| `cliente`             | string        | 255      | Nombre completo del cliente.                                                                                                        |
| `monto`               | decimal       | 12,2     | Valor total del link de pago.                                                                                                       |
| `origen_pago`         | string        | 30       | Origen desde donde fue generado el link de pago.                                                                                    |
| `tipo_tarjeta`        | string / null | 20       | Marca de la tarjeta utilizada (`VISA`, `MASTER`, `DINERS`, `AMEX`, etc.). Puede ser `null` mientras el pago no haya sido realizado. |
| `lote`                | string / null | 20       | Número de lote asignado por la pasarela de pago.                                                                                    |
| `numero_autorizacion` | string / null | 20       | Número de autorización emitido por la entidad financiera.                                                                           |
| `estado`              | string        | 15       | Estado general del pago (`GENERADO`, `PAGADO` o `ERROR`).                                                                           |
| `descripcion`         | string / null | 255      | Descripción registrada al crear el link.                                                                                            |
| `fecha_creacion`      | datetime      | -        | Fecha y hora de creación del link de pago.                                                                                          |
| `fecha_pago`          | datetime      | -        | Fecha y hora del procesamiento del pago.                                                                                            |
| `documentos`          | array         | -        | Lista de documentos asociados al pago. Puede venir vacía.                                                                           |

---

### Objeto `documentos[]`

| Campo              | Tipo    | Longitud | Descripción                                      |
|--------------------|---------|----------|--------------------------------------------------|
| `id_documento`     | integer | -        | Identificador interno del documento.             |
| `numero_documento` | string  | 100      | Número del documento.                            |
| `monto`            | decimal | 12,2     | Valor aplicado a dicho documento.                |

---

## Obtener un Link de Pago (GET)

Obtiene el detalle completo de un link de pago específico.

```bash title="URL"
GET https://app.conpronto.com/api/v1/paylinks/{id}/
```

### Ejemplo

```bash
curl --location '{{base_url}}/api/v1/paylinks/1234/' \
--header 'Authorization: Bearer {{token}}'
```

### Respuesta

La respuesta corresponde exactamente al objeto descrito anteriormente en `results[]`, sin la estructura de paginación.

```json
{
  "id": 1234,
  "cliente": "CONSUMIDOR FINAL",
  "monto": "788.59",
  "origen_pago": "Pago por Campaña",
  "tipo_tarjeta": "DINERS",
  "lote": "260424",
  "numero_autorizacion": "xxxxxx",
  "estado": "PAGADO",
  "descripcion": null,
  "fecha_creacion": "2026-07-06T18:54:36.689870",
  "fecha_pago": "2026-07-06T18:56:12.640304",
  "documentos": [
    {
      "id_documento": 5544567,
      "numero_documento": "FDIS4326",
      "monto": "788.59"
    }
  ]
}
```

---

## Reglas Generales

- El monto del link debe ser mayor a cero.
- Cada solicitud genera un link de pago único.
- Los estados expuestos por la API son únicamente:
    - `GENERADO`
    - `PAGADO`
    - `ERROR`
- La consulta de un link individual devuelve el mismo objeto utilizado en el listado, sin paginación.