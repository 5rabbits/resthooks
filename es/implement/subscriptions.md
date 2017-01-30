# Suscripciones

Implementar API REST para que un tercero pueda suscribirse

```
GET     /api/v1/webhooks/         listar suscripciones
POST    /api/v1/webhooks/         crear suscripciones
GET     /api/v1/webhooks/:id/     obtener una suscripción
PUT     /api/v1/webhooks/:id/     actualizar una suscripción
DELETE  /api/v1/webhooks/:id/     eliminar una suscripción
```

El endpoint más importante es el que permite crear una suscripción (webhook): `POST /api/v1/webhooks/`

Independiete del storage, una suscripción tiene los siguientes atributos

| Atributo  | Descripción  |
|-----------| ----------- |
| id   |  Id único del webhook  |
| resource  |String. El recurso al cual el webhook está suscrito. ej: 'clients', 'invoices'  |
| target | String. La URL que recibe el POST del webhook |
| events | String. Evento(s) a los que se está suscrito (separados por coma). Ej: 'create' o 'create,update,destroy' |
| active | **Editable** Boolean. Si está activo, el webhook enviará eventos |
| created_at  | **Oculto**. DateTime de creación |
| secret  | **Oculto**. String. Un secret key único generado en el registro |
| last_status | Integer. El status code HTTP recibido en la última ejecución |
| last_success_at  | DateTime de la última vez que el evento se envió satisfactoriamente al **target** |
| last_failure_at  | DateTime de la última vez que falló el envío del evento |
| last_failure_content  | String. El mensaje de error del último envío fallido |

Por consistencia todos los atributos, excepto el **active** son de Solo lectura.


#### Creando un  Webhook

Este endpoint debe crear una tupla en el storage de webhooks y debe validar:
  * Que sea un resource válido
  * Que el o los eventos correspondan resource. Los tipos de [Eventos](events.md) deben ser públicos para los integradores.
  * Que el target acepte peticiones POST y quiera recibir webhooks

```
POST /api/v1/webhooks/
```

##### Payload

```
  {
    resource: 'clients',
    events: 'create,update',
    target: 'https://example.com/hook-receiver/client_updates'
  }
```

##### Respuesta satisfactoria

201 Created: La aplicación pudo registrar el webhook correctamente

```
  {
    id: 12345
    resource: 'clients',
    events: 'create,update',
    target: 'https://example.com/hook-receiver/client_updates',
    active: true,
    last_status: 200,
    last_success_at: null,
    last_failure_at: null,
    last_failure_content: null
  }
```

##### Respuesta fallida

400 Bad request: Los parámetros resource, events o target son inválidos.

```
  {
    error: 'Recurso inválido'
  }
```

##### Verificación del Destino

El endpoint de creación debe considerar realizar un test al **target** antes de aceptarlo y registrarlo en el storage. El test es bastante simple y servirá para  verificar si el **target** desea recibir webhooks. El flujo es el siguiente.

Sea

* **Consumer**: La aplicación de terceros que quiere recibir webhooks
* **Sender**: Nuestra apliación web que gatilla los eventos

``` sequence
Consumer->Sender: POST /api/v1/webhooks
Sender->Consumer: POST /target-url  \n X-Hook-Secret: SECRET
Consumer-->Sender: Response 200 \n X-Hook-Secret: SECRET
Sender->Sender: Guardar Suscripción
Sender-->Consumer: Response 201 { id: 1234, ... }
```

Desde el punto de vista de nuestra aplicación web:

1. Se recibe el POST hacia el enpoint /webhooks/ para crear un webhook (suscripción a evento)
2. Se validan los parámetros: resource, events sean correctos
3. Se realiza una petición POST al **target** con header X-Hook-Secret: SECRET, un secret único generado para la validación.
4. Se espera que el **response** de **target** sea un Status HTTP 200 con un response header X-Hook-Secret idéntico al envaido.
5. Si todo es correcto, se procede a almacenar y retornar el webhook en el response con status code 201 Creado.

[Listando Eventos](events.md)