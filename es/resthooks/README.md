# REST Hooks

Es la implementación liviana de webhooks basada un API REST existente. Propuesta inicialmente por [Zapier](www.zapier.com) y adoptada por varios servicios. Zapier actualmente tiene más de 1 millón de usuarios y sobre 750 aplicaciones integradas.

REST Hooks no es una especificación, sino más bien  un conjunto de patrones y buenas prácticas. En [5rabbits](www.5rabbits.com) seguiremos estas prácticas de manera tal de tener productos fácilmente integrables a otras plataformas.

De forma general el flujo de suscripción y notificación es el siguiente:

``` sequence
Consumer->Sender: Suscribe a evento
Sender->Store: Almacena suscripción
Sender-->Consumer: suscripción
Sender->Sender: Ocurre un evento
Sender->Queue: Encola notificación
Queue->Store: Busca suscripciones
Store-->Queue: Suscripciones
Queue->Consumer: Notifica Evento mediante POST
Consumer->Sender: API REST
```

Donde:

* Sender: Es nuestra aplicación web que despacha webhooks
* Consumer: Es la aplicación de terceros que se suscribe
* Store: Donde almacena los datos nuestra aplicación
* Queue: Algún servicio de encolamiento de tareas