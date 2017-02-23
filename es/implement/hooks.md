# Enviar hooks

Al ocurrir un evento se deben gatillar los hooks registrados para dicho evento.

#### Model Driven

La lógica del envío del evento está desarrollada en callbacks del modelo. Al ocurrir cualquier acción de guardado / eliminación de datos, se debe gatillar la acción que busca las suscripciones del modelo (y sus "parents"), y las envía.

Ejemplo: El siguiente ejemplo gatilla el evento de creación de un pago para suscripciones al recurso **payment**, pero también al recurso **invoice**

```Ruby
# Invoice Model
class Invoice < NotificableModel
  has_many :payments
end

# Payment Model
class Payment < NotificableModel
  belongs_to :invoice
  after_create :notify_create
end

# Base Class with event notification Behavior
class NotificableModel < ActiveRecord::Base
  def notify_create
    notify :create
  end

  def notify(method)
    # payment
    me = class.name.underscore.to_sym.singularize

    # para :payment => [:invoice]
    parents = class
              .reflect_on_all_associations(:belongs_to)
              .map(&:name)

    # payment.create
    event_name = "#{me}.{method}"

    # Enviar el evento de creación de payment
    # para suscripciones al recurso payment
    Subscriptions.send_event event_name, self

    parents.each do |parent|
      # Enviar el evento de creación de payment
      # para suscripciones padres, por ejemplo a invoce
      Subscriptions.send_event event_name, self, parent
    end
  end
end

```

#### API Driven

Otra forma de implementación sería intervenir el API REST de manera que las notificaciones de los eventos tengan relación directa con el endpoint invocado una vez que la ejecución de este finalice satisfactoriamente.

### Legacy Driven

En un software Legado, hipotéticamente escrito en PHP, posiblemente existan entidades DTO (Data tansfer Objects) y DAO (Data Access Objects), en dicho caso se puede simular el trabajo con modelos.

* El DTO debe implementar `Notificable` y métodos callbacks
  * after_create
  * after_update
  * after_destroy
* DAO ya implementa métodos `save()`, `destroy()`, `etc()`
* Para mantener separada la lógica de notificación y evitar notificar por error en DAO se pueden implementar métodos `saveAndNotify()`, `destroyAndNotify()`
* Los métodos `<operation>AndNotify()` harán `<operation>()` (del DAO) y luego `notify(<operation>)`. Este último método hace callback del método del entity (DTO) dependiendo de qué operación se está realizando

```
Entity->after_create()
Entity->after_update()
```

* Por su parte Entity que implementa Notificable ya puede funcionar como **NotificableModel** descrito en el ejemplo Ruby.
* También la lógica de envío de notificación debiera estar en el Abstracto Notificable.

#### Async / Background

Tanto la búsqueda de suscriptores como el envío del hook vía POST debe hacerse en una tarea asíncrona (y en background) respecto del flujo (callabck del modelo) que gatilló el evento. De esta forma se establece un mecanismo elegante y a la vez eficiente.

Si tomamos el ejemplo anterior, el método `Subscriptions.send_event` debería ser capaz simplemente de:

* Instanciar un mecanismo asíncrono de envío de mensajes (Worker, Job)
* Encolar el evento con los datos básicos (event_name, resource_id, parent_id)

Luego el mecanismo asíncrono (Worker, Job, Queue) deberá ser capaz de:

* Buscar todos los suscriptores al `recurso-evento`
* Construir el **payload** de evento definido en [el capítulo anterior](events.md)
* Enviar vía POST a todos los targets de cada suscriptor encontrado
* Manejar el código de retorno para reencolar, finalizar, o eliminar la suscripción (410). Puede verlo en [Manejo de Reintentos](retries.md)

``` sequence
Model->Notificable: Enviar Evento
Notificable->Queue_1: Encola Evento
Worker_1->Database: Buscar Suscripciones
Database-->Worker_1: suscripciones
Worker_1->Worker_1: Payload del evento por suscriptor
Worker_1->Queue_2: Encola notificación
Queue_2->Suscriptor: POST Webhok
Suscriptor-->Queue_2: Status Code
Queue_2->Queue_2: Análisis Retorno
```
