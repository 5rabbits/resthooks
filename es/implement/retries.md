# Reintentos

¿Cuándo reintentar un hook enviado que falló? Todo se traduce al status_code recibido del POST.

Como pueden haber muchos suscriptores al mismo evento/recurso, no es recomendable analizar el body del response del webhook para actualizar o menajar lógica dentro de nuestra aplicación web. Sin embargo, el código http de respuesta es muy importante.


### Reintentar

Se debe reintentar si se obtiene cualquier respuesta del tipo 4xx, 5xx  o TimeOut

### No Reintentar

No se debe reintentar si se obtiene cualquier respuesta del tipo 2xx, 3xx o 410.

El código http 410 (Gone) indica que el recurso ya no está disponible, por lo que nuestra aplicación debería inmediatamente deshabilitar la suscripción que gatilló el webhook.
