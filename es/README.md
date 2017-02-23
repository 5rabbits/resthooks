# Webhooks

Un Webhook es una petición HTTP vía POST que se gatilla cuando sucede algo dentro de nuestra aplicación web.

Webhooks son una muy eficiente forma de integrar aplicaciones web, evitando el polling  y carga desmedida a nuestra API REST. Ya que la mayoría de los datos que se obtien mediante polling son basura, es decir, mientras no haya ocurrido ningún cambio en los datos (que ocurre más del 90% de las veces) no tiene sentido consultar por ellos.

Existen algunos tipos de Webhooks

* **Push**: Es el utilizado para notificar cambios en tiempo real
* **Pipes**: Notifica eventos, y quien lo recibe, usa los datos para transmitirlos
* **Plugins**: Notifica eventos, y quien lo recibe luego de procesar los datos, los devuelve en el **response**.

La implementación basada en el API REST recibe el nombre de [REST Hook](resthooks/).
