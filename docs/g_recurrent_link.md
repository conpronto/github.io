# Registro de Tarjeta y Pagos Recurrentes

Permite generar un enlace público para que un cliente registre su tarjeta y consultar las tarjetas registradas junto con los pagos recurrentes asociados.

## Requisitos

- Autenticación JWT mediante Bearer Token.
- Cliente existente en Pronto.
- El cliente debe pertenecer al partner autenticado.
- El partner debe tener habilitados los débitos recurrentes en la Pasarela configurada.

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
| `customer_id` | integer | Sí | Identificador del cliente en Pronto. |
| `fecha_inicio` | date/null | No | Fecha de inicio en formato `YYYY-MM-DD`. Si se informa, requiere `numero_cuotas`. |
| `numero_cuotas` | integer/null | Condicional | Entre 1 y 32767. Obligatorio cuando se informa `fecha_inicio`; es una referencia y no limita el número de débitos. |
| `monto` | decimal/null | No | Monto de referencia positivo, hasta 9999999999.99 y máximo dos decimales. El débito real usa el saldo del documento. |

### Ejemplo con recurrencia

```bash
curl --request POST 'https://app.conpronto.com/api/v1/recurrent/' \
  --header 'Authorization: Bearer <access_token>' \
  --header 'Content-Type: application/json' \
  --data '{"customer_id":297453,"fecha_inicio":"2026-11-06","numero_cuotas":12,"monto":"39.71"}'
```

`monto` puede omitirse o enviarse como `null`; ello no impide cobrar los saldos pendientes. Sin `fecha_inicio`, la tarjeta puede registrarse, pero no participa en la recurrencia automática. Los campos opcionales omitidos dentro de un plan se normalizan a `null`.

Si se envía una fecha sin cuotas, se devuelve HTTP **400**:

```json
{
  "numero_cuotas": ["Ingrese el número de cuotas cuando indique una fecha de inicio."]
}
```

### Comportamiento

- No crea todavía una tarjeta ni un pago recurrente.
- Valida que el cliente pertenezca al partner autenticado.
- Valida que el partner tenga habilitados los débitos recurrentes.
- Devuelve HTTP **201** y una URL pública para compartir con el cliente.
- La configuración viaja en un token firmado y se guarda al completar el registro de tarjeta. El enlace debe utilizarse completo, sin modificar su token.
- Con un plan, la respuesta incluye `fecha_inicio`, `numero_cuotas`, `monto` y `fecha_fin`. Esta última es inicio más `(numero_cuotas - 1)` meses, o `null` sin datos suficientes. No constituye un límite de cobro.

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

### Respuesta con plan (ejemplo)

```json
{
  "payment_link": "https://console.conpronto.com/register_card/<unique_id>?t=plan.<token_firmado>",
  "customer_id": 297453,
  "estado": "Disponible",
  "fecha_inicio": "2026-11-06",
  "numero_cuotas": 12,
  "monto": "39.71",
  "fecha_fin": "2027-10-06"
}
```

La URL del ejemplo es ilustrativa: usa el `payment_link` devuelto por la API.

## Reglas de ejecución automática

- Considera tarjetas habilitadas y activas, de clientes y partners activos, con fecha de inicio alcanzada y token registrado.
- Usa la **fecha de emisión del documento**, no la fecha de importación o creación del registro en Pronto. Selecciona documentos activos, no anulados y con saldo positivo desde `fecha_inicio`, inclusive, hasta el último corte mensual alcanzado.
- Mantiene el día mensual del inicio; si el mes no tiene ese día, usa el último día del mes. Recupera documentos pendientes si el proceso no se ejecutó en su fecha.
- Ejemplo: inicio **02/09/2026**, ejecución **06/11/2026**: considera documentos fechados entre **02/09/2026 y 02/11/2026**, inclusive. No se limita a documentos de noviembre.
- Genera un intento por documento y agrupa los detalles en un lote automático por partner y día. Relee el saldo antes de enviar el débito; `monto`, `numero_cuotas` y la fecha fin calculada son referencias, no límites.
- Los intentos rechazados de forma confirmada o no enviados pueden reintentarse después de **3 días**. Conserva los intentos anteriores.
- Un resultado incierto (`REVISION`) no se vuelve a debitar automáticamente. Tampoco se repite un cobro aprobado pendiente de aplicación al documento.
- Los débitos manuales no consumen los intentos automáticos; el proceso automático usa el saldo pendiente vigente.
- Solo aplica el cobro al documento después de confirmar la aprobación. Si pertenece a Contífico, intenta registrar allí el cobro; si esa aplicación no se confirma, conserva el caso para revisión sin volver a debitar.
- Crear el enlace o habilitar la tarjeta no ejecuta un cobro inmediato: la procesa la siguiente ejecución programada cuando corresponda.

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
    "is_enable": true,
    "cliente": "Nombre del cliente",
    "tipo_tarjeta": "VISA",
    "ultimos_digitos": "8211",
    "exp_month": 1,
    "exp_year": 2031,
    "cardholder_name": "NOMBRE DEL TITULAR",
    "months": 12,
    "amount": 39.71,
    "start_date": "2026-08-04",
    "end_date": "2027-07-04",
    "estado": "PAGADO",
    "tiene_pagos": true,
    "cantidad_pagos": 3,
    "pagos_exitosos": 2,
    "pagos_fallidos": 1,
    "ultimo_pago": {
      "id": 47,
      "cycle_number": 11,
      "is_automatic": true,
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
| `months` | integer/null | Número de cuotas de referencia; no limita los débitos automáticos. |
| `amount` | number/null | Monto de referencia; se debita el saldo actual del documento. |
| `start_date`        | date/null     | Fecha de inicio de la recurrencia.                |
| `end_date` | date/null | Fecha fin de referencia calculada; no desactiva automáticamente la tarjeta. |
| `is_enable` | boolean | `true`: habilitada; `false`: excluida de nuevos débitos automáticos. |
| `estado` | string | Resumen del historial de pagos; no indica si la tarjeta está habilitada. |
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

| Parámetro | Tipo    | Obligatorio | Descripción                             |
|-----------|---------|-------------|-----------------------------------------|
| `id`      | integer | Sí          | ID del registro. |

La respuesta tiene la misma estructura de cada objeto del arreglo devuelto por la consulta por cliente, pero devuelve un único registro.

Si el registro no existe o pertenece a otro partner:

```json
{
  "error": "No se encontró el registro de tarjeta o no pertenece al partner autenticado"
}
```

---

## Pagos recurrentes

| Campo              | Tipo          | Descripción                          |
|--------------------|---------------|--------------------------------------|
| `id`               | integer       | Identificador del cobro.             |
| `cycle_number` | integer | Secuencia del intento automático; no equivale necesariamente a una mensualidad. |
| `is_automatic` | boolean | Indica si el intento pertenece al proceso automático. |
| `scheduled_date`   | date/null     | Fecha programada para el cobro.      |
| `fecha_pago`       | datetime/null | Fecha y hora de procesamiento.       |
| `monto`            | decimal       | Monto cobrado.                       |
| `estado`           | string        | Estado general del cobro.            |
| `referencia`       | string/null   | Referencia devuelta por la Pasarela. |
| `error_message`    | string        | Mensaje de error, si el cobro falló. |

### Estados de los pagos recurrentes

| Estado interno | Estado expuesto | Descripción                         |
|----------------|-----------------|-------------------------------------|
| `pending`      | `GENERADO`      | Cobro pendiente de ejecución.       |
| `success`      | `PAGADO`        | Cobro procesado correctamente.      |
| `failed`       | `ERROR`         | Cobro procesado con error.          |
| `review` | `REVISION` | Resultado incierto que requiere revisión; no se reintenta automáticamente. |


## Inhabilitar tarjeta (POST)

```text
POST https://app.conpronto.com/api/v1/recurrent/{id}/disable/
```

`id` es el **ID de la tarjeta** obtenido en las consultas anteriores, no el ID del cliente ni el del intento de pago. Solo se permite operar sobre tarjetas del partner autenticado.

```bash
curl --request POST 'https://app.conpronto.com/api/v1/recurrent/10/disable/' \
  --header 'Authorization: Bearer <access_token>' \
  --header 'Content-Type: application/json' \
  --data '{}'
```

Respuesta HTTP **200**:

```json
{"id": 10, "is_enable": false}
```

- Excluye la tarjeta de nuevos débitos automáticos, incluidos los detalles aún no enviados.
- No elimina la tarjeta, los intentos, los pagos ni el historial. Sigue disponible en las consultas.
- No cancela ni revierte una transacción ya enviada a Datafast.
- Repetir la solicitud mantiene la tarjeta inhabilitada.

## Habilitar tarjeta (POST)

```text
POST https://app.conpronto.com/api/v1/recurrent/{id}/enable/
```

```bash
curl --request POST 'https://app.conpronto.com/api/v1/recurrent/10/enable/' \
  --header 'Authorization: Bearer <access_token>' \
  --header 'Content-Type: application/json' \
  --data '{}'
```

Respuesta HTTP **200**:

```json
{"id": 10, "is_enable": true}
```

La tarjeta vuelve a ser considerada en la siguiente ejecución, siempre que cumpla las reglas de elegibilidad. Puede incluir documentos pendientes acumulados desde su inicio hasta el corte mensual vigente. Habilitarla no modifica fechas, cuotas ni monto, no borra intentos y no desbloquea automáticamente resultados en revisión. Repetir la solicitud mantiene el mismo estado.

### Errores de habilitación e inhabilitación

| HTTP | Motivo |
|------|--------|
| `401` | Token ausente, inválido o vencido. |
| `404` | Tarjeta inexistente o que no pertenece al partner autenticado; también si no se encuentra el partner. |

No se requieren parámetros en el cuerpo para estas dos acciones.
