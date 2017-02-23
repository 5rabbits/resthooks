# Seguridad

> ver [Confirmación del Destino](implement/subscriptions.md#verificación-del-destino)

La seguridad del sistema de suscripciones viene dada por la seguirdad del API REST, ahora bien ¿cómo le brinados herramientas al suscriptor para que pueda verificar que se trata de un POST Válido?

### Confirmar legitimidad del webhook

#### Sender side

En cada hook enviado por nuestra aplicación,  debemos firmar el mensaje mediante el header `X-Hook-Signature`. El receptor (o suscriptor) debe verificar la firma. Previamente, debe  haber habido una negociación entre el receptor y nuestra aplicación para conocer dicha firma.

Sabemos que al realizar la suscripción al webhook, se hace la [verificación del destino](implement/subscriptions.md#verificación-del-destino) en la cual se envía un header `X-Hook-Secret`. Una forma de asegurar la legitimidad de los siguientes POST, es utilizar un a firma (X-Hook-Signature) basada en el `X-Hook-Secret` con algún dato adicional, por ejemplo:

	* X-Hook-Secret + nombre/id del recurso
	* X-Hook-Secret + El id de la suscripción
	* X-Hook-Secret + etc

De esta manera si el suscriptor almacenó esta información al momento de suscribirse, podrá verificar la autenticidad del mensaje.

#### Subscription side

El suscriptor puede tener algún mencanismo propio de autenticación por Token, en dicho caso debe proveer la información de autorización directamente en la URL (target) del webhook.

- Suscriptor (receptor) Genera su propio Personal API Token
- Suscriptor genera la URL (target) ejemplos:
	* `https://receptor.example.com/receive_updates?api_token=:personal_api_token:`
	* `https://receptor.example.com/:personal_api_token:/receive_updates`
- Receptor realiza la suscripción al webhook con la url generada como Target
- El sender envía el webhook a dicha dirección, como el token está embebido en la URL, el receptor puede verificar la autenticidad del POST y autorizar la llamada.

