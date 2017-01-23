# Webhooks

A Webhooks is an http POST that ocurrs when something happens in our Web App.

Webhooks are an efficient way for integrate web applications, avoiding polling and excesive load of our REST API.

Some general ways of implement a Webhook consumer:

* Push: Simple notification of data. No more polling.
* Pipes: Receives data and trigger actions unrelated to the original event
* Plugins: Allow others to extend you application. A Web application sending data via Webhooks and use the response to modificy its own data.

RESTHooks are a lightweight webhook layer on top of your existing REST API.

