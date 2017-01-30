# Listar e implementar eventos

Proveer un listado de eventos que el sistema de suscripciones soporta.

#### Nombres de eventos

Los nombres de los eventos deben tener la forma `sustantivo.verbo`, para mantener la consistencia con el API REST. Y esta nomenclatura debe ser indiferente del recurso al cual nos estamos suscribiendo.

Por ejemplo para el recurso `invoices`:

```
* invoice.create
* invoice.update
* invoice.delete
* payment.create
* payment.update
* payment.delete
```

Este listado de tipos de eventos debe estar disponible en la documentación de la integración y/o crear un recurso `hook_events`

```
GET /api/v1/hook_events
[
  { "resource": "invoices", "event": "invoice.create" }
  { "resource": "invoices", "event": "invoice.update" }
  { "resource": "invoices", "event": "payment.create" }
]
```

#### Payload

El payload entergado en el evento, cuando es gatillado, debe ser equivalente al que devuelve el API REST para el mismo recurso. De esta forma se puede mapear recursos REST a recursos Webhooks y vice versa de forma fácil. De todas maneras el payload del webhook debe contener información adicional sobre el evento, recurso y usuario que lo gatilló.

Ejemplo de evento cuando se crea un pago estando suscrito al recurso `invoices` y al evento `payment.create`

```
{
  "event": {
    "name": "payment.create",
    "resource": "payments"
    "resource_id": 1234,
    "parent": {
      "resource": "invoices",
      "resource_id": 1111
    },
    "user_id": 1122
  },
  "resource": {
    "id": 1234,
    "description": "payment description"
  }
}
```

Donde:

* **event**: Es donde viene contenida la información del evento notificado. Todos los atributos de **event** son requeridos excepto **parent** si no corresponde.
* **resource**: Es el recurso notificado, equivalente al expuesto por el API REST. Esto es **opcional** ya que puede decidir usar **skinny payloads** por razones de [seguridad](security.md)

En el atributo **events**:

* **name**: Es el nombre del evento lanzado (también suscrito)
* **resource**: Es la entidad que se notifica
* **resource_id**: Es el identificador de la entidad que se notifica
* **user_id**: Es el identificador del usuario que gatilla el evento, esto es muy útil para filtrar
* **parent**: Si el evento fue gatillado por una entidad de nivel superior, se informa también. Ejemplo: Suscrito al evento `payment.create` del recurso `invoices`. Esto último es posible debido a las relaciones shallow nested en el API REST
  * `/api/v1/invoices`
  * `/api/v1/invoices/:parent_id/payments/:resource_id`

Para el usuario que se suscribió es fácil llegar al endpoint correcto del API a partir de la información provista en el evento.


#### Implementación

Los eventos deben ser gatillados cuando ocurren. Dependiendo del lenguaje, framework y arquitectura utilizada en el desarrollo de la aplicación web, la implementación puede variar. Dentro de las alternativas tenemos:

* Montar algo completamente sobre nuestra API REST (middleware, filter)
* Montar algo sobre nuestro ORM (callbacks)
* Implementación personalizada o mixta

La problemática fundamental es que no siempre se tiene completamente mapeada el API REST de acuerdo a todos los recursos que pueden "cambiar" dentro de la aplicación. Así también, el API REST puede no ser fiel reflejo del modelo de datos (sobre todo en software Legacy). O simplemente nuestro software no utiliza ORM (también en legacy).
