# Seguridad API REST

El API REST Existente debe implementar algún mecanismo de seguridad

  * Basic Auth
  * API Key
  * OAuth
  * Personal API Token
  * etc.

## Personal API Token

El mecanismo de Personal API Token es uno muy utilizado, sin embargo, no es el más seguro. Los API Tokens funcionan como passwords, la ventaja es que pueden ser revocados.

Consideraciones:

1. Proveer un mecanismo para **generar** API Tokens asociados al usuario
2. Proveer un mecanismo para **revocar** API Tokens
3. El API Token generalmente viaja en cada llamada al API REST en los headers para autenticar cada llamada.